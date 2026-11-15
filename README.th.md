```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**เอเจนต์ AI ที่ปรับแต่งได้สำหรับเทอร์มินัลของคุณ**
นำ API key ของคุณมาเอง: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, Ollama ในเครื่อง หรือเซิร์ฟเวอร์แบบ OpenAI-compatible ก็ได้ รองรับ Linux และ Termux เป็นอันดับแรก

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · ไทย · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha เวอร์ชัน 0.1.0-alpha.1** TUI แบบเนทีฟ, แชตแบบ multi-provider, agent loop ที่รู้เรื่องสิทธิ์, เครื่องมือในเครื่อง และเซสชันในเครื่องที่เรียกต่อได้ ทำไว้แล้ว API และการตั้งค่ายังเปลี่ยนได้ นี่คือ prototype ไม่ใช่การมีฟีเจอร์เทียบเท่า OpenCode หรือ Hermes

*Marelle* เป็นคำภาษาฝรั่งเศสหมายถึงเกมกระโดดช่อง (hopscotch) แต่ละช่องคือหนึ่งก้าว

## การติดตั้ง

**Termux บน Android** ดาวน์โหลด binary ที่บิลด์ไว้แล้วจาก [Releases](https://github.com/S-I-L-S/marelle/releases) หรือบิลด์บนโทรศัพท์เอง ทีละขั้นตอน: [docs/TERMUX.md](docs/TERMUX.md)

**Linux และที่ใดก็ตามที่ Rust ทำงานได้** Rust 1.85 หรือใหม่กว่า:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

หรือ clone repository แล้วรัน `cargo build --release` ตัว binary จะเป็น `target/release/marelle` ขนาดราว 4 MiB

## แชตครั้งแรก

เก็บ key ไว้ในไฟล์ที่มีแต่คุณอ่านได้ แล้วบอก Marelle ว่าไฟล์นั้นอยู่ที่ไหน Marelle ไม่เก็บ key ลงในการตั้งค่าเด็ดขาด

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

เปลี่ยน `MODEL` เป็นชื่อ model จากรายการของ provider ของคุณ วิธีใช้อื่น ๆ:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

คำตอบไปที่ stdout ส่วน reasoning, จำนวน token, หมายเหตุ และข้อผิดพลาดไปที่ stderr ดังนั้น `marelle chat ... > answer.txt` จะเซฟเฉพาะคำตอบ

## เครื่องมือของเอเจนต์

Marelle ส่งนิยามเครื่องมือแบบมีโครงสร้างผ่าน API แบบ OpenAI-compatible และ Anthropic ชุดเครื่องมือในตัวมีดังนี้

| เครื่องมือ | จุดประสงค์ |
| --- | --- |
| `terminal` | รันคำสั่ง shell จากไดเรกทอรีที่เริ่มรันโปรแกรม |
| `read_file`, `write_file`, `edit_file` | อ่านและแก้ไขไฟล์ข้อความ UTF-8 |
| `list_directory`, `search_text` | ตรวจดูและค้นหาไฟล์ในโปรเจกต์ |
| `fetch_url` | ดึงหน้าเว็บ HTTP(S) สาธารณะมาเป็นข้อความ (ต้องมี `curl`) |
| `list_skills`, `load_skill` | ค้นหาและโหลดคำแนะนำ `SKILL.md` ในเครื่อง |

ใช้ `/tools` เพื่อดูรายการเครื่องมือ และ `/skills` เพื่อดูรายการ skill ที่มี คำแนะนำของโปรเจกต์จะโหลดจาก `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` และไฟล์ `.marelle/` ที่เลือกไว้ เมื่อไฟล์เหล่านั้นอยู่ในพาธที่อนุญาต

เครื่องมือไฟล์ถูกจำกัดอยู่ใน `permissions.allowed_paths` (ค่าเริ่มต้น: ไดเรกทอรีโปรเจกต์ปัจจุบัน) และเคารพ `permissions.denied_paths` การอ่านอนุญาตไว้เป็นค่าเริ่มต้น ส่วนการแก้ไฟล์ คำสั่ง shell และการดึงข้อมูลผ่านเครือข่ายจะถามก่อนรัน สิทธิ์ shell ใช้กับคำสั่งบนเครื่องจริงและไม่ใช่ sandbox ของระบบไฟล์: ควรเก็บไว้ที่ `ask` เว้นแต่คุณจะเจตนาไว้ใจให้ model รันคำสั่ง

เอเจนต์จะหยุดหลังจาก `agent.max_turns` รอบการเรียก tool ข้อความตอบของ tool ถือเป็นเนื้อหาที่ไม่น่าเชื่อถือ คำขอใช้คำตอบของ model แบบสมบูรณ์ เพื่อให้อาร์กิวเมนต์ของ tool ไม่ถูกประกอบจากเศษของ stream ที่ไม่ครบ

## เซสชัน

บทสนทนาถูกเซฟไว้ในเครื่องเป็นค่าเริ่มต้น รวมถึงประวัติการเรียก tool และการตั้งค่าของ model/session บน Unix ไดเรกทอรีเซสชันมี mode `0700` และไฟล์ JSON มี mode `0600` ไฟล์จะเก็บไว้ที่ `$XDG_STATE_HOME/marelle/sessions.json` หรือ `~/.local/state/marelle/sessions.json` หากไม่ได้ตั้ง `XDG_STATE_HOME` API key ไม่เคยถูกเขียนลงไฟล์นี้ ปิดการเก็บบันทึกด้วยคำสั่ง:

```sh
marelle config set agent.persist_sessions false
```

รหัสออก: `0` สำเร็จ `1` ข้อผิดพลาดจาก provider หรือเครือข่าย `2` ข้อผิดพลาดจากวิธีใช้หรือการตั้งค่า

## Providers

| id | โปรโตคอล | ตัวแปร key | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM) ระดับสากล |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, endpoint แบบที่รองรับ Anthropic |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel จีน |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan จีน |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | หลาย model ใน key เดียว |
| `ollama` | OpenAI-compatible | none | Ollama บนเครื่องนี้ |

endpoint ของ Z.ai และ Zhipu มาจากเอกสารของบุคคลที่สาม ไม่ใช่จากหน้าเว็บของผู้ให้บริการเอง โปรดตรวจกับแพ็กเกจของคุณ และ override ฟิลด์ใดก็ตามหากต่างกัน

ทุกฟิลด์ของ provider เปลี่ยนได้ และ provider ใหม่ก็แค่ชื่อหนึ่งพร้อมที่อยู่หนึ่ง:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

ฟิลด์อื่นของ provider: `kind` (`openai-compatible` หรือ `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` และ `extra_body` หากตัวแปรที่ระบุใน `api_key_env` ถูกตั้งไว้ ตัวนั้นจะชนะ มิฉะนั้นจะอ่านจากไฟล์

## การตั้งค่า

การตั้งค่ามาเป็นชั้น ๆ: ค่าเริ่มต้นในตัว ตามด้วยตัวแปรสภาพแวดล้อม แล้วจึงเป็น flag ต่อไปไฟล์ตั้งค่าจะแทรกอยู่ระหว่างค่าเริ่มต้นกับตัวแปรสภาพแวดล้อม

- `MARELLE_CHAT__MODEL=zai/MODEL` ตั้งค่า `chat.model` ขีดล่างสองตัวติดกันแทนจุดหนึ่งตัว
- `marelle config show [prefix]` แสดงรายการการตั้งค่าพร้อมบอกว่าแต่ละค่ามาจากชั้นใด
- `marelle config explain <key>` แสดงทุกชั้นที่ตั้งค่า key นั้น จากลำดับความสำคัญต่ำสุดไปสูงสุด

| Key | ค่าเริ่มต้น | ความหมาย |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | จะคุยกับอะไร `chat.model` เป็น `provider/model` ได้ |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | ถ้าไม่ตั้งจะปล่อยให้ provider เป็นผู้กำหนด |
| `chat.stream` | `true` | stream คำตอบของ judge ที่เป็นตัวเลือก ส่วนเทิร์นของเอเจนต์ที่มี tool ตอนนี้ใช้คำตอบแบบสมบูรณ์ เพื่อให้การเรียก tool ยังสมบูรณ์ |
| `chat.show_reasoning`, `chat.show_usage` | `false` | reasoning และจำนวน token แสดงบน stderr |
| `agent.max_turns` | `20` | จำนวนรอบเรียก tool สูงสุดต่อคำขอของผู้ใช้หนึ่งครั้ง |
| `agent.persist_sessions` | `true` | เก็บบทสนทนาในเครื่องไว้ระหว่างการรัน |
| `permissions.shell` | `ask` | การรันคำสั่ง shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | การอ่านไฟล์: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | การแก้ไฟล์: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | การดึง URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | รากที่อนุญาตสำหรับเครื่องมือไฟล์ |
| `permissions.denied_paths` | none | พาธไฟล์ที่บล็อกไว้ชัดเจน |
| `network.connect_timeout_secs` | `15` | เวลาที่อนุญาตให้เชื่อมต่อ |
| `network.idle_timeout_secs` | `300` | ความเงียบที่อนุญาตระหว่างรอข้อมูล |
| `network.proxy` | `system` | `system`, `none` หรือ URL ของ proxy |
| `network.ca_bundle` | none | ไฟล์เพิ่มเติมของหน่วยออกใบรับรองที่เชื่อถือได้ |
| `network.allow_insecure_http` | `false` | อนุญาตให้ส่ง key ผ่าน `http` ธรรมดาไปยังโฮสต์ที่ไม่ใช่เครื่องท้องถิ่น |
| `update.check` | `true` | ถาม GitHub และ GitLab หาเวอร์ชันที่ใหม่กว่าเมื่อเริ่มเซสชันแบบโต้ตอบ |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` ที่การตรวจสอบถาม release ล่าสุด |
| `update.gitlab_repo` | none | GitLab `owner/repo` ที่การตรวจสอบถามด้วย ถ้าว่างจะปิด GitLab |
| `update.timeout_secs` | `3` | จำนวนวินาทีที่การตรวจสอบใช้กับแหล่งหนึ่งได้ |

