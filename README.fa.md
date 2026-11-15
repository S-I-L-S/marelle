```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**یک عامل هوش مصنوعی پیکربندی‌شدنی برای پایانه شما.**
کلیدهای API خودتان را بیاورید: Z.ai (Zhipu GLM)، Anthropic، OpenAI، OpenRouter، یک Ollama محلی، یا هر سرور سازگار با OpenAI. نخست لینوکس و Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · فارسی
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **پیش‌آلفا، نسخه ۰.۱.۰-alpha.1.** پیاده‌شده: رابط بومی TUI، گفت‌وگوی چندتأمین‌کننده، حلقه عامل آگاه از مجوزها، ابزارهای محلی و نشست‌های محلی ازسرگرفتنی. APIها و تنظیمات ممکن است هنوز تغییر کنند. این یک نمونه اولیه است، نه هم‌ارزی ویژگی‌ها با OpenCode یا Hermes.

*Marelle* نام فرانسوی بازی پریدن روی خانه‌ها است. هر خانه یک قدم است.

## نصب

**Termux روی اندروید.** باینری آماده را از صفحه [انتشارات](https://github.com/S-I-L-S/marelle/releases) بگیرید، یا روی گوشی بسازید. راهنمای گام‌به‌گام: [docs/TERMUX.md](docs/TERMUX.md).

**لینوکس و هرجا که Rust کار می‌کند.** Rust 1.85 یا تازه‌تر لازم است:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

یا مخزن را کلون کنید و `cargo build --release` را اجرا کنید. باینری `target/release/marelle` است، حدود ۴ مِیبایت.

## نخستین گفت‌وگو

کلید را در فایلی نگه دارید که فقط شما می‌خوانید، و به Marelle بگویید کجاست. Marelle هرگز کلیدی را در تنظیماتش ذخیره نمی‌کند.

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

`MODEL` را با نام مدلی از فهرست تأمین‌کننده‌تان عوض کنید. شیوه‌های دیگر استفاده:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

پاسخ به stdout می‌رود. استدلال‌ها، شمارش توکن‌ها، یادداشت‌ها و خطاها به stderr می‌روند؛ پس `marelle chat ... > answer.txt` فقط پاسخ را ذخیره می‌کند.

## ابزارهای عامل

Marelle تعریف‌های ساخت‌یافته ابزارها را از طریق APIهای سازگار با OpenAI و Anthropic می‌فرستد. مجموعه ابزارهای درون‌ساز:

| ابزار | هدف |
| --- | --- |
| `terminal` | اجرای فرمان پوسته از پوشه راه‌اندازی |
| `read_file`, `write_file`, `edit_file` | خواندن و تغییر فایل‌های متنی UTF-8 |
| `list_directory`, `search_text` | بازدید و جست‌وجوی فایل‌های پروژه |
| `fetch_url` | واکشی صفحه عمومی HTTP(S) به‌صورت متن (نیازمند `curl`) |
| `list_skills`, `load_skill` | کشف و بارگذاری دستورالعمل‌های محلی `SKILL.md` |

فرمان `/tools` فهرست ابزارها را نشان می‌دهد و `/skills` مهارت‌های در دسترس را می‌شمارد. رهنمودهای پروژه از `AGENTS.md`، `CLAUDE.md`، `SOUL.md`، `.cursorrules` و برخی فایل‌های `.marelle/` بارگذاری می‌شود، اگر در مسیرهای مجاز باشند.

ابزارهای پرونده به `permissions.allowed_paths` محدودند (پیش‌فرض: پوشه جاری پروژه) و `permissions.denied_paths` را محترم می‌شمارند. خواندن به‌پیش‌فرض مجاز است؛ تغییر فایل‌ها، فرمان‌های پوسته و واکشی‌های شبکه پیش از اجرا می‌پرسند. اجازه پوسته به فرمان‌های روی میزبان مربوط است و محصورساز سیستم پرونده نیست: مگر آن‌که آگاهانه اجرای فرمان را به مدل بسپارید، آن را روی `ask` نگه دارید.

عامل پس از `agent.max_turns` رفت‌وبرگشت ابزارها می‌ایستد. پاسخ‌های ابزارها محتوای غیرقابل‌اعتماد تلقی می‌شوند. درخواست‌ها پاسخ‌های کامل مدل را به کار می‌برند، تا استدلال‌های ابزارها از تکه‌های ناقص جریان سرهم نشوند.

## نشست‌ها

گفت‌وگوها به‌پیش‌فرض محلی ذخیره می‌شوند، از جمله تاریخچه فراخوانی ابزارها و تنظیمات مدل و نشست. در یونیکس پوشه نشست‌ها مُد `0700` و فایل JSON مُد `0600` دارد. فایل در `$XDG_STATE_HOME/marelle/sessions.json` نگه داشته می‌شود، یا `~/.local/state/marelle/sessions.json` وقتی `XDG_STATE_HOME` تنظیم نشده. کلیدهای API هرگز در این فایل نوشته نمی‌شوند. پایداری را با این خاموش کنید:

```sh
marelle config set agent.persist_sessions false
```

کدهای خروج: `0` موفقیت، `1` خطای تأمین‌کننده یا شبکه، `2` خطای استفاده یا پیکربندی.

## تأمین‌کنندگان

| id | پروتکل | متغیر کلید | |
| --- | --- | --- | --- |
| `zai` | سازگار با OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM)، بین‌المللی |
| `zai_coding` | سازگار با OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai، نقطه پایانی سازگار با Anthropic |
| `zhipu` | سازگار با OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel، چین |
| `zhipu_coding` | سازگار با OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan، چین |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | سازگار با OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | سازگار با OpenAI | `OPENROUTER_API_KEY` | مدل‌های بسیار پشت یک کلید |
| `ollama` | سازگار با OpenAI | هیچ | Ollama روی همین ماشین |

نقاط پایانی Z.ai و Zhipu از مستندات اشخاص ثالث می‌آیند، نه از صفحات خود تأمین‌کننده. آن‌ها را با برنامه‌تان بسنجید و هر میدانی که متفاوت بود بازنویسی کنید.

هر میدانِ تأمین‌کننده تغییرپذیر است، و یک تأمین‌کننده تازه چیزی نیست جز یک نام با یک نشانی:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

دیگر میدان‌های تأمین‌کننده: `kind` (`openai-compatible` یا `anthropic`)، `api_key_file`، `auth` (`bearer`، `x-api-key`، `api-key`، `none`)، `max_tokens_field`، `stream_usage` و `extra_body`. اگر متغیری که `api_key_env` نام می‌برد تنظیم شده باشد، او پیروز است؛ وگرنه فایل خوانده می‌شود.

## تنظیمات

تنظیمات به لایه‌ها می‌آیند: مقادیر پیش‌فرض درون‌ساز، سپس متغیرهای محیطی، سپس پرچم‌ها. فایل‌های تنظیمات میان پیش‌فرض‌ها و محیط می‌نشینند.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model` را می‌گذارد. زیرخط دوتا به جای نقطه است.
- `marelle config show [prefix]` — تنظیمات و لایه‌ای که هر یک از آن آمده را فهرست می‌کند.
- `marelle config explain <key>` — هر لایه‌ای که کلیدی را گذاشته را، از کم‌اولویت‌ترین به بیش‌اولویت‌ترین نشان می‌دهد.

