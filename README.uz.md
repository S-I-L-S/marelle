```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Terminalingiz uchun sozlanadigan SI agent.**
O'z API kalitlaringizni olib keling: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, mahalliy Ollama yoki OpenAI bilan mos keladigan istalgan server. Avval Linux va Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · Oʻzbekcha
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Oldin-alfa, versiya 0.1.0-alpha.1.** Amalga oshirildi: mahalliy TUI interfeysi, bir necha ta'minotchi bilan suhbat, ruxsatlarni biluvchi agent sikli, mahalliy vositalar va davom ettiriladigan mahalliy sessiyalar. API va sozlamalar hali ham o'zgarishi mumkin. Bu prototip, OpenCode yoki Hermes bilan funksiyalar tengligi emas.

*Marelle* — bolalar sakrash o'yinining fransuzcha nomi. Har bir kvadrat — bir qadam.

## O'rnatish

**Android'dagi Termux.** Tayyor ikkilik faylni [chiqarishlar](https://github.com/S-I-L-S/marelle/releases) sahifasidan yuklab oling, yoki telefon qiling. Qadam-qadam qo'llanma: [docs/TERMUX.md](docs/TERMUX.md).

**Linux va Rust ishlaydigan hamma joyda.** Rust 1.85 yoki yangiroq kerak:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Yoki omborni klonlang va `cargo build --release` buyrug'ini bajaring. Ikkilik fayl — `target/release/marelle`, atigi 4 MiB.

## Birinchi suhbat

Kalitni faqat siz o'qiydigan faylda saqlang va Marelle'ga qayerdaligini aytng. Marelle kalitni hech qachon sozlamalarida saqlamaydi.

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

`MODEL` o'rniga ta'minotchining ro'yxatidan model nomini qo'ying. Foydalanishning boshqa yo'llari:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Javob stdout'ga boradi. Mulohazalar, tokenlar sanog'i, izohlar va xatolar stderr'ga boradi; shunday qilib `marelle chat ... > answer.txt` faqat javobni saqlaydi.

## Agent vositalari

Marelle OpenAI bilan mos keladigan va Anthropic API orqali tuzilgan vositalar ta'riflarini yuboradi. Ichiga qo'yilgan vositalar to'plami:

| Vosita | Maqsad |
| --- | --- |
| `terminal` | Ishga tushirish papkasidan qobiq buyrug'ini bajarish |
| `read_file`, `write_file`, `edit_file` | UTF-8 matnli fayllarni o'qish va o'zgartirish |
| `list_directory`, `search_text` | Loyiha fayllarini ko'zdan kechirish va izlash |
| `fetch_url` | Ommaviy HTTP(S) sahifasini matn sifatida olib kelish (`curl` kerak) |
| `list_skills`, `load_skill` | Mahalliy `SKILL.md` ko'rsatmalarini topish va yuklash |

`/tools` buyrug'i vositalar ro'yxatini, `/skills` — mavjud ko'nikmalarni ko'rsatadi. Loyiha ko'rsatmalari `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` va tanlangan `.marelle/` fayllaridan, ruxsat berilgan yo'llardalari bo'lsa, yuklanadi.

Fayl vositalari `permissions.allowed_paths` bilan cheklanadi (standartda: loyihaning joriy papkasi) va `permissions.denied_paths`'ni hisobga oladi. O'qish standartda ruxsat etilgan; fayllarni o'zgartirish, qobiq buyruqlari va tarmoqdan olish avval so'raydi. Qobiq ruxsati mezbandagi buyruqlarga tegishli va fayl tizimining qum qutisi emas: modelga buyruqlar bajarilishini bilbilara ishonmagan bo'lsangiz `ask`'da tuting.

Agent `agent.max_turns` vosita turidan keyin to'xtaydi. Vosita javoblari ishonchsiz mazmun deb hisoblanadi. So'rovlar modelning to'liq javoblaridan foydalanadi; shuning uchun vosita argumentlari oqimning tugallanmagan bo'laklaridan yig'ilmaydi.

## Sessiyalar

Suhbatlar standartda mahalliy saqlanadi; vosita chaqiruvlari tarixi bilan, model va sessiya sozlamalari bilan birga. Unix'da sessiya papkasining rejimi `0700`, JSON faylningki `0600`. Fayl `$XDG_STATE_HOME/marelle/sessions.json` joyida, yoki `XDG_STATE_HOME` qo'yilmagan bo'lsa `~/.local/state/marelle/sessions.json` joyida saqlanadi. API kalitlari bu faylga hech qachon yozilmaydi. Saqlashni shu bilan o'chirib qo'ying:

```sh
marelle config set agent.persist_sessions false
```

Chiqish kodlari: `0` muvaffaqiyat, `1` ta'minotchi yoki tarmoq xatosi, `2` ishlatish yoki tuzilish xatosi.

## Ta'minotchilar

| id | Protokol | Kalit o'zgaruvchisi | |
| --- | --- | --- | --- |
| `zai` | OpenAI bilan mos | `ZAI_API_KEY` | Z.ai (Zhipu GLM), xalqaro |
| `zai_coding` | OpenAI bilan mos | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic bilan mos oxirgi nuqta |
| `zhipu` | OpenAI bilan mos | `ZHIPUAI_API_KEY` | Zhipu BigModel, Xitoy |
| `zhipu_coding` | OpenAI bilan mos | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Xitoy |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI bilan mos | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI bilan mos | `OPENROUTER_API_KEY` | bir kalit ortida ko'p model |
| `ollama` | OpenAI bilan mos | yo'q | Shu mashinada Ollama |

Z.ai va Zhipu oxirgi nuqtalari uchinchi tomon hujjatlaridan keladi, ta'minotchining o'z sahifalaridan emas. Ularni tarifingiz bilan solishtiring, farq qilsa istalgan maydonni ustidan yozing.

Ta'minotchining har bir maydoni o'zgartiriladi; yangi ta'minotchi — oddiygina nom bilan manzil:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Ta'minotchining boshqa maydonlari: `kind` (`openai-compatible` yoki `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` va `extra_body`. `api_key_env` atagan o'zgaruvchi qo'yilgan bo'lsa, u g'olib; bo'lmasa fayl o'qiladi.

## Sozlamalar

Sozlamalar qatlamlar bilan keladi: ichki standart qiymatlar, keyin muhit o'zgaruvchilari, keyin bayroqlar. Sozlama fayllari standartlar bilan muhit orasiga tushadi.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model` qo'yadi. Ikki pastki chiziq nuqta o'rnida turadi.
- `marelle config show [prefix]` — sozlamalarni va har birining qaysi qatlamdan kelganini sanab beradi.
- `marelle config explain <key>` — kalitni qo'ygan har qatlamni, eng pastdan eng yuqori ustuvorlikkacha ko'rsatadi.

