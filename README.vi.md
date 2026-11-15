```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Một AI agent cấu hình được cho terminal của bạn.**
Tự mang API key của bạn: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, một Ollama cục bộ, hoặc bất kỳ server tương thích OpenAI nào. Linux và Termux trước tiên.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
Tiếng Việt · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, phiên bản 0.1.0-alpha.1.** TUI gốc, chat đa provider, agent loop có ý thức về quyền hạn, công cụ cục bộ và phiên cục bộ có thể tiếp tục đã được hiện thực. API và cài đặt vẫn có thể thay đổi. Đây là một prototype, chưa phải ngang tính năng với OpenCode hay Hermes.

*Marelle* trong tiếng Pháp là trò nhảy ô (hopscotch). Mỗi ô là một bước.

## Cài đặt

**Termux trên Android.** Tải binary dựng sẵn từ [Releases](https://github.com/S-I-L-S/marelle/releases), hoặc build ngay trên điện thoại. Từng bước một: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, và bất cứ nơi nào Rust chạy được.** Rust 1.85 trở lên:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Hoặc clone repository rồi chạy `cargo build --release`. Binary là `target/release/marelle`, khoảng 4 MiB.

## Cuộc chat đầu tiên

Hãy giữ key trong một file mà chỉ bạn đọc được, và nói cho Marelle biết nó ở đâu. Marelle không bao giờ lưu key trong cài đặt của mình.

```sh
mkdir -p ~/.config/marelle
( umask 077; read -rsp "API key: " key; echo; printf '%s' "$key" > ~/.config/marelle/zai.key )
export MARELLE_PROVIDERS__ZAI__API_KEY_FILE=~/.config/marelle/zai.key

