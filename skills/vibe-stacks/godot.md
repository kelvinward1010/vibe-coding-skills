# Godot 4

**Applies when:** `project.godot` exists with `config_version=5` (Godot 4.x). Check `[application] config/features` for the exact minor version and whether the project uses GDScript, C#, or both — follow it. Examples below are GDScript.

## Dev

**Code structure**
- Build with scenes as components and compose them; avoid deep inheritance chains. **Call down, signal up**: parents call methods on children, children emit signals to parents.
- Static typing everywhere: `var hp: int`, `func damage(amount: int) -> void`, `:=` for inferred types. Turn on the *untyped declaration* warning in Project Settings so new code stays typed.
- Game rules (stats, inventory, crafting, damage, save data) in `class_name` scripts extending `RefCounted` or `Resource`, not inside nodes — they run in unit tests without a scene.
- Game data (items, recipes, enemy stats, loot tables) as custom `Resource` classes saved as `.tres`. A Resource is **shared** by everyone who loads it — `duplicate()` it or set `resource_local_to_scene` before changing per-instance values.
- Node references with `@export` or unique names (`%Inventory`) cached in `@onready`. No long `$"../../Player/Body"` paths.
- Autoloads only for truly global services (event bus, save manager, settings). Not a dumping ground.
- Input through Input Map actions (`Input.is_action_pressed("jump")`), never hard-coded keys; support remapping via `InputMap`.
- Movement and physics in `_physics_process` (`CharacterBody3D.velocity` + `move_and_slide()`); visuals and UI in `_process`; always scale by `delta`.
- Remove nodes with `queue_free()`. After any `await`, check `is_instance_valid(self)` / the target before touching it.
- The scene tree is not thread-safe: background work through `WorkerThreadPool`, results applied with `call_deferred`.

**Per-frame budget** (from *Team rules*, e.g. 16.6 ms for 60 fps)
- No `get_node`, `find_child`, `get_tree().get_nodes_in_group()`, string building, or new Arrays/Dictionaries inside `_process` / `_physics_process`. Cache once.
- Turn off processing for idle nodes (`set_process(false)`); use `Timer` nodes or signals instead of polling every frame.
- Survival stats (hunger, thirst, temperature) ticked by one manager on a timer, not a `_process` on every object.
- Many repeated objects (grass, trees, rocks) with `MultiMeshInstance3D`; reuse projectiles/effects instead of instancing per shot.
- Stream world chunks with `ResourceLoader.load_threaded_request`.

**Graphics (3D)**
- Renderer: *Forward+* for desktop quality, *Mobile* for low-end, *Compatibility* for web/old GPUs. Decide early — switching later changes how lighting looks.
- `LightmapGI` for static scenery; `SDFGI` only if you need large dynamic day/night and can pay for it. Volumetric fog and SSAO are costly — expose them in graphics settings.
- Mesh LOD (auto on import) + `visibility_range` for distant objects; `OccluderInstance3D` for occlusion culling.
- Measure with the Debugger's Profiler and Monitors, not by eye.

**Save system**
- Save data as a typed Dictionary → JSON (or `FileAccess.store_var`) in `user://`, with a `version` field. Write to a temp file, then rename — a crash mid-save must not corrupt the save.
- Never load player-supplied `.tres`/`.res` files as saves: Resources can carry scripts that run on load.
- Changing the save format = bump `version` + write a migration from the previous version.

**Project hygiene**
- Gitignore `.godot/`. Commit `*.import` and `*.uid` files (4.4+). Git LFS for textures, models, audio.
- Move or rename files **inside the editor** (FileSystem dock) so references update. Moving them in the OS file manager breaks `res://` paths.
- `.tscn`/`.tres` are text, so the AI can edit them — keep those edits small, keep `ext_resource` ids consistent, and verify by running the scene. Large layout work (placing objects, painting terrain, baking lights) goes in the plan as **manual steps** for the user.

## Review checks

- **Critical:** save format changed without version bump + migration; loading `.tres`/`.res` from `user://` or downloads as save data (code execution); file moved/renamed outside the editor → broken `res://` references; hand-edited `.tscn` with mismatched `ext_resource`/`sub_resource` ids.
- Shared Resource mutated at runtime → every enemy/item using it changes at once.
- Lookup or allocation inside `_process` / `_physics_process` (see per-frame budget).
- Node accessed after `await` without a validity check; `free()` used where `queue_free()` is needed.
- Signal connected again every time a node re-enters the tree → handler runs twice; connect once or use `CONNECT_ONE_SHOT`.
- Untyped variables/functions in new code; long relative `get_node` paths.
- Physics bodies moved in `_process`, or movement not scaled by `delta`.
- Scene tree touched from a thread.
- Game rules written inside node scripts where they can't be unit-tested.
- Balance values hard-coded instead of in a Resource.

## Testing

- Test addon: use what the project has — **GUT** or **GdUnit4** (GdUnit4 also supports C#). Ask before adding one; it's a new dependency.
  - **Logic tests** — `RefCounted`/`Resource` rule classes, no scene: inventory add/remove/stack limits, crafting with missing ingredients, stat decay, damage formulas, save → load round trip and migration from each old version.
  - **Scene tests** — instantiate the scene, simulate frames and input (GUT `wait_frames`/`simulate`, GdUnit4 scene runner): pickup triggers, enemy spawn, scene change + save restore.
- Make logic deterministic: inject time and randomness (a `RandomNumberGenerator` with a fixed `seed`) instead of global `randf()` in rules.
- Run headless, e.g. GUT: `godot --headless -s addons/gut/gut_cmdln.gd -gdir=res://test -gexit`. GdUnit4 ships its own `runtest` script in the addon folder.
- Coverage: GDScript has no standard coverage tool. Instead of a % target, the test matrix must map every acceptance criterion to a test; C# code can use normal .NET coverage.
- Performance: Profiler + Monitors on the heaviest scene against the frame budget.
- Feel can't be automated — end each feature with the manual playtest checklist from *Team rules*.
