# Unity 6 + URP

**Applies when:** `ProjectSettings/ProjectVersion.txt` exists. URP parts apply when `com.unity.render-pipelines.universal` is in `Packages/manifest.json`. Check the editor version there and follow it — API names below are Unity 6.

## Dev

**Code structure**
- Keep MonoBehaviours thin: they read input, forward events, and update visuals. Game rules (stats, inventory, crafting, damage, save data) live in plain C# classes so they run in EditMode tests without a scene.
- Game data (items, recipes, enemy stats, loot tables) in ScriptableObjects, not hard-coded numbers.
- Code split into assemblies with `.asmdef` (e.g. `Game.Core`, `Game.Gameplay`, `Game.Tests`) — tests can't reference code without them, and compile times stay short.
- `[SerializeField] private` fields, not public fields. Renaming a serialized field drops its saved values in every scene and prefab — add `[FormerlySerializedAs("oldName")]`.
- Communicate with C# events or ScriptableObject event channels instead of `Find` + direct calls. Subscribe in `OnEnable`, unsubscribe in `OnDisable`.
- Input through the Input System package (`InputActionAsset`), with rebinding support.
- Async with `Awaitable` / coroutines tied to the object's life (`destroyCancellationToken`); stop them in `OnDisable`.
- Physics and Rigidbody movement in `FixedUpdate` (`linearVelocity`, `MovePosition`); input and visuals in `Update`; camera follow in `LateUpdate`.

**Per-frame budget** (from *Team rules*, e.g. 16.6 ms for 60 fps)
- No `GetComponent`, `Find*`, LINQ, string building, `new` collections, or lambdas capturing variables inside `Update` / `FixedUpdate` / `LateUpdate`. Cache in `Awake`, reuse buffers.
- Physics queries with the non-allocating overloads (`Physics.RaycastNonAlloc`, `OverlapSphereNonAlloc`).
- Spawned things (projectiles, loot, effects, enemies) through `UnityEngine.Pool.ObjectPool<T>`, not `Instantiate`/`Destroy` per event.
- Survival stats (hunger, thirst, temperature) ticked by one manager on a timer, not an `Update` on every object.
- Large content loaded with Addressables; no big assets in `Resources/`.

**URP / graphics**
- One URP Asset per quality level; keep shaders SRP Batcher compatible; enable GPU Resident Drawer for lots of repeated meshes (trees, rocks, grass).
- LOD Groups on every world mesh, occlusion culling baked, baked lighting + light probes for static scenery; few real-time shadow-casting lights.
- Terrain: limit detail density and draw distance; check with the Profiler and Frame Debugger, not by eye.

**Save system**
- Save data is a plain C# DTO with a `version` field, serialized to JSON in `Application.persistentDataPath`. Write to a temp file, then replace — a crash mid-save must not corrupt the save.
- Changing the save format = bump `version` + write a migration from the previous version.

**Project hygiene**
- Editor settings: *Asset Serialization = Force Text*, *Version Control = Visible Meta Files*. Commit every `.meta` file; Git LFS for textures, models, audio. Gitignore `Library/ Temp/ Obj/ Logs/ UserSettings/ Build*/`.
- The AI can't see the editor. Changes that need it (wiring references in the Inspector, placing objects, baking lighting) go in the plan as **manual steps** for the user, with exact names to click.

## Review checks

- **Critical:** save format changed without version bump + migration (breaks players' saves); serialized field renamed without `[FormerlySerializedAs]`; `.meta` file missing, deleted, or regenerated (breaks every reference to the asset); static state that isn't reset when *Enter Play Mode Options* skips domain reload.
- Allocation or lookup inside `Update`/`FixedUpdate`/`LateUpdate` (see per-frame budget).
- Event subscribed without unsubscribing → `MissingReferenceException` after scene change, memory leak.
- `?.` / `??` / `is null` on a `UnityEngine.Object` — skips Unity's destroyed-object check. Use `if (obj)` / `== null`.
- Physics work in `Update`, or movement not scaled by `Time.deltaTime`.
- `Instantiate`/`Destroy` in a loop or per hit instead of pooling.
- Game rules written inside MonoBehaviours where they can't be unit-tested.
- Magic numbers for balance values that belong in a ScriptableObject.
- Singletons reaching into each other everywhere — prefer explicit references or events.

## Testing

- Unity Test Framework (`com.unity.test-framework`), tests in their own `.asmdef`.
  - **EditMode** — plain C# logic, fast, no scene: inventory add/remove/stack limits, crafting with missing ingredients, stat decay over time, damage formulas, save → load round trip and migration from each old version.
  - **PlayMode** — behaviour over frames (`[UnityTest]` + `yield`): pickup triggers, spawning from a pool, scene load + save restore.
- Make logic deterministic: inject time and randomness (an `IClock`, a seeded `System.Random`) instead of calling `Time.time` / `Random.value` in rules.
- Run headless: `Unity -batchmode -projectPath . -runTests -testPlatform EditMode -testResults results.xml` (then `PlayMode`). Coverage via the Code Coverage package (`-enableCodeCoverage`); target applies to game logic assemblies only.
- Performance: Profiler capture of the heaviest scene against the frame budget; the Performance Testing package for hot systems.
- Feel can't be automated — end each feature with the manual playtest checklist from *Team rules*.
