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
| `vibe-flow` | Chạy cả pipeline cho 1 feature | tất cả bên dưới |
| `vibe-read` | Đọc & map codebase | `.vibe/CONTEXT.md` |
| `vibe-plan` | Lên plan, chia task, acceptance criteria | `.vibe/PLAN.md` |
| `vibe-dev` | Code theo plan, TDD từng task, verify có bằng chứng | code + tick checklist |
| `vibe-test` | Test đầy đủ: unit / integration / e2e, edge case, coverage, mutation, flaky | test + `.vibe/TEST-REPORT.md` |
| `vibe-review` | Review bug, security, perf, test | `.vibe/REVIEW.md` |
| `vibe-uiux-audit` | Audit UX, a11y (WCAG 2.2 AA), responsive | `.vibe/UIUX-AUDIT.md` |

## Cài đặt

**Từ GitHub (sau khi push):**

```bash
/plugin marketplace add <github-user>/vibe-coding-skills
/plugin install vibe-coding-skills@vibe-coding-skills
```

**Local để test:**

```bash
/plugin marketplace add /home/duydao/Desktop/TII/vibe-coding-skills
/plugin install vibe-coding-skills@vibe-coding-skills
```

Hoặc copy thẳng từng skill vào `~/.claude/skills/` (dùng mọi project) hoặc `<project>/.claude/skills/`.

## Cách dùng

```
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
│   ├── vibe-flow/SKILL.md
│   ├── vibe-read/SKILL.md
│   ├── vibe-plan/SKILL.md
│   ├── vibe-dev/SKILL.md
│   ├── vibe-test/SKILL.md
│   ├── vibe-review/SKILL.md
│   └── vibe-uiux-audit/
│       ├── SKILL.md
│       └── references/web-interface-rules.md
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

- Sửa checklist trong từng `SKILL.md` cho hợp stack/team (vd thêm rule riêng cho Next.js, Laravel…).
- Thêm file tham khảo cạnh `SKILL.md` (vd `skills/vibe-review/security-checklist.md`) và nhắc tới nó trong SKILL.md — Claude chỉ đọc khi cần.
