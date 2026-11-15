```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Терминалд зориулсан тохируулж болох ХОЁУГУЙ ухааны агент.**
Өөрийн API түлхүүрүүдээ авчирч тавь: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, орон нутгийн Ollama, эсвэл OpenAI-тай нийцэх бүхий л сервер. Юун түрүүнд Linux ба Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · Монгол · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Өмнөх-альфа, хувилбар 0.1.0-alpha.1.** Хэрэгжүүлсэн: уугуул TUI интерфэйс, олон ханган нийлүүлэгчтэй чат, зөвшөөрлийг мэдэрсэн агентийн цикл, орон нутгийн багаж хэрэгсэл болон үргэлжлүүлэх боломжтой орон нутгийн сешнүүд. API-ууд болон тохиргоо хүртэл өөрчлөгдөж болно. Энэ нь прототип, OpenCode эсвэл Hermes-тэй тэнцэхүүн биш.

*Marelle* — хүүхдийн үсрэлтийн тоглоомын франц нэр. Квадрат бүр нэг алхам юм.

## Суулгах

**Андройд дээрх Termux.** Бэлэн бинарийг [гаргасан хувилбаруудын](https://github.com/S-I-L-S/marelle/releases) хуудаснаас татаж ав, эсвэл утаснаас угсар. Алхам алхмаар гарын авлага: [docs/TERMUX.md](docs/TERMUX.md).

**Линукс болон Rust ажилладаг бүх газарт.** Rust 1.85 эсвэл түүнээс шинэ хэрэгтэй:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Эсвэл сан хуваарийг клон хийгээд `cargo build --release` ажиллуул. Бинар файл нь `target/release/marelle`, орчим 4 МиБ.

## Эхний чат

Түлхүүрээ зөвхөн чи унших файлд хадгалж, Marelle-т хаана байгааг хэл. Marelle түлхүүрийг тохиргоондоо хэзээ ч хадгалахгүй.

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

`MODEL`-ий оронд ханган нийлүүлэгчийн жагсаалтаас моделийн нэрийг тавь. Ашиглах бусад аргууд:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Хариулт stdout руу явна. Мэдэгдэлүүд, токений тоо, тайлбар тэмдэглэл болон алдаа stderr руу явна; тиймээс `marelle chat ... > answer.txt` зөвхөн хариултыг хадгална.

## Агентийн багажууд

Marelle OpenAI-тай нийцсэн болон Anthropic API-гаар бүтэцлэсэн багажийн тодорхойлолтууд илгээдэг. Угсарсан багажийн багц:

| Багаж | Зорилго |
| --- | --- |
| `terminal` | Ажиллуулах хавтаснаас бүрхүүлийн тушаал ажиллуулах |
| `read_file`, `write_file`, `edit_file` | UTF-8 текст файлуудыг унших, засварлах |
| `list_directory`, `search_text` | Төслийн файлуудыг харах болон хайх |
| `fetch_url` | Нийтийн HTTP(S) хуудсыг текстээр авах (`curl` шаардлагатай) |
| `list_skills`, `load_skill` | Орон нутгийн `SKILL.md` заавруудыг олж, ачаалах |

`/tools` тушаал багажийн жагсаалтыг, `/skills` боломжит чадваруудыг харуулна. Төслийн зааврууд `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` болон сонгосон `.marelle/` файлуудаас, зөвшөөрсэн замд байвал, ачигдана.

Файлын багажууд `permissions.allowed_paths`-д хязгаарлагдана (үндсэн: төслийн одоогийн хавтас) ба `permissions.denied_paths`-ийг хүндэтгэнэ. Унших үндсэндээ зөвшөөрөгдсөн; файл засварлах, бүрхүүлийн тушаал болон сүлжээнээс татах зүйлс урьд илгээх. Бүрхүүлийн зөвшөөрөл гаднах машины тушаалд хамаарна ба файлын системийн элсэн хайрцаг биш: загварыг тушаал гүйцэтгэхэд тусгайлан итгэсэн болохгүй бол `ask`-д хадгал.

Агент `agent.max_turns` багажын эргэлтийн дараа зогсно. Багажийн хариултуудыг итгэхгүй агуулга гэж үзнэ. Хүсэлтүүд загварын бүрэн хариултыг ашигладаг; тиймээс багажийн аргументууд урсгалын дутуу хэсгүүдээс бүрдэхгүй.

## Сешнүүд

Харилцаанууд үндсэндээ орон нутагт хадгалагдана; багажийн дуудлагын түүх болон загвар ба сешний тохиргоог хамтад нь. Юникс дээр сешний хавтас дэглэмтэй `0700`, JSON файл `0600` байна. Файл `$XDG_STATE_HOME/marelle/sessions.json` газар, эсвэл `XDG_STATE_HOME` тавигдаагүй үед `~/.local/state/marelle/sessions.json` газар хадгалагдана. API түлхүүрүүд энэ файл руу бичигдэхгүй. Хадгалахыг дараахаар унтраа:

```sh
marelle config set agent.persist_sessions false
```

Гарах кодууд: `0` амжилт, `1` ханган нийлүүлэгч болон сүлжээний алдаа, `2` ашиглалтын эсвэл тохиргооны алдаа.

## Ханган нийлүүлэгчид

| id | Протокол | Түлхүүрийн хувьсагч | |
| --- | --- | --- | --- |
| `zai` | OpenAI-тай нийцсэн | `ZAI_API_KEY` | Z.ai (Zhipu GLM), олон улсын |
| `zai_coding` | OpenAI-тай нийцсэн | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-тай нийцсэн төгсгөлийн цэг |
| `zhipu` | OpenAI-тай нийцсэн | `ZHIPUAI_API_KEY` | Zhipu BigModel, Хятад |
| `zhipu_coding` | OpenAI-тай нийцсэн | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Хятад |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-тай нийцсэн | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-тай нийцсэн | `OPENROUTER_API_KEY` | нэг түлхүүр ард олон загвар |
| `ollama` | OpenAI-тай нийцсэн | байхгүй | Энэ машин дээр Ollama |

Z.ai болон Zhipu төгсгөлийн цэгүүд гуравдагч этгээдийн баримт бичигээс ирдэг, ханган нийлүүлэгчийн өөрийн хуудаснаас биш. Тэдгээрийг өөрийн багцаар шалгаж, ялгаатай бол тал бүрийг дээрээс нь бич.

Ханган нийлүүлэгчийн тал бүр өөрчлөгдөх боломжтой; шинэ ханган нийлүүлэгч бол нэг нэр, нэг хаяг гэтэх юм.

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Ханган нийлүүлэгчийн бусад талбарууд: `kind` (`openai-compatible` эсвэл `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` болон `extra_body`. `api_key_env`-гаар нэрлэсэн хувьсагч тавигдсан бол тэр хождогч; үгүй бол файл уншигдахаа.

## Тохиргоо

Тохиргоонууд давхаргаар ирнэ: угсарсан анхдагч утгууд, дараа нь орчны хувьсагчид, дараа нь далбаанууд. Тохиргооны файлууд анхдагчууд ба орчны дунд ордог.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model`-ийг тогтооно. Давхар зурвас цэгийн оронд зогсно.
- `marelle config show [prefix]` — тохиргоонууд болон аль давхаргаас ирсэнийг жагсаана.
- `marelle config explain <key>` — түлхүүрийг тогтоосон давхарга бүрийг, доод зэргээс дээд хүртэл харуулна.

