# vibe-coding-skills

Bộ skill cho Claude Code để vibe coding có quy trình: **Đọc code → Lên plan → Dev → Test đầy đủ → Code review → UI/UX audit**.

```
vibe-read ─► vibe-plan ─► vibe-dev ─► vibe-test ─► vibe-review ─► vibe-uiux-audit
CONTEXT.md    PLAN.md    code+TDD   TEST-REPORT   REVIEW.md      UIUX-AUDIT.md
                 ▲ (user duyệt) ▲        │            │               │
                 │              └─ fix ◄─┴────────────┴───────────────┘
```

Mỗi bước ghi kết quả vào thư mục `.vibe/` trong project, bước sau đọc file của bước trước → context gọn, dừng/tiếp tục lúc nào cũng được.

## Skills

| Skill | Dùng khi | Output |
|-------|----------|--------|
| `vibe-init` | Setup 1 lần cho mỗi project | mục `## Vibe` trong `CLAUDE.md` |
| `vibe-flow` | Chạy cả pipeline cho 1 feature | tất cả bên dưới |
| `vibe-read` | Đọc & map codebase | `.vibe/CONTEXT.md` |
| `vibe-plan` | Lên plan, chia task, acceptance criteria | `.vibe/PLAN.md` |
| `vibe-dev` | Code theo plan, TDD từng task, verify có bằng chứng | code + tick checklist |
| `vibe-test` | Test đầy đủ: unit / integration / e2e, edge case, coverage, mutation, flaky | test + `.vibe/TEST-REPORT.md` |
| `vibe-review` | Review bug, security, perf, test | `.vibe/REVIEW.md` |
| `vibe-uiux-audit` | Audit UX, a11y (WCAG 2.2 AA), responsive | `.vibe/UIUX-AUDIT.md` |
| `vibe-stacks` | Rule theo stack, các skill khác tự nạp | — |

**Stack pack có sẵn:** React/Next.js, Vue/Nuxt, NestJS, FastAPI, Go, PostgreSQL/MySQL, MongoDB, Unity 6 (URP), Godot 4.

## Cài đặt

**Từ GitHub (khuyên dùng — cài 1 lần, mọi project đều có):**

```bash
/plugin marketplace add kelvinward1010/vibe-coding-skills
/plugin install vibe-coding-skills@vibe-coding-skills
```

**Local để test khi đang sửa skill:**

```bash
/plugin marketplace add <đường-dẫn-tới>/vibe-coding-skills
/plugin install vibe-coding-skills@vibe-coding-skills
```

Không nên copy skill vào từng project — mỗi project sẽ giữ một bản khác nhau và không cập nhật được.

## Dùng làm base cho nhiều dự án

Bộ skill chỉ chứa phần **chung**. Phần **riêng** của từng project nằm trong `CLAUDE.md` của project đó:

| Lớp | Nằm ở | Chứa gì |
|---|---|---|
| Base | repo này | Quy trình 6 bước, chuẩn test, checklist review |
| Stack pack | `skills/vibe-stacks/<stack>.md` | Rule riêng cho Next.js, FastAPI, Go… — chỉ đọc khi project dùng stack đó |
| Project | mục `## Vibe` trong `CLAUDE.md` của project | Loại project, lệnh chạy/test, bước nào áp dụng, ngưỡng coverage, team rules, thư mục cấm sửa |

Project mới: chạy `/vibe-init` một lần. Các skill khác đọc mục `## Vibe` và để nó ghi đè mặc định (vd project `api` tự bỏ bước UI/UX audit).

**Game:** `vibe-init` có loại project `game`. Quy trình giữ nguyên, nhưng bỏ UI/UX audit web (thay bằng checklist playtest), unit test cho logic game, coverage không tính code render/scene, review chú ý ngân sách mỗi frame.

Cập nhật base: sửa skill → tăng `version` trong `.claude-plugin/plugin.json` → push → các máy chạy `/plugin marketplace update`.

## Cách dùng

