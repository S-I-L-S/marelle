```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**კონფიგურირებადი ხელოვნური ინტელექტის აგენტი თქვენი ტერმინალისთვის.**
მოიტანეთ საკუთარი API-გასაღებები: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, ადგილობრივი Ollama ან ნებისმიერი OpenAI-თან თავსებადი სერვერი. ჯერ Linux და Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · ქართული · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **წინა-ალფა, ვერსია 0.1.0-alpha.1.** განხორციელებულია: მშობლიური TUI-ინტერფეისი, ჩატი რამდენიმე მომწოდებელთან, უფლებების მცოდნე აგენტის ციკლი, ადგილობრივი ინსტრუმენტები და აღდგენადი ადგილობრივი სესიები. API-ები და პარამეტრები კიდევ შეიძლება შეიცვალოს. ეს პროტოტიპია, და არა ფუნქციების თანაბრობა OpenCode-სთან ან Hermes-თან.

*Marelle* ფრანგულად ბავშვთა ხტუნვის თამაშს ნიშნავს. ყოველი უჯრა — ერთი ნაბიჯი.

## დაყენება

**Termux Android-ზე.** ჩამოტვირთეთ მზა ბინარული ფაილი [გამოშვებების](https://github.com/S-I-L-S/marelle/releases) გვერდიდან, ან ააგეთ ტელეფონზე. ნაბიჯ-ნაბიჯ გზამკვლევი: [docs/TERMUX.md](docs/TERMUX.md).

**Linux და ყველგან, სადაც Rust მუშაობს.** საჭიროა Rust 1.85 ან უფრო ახალი:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

ან დააკლონეთ საცავი და გაუშვით `cargo build --release`. ბინარული ფაილია `target/release/marelle`, დაახლოებით 4 MiB.

## პირველი ჩატი

გასაღები შეინახეთ ფაილში, რომელსაც მხოლოდ თქვენ კითხულობთ, და უთხარით Marelle-ს, სად არის. Marelle გასაღებს პარამეტრებში არასოდეს ინახავს.

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

ჩაანაცვლეთ `MODEL` თქვენი მომწოდებლის სიიდან მოდელის სახელით. გამოყენების სხვა გზები:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

პასუხი მიდის stdout-ზე. მსჯელობანი, ტოკენების რაოდენობა, შენიშვნები და შეცდომები — stderr-ზე, ამიტომ `marelle chat ... > answer.txt` მხოლოდ პასუხს ინახავს.

## აგენტის ინსტრუმენტები

Marelle უგზავნის სტრუქტურირებულ ინსტრუმენტთა აღწერებს OpenAI-თან თავსებადი და Anthropic API-ებით. ჩაშენებული ინსტრუმენტების ნაკრები:

| ინსტრუმენტი | დანიშნულება |
| --- | --- |
| `terminal` | გარსის ბრძანების გაშვება გაშვების საქაღალდიდან |
| `read_file`, `write_file`, `edit_file` | UTF-8 ტექსტური ფაილების წაკითხვა და შეცვლა |
| `list_directory`, `search_text` | პროექტის ფაილების დათვალიერება და ძებნა |
| `fetch_url` | საჯარო HTTP(S) გვერდის ჩამოტვირთვა ტექსტად (სჭირდება `curl`) |
| `list_skills`, `load_skill` | ადგილობრივი `SKILL.md` ინსტრუქციების აღმოჩენა და ჩატვირთვა |

ბრძანება `/tools` აჩვენებს ინსტრუმენტების სიას, `/skills` — ხელმისაწვდომ უნარებს. პროექტის მითითებები იტვირთება `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` და არჩეული `.marelle/` ფაილებიდან, თუ ისინი ნებადართულ გზებზეა.

ფაილური ინსტრუმენტები შეზღუდულია `permissions.allowed_paths`-ით (ნაგულისხმევად: პროექტის მიმდინარე საქაღალდე) და ითვალისწინებს `permissions.denied_paths`-ს. კითხვა ნაგულისხმევად ნებადართულია; ფაილების შეცვლა, გარსის ბრძანებები და ქსელური ჩამოტვირთვები ჯერ კითხავს. გარსის ნებართვა ეხება ბრძანებებს მასპინძელზე და ფაილური სისტემის სავარცხელი არაა: დაიჭირეთ `ask`-ზე, თუ განზრახ არ ენდოთ მოდელს ბრძანებების შესრულებას.

აგენტი ჩერდება `agent.max_turns` ინსტრუმენტული ტურის შემდეგ. ინსტრუმენტების პასუხები უნდობელ შიგთავსად ითვლება. მოთხოვნები იყენებს მოდელის სრულ პასუხებს, ამიტომ ინსტრუმენტების არგუმენტები ნაკადის არასრული ნაჭრებისგან არ იკრიბება.

## სესიები

საუბრები ნაგულისხმევად ადგილობრივად ინახება, ინსტრუმენტთა გამოძახებების ისტორიით და მოდელისა და სესიის პარამეტრებით. Unix-ზე სესიების საქაღალდეს რეჟიმი `0700` აქვს, JSON ფაილს — `0600`. ფაილი ინახება `$XDG_STATE_HOME/marelle/sessions.json`-ში, ან `~/.local/state/marelle/sessions.json`-ში, როცა `XDG_STATE_HOME` არაა დაყენებული. API-გასაღებები ამ ფაილში არასოდეს იწერება. შენახვა გამორთეთ:

```sh
marelle config set agent.persist_sessions false
```

გასვლის კოდები: `0` წარმატება, `1` მომწოდებლის ან ქსელის შეცდომა, `2` გამოყენების ან კონფიგურაციის შეცდომა.

## მომწოდებლები

| id | ოქმი | გასაღების ცვლადი | |
| --- | --- | --- | --- |
| `zai` | OpenAI-თან თავსებადი | `ZAI_API_KEY` | Z.ai (Zhipu GLM), საერთაშორისო |
| `zai_coding` | OpenAI-თან თავსებადი | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-თან თავსებადი ბოლო წერტილი |
| `zhipu` | OpenAI-თან თავსებადი | `ZHIPUAI_API_KEY` | Zhipu BigModel, ჩინეთი |
| `zhipu_coding` | OpenAI-თან თავსებადი | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, ჩინეთი |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-თან თავსებადი | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-თან თავსებადი | `OPENROUTER_API_KEY` | ბევრი მოდელი ერთი გასაღების უკან |
| `ollama` | OpenAI-თან თავსებადი | არაფერი | Ollama ამ მანქანაზე |

Z.ai-ს და Zhipu-ს ბოლო წერტილები მესამე მხარეების დოკუმენტაციიდანაა, და არა მომწოდებლის საკუთარი გვერდებიდან. შეადარეთ თქვენს ტარიფს და გადააწერეთ რომელიმე ველი, თუ განსხვავდება.

მომწოდებლის ყოველი ველი შეცვლადია, და ახალი მომწოდებელი უბრალოდ სახელია მისამართით:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

მომწოდებლის სხვა ველები: `kind` (`openai-compatible` ან `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` და `extra_body`. თუ `api_key_env`-ის მიერ დასახელებული ცვლადი დაყენებულია, ის იმარჯვებს; თორემ ფაილი იკითხება.

## პარამეტრები

პარამეტრები ფენებად მოდის: ჩაშენებული ნაგულისხმევი მნიშვნელობები, შემდეგ გარემოს ცვლადები, შემდეგ დროშები. პარამეტრების ფაილები ნაგულისხმევებსა და გარემოს შორის ჩაერთმევა.

- `MARELLE_CHAT__MODEL=zai/MODEL` აყენებს `chat.model`-ს. ორმაგი ქვედა ხაზი წერტილის ნაცვლად დგას.
- `marelle config show [prefix]` ჩამოთვლის პარამეტრებს და ფენას, საიდანაც თითოეული მოვიდა.
- `marelle config explain <key>` აჩვენებს ყოველ ფენას, რომელმაც გასაღები დააყენა, ყველაზე დაბალი პრიორიტეტიდან უმაღლესამდე.

| გასაღები | ნაგულისხმევი | მნიშვნელობა |
| --- | --- | --- |
| `chat.provider`, `chat.model` | არცერთი | ვისთან ვლაპარაკობთ. `chat.model` შეიძლება `provider/model` იყოს. |
| `chat.system` | არცერთი | სისტემური პრომპტი |
| `chat.max_tokens`, `chat.temperature` | არცერთი | დაუყენებელია — მომწოდებელს რჩება |
| `chat.stream` | `true` | მსაჯულის სურვილისამებრ პასუხის დინება; ინსტრუმენტიანი აგენტური სვლები ამჟამად სრულ პასუხებს იყენებს, რომ ინსტრუმენტთა გამოძახებები მთლიანი დარჩეს |
| `chat.show_reasoning`, `chat.show_usage` | `false` | მსჯელობანი და ტოკენების რაოდენობა stderr-ზე |
| `agent.max_turns` | `20` | მაქსიმუმი ინსტრუმენტული ტურებისა ერთ მომხმარებლის მოთხოვნაზე |
| `agent.persist_sessions` | `true` | ადგილობრივი საუბრების შენახვა გაშვებებს შორის |
| `permissions.shell` | `ask` | გარსის ბრძანებების შესრულება: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | ფაილების კითხვა: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | ფაილების შეცვლა: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-ის ჩამოტვირთვა: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | ნებადართული ფუძეები ფაილური ინსტრუმენტებისთვის |
| `permissions.denied_paths` | არცერთი | აშკარად დაბლოკილი ფაილების გზები |
| `network.connect_timeout_secs` | `15` | დაკავშირებაზე ნებადართული დრო |
| `network.idle_timeout_secs` | `300` | ნებადართული დუმილი მონაცემების ლოდინში |
| `network.proxy` | `system` | `system`, `none`, ან პროქსის მისამართი |
| `network.ca_bundle` | არცერთი | სანდო სერთიფიკაციული ორგანოების დამატებითი ფაილი |
| `network.allow_insecure_http` | `false` | გასაღების უფლება მარტივ `http`-ზე უცხო ჰოსტამდე |
| `update.check` | `true` | GitHub-ს და GitLab-ს ახალი ვერსიის კითხვა ინტერაქტიული სესიის დასაწყისში |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, სადაც შემოწმება უახლეს გამოშვებას კითხავს |
| `update.gitlab_repo` | არცერთი | GitLab `owner/repo` იმავე შემოწმებისთვის. ცარიელი გამორთავს GitLab-ს |
| `update.timeout_secs` | `3` | რამდენი წამი შეუძლია შემოწმებას ერთ წყაროზე დახარჯოს |

Marelle თან ატარებს სანდო სერთიფიკაციული ორგანოების საკუთარ სიას, ამიტომ იქაც მუშაობს, სადაც სისტემას არა აქვს, როგორც Termux-ში. `SSL_CERT_FILE` კიდევ დაამატებს.

როცა უფრო ახალი ვერსია მოიძებნება, სესია ერთხელ მოიხსენიებს — ვერსიას, იმას, რაც თქვენ გაქვთ, და სად აღება. ჯერ გამოშვებას კითხავს; საცავი, რომელიც თავის კომიტებს მხოლოდ ტეგებით ნიშნავს, უახლეს ტეგს იძლევა. რაც უნდა ჩავარდეს, მუნჯად რჩება.

## თქვენი გასაღებები თქვენივე რჩება

- პარამეტრები ცვლადს ან ფაილს ასახელებს, სადაც გასაღები წევს, და არა თავად გასაღებს. ასეთი პარამეტრი, როგორც `providers.x.api_key`, უარყოფილია.
- გასაღებს `config show`, `config explain` ან `providers` არ ბეჭდავს, და ის ამოიჭრება სერვერისგან დაბრუნებული შეცდომის ყოველი ტექსტიდან.
- იგზავნება მხოლოდ მომწოდებლის მისამართზე, ერთ სათაურში, და არასოდეს უბრალო `http`-ზე სხვა მანქანამდე, თუ ამის ნება არ დართეთ.
- გადამისამართებები არ მიჰყვება, ამიტომ გასაღები სხვაგან ვერ გადაიგზავნება.

## ენები

ინტერფეისი ლაპარაკობს ინგლისურად, რუსულად, ფრანგულად და უკრაინულად. აირჩიეთ `--lang <code>`-ით ან `MARELLE_LANG`-ით; თორემ `LC_ALL`, `LC_MESSAGES` და `LANG` გადაწყვეტს. ყოველი შეტყობინება კატალოგში ცხოვრობს, და ტესტი ვარდება, თუ რომელიმე ენა ერთს გამოტოვებს. მიზანია ევრაზიის ყოველი ენა და შემდეგ მეტიც.

## გზამკვლევი

1. **ლაპარაკი.** OpenAI-თან თავსებადი და Anthropic ადაპტერები, SSE-დინებითაც და სრული პასუხების კითხვითაც. ინსტრუმენტიანი აგენტური სვლები ამჟამად სრულ პასუხებს იყენებს. *მზადაა.*
2. **პარამეტრები და სახე.** TOML პარამეტრების ფაილები, გლობალური და პროექტისთვის, `config set`, `get` და `unset`-ით და ყოველი გასაღების აღწერით. თემატიზებული ინტერფეისი [Hermes Agent](https://github.com/NousResearch/hermes-agent)-ის სულისკვეთებით: ბანერი, ინფორმაციული ფილა, ჩარჩოში მოქცეული პასუხები, მბრუნავი, ნაგულისხმევად გლიცინიის იისფერში, ნებისმიერი ფერი HEX-ად. *მზადაა.*
3. **ბევრი მოდელი.** მოდელთა მეტსახელები, რამდენიმე მოდელის ერთდროული კითხვა და პასუხების შედარება, სათადარიგო ჯაჭვები, და მსაჯულ-მოდელი, რომელიც მათ ერთად ადნობს. *მზადაა.*
4. **მოქმედება.** სტრუქტურირებული ინსტრუმენტთა გამოძახებები OpenAI-თან თავსებადი და Anthropic მომწოდებლებისთვის, ადგილობრივი ფაილური, საძიებო და ტერმინალის ინსტრუმენტები, URL-ის ჩამოტვირთვა, ადგილობრივი უნარები და allow/ask/deny უფლებები; ტრანსკრიპტი აჩვენებს ყოველ ინსტრუმენტთა გამოძახებას და მის შედეგს. *მზადაა.*
5. **დამახსოვრება.** დასახელებული საუბრები გაშვებებს შორის ადგილობრივად რჩება. გრძელვადიანი სემანტიკური მეხსიერება ჯერ წინ გვიდგას.
6. **გაფართოება.** MCP სერვერები, გარე მოდულები, ქვე-აგენტები და უნარების უფრო მდიდარი მხარდაჭერა.
7. **მიღწევა.** ბრაუზერის ავტომატიზაცია, მესენჯერთა კარიბჭეები და დაგეგმილი ამოცანები.

დიზაინის უკან მდგომი მსჯელობა [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-შია.

## ცნობილი შეზღუდვები

- მომწოდებელთა და მოდელთა ამჩევები დაყენებულ მომწოდებლებს და მოდელთა მეტსახელებს ჩამოთვლიან. Marelle ჯერ შორეულ მოდელთა კატალოგებს არ აღმოაჩენს.
- ამჩევები ფოკუსისთვის მაუსის დაწკაპუნებას და ბორბლიაკით გადახვევას უჭერენ მხარს; ტრანსკრიპტი და შემავალი რედაქტორი კლავიატურით იმართება.
- Ctrl-C ინტერაქტიული სესიიდან გამოდის. ჯერ მხოლოდ მიმდინარე პასუხს არ წყვეტს.
- აგენტის ციკლი, უფლებების მცოდნე ადგილობრივი ინსტრუმენტები და მუდმივი ადგილობრივი სესიები განხორციელებულია, მაგრამ Marelle წინა-ალფა პროტოტიპად რჩება; ამჟამინდელი დიზაინისა და საზღვრებისთვის წაიკითხეთ [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## განვითარება

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

ტესტები ქსელს არასოდეს ეხება: ისინი მოჩვენებით სერვერთან საუბრობენ `crates/marelle-testkit`-ში. GitHub Actions ასევე ააგებს Android-ის ბინარებს და გამოაცხადებულ მინიმალურ Rust-ის ვერსიას შეამოწმებს. GitLab-ის სარკისთვის GitLab CI-ს ხაზია; იხილეთ [docs/GITLAB.md](docs/GITLAB.md).

## ლიცენზია

[AGPL-3.0-only](LICENSE). მესამე მხარეთა შენიშვნები [NOTICE](NOTICE)-შია. გამოშვების არქივები მოიცავს `THIRD-PARTY-LICENSES.txt`-ს ბინარში მყოფი Rust-ყუთებისთვის.
