```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Настроим ИИ-агент за вашия терминал.**
Донесете своите API-ключове: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, локална Ollama или какъвто и да е сървър, съвместим с OpenAI. Преди всичко Linux и Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · Български · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Пре-алфа, версия 0.1.0-alpha.1.** Реализирано: собствено TUI-интерфейс, чат с няколко доставчика, агентски цикъл, съобразен с правата, локални инструменти и възобновяеми локални сесии. API-тата и настройките може още да се променят. Това е прототип, а не паритет на функциите с OpenCode или Hermes.

*Marelle* е френското название на детската игра с прескачане по квадратчета. Всяко квадратче е стъпка.

## Инсталиране

**Termux на Android.** Изтеглете готов бинарник от страницата с [изданията](https://github.com/S-I-L-S/marelle/releases) или го съберете на телефона. Стъпка по стъпка: [docs/TERMUX.md](docs/TERMUX.md).

**Linux и навсякъде, където върви Rust.** Нужен е Rust 1.85 или по-нов:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Или клонирайте хранилището и изпълнете `cargo build --release`. Бинарникът е `target/release/marelle`, около 4 МиБ.

## Първи чат

Пазете ключа във файл, който четете само вие, и кажете на Marelle къде е. Marelle никога не записва ключ в своите настройки.

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

Заменете `MODEL` с име на модел от списъка на вашия доставчик. Други начини за ползване:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Отговорът иде в stdout. Мисленето, броенето на токени, бележките и грешките иде в stderr, така че `marelle chat ... > answer.txt` записва само отговора.

## Инструменти на агента

Marelle праща структурирани определения на инструменти през OpenAI-съвместимия и Anthropic API. Вграденият набор инструменти:

| Инструмент | Предназначение |
| --- | --- |
| `terminal` | Изпълнение на команда от шелла в изходната директория |
| `read_file`, `write_file`, `edit_file` | Четене и промяна на текстови файлове в UTF-8 |
| `list_directory`, `search_text` | Преглед и търсене в файловете на проекта |
| `fetch_url` | Изтегляне на публична HTTP(S)-страница като текст (нужен е `curl`) |
| `list_skills`, `load_skill` | Откриване и зареждане на локални инструкции `SKILL.md` |

Командата `/tools` ще покаже списъка с инструменти, а `/skills` — наличните умения. Указанията на проекта се зареждат от `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` и избрани файлове `.marelle/`, ако са в разрешените пътища.

Файловите инструменти са ограничени до `permissions.allowed_paths` (по подразбиране: текущата директория на проекта) и съобразяват `permissions.denied_paths`. Четенето е разрешено по подразбиране; промяната на файлове, командите на шелла и изтеглянията от мрежата питат първо. Разрешението за шелла касае команди на хоста и не е пясъчник на файловата система: дръжте го на `ask`, освен ако съзнателно не доверите на модела изпълнението на команди.

Агентът спира след `agent.max_turns` размена с инструментите. Отговорите на инструментите се смятат за недоверен съдържание. Заявките използват пълни отговори на модела, така че аргументите на инструментите не се сглобяват от непълни парченца от потока.

## Сесии

Разговорите по подразбиране се записват локално, заедно с историята на извикванията на инструментите и настройките на модела и сесията. На Unix директорията на сесиите е с режим `0700`, а JSON-файлът — с `0600`. Файлът се съхранява в `$XDG_STATE_HOME/marelle/sessions.json`, или `~/.local/state/marelle/sessions.json`, когато `XDG_STATE_HOME` не е зададено. API-ключовете никога не се записват в този файл. Изключете записването с:

```sh
marelle config set agent.persist_sessions false
```

Кодове на изход: `0` успех, `1` грешка на доставчика или мрежата, `2` грешка на използване или настройки.

## Доставчици

| id | Протокол | Променлива на ключа | |
| --- | --- | --- | --- |
| `zai` | съвместим с OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), международен |
| `zai_coding` | съвместим с OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, крайна точка, съвместима с Anthropic |
| `zhipu` | съвместим с OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Китай |
| `zhipu_coding` | съвместим с OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Китай |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | съвместим с OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | съвместим с OpenAI | `OPENROUTER_API_KEY` | много модели зад един ключ |
| `ollama` | съвместим с OpenAI | няма | Ollama на тази машина |

Крайните точки на Z.ai и Zhipu идват от документация на трети страни, не от страниците на самия доставчик. Сверете ги с вашия план и презапишете всяко поле, ако различават.

Всяко поле на доставчика може да се промени, а нов доставчик е просто име с адрес:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Другите полета на доставчика: `kind` (`openai-compatible` или `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` и `extra_body`. Ако променливата, посочена в `api_key_env`, е зададена, тя печели; иначе се чете файлът.

## Настройки

Настройките идват на слоеве: вградени по подразбиране стойности, после променливи на средата, после флагове. Файловете с настройки застават между стойностите по подразбиране и средата.

- `MARELLE_CHAT__MODEL=zai/MODEL` задава `chat.model`. Двойното долно чертище стои за точка.
- `marelle config show [prefix]` изброява настройките и слоя, от който всяка е дошла.
- `marelle config explain <key>` показва всеки слой, задал ключа, от най-нисък приоритет до най-висок.

| Ключ | По подразбиране | Смисъл |
| --- | --- | --- |
| `chat.provider`, `chat.model` | никакъв | С кого да говорите. `chat.model` може да е `provider/model`. |
| `chat.system` | никакъв | Системен промпт |
| `chat.max_tokens`, `chat.temperature` | никакъв | Не е зададено — оставено на доставчика |
| `chat.stream` | `true` | Поточно предаване на отговора на съдията, когато е включен; ходовете на агента с инструменти в момента използват пълни отговори, за да останат извикванията на инструментите цели |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Мислене и броене на токени в stderr |
| `agent.max_turns` | `20` | Максимум размена с инструменти на една заявка от потребителя |
| `agent.persist_sessions` | `true` | Запазване на локалните разговори между пускания |
| `permissions.shell` | `ask` | Изпълнение на команди на шелла: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Четене на файлове: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Промяна на файлове: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Изтегляне на URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Разрешени корени за файловите инструменти |
| `permissions.denied_paths` | никакъв | Изрично блокирани пътища на файлове |
| `network.connect_timeout_secs` | `15` | Време, разрешено за свързване |
| `network.idle_timeout_secs` | `300` | Позволено мълчание при чакане на данни |
| `network.proxy` | `system` | `system`, `none` или адрес на прокси |
| `network.ca_bundle` | никакъв | Допълнителен файл с доверени сертификатни органи |
| `network.allow_insecure_http` | `false` | Позволяване на ключ през обикновен `http` към чуж хост |
| `update.check` | `true` | Пита GitHub и GitLab за по-нова версия при старта на интерактивна сесия |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, на който проверката пита за най-новото издание |
| `update.gitlab_repo` | никакъв | GitLab `owner/repo` за същата проверка. Празно изключва GitLab |
| `update.timeout_secs` | `3` | Колко секунди проверката може да похарчи на един източник |

Marelle носи собствен списък доверени сертификатни органи, затова работи и там, където системата няма, като в Termux. `SSL_CERT_FILE` добавя още.

Когато бъде намерена по-нова версия, сесията ще я спомене веднъж — версията, това, което имате, и откъде да я вземете. Първо се пита изданието; хранилище, което само тагва комитите си, дава новия таг. Каквото и да се провали, остава мълчаливо.

## Вашите ключове остават ваши

- Настройките назовават променлива или файл, в който лежи ключът, никога самия ключ. Настройка като `providers.x.api_key` се отхвърля.
- Ключът не се печата в `config show`, `config explain` или `providers` и се изрязва от всеки текст на грешка, дошъл от сървър.
- Той се праща само на адреса на доставчика, в един заглавен ред, и никога през обикновен `http` на друга машина, освен ако не позволите.
- Пренасочванията не се следват, така че ключът не може да бъде препратен другаде.

## Езици

Интерфейсът говори английски, руски, френски и украински. Изберете с `--lang <code>` или `MARELLE_LANG`; иначе решават `LC_ALL`, `LC_MESSAGES` и `LANG`. Всяко съобщение живее в каталог, и тест пада, ако някой език пропусне едно. Целта са всички езици на Евразия, а после още.

## Пътна карта

1. **Говори.** Адаптери, съвместими с OpenAI, и Anthropic, с SSE-поточност и разбор на пълни отговори. Ходовете на агента с инструменти в момента използват пълни отговори. *Готово.*
2. **Настройки и външност.** TOML-файлове с настройки, глобални и за проект, с `config set`, `get` и `unset` и описание за всеки ключ. Тематизиран интерфейс в духа на [Hermes Agent](https://github.com/NousResearch/hermes-agent): банер, информационен панел, отговори в рамка, въртяща се точка, по подразбиране в лилавото на глицинията, с всеки цвят задаваем като HEX. *Готово.*
3. **Много модели.** Алиаси на модели, едновременно питане на няколко модела и сравняване на отговорите, вериги за резервен избор и модел-съдия, който ги събира. *Готово.*
4. **Действай.** Структурирани извиквания на инструменти за доставчици, съвместими с OpenAI и Anthropic, локални инструменти за файлове, търсене и терминал, изтегляне на URL, локални умения и права allow/ask/deny; транскриптът показва всяко извикване на инструмент и резултата му. *Готово.*
5. **Помни.** Именувани разговори се запазват локално между пусканията. Дълготрайната семантична памет е още напред.
6. **Разширявай.** Сървъри MCP, външни плъгини, подагенти и по-богата поддръжка на умения.
7. **Достигай.** Автоматизация на браузър, шлюзове към месинджъри и планирани задачи.

Мисълта зад дизайна е в [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Известни ограничения

- Избирачите на доставчици и модели изброяват настроените доставчици и алиасите на модели. Marelle още не открива отдалечени каталоги на модели.
- Избирачите поддържат клик с мишката за фокус и въртене с колелцето; транскриптът и редакторът на вход се управляват от клавиатурата.
- Ctrl-C излиза от интерактивната сесия. Все още не прекъсва само текущия отговор.
- Агентският цикъл, локалните инструменти, съобразени с правата, и устойчивите локални сесии са реализирани, но Marelle остава пре-алфа прототип; за настоящия дизайн и граници четете [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Разработка

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Тестовете никога не докосват мрежата: те говорят с фалшив сървър в `crates/marelle-testkit`. GitHub Actions също събира Android-бинарниците и проверява заявената минимална версия на Rust. За огледало на GitLab има тръбопровод GitLab CI; вижте [docs/GITLAB.md](docs/GITLAB.md).

## Лиценз

[AGPL-3.0-only](LICENSE). Известия на трети страни са в [NOTICE](NOTICE). Архивите на изданията включват `THIRD-PARTY-LICENSES.txt` за Rust-крейтовете в бинарника.