| Түлхүүр | Анхдагч | Утга |
| --- | --- | --- |
| `chat.provider`, `chat.model` | байхгүй | Хэнтэй ярилцах. `chat.model` нь `provider/model` байж болно. |
| `chat.system` | байхгүй | Системийн өгүүлбэр |
| `chat.max_tokens`, `chat.temperature` | байхгүй | Тогтоогдоогүй — ханган нийлүүлэгчид үлдэгдэнэ |
| `chat.stream` | `true` | Шүүгчийн үзэсэл өгөх хариултын урсгал; багаж бүхий агентийн шат өнөөдөр бүрэн хариултуудыг ашигладаг, багажийн дуудлагууд бүхэлээрээ үлдэх тул |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Мэдэгдлүүд болон токений тоо stderr дээр |
| `agent.max_turns` | `20` | Нэг хэрэглэгчийн хүсэлт тутамд багажийн эргэлтийн дээд хязгаар |
| `agent.persist_sessions` | `true` | Орон нутгийн яриа харилцааг ажиллуулалт хооронд хадгалах |
| `permissions.shell` | `ask` | Бүрхүүлийн тушаал ажиллуулах: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Файл унших: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Файл өөрчлөх: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL татах: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Файлын багажуудад зөвшөөрсэн үндсүүд |
| `permissions.denied_paths` | байхгүй | Тодорхой блоклосон файлын замууд |
| `network.connect_timeout_secs` | `15` | Холбох зөвшөөрөгдсөн хугацаа |
| `network.idle_timeout_secs` | `300` | Мэдээлэл хүлээхдээ зөвшөөрсэн чимээгүй байдал |
| `network.proxy` | `system` | `system`, `none`, эсвэл прокси хаяг |
| `network.ca_bundle` | байхгүй | Итгэмжлэгдсэн гэрчилгээ олгох байгууллагын нэмэлт файл |
| `network.allow_insecure_http` | `false` | Түлхүүрийг энгийн `http`-ээр өөр хост руу зөвшөөрөх |
| `update.check` | `true` | Интерактив сешний эхлэлд GitHub болон GitLab-аас шинэ хувилбар байгаа эсэхийг асуух |
| `update.github_repo` | `S-I-L-S/marelle` | Шалгалт хамгийн сүүлчийн гаргасан хувилбарыг асуух GitHub `owner/repo` |
| `update.gitlab_repo` | байхгүй | Мөнхүү шалгалтын GitLab `owner/repo`. Хоосон бол GitLab унтраана |
| `update.timeout_secs` | `3` | Шалгалт нэг эх сурвалжад хэдэн секунд зарцуулах вэ |