| کلید | پیش‌فرض | معنا |
| --- | --- | --- |
| `chat.provider`, `chat.model` | هیچ | با که گفت‌وگو. `chat.model` می‌تواند `provider/model` باشد. |
| `chat.system` | هیچ | اعلان سیستم |
| `chat.max_tokens`, `chat.temperature` | هیچ | تنظیم‌نشده — به تأمین‌کننده واگذار |
| `chat.stream` | `true` | جریان پاسخ اختیاری داور؛ نوبت‌های عامل با ابزارها هماکنون پاسخ‌های کامل را به کار می‌برند، تا فراخوانی‌های ابزار سالم بمانند |
| `chat.show_reasoning`, `chat.show_usage` | `false` | استدلال‌ها و شمارش توکن‌ها روی stderr |
| `agent.max_turns` | `20` | بیشینه رفت‌وبرگشت‌های ابزار به ازای هر درخواست کاربر |
| `agent.persist_sessions` | `true` | ذخیره گفت‌وگوهای محلی میان اجراها |
| `permissions.shell` | `ask` | اجرای فرمان‌های پوسته: `allow`، `ask`، `deny` |
| `permissions.files_read` | `allow` | خواندن فایل‌ها: `allow`، `ask`، `deny` |
| `permissions.files_write` | `ask` | تغییر فایل‌ها: `allow`، `ask`، `deny` |
| `permissions.network` | `ask` | واکشی URL: `allow`، `ask`، `deny` |
| `permissions.allowed_paths` | `["."]` | ریشه‌های مجاز برای ابزارهای پرونده |
| `permissions.denied_paths` | هیچ | مسیرهای فایل صریحاً بسته |
| `network.connect_timeout_secs` | `15` | زمان مجاز برای اتصال |
| `network.idle_timeout_secs` | `300` | سکوت مجاز در انتظار داده |
| `network.proxy` | `system` | `system`، `none`، یا نشانی پراکسی |
| `network.ca_bundle` | هیچ | فایل افزوده از مراجع گواهی مورداعتماد |
| `network.allow_insecure_http` | `false` | اجازه کلید روی `http` خام به میزبان بیگانه |
| `update.check` | `true` | پرسیدن از GitHub و GitLab درباره نسخه‌ای نوتر در آغاز نشست تعاملی |
| `update.github_repo` | `S-I-L-S/marelle` | `owner/repo` گیت‌هابی که بازرسی از او جدیدترین انتشار را می‌پرسد |
| `update.gitlab_repo` | هیچ | `owner/repo` گیت‌لبی برای همان بازرسی. خالی یعنی GitLab خاموش |
| `update.timeout_secs` | `3` | بازرسی چند ثانیه می‌تواند برای یک منبع بگذارد |

