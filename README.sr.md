```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Конфигурабилни ИИ-агент за ваш терминал.**
Донесите своје API-кључеве: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, локалну Ollamу или било који сервер компатибилан са OpenAI. Првенствено Linux и Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · Српски · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Пре-алфа, верзија 0.1.0-alpha.1.** Реализовано: сопствени TUI-интерфејс, чет са више добављача, агентска петља свесна овлашћења, локални алати и обновљиве локалне сесије. API-ји и подешавања могу још да се промене. Ово је прототип, а не паритет функција са OpenCode или Hermes.

*Marelle* је француски назив дечје игре скакања по квадратићима. Сваки квадратић је корак.

## Инсталација

**Termux на Андроиду.** Преузмите готови бинарник са странице [издања](https://github.com/S-I-L-S/marelle/releases) или га саградите на телефону. Упутство корак по корак: [docs/TERMUX.md](docs/TERMUX.md).

**Linux и свуда где ради Rust.** Потребан је Rust 1.85 или новији:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Или клонирајте репозиторијум и покрените `cargo build --release`. Бинарник је `target/release/marelle`, око 4 MiB.

## Први чет

Чувајте кључ у датотеци коју читате само ви, и реците Marelle где је. Marelle никада не чува кључ у својим подешавањима.

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

Замените `MODEL` називом модела са списка вашег добављача. Други начини коришћења:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Одговор иде на stdout. Развиђања, број токена, напомене и грешке иду на stderr, тако да `marelle chat ... > answer.txt` чува само одговор.

## Алати агента

Marelle шаље структуриране дефиниције алата преко API-ја компатибилних са OpenAI и Anthropic. Уграђени сет алата:

| Алат | Сврха |
| --- | --- |
| `terminal` | Покретање команде љуске из директоријума покретања |
| `read_file`, `write_file`, `edit_file` | Читање и мењање текстуалних датотека UTF-8 |
| `list_directory`, `search_text` | Преглед и претрага датотека пројекта |
| `fetch_url` | Преузимање јавне HTTP(S)-стране као текста (треба `curl`) |
| `list_skills`, `load_skill` | Откривање и учитавање локалних упутстава `SKILL.md` |

Команда `/tools` приказује списак алата, а `/skills` доступне вештине. Упутства пројекта се учитавају из `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` и одабраних датотека `.marelle/`, ако су у дозвољеним путањама.

Алати за датотеке су ограничени на `permissions.allowed_paths` (подразумевано: текући директоријум пројекта) и поштују `permissions.denied_paths`. Читање је подразумевано дозвољено; мењање датотека, команде љуске и мрежна преузимања прво питају. Овлашћење за љуску се односи на команде на домаћину и није песак sistema датотека: држите га на `ask`, осим ако намерно не поверите моделу извршавање команди.

Агент се зауставља после `agent.max_turns` размена са алатима. Одговори алата се третирају као неповерљив садржај. Захтеви користе целе одговоре модела, тако да аргументи алата не састављају се из непотпуних комада тока.

## Сесије

Разговори се подразумевано чувају локално, укључујући историју позива алата и подешавања модела и сесије. На Unix-у директоријум сесија има режим `0700`, а JSON датотека `0600`. Датотека се чува у `$XDG_STATE_HOME/marelle/sessions.json`, или `~/.local/state/marelle/sessions.json` када `XDG_STATE_HOME` није задато. API-кључеви се никада не уписују у ту датотеку. Искључите трајност са:

```sh
marelle config set agent.persist_sessions false
```

Кодови изласка: `0` успех, `1` грешка добављача или мреже, `2` грешка употребе или подешавања.

## Добављачи

| id | Протокол | Променљива кључа | |
| --- | --- | --- | --- |
| `zai` | компатибилан са OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), међународни |
| `zai_coding` | компатибилан са OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, завршетак компатибилан са Anthropic |
| `zhipu` | компатибилан са OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Кина |
| `zhipu_coding` | компатибилан са OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Кина |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | компатибилан са OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | компатибилан са OpenAI | `OPENROUTER_API_KEY` | много модела иза једног кључа |
| `ollama` | компатибилан са OpenAI | нема | Ollama на овој машини |

Завршеци Z.ai и Zhipu долазе из документације трећих страна, не са страница самог добављача. Проверите их према свом плану и препишите било које поље ако се разликују.

Свако поље добављача може да се промени, а нови добављач је просто име са адресом:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Остала поља добављача: `kind` (`openai-compatible` или `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` и `extra_body`. Ако је променљива названа у `api_key_env` задата, она побеђује; иначе се чита датотека.

## Подешавања

Подешавања долазе у слојевима: уграђене подразумеване вредности, затим променљиве окружења, затим заставице. Датотеке подешавања ставају се између подразумеваних вредности и окружења.

- `MARELLE_CHAT__MODEL=zai/MODEL` задаје `chat.model`. Двоструко доње цртање стоји за тачку.
- `marelle config show [prefix]` излистава подешавања и слој из којег је свако дошло.
- `marelle config explain <key>` приказује сваки слој који је задао кључ, од најнижег приоритета до највишег.

| Кључ | Подразумевано | Смисао |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ништа | Са ким причати. `chat.model` може бити `provider/model`. |
| `chat.system` | ништа | Системски промпт |
| `chat.max_tokens`, `chat.temperature` | ништа | Незадато — остављено добављачу |
| `chat.stream` | `true` | Ток одговора судије кад је укључен; ходови агента са алатима тренутно користе целe одговоре, да позиви алата остају цели |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Развиђања и број токена на stderr |
| `agent.max_turns` | `20` | Максимум размена са алатима на један захтев корисника |
| `agent.persist_sessions` | `true` | Чување локалних разговора између покретања |
| `permissions.shell` | `ask` | Извршавање команди љуске: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Читање датотека: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Мењање датотека: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Преузимање URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Дозвољени корени за алате за датотеке |
| `permissions.denied_paths` | ништа | Изрично блокиране путање датотека |
| `network.connect_timeout_secs` | `15` | Време дозвољено за везу |
| `network.idle_timeout_secs` | `300` | Дозвољено ћутање при чекању података |
| `network.proxy` | `system` | `system`, `none` или адреса проксија |
| `network.ca_bundle` | ништа | Додатна датотека поузданих сертификатних органа |
| `network.allow_insecure_http` | `false` | Дозволити кључ преко обичног `http` на туђ хост |
| `update.check` | `true` | Питати GitHub и GitLab за новију верзију при почетку интерактивне сесије |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` од којег провера пита за најновије издање |
| `update.gitlab_repo` | ништа | GitLab `owner/repo` за исти проверу. Празно искључује GitLab |
| `update.timeout_secs` | `3` | Колико секунди провера може потрошити на један извор |