| Kalit | Standart | Ma'nosi |
| --- | --- | --- |
| `chat.provider`, `chat.model` | yo'q | Kim bilan gaplashish. `chat.model` `provider/model` bo'lishi mumkin. |
| `chat.system` | yo'q | Tizimli taklif |
| `chat.max_tokens`, `chat.temperature` | yo'q | Qo'yilmagan — ta'minotchiga qoldiriladi |
| `chat.stream` | `true` | Hakim modelning xohishi bo'yicha javob oqimi; vositali agent qadamlari hozircha to'liq javoblardan foydalanadi, vosita chaqiruvlari butun qolishi uchun |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Mulohazalar va tokenlar sanog'i stderr'da |
| `agent.max_turns` | `20` | Foydalanuvchi bitta so'roviga vosita aylanmalarining eng ko'pi |
| `agent.persist_sessions` | `true` | Mahalliy suhbatlarni ishga tushirishlar oralig'ida saqlash |
| `permissions.shell` | `ask` | Qobiq buyruqlarini bajarish: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Fayl o'qish: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Fayl o'zgartirish: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL yuklab olish: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Fayl vositalari uchun ruxsat etilgan ildizlar |
| `permissions.denied_paths` | yo'q | Aniq bloklangan fayl yo'llari |
| `network.connect_timeout_secs` | `15` | Ulanish uchun berilgan vaqt |
| `network.idle_timeout_secs` | `300` | Ma'lumot kutayotganda ruxsat etilgan sukut |
| `network.proxy` | `system` | `system`, `none`, yoki proksi manzili |
| `network.ca_bundle` | yo'q | Ishonchli sertifikat beruvchi markazlarning qo'shimcha fayli |
| `network.allow_insecure_http` | `false` | Kalitning yalang'och `http` orqali boshqa xostga borishiga ruxsat |
| `update.check` | `true` | Interaktiv sessiya boshlanishida GitHub va GitLab'dan yangiroq versiya so'rash |
| `update.github_repo` | `S-I-L-S/marelle` | Tekshiruv eng yangi chiqarishni so'raydigan GitHub `owner/repo` |
| `update.gitlab_repo` | yo'q | O'sha tekshiruv uchun GitLab `owner/repo`. Bo'sh bo'lsa GitLab o'chiriladi |
| `update.timeout_secs` | `3` | Tekshiruv bitta manbaga necha sekund sarflashi mumkin |

Marelle o'zi bilan ishonchli sertifikat markazlarining o'z ro'yxatini olib yuradi; shuning uchun tizimda bo'lmagan joyda ham, Termux'dagi kabi ishlaydi. `SSL_CERT_FILE` yana qo'shadi.

Yangiroq versiya topsa, sessiya uni bir marta eslatadi — versiyani, sizda turganini va qayerdan olish kerakligini. Avval chiqarish so'raladi; commitlarini faqat yorliqlar bilan belgilaydigan ombor eng yangi yorliqni beradi. Nima g'oyb bo'lsa, jim qoladi.

