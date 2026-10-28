# vibe-coding-skills

Bộ skill cho [Claude Code](https://claude.com/claude-code) giúp vibe coding **có quy trình**: Claude đọc code → lên plan cho bạn duyệt → code kèm test → test đầy đủ → tự review → audit giao diện. Cài một lần, dùng cho mọi project — web, API hay game.

```
vibe-read ─► vibe-plan ─► vibe-dev ─► vibe-test ─► vibe-review ─► vibe-uiux-audit
CONTEXT.md    PLAN.md    code+TDD   TEST-REPORT   REVIEW.md      UIUX-AUDIT.md
                 ▲ (bạn duyệt) ▲         │            │               │
                 │             └─ fix ◄──┴────────────┴───────────────┘
```

Mỗi bước ghi kết quả vào thư mục `.vibe/` của project, bước sau đọc file của bước trước. Nhờ vậy context gọn và bạn có thể dừng, sửa tay, hoặc tiếp tục bất cứ lúc nào.

**Mục lục:** [Cài đặt](#1-cài-đặt) · [Project mới](#2-bắt-đầu-một-project-mới) · [Dùng hằng ngày](#3-dùng-hằng-ngày) · [Danh sách skill](#4-danh-sách-skill) · [Stack pack](#5-stack-pack) · [Tiêu chuẩn test](#6-tiêu-chuẩn-test) · [Cập nhật bộ skill](#7-cập-nhật-và-phát-triển-bộ-skill) · [Câu hỏi thường gặp](#8-câu-hỏi-thường-gặp)

---

## 1. Cài đặt

**Yêu cầu:** đã cài Claude Code (CLI, app desktop hoặc extension VS Code/JetBrains).

Cài **một lần cho cả máy** — không cần clone repo này. Mở Claude Code ở thư mục bất kỳ và chạy:

```
/plugin marketplace add kelvinward1010/vibe-coding-skills
/plugin install vibe-coding-skills@vibe-coding-skills
```

**Kiểm tra:** gõ `/vibe` trong Claude Code — phải thấy gợi ý `/vibe-init`, `/vibe-flow`, `/vibe-read`… Nếu chưa thấy, khởi động lại Claude Code.

> Đừng copy thư mục skill vào từng project. Mỗi project sẽ giữ một bản riêng và không nhận được bản cập nhật.

---

## 2. Bắt đầu một project mới

Mỗi project là **một thư mục riêng**, không nằm trong repo này:

```
D:\Work\
├── vibe-coding-skills\   ← bộ skill (chỉ mở khi muốn sửa skill)
├── web-ban-hang\         ← project web
└── game-sinh-ton\        ← project game
```

### Bước 1 — Tạo project bằng công cụ của stack

| Loại | Cách tạo |
|---|---|
| Next.js | `npx create-next-app@latest web-ban-hang` |
| Vue / Nuxt | `npm create vue@latest` / `npx nuxi@latest init` |
| NestJS | `npx @nestjs/cli new api-ban-hang` |
| FastAPI | tạo thư mục + `pyproject.toml` (hoặc nhờ Claude dựng khung) |
| Go | `go mod init <module>` |
| Unity 6 | Unity Hub → *New project* → template **Universal 3D**. Sau đó *Edit → Project Settings → Editor*: **Asset Serialization = Force Text** |
| Godot 4 | Godot → *New Project* → chọn renderer (*Forward+* cho game 3D desktop) |

Với game, luôn tạo project bằng Editor trước — Editor sinh ra nhiều file cấu hình mà AI không nên tự viết.

### Bước 2 — Mở project trong Claude Code

Mở thư mục project (VS Code → *File → Open Folder*, hoặc `cd` vào thư mục rồi chạy `claude`).

### Bước 3 — Chạy `/vibe-init` (một lần)

```
/vibe-init
```

Claude sẽ:
1. Nhận diện stack và loại project (`web-ui`, `api`, `game`, `cli`…).
2. Đọc lệnh chạy/test/build từ project.
3. Hỏi những gì không tự đoán được — từng câu một (rule riêng của team, ngưỡng coverage, FPS mục tiêu nếu là game…).
4. Cho bạn xem rồi ghi mục `## Vibe` vào `CLAUDE.md` của project, và thêm `.vibe/` vào `.gitignore`.

Ví dụ mục `## Vibe` được tạo:

```markdown
## Vibe

- Type: web-ui, api
- Stack: Next.js 15, Prisma, Postgres
- Stages: read, plan, dev, test, review, uiux-audit
- Test levels: unit (Vitest), integration (Vitest + test DB), e2e (Playwright)
- Coverage: default
- Stack packs: react-nextjs, sql

| Task | Command |
|---|---|
| Dev | `pnpm dev` |
| Test | `pnpm test` |
| E2E | `pnpm e2e` |

### Team rules
- Mọi API mới phải có rate limit

### Off-limits
- `src/generated/` — Prisma sinh ra, không sửa tay
```

Bạn có thể sửa tay mục này bất cứ lúc nào. Khi project đổi nhiều (thêm DB, đổi framework), chạy lại `/vibe-init` để cập nhật.

### Bước 4 — Làm tính năng đầu tiên

```
/vibe-flow tạo trang đăng nhập bằng email + mật khẩu
```

Xem chi tiết ở phần tiếp theo.

---

## 3. Dùng hằng ngày

### Cách 1 — Chạy cả quy trình với `/vibe-flow` (khuyên dùng)

```
/vibe-flow thêm bộ lọc theo trạng thái cho trang danh sách đơn hàng
```

Claude chọn một trong ba **track** và báo cho bạn:

| Track | Khi nào | Làm gì |
|---|---|---|
| **Quick** | Sửa chữ, sửa 1 file rõ ràng | Code + test + review ngắn, không có plan |
| **Standard** | Tính năng hoặc bug bình thường | Đủ 6 bước |
| **Spike** | Bạn muốn thử nhanh một ý tưởng | Code nháp, không test/review. Muốn giữ lại thì làm lại theo Standard |

Với **Standard**, đây là những gì xảy ra và khi nào bạn cần làm gì:

| # | Bước | Claude làm | Bạn làm |
|---|---|---|---|
| 1 | Read | Đọc code liên quan, chạy test hiện có để ghi lại tình trạng ban đầu | — |
| 2 | Plan | Hỏi lại những chỗ chưa rõ, viết plan: task, tiêu chí hoàn thành, cách kiểm tra | **Trả lời câu hỏi, đọc plan, duyệt hoặc yêu cầu sửa.** Chưa duyệt thì Claude không code. |
| 3 | Dev | Code từng task, viết test trước, chạy test làm bằng chứng | Làm các bước thủ công nếu plan có (vd kéo thả trong Unity Editor) |
| 4 | Test | Bổ sung test còn thiếu, đo coverage, cố tình làm sai code để chắc test bắt được lỗi | — |
| 5 | Review | Một agent mới (không phải agent vừa code) review bug, bảo mật, hiệu năng | — |
| 6 | UI/UX audit | Chạy app, chụp màn hình 3 kích thước, kiểm tra bàn phím, accessibility | — (bỏ qua nếu không có giao diện web) |

Lỗi tìm thấy ở bước 4–6 → Claude tự quay lại bước 3 sửa, rồi chạy lại. Cuối cùng Claude báo cáo: đã làm gì, file nào đổi, kết quả test, vấn đề còn mở. **Claude không tự commit** — bạn quyết định.

**Dừng giữa chừng?** Gõ lại `/vibe-flow` — Claude đọc `.vibe/` và tiếp tục từ bước chưa xong.

### Cách 2 — Chạy từng bước riêng

```
/vibe-read                                  # hiểu một codebase lạ
/vibe-plan sửa bug bị logout sau 5 phút     # chỉ lên plan
/vibe-dev                                   # code theo PLAN.md đã duyệt
/vibe-test                                  # bổ sung test cho thay đổi hiện tại
/vibe-review                                # review diff hiện tại
/vibe-uiux-audit trang /dashboard           # audit một màn hình
```

Hoặc nói tự nhiên: *"review code giúp"*, *"test đầy đủ cho phần này"*, *"check giao diện trang này"* — Claude tự chọn skill phù hợp.

### Mẹo viết yêu cầu

- Nói **kết quả mong muốn**, không cần nói cách làm: *"người dùng lọc được đơn theo trạng thái, giữ bộ lọc khi tải lại trang"*.
- Một yêu cầu = một tính năng. Việc lớn → chia nhỏ, mỗi phần một lần `/vibe-flow`.
- Có ảnh thiết kế, link API, ví dụ dữ liệu → đưa luôn vào yêu cầu.

---

## 4. Danh sách skill

| Skill | Dùng khi | Kết quả |
|---|---|---|
| `vibe-init` | Setup 1 lần cho mỗi project | mục `## Vibe` trong `CLAUDE.md` |
| `vibe-flow` | Chạy cả quy trình cho 1 tính năng | tất cả bên dưới |
| `vibe-read` | Đọc và vẽ bản đồ codebase | `.vibe/CONTEXT.md` |
| `vibe-plan` | Lên plan, chia task, tiêu chí hoàn thành | `.vibe/PLAN.md` |
| `vibe-dev` | Code theo plan, test trước từng task, có bằng chứng | code + checklist đã tick |
| `vibe-test` | Test đủ tầng unit / integration / e2e, edge case, coverage, mutation, flaky | test + `.vibe/TEST-REPORT.md` |
| `vibe-review` | Review bug, bảo mật, hiệu năng, chất lượng test | `.vibe/REVIEW.md` |
| `vibe-uiux-audit` | Audit UX, accessibility (WCAG 2.2 AA), responsive | `.vibe/UIUX-AUDIT.md` |
| `vibe-stacks` | Rule theo từng stack — các skill khác tự nạp, không cần gọi | — |

---

## 5. Stack pack

Mỗi pack là một file rule riêng cho một công nghệ, gồm 3 phần: **cách viết code đúng** (dùng ở bước Dev), **lỗi cần bắt** (bước Review), **cách test** (bước Test). Project chỉ nạp pack của stack mình dùng — `/vibe-init` tự chọn.

| Pack | Stack | Ví dụ lỗi bắt được |
|---|---|---|
| `react-nextjs` | React, Next.js | Secret lộ vào code client, `useEffect` dùng sai, quên revalidate sau khi lưu |
| `vue` | Vue 3, Nuxt | `v-html` chưa sanitize, mất reactivity, hydration mismatch |
| `nestjs` | NestJS (Node.js) | Endpoint thiếu guard, sửa được dữ liệu của user khác, trả entity có password hash |
| `fastapi` | FastAPI (Python) | Gọi hàm chặn trong `async def`, thiếu `response_model`, đổi model không có migration |
| `go` | Go | Bỏ qua `err`, goroutine rò rỉ, data race |
| `sql` | PostgreSQL, MySQL | Migration khoá bảng lớn, N+1 query, thiếu index |
| `mongodb` | MongoDB | NoSQL injection, mảng phình vô hạn, query không có index |
| `unity` | Unity 6 + URP | Cấp phát bộ nhớ trong `Update()`, đổi format save làm hỏng save cũ, mất file `.meta` |
| `godot` | Godot 4 | Sửa Resource dùng chung, load file save chứa script, di chuyển file ngoài Editor |

**Game:** `vibe-init` có loại project `game`. Quy trình giữ nguyên nhưng:
- Bỏ UI/UX audit web, thay bằng checklist playtest (cảm giác điều khiển, HUD, cài đặt, lưu game).
- Unit test cho logic game (chỉ số, inventory, chế tạo, save). Cảm giác chơi kiểm tra bằng tay.
- Review chú ý ngân sách thời gian mỗi frame (vd 60 FPS = 16.6 ms).
- AI không nhìn thấy Editor: việc kéo thả, đặt object, bake ánh sáng được ghi trong plan thành **bước thủ công** cho bạn.

---

## 6. Tiêu chuẩn test

Mọi code được giữ lại đều phải có test (chỉ Spike được bỏ qua). `vibe-test` yêu cầu:

- **Ma trận test** — mỗi tiêu chí hoàn thành trong plan có test ở đúng tầng (unit / integration / e2e).
- **Đủ chiều** — đường chính, giá trị biên, input sai, lỗi mạng/DB, phân quyền, chạy đồng thời, lỗi cũ không quay lại.
- **Diff coverage** — ≥ 90% dòng, ≥ 80% nhánh trên **code thay đổi** (đổi được trong mục `## Vibe`; ngôn ngữ không có công cụ coverage như GDScript thì đặt `Coverage: none`).
- **Mutation check** — cố tình làm sai code, test phải đỏ.
- **Không flaky** — test mới chạy 3 lần đều xanh.
- Cấm skip, xoá hoặc nới lỏng assertion để test xanh.

---

## 7. Cập nhật và phát triển bộ skill

### Phân lớp: sửa ở đâu?

| Muốn thay đổi | Sửa ở |
|---|---|
| Rule chỉ cho **một project** | mục `## Vibe` trong `CLAUDE.md` của project đó — **không** sửa repo này |
| Rule cho **cả một stack** | `skills/vibe-stacks/<stack>.md` |
| Quy trình chung cho **mọi project** | `skills/<skill>/SKILL.md` |

### Quy trình sửa skill

1. Mở thư mục `vibe-coding-skills` và sửa file.
2. Thử trên máy trước khi đẩy lên — trỏ marketplace vào thư mục local:
   ```
   /plugin marketplace add D:\Work\vibe-coding-skills
   /plugin install vibe-coding-skills@vibe-coding-skills
   ```
3. Tăng `version` trong `.claude-plugin/plugin.json` (sửa nhỏ: `0.4.0 → 0.4.1`, thêm skill/pack: `0.4.0 → 0.5.0`).
4. Commit và push lên GitHub.
5. Trên mỗi máy: `/plugin marketplace update vibe-coding-skills`, rồi mở `/plugin` để cập nhật plugin nếu chưa tự cập nhật. Mọi project nhận bản mới.

### Thêm stack pack mới

1. Tạo `skills/vibe-stacks/<stack>.md` với 4 mục: **Applies when** (dấu hiệu nhận biết, vd file nào tồn tại), **Dev**, **Review checks** (lỗi nghiêm trọng đánh dấu **Critical**), **Testing**. Giữ dưới ~60 dòng, chỉ ghi rule riêng của stack đó.
2. Thêm 1 dòng vào bảng trong `skills/vibe-stacks/SKILL.md`.
3. Thêm vào bảng ở mục [5](#5-stack-pack) của README này.

### Thêm tài liệu tham khảo cho một skill

Đặt file cạnh `SKILL.md` (vd `skills/vibe-review/security-checklist.md`) và nhắc tới nó trong `SKILL.md`. Claude chỉ đọc file đó khi cần.

---

## 8. Câu hỏi thường gặp

**Có phải clone repo này về rồi code trong đó không?**
Không. Cài plugin một lần (mục 1), code ở thư mục project riêng. Chỉ clone repo này khi muốn sửa skill.

**Project đang làm dở có dùng được không?**
Được. Mở project, chạy `/vibe-init`, rồi `/vibe-flow` như bình thường. Bước Read sẽ ghi lại tình trạng test hiện tại, nên lỗi có sẵn từ trước không bị tính cho thay đổi mới.

**Có cần chạy `/vibe-init` mỗi lần không?**
Không, chỉ một lần mỗi project. Chạy lại khi project đổi stack hoặc thêm công nghệ mới.

**Có nên commit thư mục `.vibe/` không?**
Mặc định không (đã thêm vào `.gitignore`) — đó là ghi chú làm việc. Muốn lưu plan/review cho team thì xoá dòng `.vibe/` khỏi `.gitignore`.

**Stack của tôi chưa có pack?**
Vẫn dùng được — các skill có rule chung cho mọi ngôn ngữ. Pack chỉ thêm rule riêng. Muốn có pack thì xem mục [7](#thêm-stack-pack-mới).

**Project không có giao diện (API, CLI) thì sao?**
`/vibe-init` ghi loại project là `api`/`cli`, bước UI/UX audit và test trình duyệt tự bỏ qua.

**Claude có tự commit/push không?**
Không. Cuối mỗi lần chạy, Claude hỏi bạn muốn commit, mở PR hay làm tiếp.

---

## Cấu trúc repo

```
vibe-coding-skills/
├── .claude-plugin/
│   ├── plugin.json               # tên, version, tác giả
│   └── marketplace.json          # để cài qua /plugin marketplace add
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

## Ý tưởng lấy từ cộng đồng

| Ý tưởng | Áp dụng ở | Nguồn |
|---|---|---|
| Chưa duyệt plan thì không code; hỏi từng câu một | `vibe-plan` | [obra/superpowers](https://github.com/obra/superpowers) — brainstorming |
| Task viết test trước, có lệnh kiểm tra + kết quả mong đợi, cấm "TBD" | `vibe-plan` | superpowers — writing-plans |
| Không có bằng chứng thì không được nói "xong" | `vibe-dev` | superpowers — verification-before-completion |
| Review 5 trục, giới hạn kích thước thay đổi, mutation check | `vibe-review` | [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) — code-review-and-quality |
| Bảng chặn các lý do để làm ẩu | `vibe-dev`, `vibe-review` | addyosmani + superpowers |
| Review bằng agent mới, không bị lệch theo người viết code | `vibe-review`, `vibe-flow` | superpowers — subagent-driven-development |
| 100+ rule giao diện, kết quả dạng `file:line` | `vibe-uiux-audit` | [vercel-labs/web-interface-guidelines](https://github.com/vercel-labs/web-interface-guidelines) |

Cả ba repo đều dùng license MIT.

**Khi nào nên dùng bộ khác:** cần quy trình nặng hơn (git worktree, debug có hệ thống, TDD nghiêm ngặt) → `superpowers`. Cần thêm CI/CD, ship, security chuyên sâu → `addyosmani/agent-skills`. Bộ này gọn hơn, tập trung vào quy trình 6 bước, có bước test đầy đủ, UI/UX audit và stack pack cho cả web lẫn game.

## License

MIT — Kelvin Ward