Marelle итгэмжлэгдсэн гэрчилгээ олгох байгууллагуудын өөрийн жагсаалтыг зөөдөг; тиймээс системд байхгүй газар, Termux-ын адил ажиллана. `SSL_CERT_FILE` бас нэмж өгнө.

Шинэ хувилбар олдоход сешний хувилбарын тухай нэг л удаа дурдана — хувилбар, таны байгааг, хаанаас авахыг. Хамгийн түрүүнд гаргасан хувилбарыг асууна; зөвхөн бичил өөрчлөлтөө тэмдэгтээр тэмдэглэдэг сан эхний тэмдэгтийг өгнө. Ямар ч алдаа гарсан чүү байгаатай.

## Түлхүүрүүд чиний байсаар байна

- Тохиргоонууд түлхүүрийг агуулсан хувьсагч эсвэл файлыг нэрлэдэг, хэзээ ч түлхүүр биш. `providers.x.api_key` мэт тохиргоог үүрэг бүрдүүлэгдэхгүй.
- Түлхүүрийг `config show`, `config explain` эсвэл `providers` хэвлэдэггүй, серверээс эргэн ирсэн алдааны бүх текстээс таслана.
- Энэ ханган нийлүүлэгчийн хаяг руу л, нэг толгойд илгээгддэг, ба зөвшөөрсөн тохиолдолдоо энгийн `http`-ээр өөр машин руу биш.
- Даиретуудийг дагадаггүй; тиймээс түлхүүр өөр газар хүргэгдэхгүй.

## Хэлүүд

Интерфэйс англи, орос, франц, украин хэл дээр ярьдаг. `--lang <code>` эсвэл `MARELLE_LANG`-аар сонго; үгүй бол `LC_ALL`, `LC_MESSAGES` болон `LANG` шийддэг. Бүх мэссэж каталогт амьдардаг, хэл нэг мессэж үлдээвэл тест унана. Зорилго — Евразийн бүх хэл, дараа нь дахин олон.

## Замын зураг