marelle providers                                    # what exists, and whether the key was found
marelle config set chat.provider zai
marelle config set chat.model glm-5.2
marelle                                                # open the interactive terminal UI
marelle chat -m zai/MODEL "Explain hopscotch in one sentence"
```

Thay `MODEL` bằng tên một model trong danh sách của provider bạn. Các cách dùng khác:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Câu trả lời đi ra stdout. Phần suy luận, số token, ghi chú và lỗi đi ra stderr, nên `marelle chat ... > answer.txt` chỉ lưu lại câu trả lời.

## Công cụ của agent

Marelle gửi định nghĩa công cụ có cấu trúc qua các API tương thích OpenAI và Anthropic. Bộ công cụ dựng sẵn gồm:

| Công cụ | Mục đích |
| --- | --- |
| `terminal` | Chạy một lệnh shell từ thư mục khởi chạy |
| `read_file`, `write_file`, `edit_file` | Đọc và sửa các file text UTF-8 |
| `list_directory`, `search_text` | Xem và tìm kiếm trong file của dự án |
| `fetch_url` | Tải một trang HTTP(S) công khai dưới dạng text (cần `curl`) |
| `list_skills`, `load_skill` | Tìm và nạp các hướng dẫn `SKILL.md` cục bộ |

Dùng `/tools` để xem danh sách công cụ và `/skills` để liệt kê các skill có sẵn. Hướng dẫn của dự án được nạp từ `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` và các file `.marelle/` được chọn khi chúng nằm trong các path được phép.

Các công cụ file bị giới hạn trong `permissions.allowed_paths` (mặc định: thư mục dự án hiện tại) và tôn trọng `permissions.denied_paths`. Mặc định được phép đọc; sửa file, chạy lệnh shell và tải qua mạng sẽ hỏi trước khi thực hiện. Quyền shell áp dụng cho lệnh chạy trên host và không phải là sandbox filesystem: hãy giữ ở mức `ask` trừ khi bạn cố ý giao quyền thực thi lệnh cho model.

Agent dừng sau `agent.max_turns` lượt tool round-trip. Phản hồi của tool được coi là nội dung không tin cậy. Các request dùng phản hồi model đầy đủ để tham số tool không bị ghép từ những mảnh stream không hoàn chỉnh.

## Phiên

Cuộc hội thoại được lưu cục bộ theo mặc định, kể cả lịch sử gọi tool và cài đặt model/session. Trên Unix, thư mục session có mode `0700` và file JSON có mode `0600`. File được lưu tại `$XDG_STATE_HOME/marelle/sessions.json`, hoặc `~/.local/state/marelle/sessions.json` khi `XDG_STATE_HOME` không được đặt. API key không bao giờ được ghi vào file này. Để tắt lưu trữ:

```sh
marelle config set agent.persist_sessions false
```

Mã thoát: `0` thành công, `1` lỗi provider hoặc mạng, `2` lỗi usage hoặc cấu hình.

## Providers

| id | Giao thức | Biến chứa key | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM), quốc tế |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, endpoint tương thích Anthropic |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel, Trung Quốc |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Trung Quốc |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | nhiều model sau một key duy nhất |
| `ollama` | OpenAI-compatible | none | Ollama trên chính máy này |

Các endpoint Z.ai và Zhipu lấy từ tài liệu của bên thứ ba, không phải từ trang của chính nhà cung cấp. Hãy đối chiếu với gói của bạn và ghi đè trường nào khác biệt.

Mọi trường của provider đều đổi được, và một provider mới chỉ là một cái tên kèm một địa chỉ:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Các trường khác của provider: `kind` (`openai-compatible` hoặc `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` và `extra_body`. Nếu biến được nêu trong `api_key_env` được đặt thì nó thắng; nếu không, file sẽ được đọc.

## Thiết lập

Cài đặt theo từng lớp: mặc định dựng sẵn, rồi biến môi trường, rồi cờ. File cài đặt sẽ chen vào giữa mặc định và biến môi trường ở bước tiếp theo.

- `MARELLE_CHAT__MODEL=zai/MODEL` đặt `chat.model`. Cặp underscore kép đứng cho một dấu chấm.
- `marelle config show [prefix]` liệt kê các cài đặt và mỗi cái đến từ lớp nào.
- `marelle config explain <key>` cho thấy mọi lớp đã đặt một key, từ mức ưu tiên thấp nhất đến cao nhất.

| Key | Mặc định | Ý nghĩa |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | Nói chuyện với ai. `chat.model` có thể là `provider/model`. |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | Để provider tự quyết khi không đặt |
| `chat.stream` | `true` | Stream phần trả lời của judge tùy chọn; các lượt agent có tool hiện dùng phản hồi đầy đủ để các lệnh gọi tool giữ nguyên vẹn |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Suy luận và số token trên stderr |
| `agent.max_turns` | `20` | Số lượt tool round-trip tối đa cho mỗi yêu cầu của người dùng |
| `agent.persist_sessions` | `true` | Lưu hội thoại cục bộ giữa các lần chạy |
| `permissions.shell` | `ask` | Thực thi lệnh shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Đọc file: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Sửa file: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Tải URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Các gốc được phép cho công cụ file |
| `permissions.denied_paths` | none | Các file path bị chặn tường minh |
| `network.connect_timeout_secs` | `15` | Thời gian được phép để kết nối |
| `network.idle_timeout_secs` | `300` | Khoảng lặng được phép khi đang chờ dữ liệu |
| `network.proxy` | `system` | `system`, `none`, hoặc một URL proxy |
| `network.ca_bundle` | none | Một file bổ sung các certificate authority đáng tin |
| `network.allow_insecure_http` | `false` | Cho phép gửi key qua `http` thường tới host không phải cục bộ |
| `update.check` | `true` | Hỏi GitHub và GitLab về phiên bản mới hơn khi một phiên tương tác bắt đầu |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` mà bước kiểm tra hỏi bản phát hành mới nhất |
| `update.gitlab_repo` | none | GitLab `owner/repo` mà bước kiểm tra cũng hỏi. Bỏ trống thì tắt GitLab |
| `update.timeout_secs` | `3` | Số giây bước kiểm tra được dành cho một nguồn |

Marelle mang theo danh sách certificate authority đáng tin của riêng mình, nên nó vẫn chạy được ở nơi hệ thống không có, như Termux. `SSL_CERT_FILE` thêm vào.

Khi tìm thấy phiên bản mới hơn, session nhắc một lần — phiên bản, bản bạn đang có, và lấy ở đâu. Nó hỏi bản release trước; repository nào chỉ tag các commit sẽ lùi về tag mới nhất của mình. Bất cứ trục trặc gì cũng giữ im lặng.