Marelle พกรายการหน่วยออกใบรับรองที่เชื่อถือได้มาเอง จึงใช้งานได้แม้ในระบบที่ไม่มี เช่น Termux `SSL_CERT_FILE` เพิ่มเข้ามาได้

เมื่อพบเวอร์ชันที่ใหม่กว่า เซสชันจะพูดถึงเพียงครั้งเดียว — เวอร์ชัน สิ่งที่คุณมี และหาได้จากไหน จะถามหา release ก่อน ถ้า repository แค่ใส่ tag ให้ commit ก็จะไปหา tag ล่าสุดแทน อะไรที่ผิดพลาดจะเงียบไว้

## key ของคุณยังเป็นของคุณ

- การตั้งค่าระบุเพียงชื่อตัวแปรหรือไฟล์ที่เก็บ key เท่านั้น ไม่ใช่ตัว key เอง การตั้งค่าอย่าง `providers.x.api_key` จะถูกปฏิเสธ
- Key ไม่ถูกพิมพ์โดย `config show`, `config explain` หรือ `providers` และถูกลบออกจากข้อความ error ใด ๆ ที่กลับมาจากเซิร์ฟเวอร์
- Key ถูกส่งไปเฉพาะที่อยู่ของ provider ในหนึ่ง header เท่านั้น และไม่เคยส่งผ่าน `http` ธรรมดาไปยังเครื่องอื่น เว้นแต่คุณอนุญาต
- Redirect ไม่ถูกตาม จึงส่งต่อ key ไปที่อื่นไม่ได้