## Kalitlaringiz sizniki bo'lib qoladi

- Sozlamalar kalit yotgan o'zgaruvchini yoki faylni nomlaydi, hech qachon kalitning o'zini emas. `providers.x.api_key` kabi sozlama rad etiladi.
- Kalitni `config show`, `config explain` yoki `providers` chop etmaydi, va u serverdan kelgan har xato matnidan kesib olinadi.
- U faqat ta'minotchining manziliga, bir sarlavhada yuboriladi, va siz ruxsat bermaguningizcha aslo yalang'och `http` orqali boshqa mashinaga bormaydi.
- Qayta yo'naltirishlarga ergashilmaydi; shuning uchun kalit boshqa yoqqa o'tkazilishi mumkin emas.

## Tillar

Interfeys ingliz, rus, fransuz va ukrain tillarida gapiradi. `--lang <code>` yoki `MARELLE_LANG` bilan tanlang; aks holda `LC_ALL`, `LC_MESSAGES` va `LANG` hal qiladi. Har bir xabar katalogda yashaydi, va bir til bittasini tashlab qo'ysa test yiqiladi. Maqsad — Yevrosiyo ning har bir tili, keyin ortiq.

## Yo'l xaritasi

1. **Gapirish.** OpenAI bilan mos va Anthropic moslashuvchilari, SSE oqimi ham, to'liq javoblarni o'qish ham bo'lgan. Vositali agent qadamlari hozircha to'liq javoblardan foydalanadi. *Bajarildi.*
2. **Sozlamalar va ko'rinish.** TOML sozlama fayllari, umumiy va loyiha bo'yicha, `config set`, `get` va `unset` bilan, har kalitning ta'rifi bilan. [Hermes Agent](https://github.com/NousResearch/hermes-agent) ruhida mavzulashtirilgan interfeys: banner, ma'lumot paneli, ramkadagi javoblar, aylanuvchi, standartda gliciniya binafsharangida, istalgan rang HEX sifatida. *Bajarildi.*
3. **Ko'p model.** Model taxalluslari, bir necha modelga birdan so'rash va javoblarni solishtirish, zaxira zanjirlari, va ularni erituvchi hakim model. *Bajarildi.*
4. **Ish tutish.** OpenAI bilan mos va Anthropic ta'minotchilari uchun tuzilgan vosita chaqiruvlari, mahalliy fayl/izlash/terminal vositalari, URL yuklab olish, mahalliy ko'nikmalar va allow/ask/deny ruxsatlari; yozuv har vosita chaqiruvini va natijasini ko'rsatadi. *Bajarildi.*
5. **Esda saqlash.** Nomlangan suhbatlar ishga tushirishlar oralig'ida mahalliy qoladi. Uzoq muddatli ma'noli xotira hali oldinda.
6. **Kengaytirish.** MCP serverlari, tashqi qo'shimchalar, sub-agentlar va ko'nikmalarga boyroq qo'llab-quvvatlash.
7. **Yetib borish.** Brauzerni avtomatlashtirish, xabar yuboruvchi darvozalari va rejalashtirilgan vazifalar.

Dizayn ortidagi mulohaza [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-da.

## Ma'lum cheklovlar

- Ta'minotchi va model tanlagichlar qo'yilgan ta'minotchilarni va model taxalluslarini sanab o'tadi. Marelle hali masofadagi model kataloglarini ochmaydi.
- Tanlagichlar fokus uchun sichqoncha bosishini va g'ildirak bilan aylantirishni qo'llaydi; yozuv va kiritish muharriri klaviaturadan boshqariladi.
- Ctrl-C interaktiv sessiyadan chiqadi. Hali joriy javobni to'xtatmaydi.
- Agent sikli, ruxsatlardan xabardor mahalliy vositalar va doimiy mahalliy sessiyalar amalga oshirildi; lekin Marelle oldin-alfa prototip bo'lib qoladi. Hozirgi dizayn va chegaralar uchun [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) o'qing.

## Rivojlantirish

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Sinovlar tarmoqqa hech qachon tegmaydi: ular `crates/marelle-testkit` dagi soxta server bilan gaplashadi. GitHub Actions Android ikkiliklarini ham tayyorlaydi va e'lon qilingan eng past Rust versiyasini tekshiradi. GitLab oynasi uchun GitLab CI liniyasi bor; ko'ring [docs/GITLAB.md](docs/GITLAB.md).

## Litsenziya

[AGPL-3.0-only](LICENSE). Uchinchi tomon xabarnomalari [NOTICE](NOTICE)-da. Chiqarish arxivlari ikkilik ichidagi Rust qutilari uchun `THIRD-PARTY-LICENSES.txt` o'z ichiga oladi.