Marelle носи сопствени списак поузданих сертификатних органа, тако да ради и тамо где их систем нема, као у Termux-у. `SSL_CERT_FILE` додаје још.

Кад се нађе новија верзија, сесија ће је споменути једном — верзију, оно што имате и где да је узмете. Прво се пита издање; репозиторијум који само тагује своје комите даје најновији таг. Шта год закзаца, остаје нечујно.

## Ваше кључеве остају ваше

- Подешавања називају променљиву или датотеку где лежи кључ, никада сам кључ. Подешавање као `providers.x.api_key` се одбија.
- Кључ се не штампа у `config show`, `config explain` или `providers` и се ишчупава из сваког текста грешке од сервера.
- Он се шаље само на адресу добављача, у једном заглављу, и никада преко обичног `http` на другу машину, ако то не дозволите.
- Пренаширења се не прате, тако да кључ не може да пође негде другоме.

## Језици

Интерфејс говори енглески, руски, француски и украјински. Бирајте преко `--lang <code>` или `MARELLE_LANG`; иначе одлучују `LC_ALL`, `LC_MESSAGES` и `LANG`. Свака порука живи у каталогу, и тест пада ако нека језик пропусти једну. Циљ су сви језици Евроазије, а онда још.

## Путна карта

1. **Причати.** Адаптери компатибилни са OpenAI и Anthropic, са SSE-током и разбором целих одговора. Ходови агента са алатима тренутно користе целе одговоре. *Готово.*
2. **Подешавања и изглед.** TOML датотеке подешавања, глобалне и по пројекту, са `config set`, `get` и `unset` и описом сваког кључа. Тематизован интерфејс у духу [Hermes Agent](https://github.com/NousResearch/hermes-agent): банер, информативна плоча, одговори у оквиру, спинер, подразумевано у љубичастој глициније, са било којом бојом као HEX. *Готово.*
3. **Много модела.** Алиаси модела, истовремено питање неколико модела и поређење одговора, ланци резерве и модел-судија који их састаавља. *Готово.*
4. **Делати.** Структурирани позиви алата за добављаче компатибилне са OpenAI и Anthropic, локални алати за датотеке, претрагу и терминал, преузимање URL, локалне вештине и овлашћења allow/ask/deny; транскрипт приказује сваки позив алата и његов резултат. *Готово.*
5. **Сећати се.** Именовани разговори трају локално између покретања. Дуготрајна семантичка меморија је још пред нама.
6. **Проширивати.** MCP сервери, спољни прикључци, подагенти и богатија подршка вештина.
7. **Досежити.** Аутоматизација прегледача, капије гласника и заказани послови.

Мишљење иза дизајна је у [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Позната ограничења

- Бирачи добављача и модела излиставају подешене добављаче и алиасе модела. Marelle још не открива удаљене каталоге модела.
- Бирачи подржавају клик мишом за фокус и скроловање точки; транскрипт и уредник уноса остају вођени тастатуром.
- Ctrl-C излази из интерактивне сесије. Још не прекида само текући одговор.
- Агентска петља, локални алати свесни овлашћења и трајне локалне сесије су спроведени, али Marelle остаје пре-алфа прототип; о садашњем дизајну и границама читајте [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Развој

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Проби никада не дирају мрежу: говоре са макетом сервера у `crates/marelle-testkit`. GitHub Actions такође гради Android бинарнике и проверява задекларисану минималну верзију Rust. За огледало на GitLab-у постоји ток GitLab CI; видети [docs/GITLAB.md](docs/GITLAB.md).

## Лиценца

[AGPL-3.0-only](LICENSE). Обавештења трећих страна су у [NOTICE](NOTICE). Архиви издања укључују `THIRD-PARTY-LICENSES.txt` за Rust-сандуке у бинарнику.
