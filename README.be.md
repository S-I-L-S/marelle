```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Наладны ІІ-агент для вашага тэрмінала.**
Прыносіце свае API-ключы: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, лакальная Ollama ці любы OpenAI-сумяшчальны сервер. Перш за ўсё Linux і Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · Беларуская · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Пре-альфа, версія 0.1.0-alpha.1.** Рэалізавана: уласны TUI-інтэрфейс, чат з некалькімі правайдэрамі, агенцкі цыкл з улікай правоў, лакальныя інструменты і аднаўляльныя лакальныя сесіі. API і налады яшчэ могуць змяніцца. Гэта прататып, а не парытэт функцый з OpenCode ці Hermes.

*Marelle* — французская назва гры ў класікі. Кожная клетачка — гэта крок.

## Усталёўка

**Termux на Android.** Спампуйце гатовы бінарнік са старонкі [рэлізаў](https://github.com/S-I-L-S/marelle/releases) ці збярыце на тэлефоне. Пакрокавая інструкцыя: [docs/TERMUX.md](docs/TERMUX.md).

**Linux і ўсюды, дзе працуе Rust.** Патрэбен Rust 1.85 або навейшы:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ці склануйце рэпазітарый і выканайце `cargo build --release`. Бінарнік — `target/release/marelle`, каля 4 МіБ.

## Першы чат

Трымайце ключ у файле, які чытаеце толькі вы, і скажыце Marelle, дзе ён. Marelle ніколі не захоўвае ключ у сваіх наладах.

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

Замяніце `MODEL` на назву мадэлі са спісу вашага правайдэра. Іншыя спосабы карыстання:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Адказ ідзе ў stdout. Разважанні, колькасць токенаў, заўвагі і памылкі — у stderr, тож `marelle chat ... > answer.txt` захоўвае толькі адказ.

## Інструменты агента

Marelle дасылае структураваныя азначэнні інструментаў праз OpenAI-сумяшчальны і Anthropic API. Убудаваны набор інструментаў:

| Інструмент | Прызначэнне |
| --- | --- |
| `terminal` | Выканаць каманду абалонкі з каталогу запуску |
| `read_file`, `write_file`, `edit_file` | Чытаць і змяняць тэкставыя файлы UTF-8 |
| `list_directory`, `search_text` | Агляд і пошук у файлах праекта |
| `fetch_url` | Атрымаць публічную HTTP(S)-старонку як тэкст (патрэбен `curl`) |
| `list_skills`, `load_skill` | знаходзіць і загружае лакальныя інструкцыі `SKILL.md` |

Каманда `/tools` пакажа спіс інструментаў, `/skills` — даступныя навыкі. Настановы праекта загружаюцца з `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` і асобных файлаў `.marelle/`, калі яны ў дазволеных шляхах.

Файлавыя інструменты абмежаваны `permissions.allowed_paths` (тыпова: бягучы каталог праекта) і ўлічваюць `permissions.denied_paths`. Чытанне дазволена тыпова; змена файлаў, каманды абалонкі і сеткавыя загрузкі спачатку пытаюцца. Дазвол на абалонку датычыцца каманд на хосце і не з'яўляецца пясочніцай файлавай сістэмы: трымайце `ask`, калі не свядома давяраеце мадэлі выкананне каманд.

Агент спыняецца пасля `agent.max_turns` абменаў з інструментамі. Адказы інструментаў лічацца нядавераным зместам. Запыты выкарыстоўваюць поўныя адказы мадэлі, таму аргументы інструментаў не збіраюцца з няпоўных фрагментаў патоку.

## Сесіі

Размовы тыпова захоўваюцца лакальна, разам з гісторыяй выклікаў інструментаў і наладамі мадэлі і сеансу. На Unix каталог сесій мае рэжым `0700`, а JSON-файл — `0600`. Файл захоўваецца ў `$XDG_STATE_HOME/marelle/sessions.json`, ці `~/.local/state/marelle/sessions.json`, калі `XDG_STATE_HOME` не зададзены. API-ключы ніколі не запісваюцца ў гэты файл. Адключыць захаванне:

```sh
marelle config set agent.persist_sessions false
```

Коды выхаду: `0` поспех, `1` памылка правайдэра ці сеткі, `2` памылка выкарыстання ці налад.

## Правайдэры

| id | Пратакол | Зменная ключа | |
| --- | --- | --- | --- |
| `zai` | OpenAI-сумяшчальны | `ZAI_API_KEY` | Z.ai (Zhipu GLM), міжнародны |
| `zai_coding` | OpenAI-сумяшчальны | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-сумяшчальны эндпоінт |
| `zhipu` | OpenAI-сумяшчальны | `ZHIPUAI_API_KEY` | Zhipu BigModel, Кітай |
| `zhipu_coding` | OpenAI-сумяшчальны | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Кітай |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-сумяшчальны | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-сумяшчальны | `OPENROUTER_API_KEY` | шмат мадэляў за адным ключом |
| `ollama` | OpenAI-сумяшчальны | няма | Ollama на гэтай машыне |

Эндпоінты Z.ai і Zhipу ўзятыя са старонняй дакументацыі, а не са старонак пастаўшчыка. Зверце іх з вашым тарыфам і пераазначце любое поле, калі яны розняцца.

Люба поле правайдэра можна змяніць, а новы правайдэр — гэта проста імя з адрасам:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Іншыя поля правайдэра: `kind` (`openai-compatible` ці `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` і `extra_body`. Калі зменная з імя `api_key_env` зададзена, яна выйграе; інакш чытаецца файл.

## Налады

Налады прыходзяць слаямі: убудаваныя тыповыя значэнні, потым зменныя асяроддзя, потым сцягі. Файлы налад становяцца паміж тыповымі значэннямі і асяроддзем.

- `MARELLE_CHAT__MODEL=zai/MODEL` задае `chat.model`. Падвойнае падкрэсленне азначае кропку.
- `marelle config show [prefix]` пералічвае налады і слой, з якога кожная прыйшла.
- `marelle config explain <key>` паказвае кожны слой, што задаў ключ, ад найніжэйшага прыярытэту да найвышэйшага.

| Ключ | Тыпова | Значэнне |
| --- | --- | --- |
| `chat.provider`, `chat.model` | няма | З кім размаўляць. `chat.model` можа быць `provider/model`. |
| `chat.system` | няма | Сістэмны промпт |
| `chat.max_tokens`, `chat.temperature` | няма | Не зададзена — на рашэнне правайдэра |
| `chat.stream` | `true` | Патокавы адказ суддзі, калі ён уключаны; хады агента з інструментамі зараз выкарыстоўваюць поўныя адказы, каб выклікі інструментаў заставаліся цэлымі |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Разважанні і колькасць токенаў у stderr |
| `agent.max_turns` | `20` | Максімум абменаў з інструментамі на адзін запыт карыстальніка |
| `agent.persist_sessions` | `true` | Захоўваць лакальныя размовы паміж запускамі |
| `permissions.shell` | `ask` | Выкананне каманд абалонкі: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Чытанне файлаў: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Змена файлаў: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Загрузка URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Дазволеныя карані для файлавых інструментаў |
| `permissions.denied_paths` | няма | Яўна забароненыя шляхі файлаў |
| `network.connect_timeout_secs` | `15` | Час, дазволены на злучэнне |
| `network.idle_timeout_secs` | `300` | Дазволенае маўчанне пры чаканні даных |
| `network.proxy` | `system` | `system`, `none` ці адрас проксі |
| `network.ca_bundle` | няма | Дадатковы файл давераных цэнтраў сертыфікацыі |
| `network.allow_insecure_http` | `false` | Дазволіць ключ праз звычайны `http` на чужы хост |
| `update.check` | `true` | Пытаць GitHub і GitLab пра навейшую версію на старце інтэрактыўнага сеансу |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, у якога праверка пытае свежы рэліз |
| `update.gitlab_repo` | няма | GitLab `owner/repo` для той жа праверкі. Пуста — GitLab адключаны |
| `update.timeout_secs` | `3` | Колькі секунд праверка можа патраціць на адну крыніцу |

Marelle нясе ўласны спіс давераных цэнтраў сертыфікацыі, таму працуе і там, дзе ў сістэме іх няма, як у Termux. `SSL_CERT_FILE` дадае яшчэ.

Калі знойдзена навейшая версія, сеанс згадае пра яе адзін раз — версію, тое, што ў вас стаіць, і дзе ўзяць. Спачатку пытаецца рэліз; рэпазітарый, што толькі тэгуе коміты, аддае навейшы тэг. Любы невялікі правал заставаецца маўклівым.

## Вашы ключы застаюцца вашымі

- Налады называюць зменную ці файл, дзе ляжыць ключ, але не сам ключ. Такая налада, як `providers.x.api_key`, адхіляецца.
- Ключ не друкуецца ў `config show`, `config explain` ці `providers` і выбракоўваецца з любога тэксту памылкі ад сервера.
- Ён дасылаецца толькі на адрас правайдэра, адным загалоўкам, і ніколі праз звычайны `http` на іншую машыну, калі вы гэтага не дазволілі.
- Перанакіраванні не праводзяцца, таму ключ не можа пайсці кудысь яшчэ.

## Мовы

Інтэрфейс размаўляе англійскай, рускай, французскай і ўкраінскай. Выбірайце праз `--lang <code>` ці `MARELLE_LANG`; інакш вырашаюць `LC_ALL`, `LC_MESSAGES` і `LANG`. Кожнае паведамленне жыве ў каталогу, і тэст падае, калі якая-небудзь мова прапусціла адно. Мэта — усе мовы Еўразіі, а потым яшчэ.

## Дарожная карта

1. **Гаварыць.** Адаптары OpenAI-сумяшчальнага і Anthropic, з SSE-патокавасцю і разборам поўных адказаў. Хады агента з інструментамі зараз выкарыстоўваюць поўныя адказы. *Гатова.*
2. **Налады і выгляд.** TOML-файлы налад, глабальныя і на праект, з `config set`, `get` і `unset` і апісаннем кожнага ключа. Тэматызаваны інтэрфейс у духу [Hermes Agent](https://github.com/NousResearch/hermes-agent): банер, інфармацыйная панэль, адказы ў рамцы, спінер, тыпова ў фіялетавым колеры глициніі, з любым колерам праз HEX. *Гатова.*
3. **Шмат мадэляў.** Аліасы мадэляў, адначасовае пытанне некалькіх мадэляў і параўнанне адказаў, ланцугі fallback і суддзя-мадэль, што злучае іх. *Гатова.*
4. **Дзейнічаць.** Структураваныя выклікі інструментаў для OpenAI-сумяшчальных і Anthropic, лакальныя інструменты файлаў, пошуку і тэрмінала, загрузка URL, лакальныя навыкі і правы allow/ask/deny; транскрыпт паказвае кожны выклік інструмента і яго вынік. *Гатова.*
5. **Памятаць.** Іменаваныя размовы захоўваюцца лакальна паміж запускамі. Доўгатэрміновая семантычная памяць яшчэ наперадзе.
6. **Пашыраць.** Серверы MCP, вонкавыя плагіны, субагенты і багацейшая падтрымка навыкаў.
7. **Дасягаць.** Аўтаматызацыя браўзера, шлюзы мессенджараў і запланаваныя задачы.

Разважанні за дызайнам — у [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Вядомыя абмежаванні

- Селектары правайдэраў і мадэляў паказваюць наладжаных правайдэраў і аліасы мадэляў. Marelle яшчэ не адкрывае аддаленыя каталогі мадэляў.
- Селектары падтрымліваюць клік мышкай для фокусу і пракрутку колёсікам; транскрыпт і рэдактар уводу кіруюцца клавіятурай.
- Ctrl-C выходзіць з інтэрактыўнага сеансу. Пакуль што ён не адмяняе толькі бягучы адказ.
- Агентскі цыкл, лакальныя інструменты з улікай правоў і трывалыя лакальныя сесіі рэалізаваны, але Marelle застаецца пре-альфа прататыпам; пра бягучы дызайн і межы чытайце [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Распрацоўка

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Тэсты ніколі не датыкаюцца з сеткай: яны гавораць з мок-серверам у `crates/marelle-testkit`. GitHub Actions таксама збірае Android-бінарнікі і правярае заяўленую мінімальную версію Rust. Для люстэрка на GitLab ёсць канвеер GitLab CI; глядзіце [docs/GITLAB.md](docs/GITLAB.md).

## Ліцэнзія

[AGPL-3.0-only](LICENSE). Староннія заўвагі — у [NOTICE](NOTICE). Архівы рэлізаў утрымліваюць `THIRD-PARTY-LICENSES.txt` для Rust-крейтаў усярэдзіне бінарніка.