1. **Ярих.** OpenAI-тай нийцсэн болон Anthropic адаптерүүд, SSE урсгал болон бүрэн хариултын тайлал хоёуланг ашигласан. Багаж бүхий агентийн шат өнөөдөр бүрэн хариултуудыг ашигладаг. *Бүтсэн.*
2. **Тохиргоо болон үзэмж.** TOML тохиргооны файлууд, ерөнхий болон төсөө бүрээр, `config set`, `get` болон `unset`-тэй, бүхий л түлхүүрийн тайлбар бүхий. [Hermes Agent](https://github.com/NousResearch/hermes-agent)-ын сэтгэл хөдлөлүүдийн сэтгэлзүйн темабилдаг интерфэйс: баннер, мэдээллийн самбар, хүрээнд багтсан хариултууд, эргэдэгч, анхдагч гүнлэг ногоо цэцгийн ногоонд, бүхий л өнгө HEX түвшинд тохируулагдах. *Бүтсэн.*
3. **Олон загварууд.** Загварын нэрс, хэд хэдэн загварыг зэрэгцүүлэн асуух, хариултуудыг харьцуулах, дэд шийдвэрүүд, болон тэдгээрийг нэгтгэдэг шүүгч загвар. *Бүтсэн.*
4. **Үйлдэл хийх.** OpenAI-тай нийцсэн болон Anthropic ханган нийлүүлэгчдийн зориулалттай бүтэцлэсэн багажийн дуудлагууд, орон нутгийн файл/хайлт/терминал багажууд, URL татах, орон нутгийн чадварууд болон allow/ask/deny зөвшөөрлүүд; бичилт бүхий багажийн дуудлага болон түүний үр дүнг харуулна. *Бүтсэн.*
5. **Санах.** Нэрлэсэн харилцаанууд ажиллуулалт хооронд орон нутагт үлдэж байна. Урт хугацааны семантик ой хараахан урд нь.
6. **Өргөтгөх.** MCP серверүүд, гаднах нэмэлтүүд, дэд-агентүүд болон чадваруудын илүү баян дэмжлэг.
7. **Хүрэх.** Хөтчийн автоматжуулалт, мессенжерийн гарцууд болон төлөвлөсөн ажлууд.

Дизайны ард байгаа бодол [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-д байдаг.

## Мэдэгдэж байгаа хязгаарлалтууд

- Ханган нийлүүлэгч болон загварын сонгогчид тохируулсан ханган нийлүүлэгчдийг болон загварын нэрийг жагсаана. Marelle алсын загварын каталогуудыг хараахан олддоггүй.
- Сонгогчид фокус авахад хулганы товшилт болон дугуйн гүйлгээг дэмждэг; бичилт болон оролт засварлагч гарын авлагаар удирдагддаг.
- Ctrl-C интерактив сессийн бүтэнээс гадагшаа гарна. Одоогоор зөвхөн үргэлжилсэн хариултыг зогсоодоггүй.
- Агентийн цикл, зөвшөөрлийг мэддэг орон нутгийн багажууд болон оршин тогтнох орон нутгийн сешнүүд хэрэгжсэн; гэхдээ Marelle өмнөх-альфа прототип хэвээрээ. Одоогийн дизайны болон хязгаарыг [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-с уншина уу.

## Хөгжүүлэлт

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Тестүүд сүлжээнд хэзээ ч хүрдэггүй: тэд `crates/marelle-testkit` доторх зохиомол сервертэй харьцдаг. GitHub Actions Android-ын бинар файлуудыг ч мөн бүтээд, зарлагдсан хамгийн бага Rust-ийн хувилбарыг шалгадаг. GitLab тольд зориулсан GitLab CI хоолойд байдаг; үзэх [docs/GITLAB.md](docs/GITLAB.md).

## Лиценз

[AGPL-3.0-only](LICENSE). Гуравдагч этгээдийн тэмдэглэлүүд [NOTICE](NOTICE)-д байдаг. Гаргасан хувилбарын архивууд бинар доторх Rust хайрцагнуудад зориулсан `THIRD-PARTY-LICENSES.txt` агуулдаг.
