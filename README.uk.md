```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Налаштовуваний ШІ-агент для вашого терміналу.**
Приносіть свої API-ключі: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, локальна Ollama або будь-який OpenAI-сумісний сервер. Перш за все Linux і Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · Українська · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Пре-альфа, версія 0.1.0-alpha.1.** Реалізовано: власний TUI-інтерфейс, чат із кількома провайдерами, агентський цикл з урахуванням прав, локальні інструменти та поновлювані локальні сесії. API й налаштування можуть ще змінитися. Це прототип, а не паритет функцій з OpenCode чи Hermes.

*Marelle* — французька назва гри в класики. Кожна клітинка — це крок.

## Встановлення

**Termux на Android.** Завантажте готовий бінарник зі сторінки [релізів](https://github.com/S-I-L-S/marelle/releases) або зберіть на телефоні. Покрокова інструкція: [docs/TERMUX.md](docs/TERMUX.md).

**Linux та будь-де працює Rust.** Потрібен Rust 1.85 або новіший:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Або склонуйте репозиторій і виконайте `cargo build --release`. Бінарник — `target/release/marelle`, близько 4 МіБ.

## Перший чат

Зберігайте ключ у файлі, який читаєте тільки ви, і скажіть Marelle, де він. Marelle ніколи не зберігає ключ у своїх налаштуваннях.

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

Замініть `MODEL` на назву моделі зі списку вашого провайдера. Інші способи користування:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Відповідь іде у stdout. Міркування, кількість токенів, примітки та помилки — у stderr, тож `marelle chat ... > answer.txt` зберігає тільки відповідь.

## Інструменти агента

Marelle надсилає структуровані означення інструментів через OpenAI-сумісний та Anthropic API. Вбудований набір інструментів:

| Інструмент | Призначення |
| --- | --- |
| `terminal` | Виконати команду оболонки з каталогу запуску |
| `read_file`, `write_file`, `edit_file` | Читати та змінювати текстові файли UTF-8 |
| `list_directory`, `search_text` | Огляд і пошук у файлах проєкту |
| `fetch_url` | Отримати публічну HTTP(S)-сторінку як текст (потрібен `curl`) |
| `list_skills`, `load_skill` | Знаходити та завантажувати локальні інструкції `SKILL.md` |

Команда `/tools` покаже список інструментів, `/skills` — доступні навички. Настанови проєкту завантажуються з `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` і окремих файлів `.marelle/`, якщо вони в дозволених шляхах.

Файлові інструменти обмежені `permissions.allowed_paths` (типово: поточний каталог проєкту) і зважають на `permissions.denied_paths`. Читання дозволено типово; зміна файлів, команди оболонки та мережеві завантаження спершу питають. Дозвіл на оболонку стосується команд на хості й не є пісочницею файлової системи: тримайте `ask`, якщо не свідомо довіряєте моделі виконання команд.

Агент зупиняється після `agent.max_turns` обмінів з інструментами. Відповіді інструментів вважаються недовіреним вмістом. Запити використовують повні відповіді моделі, тож аргументи інструментів не збираються з неповних фрагментів потоку.

## Сесії

Розмови типово зберігаються локально, разом з історією викликів інструментів та налаштуваннями моделі й сеансу. На Unix каталог сесій має режим `0700`, а JSON-файл — `0600`. Файл зберігається у `$XDG_STATE_HOME/marelle/sessions.json`, або `~/.local/state/marelle/sessions.json`, якщо `XDG_STATE_HOME` не задано. API-ключі ніколи не записуються в цей файл. Вимкнути збереження:

```sh
marelle config set agent.persist_sessions false
```

Коди виходу: `0` успіх, `1` помилка провайдера чи мережі, `2` помилка використання чи налаштувань.

## Провайдери

| id | Протокол | Змінна ключа | |
| --- | --- | --- | --- |
| `zai` | OpenAI-сумісний | `ZAI_API_KEY` | Z.ai (Zhipu GLM), міжнародний |
| `zai_coding` | OpenAI-сумісний | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-сумісний ендпоінт |
| `zhipu` | OpenAI-сумісний | `ZHIPUAI_API_KEY` | Zhipu BigModel, Китай |
| `zhipu_coding` | OpenAI-сумісний | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Китай |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-сумісний | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-сумісний | `OPENROUTER_API_KEY` | багато моделей за одним ключем |
| `ollama` | OpenAI-сумісний | немає | Ollama на цій машині |

Ендпоінти Z.ai і Zhipu узяті зі сторонньої документації, а не зі сторінок постачальника. Звірьте їх із вашим тарифом і перевизначте будь-яке поле, якщо вони різняться.

Будь-яке поле провайдера можна змінити, а новий провайдер — це просто ім'я з адресою:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Інші поля провайдера: `kind` (`openai-compatible` або `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` і `extra_body`. Якщо змінна з імені `api_key_env` задана, вона виграє; інакше читається файл.

## Налаштування

Налаштування приходять шарами: вбудовані типові значення, потім змінні середовища, потім прапорці. Файли налаштувань стають між типовими значеннями та середовищем.

- `MARELLE_CHAT__MODEL=zai/MODEL` задає `chat.model`. Подвійне підкреслення означає крапку.
- `marelle config show [prefix]` перелічує налаштування та шар, з якого кожне прийшло.
- `marelle config explain <key>` показує кожен шар, що задав ключ, від найнижчого пріоритету до найвищого.

| Ключ | Типове | Значення |
| --- | --- | --- |
| `chat.provider`, `chat.model` | немає | З ким розмовляти. `chat.model` може бути `provider/model`. |
| `chat.system` | немає | Системний промпт |
| `chat.max_tokens`, `chat.temperature` | немає | Не задано — на розсуд провайдера |
| `chat.stream` | `true` | Потокова відповідь судді, коли він увімкнений; ходи агента з інструментами зараз використовують повні відповіді, щоб виклики інструментів лишалися цілими |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Міркування та кількість токенів у stderr |
| `agent.max_turns` | `20` | Максимум обмінів з інструментами на один запит користувача |
| `agent.persist_sessions` | `true` | Зберігати локальні розмови між запусками |
| `permissions.shell` | `ask` | Виконання команд оболонки: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Читання файлів: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Зміна файлів: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Завантаження URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Дозволені корені для файлових інструментів |
| `permissions.denied_paths` | немає | Явно заборонені шляхи файлів |
| `network.connect_timeout_secs` | `15` | Час, дозволений на з'єднання |
| `network.idle_timeout_secs` | `300` | Дозволене мовчання при очікуванні даних |
| `network.proxy` | `system` | `system`, `none` або адреса проксі |
| `network.ca_bundle` | немає | Додатковий файл довірених центрів сертифікації |
| `network.allow_insecure_http` | `false` | Дозволити ключ через звичайний `http` на чужий хост |
| `update.check` | `true` | Питати GitHub і GitLab про новішу версію на старті інтерактивного сеансу |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, у якого перевірка питає свіжий реліз |
| `update.gitlab_repo` | немає | GitLab `owner/repo` для тієї ж перевірки. Порожньо — GitLab вимкнено |
| `update.timeout_secs` | `3` | Скільки секунд перевірка може витратити на одне джерело |

Marelle несе власний список довірених центрів сертифікації, тому працює і там, де в системі їх немає, як-от у Termux. `SSL_CERT_FILE` додає ще.

Коли знайдено новішу версію, сеанс згадає про неї один раз — версію, те, що у вас стоїть, і де її взяти. Спершу питається реліз; репозиторій, що лише тегує комміти, віддає новіший тег. Будь-яка невдача лишається мовчазною.

## Ваші ключі лишаються вашими

- Налаштування називають змінну або файл, де лежить ключ, але не сам ключ. Таке налаштування, як `providers.x.api_key`, відхилюється.
- Ключ не друкується у `config show`, `config explain` чи `providers` і викидається з будь-якого тексту помилки, що повернувся від сервера.
- Він надсилається лише на адресу провайдера, одним заголовком, і ніколи через звичайний `http` на іншу машину, якщо ви це не дозволили.
- Перенаправлення не провідуються, тож ключ не може піти кудись іще.

## Мови

Інтерфейс розмовляє англійською, російською, французькою та українською. Вибирайте через `--lang <code>` або `MARELLE_LANG`; інакше вирішують `LC_ALL`, `LC_MESSAGES` і `LANG`. Кожне повідомлення живе в каталозі, і тест падає, якщо якась мова пропустила одне. Мета — всі мови Євразії, а потім іще.

## Дорожня карта

1. **Говорити.** Адаптери OpenAI-сумісного та Anthropic, з SSE-потоковістю та розбором повних відповідей. Ходи агента з інструментами зараз використовують повні відповіді. *Готово.*
2. **Налаштування й вигляд.** TOML-файли налаштувань, глобальні й на проєкт, з `config set`, `get` та `unset` і описом кожного ключа. Тематизований інтерфейс у дусі [Hermes Agent](https://github.com/NousResearch/hermes-agent): банер, інформаційна панель, відповіді в рамці, спінер, типово у фіолетовому кольорі глицинії, з будь-яким кольором через HEX. *Готово.*
3. **Багато моделей.** Аліаси моделей, одночасне питання кількох моделей і порівняння відповідей, ланцюги fallback та суддя-модель, що поєднує їх. *Готово.*
4. **Діяти.** Структуровані виклики інструментів для OpenAI-сумісних та Anthropic, локальні інструменти файлів, пошуку та терміналу, завантаження URL, локальні навички й права allow/ask/deny; транскрипт показує кожен виклик інструмента та його результат. *Готово.*
5. **Пам'ятати.** Іменовані розмови зберігаються локально між запусками. Довготривала семантична пам'ять ще попереду.
6. **Розширювати.** Сервери MCP, зовнішні плагіни, субагенти та багатша підтримка навичок.
7. **Досягати.** Автоматизація браузера, шлюзи месенджерів і заплановані задачі.

Міркування за дизайном — у [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Відомі обмеження

- Селектори провайдерів і моделей показують налаштованих провайдерів та аліаси моделей. Marelle ще не відкриває віддалені каталоги моделей.
- Селектори підтримують клік мишею для фокуса та прокрутку коліщатком; транскрипт і редактор вводу керуються клавіатурою.
- Ctrl-C виходить з інтерактивного сеансу. Поки що він не скасовує лише поточну відповідь.
- Агентський цикл, локальні інструменти з урахуванням прав і сталі локальні сесії реалізовані, але Marelle лишається пре-альфа прототипом; про поточний дизайн і межі читайте [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Розробка

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Тести ніколи не торкаються мережі: вони говорять із мок-сервером у `crates/marelle-testkit`. GitHub Actions також збирає Android-бінарники та перевіряє заявлену мінімальну версію Rust. Для дзеркала на GitLab є конвеєр GitLab CI; дивіться [docs/GITLAB.md](docs/GITLAB.md).

## Ліцензія

[AGPL-3.0-only](LICENSE). Сторонні нотатки — в [NOTICE](NOTICE). Архіви релізів містять `THIRD-PARTY-LICENSES.txt` для Rust-крейтів усередині бінарника.
