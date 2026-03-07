# OpenClaw Skills cho Dev — Bản đầy đủ (không còn kiểu 3 skill cho có)

Đây là bản cập nhật sau khi rà lại danh sách skills thực tế đang có trong OpenClaw runtime.

---

## 1) Trước hết: OpenClaw có bao nhiêu skill “đáng dùng”?

Tại máy hiện tại, danh sách skills phát hiện được gồm (rút gọn nhóm chính):

- **Core/dev**: `coding-agent`, `skill-creator`, `model-usage`, `session-logs`, `summarize`
- **LLM bridge**: `gemini`, `oracle`
- **Git/dev platform**: `github`, `gh-issues`, `clawhub`
- **Messaging/collab**: `discord`, `slack`, `imsg`, `bluebubbles`, `wacli`
- **Productivity**: `notion`, `obsidian`, `things-mac`, `apple-notes`, `apple-reminders`, `trello`
- **Media/AI content**: `openai-image-gen`, `openai-whisper`, `openai-whisper-api`, `video-frames`, `nano-pdf`, `nano-banana-pro`, `gifgrep`
- **Infra/device/home**: `healthcheck`, `camsnap`, `openhue`, `sonoscli`, `spotify-player`, `voice-call`
- **System/terminal**: `tmux`, `xurl`, `ordercli`, `eightctl`, `mcporter`, `blucli`
- **Utility**: `weather`, `goplaces`, `blogwatcher`, `peekaboo`, `sag`, `sherpa-onnx-tts`, `songsee`

=> Nói thẳng: hệ sinh thái skill khá rộng, không chỉ vài cái cơ bản.

---

## 2) Cách cài đặt / bật skill cho đúng

## 2.1 Skill built-in
- Nếu skill đã có sẵn trong thư mục `openclaw/skills`, thường chỉ cần dùng đúng ngữ cảnh là agent tự chọn.

## 2.2 Skill community (ClawHub)
- Cài thêm từ marketplace/community khi cần workflow chuyên sâu.
- Sau khi cài:
  1. kiểm tra thư mục skill có `SKILL.md`,
  2. restart gateway nếu cần nạp lại.

## 2.3 Skill nội bộ (team skill)
- Tạo skill riêng cho quy trình team (release, incident, migration, hotfix).
- Tối thiểu cần:
  - scope,
  - non-scope,
  - checklist,
  - format output chuẩn.

---

## 3) Nhóm skill hay cho lập trình viên (chi tiết)

## 3.1 Nhóm coding trực tiếp

### `coding-agent`
- Dùng khi: feature lớn, refactor nhiều file, bug khó tái hiện.
- Mạnh ở vòng lặp: đọc code -> sửa -> build/test -> sửa tiếp.

### `session-logs`
- Dùng để soi lại lịch sử phiên trước, trace quyết định và lỗi.
- Rất hợp để debug “hôm qua chạy được, hôm nay hỏng”.

### `summarize`
- Tóm tắt logs dài, PR lớn, output test dài.
- Giúp giảm thời gian đọc output rác.

### `model-usage`
- Theo dõi usage/token/cost/model behavior.
- Hữu ích khi tối ưu chi phí agent cho team.

---

## 3.2 Nhóm Git/GitHub workflow

### `github`
- Tạo/cập nhật issue, PR context, lấy metadata repo.

### `gh-issues`
- Tập trung xử lý issue list, triage, mapping bug -> task.

### `clawhub`
- Khám phá, cài và quản lý skills từ ecosystem.

---

## 3.3 Nhóm tri thức & tài liệu dev

### `notion`, `obsidian`, `trello`
- Đồng bộ checklist công việc, technical notes, release note.

### `apple-notes`, `apple-reminders`, `things-mac`
- Hợp cho workflow cá nhân trên Mac.

---

## 3.4 Nhóm AI content cho dev

### `openai-whisper` / `openai-whisper-api`
- Chuyển audio họp kỹ thuật thành text action items.

### `openai-image-gen`
- Tạo ảnh minh hoạ docs/demo.

### `video-frames`
- Trích frame video để debug UI/bug demo theo timeline.

### `nano-pdf`
- Parse/tóm tắt PDF spec, tài liệu tích hợp.

---

## 3.5 Nhóm ops/hạ tầng

### `healthcheck`
- Audit security baseline, hardening, risk posture.

### `tmux`
- Quản lý các phiên terminal dài, job nền, tail logs.

### `xurl`
- Workflow gọi API/debug request-response nhanh.

---

## 3.6 Nhóm giao tiếp & automation

### `discord`, `slack`, `imsg`, `wacli`, `bluebubbles`
- Bot hóa thông báo build, incident, release.

### `voice-call`, `sag`, `sherpa-onnx-tts`
- Tạo voice summary cho cảnh báo/briefing.

---

## 4) Bộ skill stack gợi ý cho dev team

## Gói A — Solo dev (nhẹ, hiệu quả)
- `coding-agent`
- `session-logs`
- `summarize`
- `github`

## Gói B — Team product (có quy trình)
- Gói A + `gh-issues`, `notion`, `trello`, `model-usage`

## Gói C — Team vận hành nghiêm túc
- Gói B + `healthcheck`, `tmux`, `xurl`, `slack`/`discord`

---

## 5) Cách chọn skill đúng (tránh lạm dụng)

1. Task coding sâu => ưu tiên `coding-agent`.
2. Task tài liệu/tri thức => `summarize` + `notion/obsidian`.
3. Task repo/issue => `github` + `gh-issues`.
4. Task ops/security => `healthcheck` + `tmux/xurl`.
5. Đừng nhét 10 skill cho 1 việc nhỏ (tăng noise).

---

## 6) Ví dụ prompt thực dụng

### Fix bug nhiều file
> “Dùng coding-agent, sửa bug sync trong module X, chạy build/test, trả root cause + patch summary.”

### Review release nhanh
> “Dùng summarize + github, tóm tắt PR trong milestone Y và liệt kê risk còn mở.”

### Lập checklist bảo mật
> “Dùng healthcheck, audit host chạy OpenClaw, xuất checklist hardening theo mức ưu tiên.”

---

## 7) Sai lầm phổ biến

- Chọn sai skill (ví dụ bug code phức tạp nhưng lại làm kiểu one-shot).
- Không có acceptance criteria rõ.
- Không verify output bằng build/test thực tế.
- Không chốt format kết quả (dẫn đến trả lời lan man).

---

## 8) Kết luận

Skill là lớp “chuyên môn hóa hành vi agent”.
Dùng đúng skill = tăng tốc + giảm lỗi quy trình.

Với dev, bộ skill tối thiểu nên có: `coding-agent`, `session-logs`, `summarize`, `github`.
Sau đó mở rộng theo nhu cầu team sang `gh-issues`, `healthcheck`, `notion/trello`, `tmux/xurl`.
