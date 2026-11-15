```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Ձեր տերմինալի համար կարգավորելի արհեստական բանականության գործակալ։**
Բերեք ձեր սեփական API բանալիները․ Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, տեղական Ollama կամ OpenAI-ի հետ համատեղելի ցանկացած սերվեր։ Նախ ՝ Linux և Termux։

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · Հայերեն
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Նախ-ալֆա, տարբերակ 0.1.0-alpha.1։** Իրագործված է․ բնիկ TUI միջերես, զրույց մի քանի մատակարարների հետ, թույլտվություններին տեղյակ գործակալի շրջանակ, տեղական գործիքներ և վերսկսելի տեղական նիստեր։ API-ները և կարգավորումները դեռ կարող են փոխվել։ Սա նախատիպ է, և ոչ թե OpenCode-ի կամ Hermes-ի հետ հնարավորությունների համապատասխանություն։

*Marelle*-ն ֆրանսերեն անվանումն է երեխաների ցատկելու խաղի։ Յուրաքանչյուր քառակուսին մեկ քայլ է։

## Տեղադրում

**Termux Android-ի վրա։** Բեռնեք պատրաստի երկուական ֆայլը [թողարկումների](https://github.com/S-I-L-S/marelle/releases) էջից, կամ հավաքեք հեռախոսում։ Քայլ առ քայլ ուղեցույց․ [docs/TERMUX.md](docs/TERMUX.md)։

**Linux և ամենուր, որտեղ Rust-ն աշխատում է։** Անհրաժեշտ է Rust 1.85 կամ ավելի նոր․

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Կամ կլոնավորեք պահոցը և գործարկեք `cargo build --release`։ Երկուական ֆայլը `target/release/marelle`-ն է, մոտ 4 ՄԲ։

## Առաջին զրույցը

Բանալին պահեք այնպիսի ֆայլում, որը կարդում է միայն ձեզ, և ասեք Marelle-ին, որտեղ է այն։ Marelle-ն բանալին երբեք չի պահում իր կարգավորումներում։

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

Փոխարինեք `MODEL`-ը ձեր մատակարարի ցուցակից մոդելի անունով։ Օգտագործման այլ եղանակներ․

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Պատասխանը գնում է stdout։ Մտածողությունները, տոքենների հաշիվը, նշումները և սխալները գնում են stderr, որոնին ՝ `marelle chat ... > answer.txt` պահում է միայն պատասխանը։

## Գործակալի գործիքները

Marelle-ն ուղարկում է կառուցվածքավորված գործիքների սահմանումներ OpenAI-ի հետ համատեղելի և Anthropic API-ներով։ Ներդրված գործիքների հավաքածուն․

| Գործիք | Նպատակ |
| --- | --- |
| `terminal` | Կառավարամեջբի հրաման գործարկել մեկնարկի թղթապանակից |
| `read_file`, `write_file`, `edit_file` | Կարդալ և փոփոխել UTF-8 տեքստային ֆայլերը |
| `list_directory`, `search_text` | Զննել և որոնել նախագծի ֆայլերում |
| `fetch_url` | Բեռնել հանրային HTTP(S) էջը որպես տեքստ (պետք է `curl`) |
| `list_skills`, `load_skill` | Հայտնաբերել և բեռնել տեղական `SKILL.md` ցուցումները |

`/tools` հրամանը ցույց է տալիս գործիքների ցանկը, `/skills`-ը՝ մատչելի հմտությունները։ Նախագծի ուղեցույցները բեռնվում են `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` և ընտրված `.marelle/` ֆայլերից, եթե դրանք թույլատրելի ուղիներում են։

Ֆայլային գործիքները սահմանափակված են `permissions.allowed_paths`-ով (լռելյայն․ նախագծի ընթացիկ թղթապանակը) և հաշվի են առնում `permissions.denied_paths`-ը։ Կարդալը լռելյայն թույլատրված է․ ֆայլերի փոփոխությունները, համակարգչային հրամանները և ցանցից բեռնումները նախ հարցնում են։ Համակարգչային թույլտվությունը վերաբերում է հյուրընկալողի հրամաններին և ֆայլային համակարգի ավազան չէ․ պահեք այն `ask`-ի վրա, եթե միտմամբ մոդելին չեք վստահում հրամանների կատարումը։

Գործակալը կանգնում է `agent.max_turns` գործիքային փոխանակումներից հետո։ Գործիքների պատասխանները համարվում են անվստահելի բովանդակություն։ Հարցումներն օգտագործում են մոդելի ամբողջական պատասխաններ, որոնին գործիքների փաստարկները հոսքի թերի կտորներից չեն հավաքվում։

## Նիստերը

Զրույցները լռելյայն պահվում են տեղում, ներառյալ գործիքների կանչերի պատմությունը և մոդելի ու նիստի կարգավորումները։ Unix-ում նիստերի թղթապանակը `0700` եղանակ ունի, JSON ֆայլը՝ `0600`։ Ֆայլը պահվում է `$XDG_STATE_HOME/marelle/sessions.json`-ում, կամ `~/.local/state/marelle/sessions.json`-ում, երբ `XDG_STATE_HOME`-ը չի դրված։ API բանալիները երբեք չեն գրվում այդ ֆայլում։ Պահպանումն անջատեք․

```sh
marelle config set agent.persist_sessions false
```

Ելքի կոդերը․ `0` հաջողություն, `1` մատակարարի կամ ցանցի սխալ, `2` օգտագործման կամ կարգավորման սխալ։

## Մատակարարները

| id | Արձանագրություն | Բանալու փոփոխական | |
| --- | --- | --- | --- |
| `zai` | OpenAI-ի հետ համատեղելի | `ZAI_API_KEY` | Z.ai (Zhipu GLM), միջազգային |
| `zai_coding` | OpenAI-ի հետ համատեղելի | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-ի հետ համատեղելի վերջնակետ |
| `zhipu` | OpenAI-ի հետ համատեղելի | `ZHIPUAI_API_KEY` | Zhipu BigModel, Չինաստան |
| `zhipu_coding` | OpenAI-ի հետ համատեղելի | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Չինաստան |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-ի հետ համատեղելի | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-ի հետ համատեղելի | `OPENROUTER_API_KEY` | շատ մոդելներ մի բանալու ետևում |
| `ollama` | OpenAI-ի հետ համատեղելի | ոչ մի | Ollama այս մեքենայի վրա |

Z.ai-ի և Zhipu-ի վերջնակետերը երրորդ կողմերի փաստաթղթերից են, ոչ թե մատակարարի սեփական էջերից։ Համեմատեք դրանք ձեր փաթեթի հետ և գերագրեք որևէ դաշտ, եթե տարբերվում են։

Մատակարարի յուրաքանչյուր դաշտ փոփոխելի է, իսկ նոր մատակարարը պարզապես անուն է հասցեով․

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Մատակարարի մյուս դաշտերը․ `kind` (`openai-compatible` կամ `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` և `extra_body`։ Եթե `api_key_env`-ի անվանած փոփոխականը դրված է, նա հաղթում է․ այլապես կարդացվում է ֆայլը։

## Կարգավորումները

Կարգավորումները գալիս են շերտերով․ ներդրված լռելյայն արժեքներ, հետո միջավայրի փոփոխականներ, հետո դրոշներ։ Կարգավորումների ֆայլերը դնում են իրենց լռելյայնների և միջավայրի միջև։

- `MARELLE_CHAT__MODEL=zai/MODEL` դնում է `chat.model`-ը։ Կրկնակի տողը կետի փոխարեն կանգնում է։
- `marelle config show [prefix]` ցուցակագրում է կարգավորումները և շերտը, որից յուրաքանչյուրը եկել է։
- `marelle config explain <key>` ցույց է տալիս ամեն շերտ, որը բանալին դրել է, ամենացածր առաջնահերթությունից մինչև ամենաբարձրը։

| Բանալի | Լռելյայն | Իմաստ |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ոչ մի | Ում հետ խոսել։ `chat.model`-ը կարող է `provider/model` լինել։ |
| `chat.system` | ոչ մի | Համակարգային հուշում |
| `chat.max_tokens`, `chat.temperature` | ոչ մի | Դրված չէ — մատակարարին է մնում |
| `chat.stream` | `true` | Դատավորի ոչ պարտադիր պատասխանի հոսքը․ գործիքներով գործակալի քայլերը ներկա պահին ամբողջական պատասխաններ են օգտագործում, որոնին գործիքների կանչերը ամբողջ մնան |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Մտածողություններ և տոքենների հաշիվ stderr-ի վրա |
| `agent.max_turns` | `20` | Առավելագույնը գործիքային փոխանակումների՝ օգտագործողի մի հարցման համար |
| `agent.persist_sessions` | `true` | Տեղական զրույցները պահել գործարկումների միջև |
| `permissions.shell` | `ask` | Համակարգչային հրամանների կատարում․ `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Ֆայլերի կարդալը․ `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Ֆայլերի փոփոխումը․ `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL բեռնումը․ `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Թույլատրված արմատներ ֆայլային գործիքների համար |
| `permissions.denied_paths` | ոչ մի | Բացահայտ արգելափակված ֆայլերի ուղիներ |
| `network.connect_timeout_secs` | `15` | Միանալու թույլատրված ժամանակը |
| `network.idle_timeout_secs` | `300` | Թույլատրված լռությունը տվյալների սպասման ժամանակ |
| `network.proxy` | `system` | `system`, `none`, կամ միջնորդի հասցեն |
| `network.ca_bundle` | ոչ մի | Վստահելի վկայագրերի կենտրոնների հավելյալ ֆայլ |
| `network.allow_insecure_http` | `false` | Թույլատրել բանալին մերկ `http`-ով օտար հյուրընկալողին |
| `update.check` | `true` | GitHub-ին և GitLab-ին հարցնել ավելի նոր տարբերակի մասին ինտերակտիվ նիստի սկզբում |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`-ն, որից ստուգումը ամենանոր թողարկումը հարցնում է |
| `update.gitlab_repo` | ոչ մի | GitLab `owner/repo`-ն նույն ստուգման համար։ Դատարկը անջատում է GitLab-ը |
| `update.timeout_secs` | `3` | Քանի վայրկյան ստուգումը կարող է մի աղբյուրի վրա ծախսել |