Marelle سیاهه خودش از مراجع گواهی مورداعتماد را با خود حمل می‌کند؛ پس آنجا هم کار می‌کند که سیستم ندارد، مثل Termux. `SSL_CERT_FILE` بیشتر می‌افزاید.

وقتی نسخه‌ای نوتر یافت شود، نشست یک بار یادش می‌کند — نسخه را، آنچه دارید را، و از کجا می‌توان گرفت. نخست انتشار پرسیده می‌شود؛ مخزنی که فرمان‌هایش را تنها برچسب می‌زند تازه‌ترین برچسب را می‌دهد. هرچه شکست بخورد بی‌صدا می‌ماند.

## کلیدهایتان مالِ خودتان می‌ماند

- تنظیمات متغیری یا پرونده‌ای را نام می‌برند که کلید در آن آرام گرفته، هرگز خودِ کلید را نه. تنظیمی چون `providers.x.api_key` رد می‌شود.
- کلید را `config show`، `config explain` یا `providers` چاپ نمی‌کنند، و از هر متن خطایی که از سرور برمی‌گردد کنار گذاشته می‌شود.
- تنها به نشانی تأمین‌کننده، در یک سرصفحه فرستاده می‌شود، و مگر خودتان اجازه دهید هرگز روی `http` خام به ماشین دیگری نمی‌رود.
- به بازنشانی‌ها پیروی نمی‌شود؛ پس کلید جایی دیگر فرستاده نمی‌شود.

## زبان‌ها

واسط به انگلیسی، روسی، فرانسوی و اوکراینی سخن می‌گوید. با `--lang <code>` یا `MARELLE_LANG` برگزینید؛ وگرنه `LC_ALL`، `LC_MESSAGES` و `LANG` تصمیم می‌گیرند. هر پیام در یک کتابچه زندگی می‌کند، و اگر زبانی یکی را جا بگذارد آزمونی فرو می‌ریزد. هدف هر زبان اوراسیا است و پس از آن بیشتر.