## Key của bạn vẫn là của bạn

- Cài đặt chỉ nêu tên biến hay file chứa key, không bao giờ nêu key. Một cài đặt kiểu `providers.x.api_key` sẽ bị từ chối.
- Key không được `config show`, `config explain` hay `providers` in ra, và bị loại khỏi mọi văn bản lỗi trả về từ server.
- Nó chỉ được gửi tới địa chỉ của provider, trong một header, và không bao giờ qua `http` thường tới máy khác trừ khi bạn cho phép.
- Redirect không được đi theo, nên key không thể bị chuyển tiếp đi nơi khác.

## Ngôn ngữ

Giao diện nói tiếng Anh, tiếng Nga, tiếng Pháp và tiếng Ukraina. Chọn bằng `--lang <code>` hoặc `MARELLE_LANG`; nếu không thì `LC_ALL`, `LC_MESSAGES` và `LANG` quyết định. Mọi thông điệp nằm trong một catalog, và một test sẽ fail nếu một ngôn ngữ thiếu thông điệp nào. Mục tiêu là mọi ngôn ngữ của Eurasia, rồi nhiều hơn nữa.

## Lộ trình

1. **Trò chuyện.** Các adapter tương thích OpenAI và Anthropic, với cả streaming SSE lẫn giải mã phản hồi đầy đủ. Các lượt agent có tool hiện dùng phản hồi đầy đủ. *Xong.*
2. **Cài đặt và diện mạo.** File cài đặt TOML, toàn cục và theo từng dự án, với `config set`, `get`, `unset` và mô tả cho từng key. Giao diện có chủ đề theo tinh thần [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, bảng thông tin, câu trả lời có khung, spinner, mặc định tím wisteria với mọi màu đặt được dạng HEX. *Xong.*
3. **Nhiều model.** Alias model, hỏi nhiều model cùng lúc và so đáp án, chuỗi fallback, và một model judge gộp chúng lại. *Xong.*
4. **Hành động.** Gọi tool có cấu trúc cho các provider tương thích OpenAI và Anthropic, công cụ file/tìm kiếm/terminal cục bộ, tải URL, skill cục bộ và quyền allow/ask/deny, với transcript hiển thị từng lệnh gọi tool và kết quả của nó. *Xong.*
5. **Ghi nhớ.** Các hội thoại đặt tên được giữ cục bộ giữa các lần chạy. Trí nhớ ngữ nghĩa lâu dài còn ở phía trước.
6. **Mở rộng.** Server MCP, plugin ngoài, subagent và hỗ trợ skill phong phú hơn.
7. **Vươn xa.** Tự động hóa trình duyệt, cổng nhắn tin và tác vụ định kỳ.

Lý do đằng sau thiết kế nằm trong [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Giới hạn đã biết

- Bộ chọn provider và model liệt kê các provider đã cấu hình và alias model. Marelle chưa tự khám phá catalog model từ xa.
- Bộ chọn hỗ trợ nhấp chuột để lấy tiêu điểm và cuộn bằng bánh xe; transcript và trình soạn nhập vẫn điều khiển bằng bàn phím.
- Ctrl-C thoát khỏi phiên tương tác. Nó chưa thể chỉ hủy riêng phản hồi hiện tại.
- Agent loop, công cụ cục bộ có ý thức về quyền hạn và phiên cục bộ bền vững đã được hiện thực, nhưng Marelle vẫn là một prototype pre-alpha; xem [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) để biết thiết kế và ranh giới hiện tại.

## Phát triển

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Test không bao giờ chạm vào mạng: chúng nói với một mock server trong `crates/marelle-testkit`. GitHub Actions cũng build các binary Android và kiểm tra phiên bản Rust tối thiểu đã khai báo. Một pipeline GitLab CI được cung cấp cho bản mirror GitLab; xem [docs/GITLAB.md](docs/GITLAB.md).

## Giấy phép

[AGPL-3.0-only](LICENSE). Thông báo bên thứ ba nằm trong [NOTICE](NOTICE). Bản lưu trữ phát hành có kèm `THIRD-PARTY-LICENSES.txt` cho các crate Rust bên trong binary.