Marelle-ն իր հետ տանում է վստահելի վկայագրերի կենտրոնների սեփական ցուցակը, որոնին աշխատում է նաև այնտեղ, որտեղ համակարգը չունի, ինչպես Termux-ում։ `SSL_CERT_FILE`-ը ավելացնում է։

Երբ ավելի նոր տարբերակ է գտնվում, նիստը մեկ անգամ կհիշատակի այն — տարբերակը, ձեզ մոտ եղածը և որտեղից վերցնել։ Նախ թողարկումն է հարցվում․ պահոցը, որն իր փոփոխությունները միայն պիտակներով է նշում, ամենանոր պիտակը տալիս է։ Ինչ անհաջող լինի, լուռ մնա։

## Ձեր բանալիները ձերն են մնում

- Կարգավորումները անուն են տալիս փոփոխականի կամ ֆայլի, որում բանալին է պառկած, և ոչ մի դեպքում ինքնին բանալիին։ Այնպիսի կարգավորում, ինչպես `providers.x.api_key`, մերժվում է։
- Բանալին `config show`, `config explain` կամ `providers` չեն տպում, և այն կտրվում է սերվերից վերադարձած ամեն սխալի տեքստից։
- Այն ուղարկվում է միայն մատակարարի հասցեին, մի գլխագրում, և երբեք մերկ `http`-ով այլ մեքենա, եթե դուք թույլ չտաք։
- Վերահասցեավորումներին հետևում չեն, որոնին բանալին ուրիշ տեղ հասցնել հնարավոր չէ։