## نقشه راه

1. **گفت‌وگو.** مبدل‌های سازگار با OpenAI و Anthropic، هم با جریان SSE هم با خوانش پاسخ‌های کامل. نوبت‌های عامل با ابزارها هم‌اکنون پاسخ‌های کامل را به کار می‌برند. *انجام شد.*
2. **تنظیمات و ظاهر.** فایل‌های تنظیمات TOML، سراسری و برای هر پروژه، با `config set`، `get` و `unset` و توصیف هر کلید. واسطی تم‌بندی‌شده در روح [Hermes Agent](https://github.com/NousResearch/hermes-agent): سرلوحه، پنل اطلاعات، پاسخ‌های قاب‌دار، چرخنده، به‌پیش‌فرض در بنفشِ گل یاس، هر رنگ نهفته به‌صورت HEX. *انجام شد.*
3. **مدل‌های بسیار.** نام‌های مستعار مدل‌ها، پرسش هم‌زمان از چند مدل و سنجش پاسخ‌ها، زنجیرهای جایگزین، و مدل داوری که آنها را می‌گدازد. *انجام شد.*
4. **کنش‌گری.** فراخوانی‌های ساخت‌یافته ابزارها برای تأمین‌کنندگان سازگار با OpenAI و Anthropic، ابزارهای محلی پرونده/جست‌وجو/پایانه، واکشی URL، مهارت‌های محلی و مجوزهای allow/ask/deny؛ نوشتار هر فراخوانی ابزار و نتیجه‌اش را نشان می‌دهد. *انجام شد.*
5. **به یاد سپردن.** گفت‌وگوهای نام‌دار میان اجراها محلی می‌مانند. حافظه معنایی بلندمدت هنوز پیش روی است.
6. **گسترش.** سرورهای MCP، افزودنی‌های بیرونی، زیرعامل‌ها و پشتیبانی بسنده‌تری مهارت‌ها.
7. **رسیدن.** خودکارِسازی مرورگر، دروازه‌های پیام‌رسان‌ها و کارهای زمان‌بندی‌شده.

اندیشه پسِ طراحی در [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) است.

## محدودیت‌های شناخته‌شده

- برگزیننده‌های تأمین‌کننده و مدل، تأمین‌کنندگان تنظیم‌شده و نام‌های مستعار مدل‌ها را می‌شمارند. Marelle هنوز کاتالوگ‌های دورِ مدل‌ها را کشف نمی‌کند.
- برگزیننده‌ها کلیک موش را برای کانونی‌کردن و غلتاندن با چرخه پشتیبانی می‌کنند؛ نوشتار و ویرایشگر ورود همچنان از صفحه‌کلید فرمان می‌برند.
- Ctrl-C از نشست تعاملی بیرون می‌رود. هنوز تنها پاسخ جاری را نمی‌بُرد.
- حلقه عامل، ابزارهای محلی آگاه از مجوزها و نشست‌های محلی پایدار پیاده شده‌اند، اما Marelle نمونه اولیه پیش‌آلفا می‌ماند؛ برای طراحی کنونی و مرزهایش بخوانید [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## گسترش

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

آزمون‌ها هرگز به شبکه دست نمی‌زنند: با یک سرور ساختگی در `crates/marelle-testkit` گفت‌وگو می‌کنند. GitHub Actions باینری‌های اندروید را نیز می‌سازد و کمینه نسخه اعلام‌شده Rust را می‌سنجد. برای آینه در GitLab یک خط GitLab CI هست؛ ببینید [docs/GITLAB.md](docs/GITLAB.md).

## مجوز

[AGPL-3.0-only](LICENSE). یادداشت‌های ثالث در [NOTICE](NOTICE). بسته‌های انتشار `THIRD-PARTY-LICENSES.txt` را برای جعبه‌های Rust درون باینری می‌آورند.