## ภาษา

อินเทอร์เฟซพูดได้ทั้งภาษาอังกฤษ รัสเซีย ฝรั่งเศส และยูเครน เลือกด้วย `--lang <code>` หรือ `MARELLE_LANG` ถ้าไม่ระบุ `LC_ALL`, `LC_MESSAGES` และ `LANG` จะเป็นผู้ตัดสิน ทุกข้อความอยู่ใน catalog และจะมี test ล้มเหลวทันทีที่ภาษาใดขาดข้อความสักข้อ เป้าหมายคือทุกภาษาในยูเรเซีย แล้วไปให้ถึงภาษาอื่นต่อไป

## แผนงาน

1. **คุย** Adapter สำหรับ OpenAI-compatible และ Anthropic รองรับทั้ง SSE streaming และการถอดรหัสคำตอบแบบสมบูรณ์ เทิร์นของเอเจนต์ที่มี tool ตอนนี้ใช้คำตอบแบบสมบูรณ์ *เสร็จแล้ว*
2. **การตั้งค่าและหน้าตา** ไฟล์ตั้งค่า TOML ทั้งระดับ global และรายโปรเจกต์ พร้อม `config set`, `get`, `unset` และคำอธิบายของทุก key อินเทอร์เฟซมีธีมตามแนวคิดของ [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, แผงข้อมูล, คำตอบมีกรอบ, spinner, สีม่วง wisteria เป็นค่าเริ่มต้น และตั้งสีใดก็ได้เป็น HEX *เสร็จแล้ว*
3. **หลาย model** Model alias การถามหลาย model พร้อมกันแล้วเทียบคำตอบ ลูกโซ่ fallback และ model ตัวตัดสินที่รวมคำตอบให้ *เสร็จแล้ว*
4. **ลงมือทำ** การเรียก tool แบบมีโครงสร้างสำหรับ provider แบบ OpenAI-compatible และ Anthropic เครื่องมือไฟล์/ค้นหา/เทอร์มินัลในเครื่อง การดึง URL, skill ในเครื่อง และสิทธิ์แบบ allow/ask/deny โดย transcript แสดงทุกการเรียก tool พร้อมผลลัพธ์ *เสร็จแล้ว*
5. **จดจำ** บทสนทนาที่ตั้งชื่อไว้อยู่คงอยู่ในเครื่องระหว่างการรัน ความจำเชิงความหมายระยะยาวยังอยู่ข้างหน้า
6. **ขยายตัว** MCP server, ปลั๊กอินภายนอก, subagent และการรองรับ skill ที่หลากหลายขึ้น
7. **ไปให้ไกล** การควบคุมเบราว์เซอร์อัตโนมัติ, ทางเข้าสำหรับข้อความ และงานตามกำหนดเวลา

เหตุผลเบื้องหลังการออกแบบอยู่ใน [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

## ข้อจำกัดที่ทราบแล้ว

- ตัวเลือก provider และ model แสดง provider ที่ตั้งค่าไว้และ model alias Marelle ยังไม่ค้นหา catalog ของ model จากระยะไกล
- ตัวเลือกรองรับการคลิกด้วยเมาส์เพื่อโฟกัสและการเลื่อนด้วยล้อเลื่อน ส่วน transcript และตัวแก้ข้อความป้อนยังใช้คีย์บอร์ดเป็นหลัก
- Ctrl-C ออกจากเซสชันแบบโต้ตอบ ยังไม่ยกเลิกเฉพาะคำตอบที่กำลังรับ
- Agent loop, เครื่องมือในเครื่องที่รู้เรื่องสิทธิ์ และเซสชันในเครื่องที่อยู่ถาวร ทำไว้แล้ว แต่ Marelle ยังเป็น prototype ระดับ pre-alpha ดู [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) สำหรับการออกแบบและขอบเขตปัจจุบัน

## การพัฒนา

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Test ไม่แตะเครือข่ายเลย: พวกมันคุยกับ mock server ใน `crates/marelle-testkit` GitHub Actions ก็บิลด์ binary ของ Android และตรวจเวอร์ชัน Rust ขั้นต่ำที่ประกาศไว้ด้วย มี pipeline ของ GitLab CI ให้สำหรับ mirror บน GitLab ดู [docs/GITLAB.md](docs/GITLAB.md)

## สัญญาอนุญาต

[AGPL-3.0-only](LICENSE) ข้อความแจ้งของบุคคลที่สามอยู่ใน [NOTICE](NOTICE) ไฟล์เก็บถาวรของ release มี `THIRD-PARTY-LICENSES.txt` สำหรับ crate Rust ที่อยู่ใน binary