## Լեզուներ

Միջերեսը խոսում է անգլերեն, ռուսերեն, ֆրանսերեն և ուկրաիներեն։ Ընտրեք `--lang <code>`-ով կամ `MARELLE_LANG`-ով․ այլապես `LC_ALL`-ը, `LC_MESSAGES`-ը և `LANG`-ը որոշում են։ Ամեն հաղորդագրություն կատալոգում է ապրում, և թեստը ընկնում է, եթե որևէ լեզու մեկն անց կացնի։ Նպատակը Եվրասիայի ամեն լեզուն է, հետո ավելին։

## Ճանապարհի քարտեզը

1. **Խոսել։** OpenAI-ի հետ համատեղելի և Anthropic հարմարեցուցիչներ, ինչպես SSE հոսքով, այնպես էլ ամբողջական պատասխանների ընթերցումով։ Գործիքներով գործակալի քայլերը ներկա պահին ամբողջական պատասխաններ են օգտագործում։ *Պատրաստ է։*
2. **Կարգավորումներ և տեսք։** TOML կարգավորման ֆայլեր, համապարփակ և ըստ նախագծի, `config set`, `get` և `unset`-ով և ամեն բանալու նկարագրությամբ։ Թեմավորված միջերես [Hermes Agent](https://github.com/NousResearch/hermes-agent)-ի ոգով․ ցուցատախտակ, տեղեկատվական վահանակ, շրջանակի մեջ առած պատասխաններ, պտուտակ, լռելյայն գլիցինիայի մանուշակագույնով, ամեն գույն HEX-ով դնելի։ *Պատրաստ է։*
3. **Բազմաթիվ մոդելներ։** Մոդելների մականուններ, մի քանի մոդելի միաժամանակ հարցումը և պատասխանների համեմատումը, պահեստային շղթաները, և դատավոր մոդել, որը դրանք միաձուլում է։ *Պատրաստ է։*
4. **Գործել։** Կառուցվածքավորված գործիքների կանչեր OpenAI-ի հետ համատեղելի և Anthropic մատակարարների համար, տեղական ֆայլային, որոնման և տերմինալի գործիքներ, URL բեռնում, տեղական հմտություններ և allow/ask/deny թույլտվություններ․ գրառումը ցույց է տալիս ամեն գործիքային կանչ և նրա արդյունքը։ *Պատրաստ է։*
5. **Հիշել։** Անվանված զրույցները գործարկումների միջև տեղում են մնում։ Երկարաժամկետ իմաստային հիշողությունը դեռ առջևում է։
6. **Ընդարձակել։** MCP սերվերներ, արտաքին խցանումներ, ենթագործակալներ և հմտությունների ավելի հարուստ աջակցություն։
7. **Հասնել։** Զննիչի ավտոմատացում, նամականի դարպասներ և ժամանակացույցով դրված առաջադրանքներ։

Ձևավորման ետևում ընկած մտորումները [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-ում են։

## Հայտնի սահմանափակումները

- Մատակարարների և մոդելների ընտրիչները դրված մատակարարներին և մոդելների մականունները ցուցակագրում են։ Marelle-ն դեռ հեռավոր մոդելների ցուցակներ չի հայտնաբերում։
- Ընտրիչները ուշադրության համար մկնիկի սեղմում և անիվով ոլորում են սատարում․ գրառումը և մուտքի խմբագրիչը ստեղնաշարով են մնում։
- Ctrl-C-ն դուրս է գալիս ինտերակտիվ նիստից։ Դեռ միայն ընթացիկ պատասխանն է չի ընդհատում։
- Գործակալի շրջանակը, թույլտվություններին տեղյակ տեղական գործիքները և հավերժական տեղական նիստերը իրագործված են, բայց Marelle-ն նախ-ալֆա նախատիպ է մնում․ ներկա ձևավորման և սահմանների մասին կարդացեք [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-ում։

## Զարգացում

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Փորձարկումները ցանցին երբեք չեն դիպչում․ նրանք խոսում են կեղծ սերվերի հետ `crates/marelle-testkit`-ում։ GitHub Actions-ը նաև Android-ի երկուականներ է հավաքում և հայտարարված նվազագույն Rust-ի տարբերակը ստուգում։ GitLab-ի հայելու համար GitLab CI խողովակագիծ կա․ տեսեք [docs/GITLAB.md](docs/GITLAB.md)։

## Արտոնագիր

[AGPL-3.0-only](LICENSE)։ Երրորդ կողմերի նշումները [NOTICE](NOTICE)-ում են։ Թողարկումների արխիվները ներառում են `THIRD-PARTY-LICENSES.txt` երկուականում գտնվող Rust տուփերի համար։