```
/vibe-init
/vibe-flow thêm trang quản lý ứng viên có filter theo trạng thái
/vibe-read
/vibe-plan sửa bug login bị logout sau 5 phút
/vibe-test
/vibe-review
/vibe-uiux-audit trang /dashboard
```

Hoặc nói tự nhiên ("review code giúp", "audit UI trang này") — Claude tự chọn skill theo `description`.

## Cấu trúc

```
vibe-coding-skills/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── vibe-init/SKILL.md
│   ├── vibe-flow/SKILL.md
│   ├── vibe-read/SKILL.md
│   ├── vibe-plan/SKILL.md
│   ├── vibe-dev/SKILL.md
│   ├── vibe-test/SKILL.md
│   ├── vibe-review/SKILL.md
│   ├── vibe-uiux-audit/
│   │   ├── SKILL.md
│   │   └── references/web-interface-rules.md
│   └── vibe-stacks/
│       ├── SKILL.md              # bảng chọn pack
│       └── <stack>.md            # react-nextjs, vue, nestjs, fastapi, go, sql, mongodb, unity, godot
└── README.md
```

## Tiêu chuẩn test

Mọi code ship đều phải có test (chỉ spike được bỏ qua). `vibe-test` bắt buộc:

- **Ma trận test**: mỗi acceptance criterion → test ở đúng tầng (unit / integration / e2e)
- **Đủ chiều**: happy path, biên, input sai, lỗi mạng/DB, phân quyền, concurrency, regression
- **Diff coverage** ≥ 90% lines, ≥ 80% branches trên code thay đổi
- **Mutation check**: cố tình làm sai code → test phải đỏ
- **Không flaky**: test mới chạy 3 lần đều xanh
- Cấm skip / xoá / nới assertion để cho xanh

## Điểm mạnh lấy từ cộng đồng

| Ý tưởng | Áp dụng ở | Nguồn |
|---|---|---|
| Hard gate: chưa duyệt plan thì không code; hỏi từng câu một | `vibe-plan` | [obra/superpowers](https://github.com/obra/superpowers) — brainstorming |
| Task test-first, có lệnh verify + output mong đợi, cấm "TBD" | `vibe-plan` | superpowers — writing-plans |
| Verification gate: không có bằng chứng thì không được nói "xong" | `vibe-dev` | superpowers — verification-before-completion |
| Review 5 trục, giới hạn kích thước change, mutation check test | `vibe-review` | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — code-review-and-quality |
| Bảng "anti-rationalization" chặn các lý do để làm ẩu | `vibe-dev`, `vibe-review` | addyosmani + superpowers |
| Review bằng subagent mới (không bị lệch theo người viết code) | `vibe-review`, `vibe-flow` | superpowers — subagent-driven-development |
| 100+ rule giao diện, output dạng `file:line` | `vibe-uiux-audit` | [vercel-labs/web-interface-guidelines](https://github.com/vercel-labs/web-interface-guidelines) |

Cả ba repo đều dùng license MIT.

**Khi nào nên dùng bộ khác:** cần quy trình đầy đủ hơn (git worktree, debug có hệ thống, TDD nghiêm ngặt) thì cài `superpowers`. Cần thêm security, perf, CI/CD, ship thì xem `addyosmani/agent-skills`. Bộ này gọn hơn, tập trung vào pipeline 6 bước, có bước test đầy đủ và UI/UX audit riêng, thứ mà hai bộ kia không có.

## Tuỳ biến

- Rule riêng của 1 project → mục `## Vibe` trong `CLAUDE.md` của project đó, không sửa base.
- Rule cho cả 1 stack → thêm `skills/vibe-stacks/<stack>.md` (4 mục: Applies when, Dev, Review checks, Testing) và 1 dòng vào bảng trong `vibe-stacks/SKILL.md`.
- Thêm file tham khảo cạnh `SKILL.md` (vd `skills/vibe-review/security-checklist.md`) và nhắc tới nó trong SKILL.md — Claude chỉ đọc khi cần.
