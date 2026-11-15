```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigurējams MI aģents jūsu terminālī.**
Atvediet savus API atslēgas: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokāla Ollama vai jebkurš OpenAI saderīgs serveris. Vispirms Linux un Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · Latviešu · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Priekš-alfa, versija 0.1.0-alpha.1.** Ieviests: natīvs TUI saskarne, čats ar vairākiem piegādātājiem, aģenta cikls, kas apzina atļaujas, lokāli rīki un atjaunojamas lokālas sesijas. API un iestatījumi vēl var mainīties. Šis ir prototips, nevis funkciju paritāte ar OpenCode vai Hermes.

*Marelle* ir bērnu lēkšanas spēles franču nosaukums. Katrs kvadrāts ir solis.

## Instalēšana

**Termux uz Androida.** Lejupielādējiet gatavu binārfailu no [laidienu](https://github.com/S-I-L-S/marelle/releases) lapas vai uzbūvējiet to telefonā. Solis pa solim ceļvedis: [docs/TERMUX.md](docs/TERMUX.md).

**Linux un visur, kur strādā Rust.** Vajag Rust 1.85 vai jaunāku:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Vai arī klonējiet repozitoriju un palaidiet `cargo build --release`. Binārā fails ir `target/release/marelle`, apmēram 4 MiB.

## Pirmais čats

Glabājiet atslēgu failā, ko lasa tikai jūs, un pasakiet Marelle, kur tā ir. Marelle nekad nesaglabā atslēgu savos iestatījumos.

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

Aizvietojiet `MODEL` ar modeļa nosaukumu no jūsu piegādātāja saraksta. Citi lietošanas veidi:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Atbilde ies uz stdout. Pārdomas, tokenu skaiti, piezīmes un kļūdas ies uz stderr, tāpēc `marelle chat ... > answer.txt` saglabā tikai atbildi.

## Aģenta rīki

Marelle sūta strukturētus rīku aprakstus caur OpenAI saderīgiem un Anthropic API. Iebūvētais rīku komplekts:

| Rīks | Nolūks |
| --- | --- |
| `terminal` | Izpildīt apvalka komandu no palaišanas mapes |
| `read_file`, `write_file`, `edit_file` | Lasīt un mainīt UTF-8 teksta failus |
| `list_directory`, `search_text` | Pārskatīt un meklēt projekta failos |
| `fetch_url` | Lejupielādēt publisku HTTP(S) lapu kā tekstu (vajag `curl`) |
| `list_skills`, `load_skill` | Atrast un ielādēt lokālus `SKILL.md` norādījumus |

Komanda `/tools` parāda rīku sarakstu, `/skills` — pieejamās prasmes. Projekta norādes ielādējas no `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` un izvēlētiem `.marelle/` failiem, ja tie ir atļautajos ceļos.

Failu rīki ir ierobežoti ar `permissions.allowed_paths` (pēc noklusējuma: pašreizējā projekta mape) un ņem vērā `permissions.denied_paths`. Lasīšana pēc noklusējuma ir atļauta; failu mainīšana, apvalka komandas un tīkla lejupielādes vispirms jautā. Apvalka atļauja attiecas uz komandām uz resursdatora un nav failu sistēmas smilškaste: turēt to uz `ask`, ja vien tīšā neuzticat modelim komandu izpildi.

Aģents apstājas pēc `agent.max_turns` rīku apmaiņām. Rīku atbildes uzskata par neuzticamu saturu. Pieprasījumi izmanto pilnus modeļa atbildes, tāpēc rīku argumenti nesastādas no nepilnīgiem plūsmas gabaliem.

## Sesijas

Sarunas pēc noklusējuma saglabājas lokāli, tostarp rīku izsaukumu vēsture un modeļa un sesijas iestatījumi. Unix sesiju mapei ir režīms `0700`, JSON failam — `0600`. Fails glabājas `$XDG_STATE_HOME/marelle/sessions.json`, vai `~/.local/state/marelle/sessions.json`, kad `XDG_STATE_HOME` nav iestatīts. API atslēgas nekad neierakstās šajā failā. Atslēdziet saglabāšanu ar:

```sh
marelle config set agent.persist_sessions false
```

Iziešanas kodi: `0` panākums, `1` piegādātāja vai tīkla kļūda, `2` lietošanas vai iestatījumu kļūda.

## Piegādātāji

| id | Protokols | Atslēgas mainīgais | |
| --- | --- | --- | --- |
| `zai` | saderīgs ar OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), starptautisks |
| `zai_coding` | saderīgs ar OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, galapunkts, saderīgs ar Anthropic |
| `zhipu` | saderīgs ar OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Ķīna |
| `zhipu_coding` | saderīgs ar OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Ķīna |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | saderīgs ar OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | saderīgs ar OpenAI | `OPENROUTER_API_KEY` | daudzi modeļi aiz vienas atslēgas |
| `ollama` | saderīgs ar OpenAI | nav | Ollama šajā mašīnā |

Z.ai un Zhipu galapunkti nāk no trešo pušu dokumentācijas, nevis no piegādātāja paša lapām. Pārbaudiet tos pret savu plānu un pārrakstiet jebkuru lauku, ja tie atšķiras.

Katrs piegādātāja lauks ir maināms, un jauns piegādātājs ir vienkārši vārds ar adresi:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Pārējie piegādātāja lauki: `kind` (`openai-compatible` vai `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` un `extra_body`. Ja mainīgais, nosaukts `api_key_env`, ir uzstādīts, tas uzvar; citādi tiek lasīts fails.

## Iestatījumi

Iestatījumi nāk slāņos: iebūvētas noklusējuma vērtības, tad vides mainīgie, tad karodziņi. Iestatījumu faili iejaucas starp noklusējuma vērtībām un vidi.

- `MARELLE_CHAT__MODEL=zai/MODEL` uzstāda `chat.model`. Dubults pasvītra triks nozīmē punktu.
- `marelle config show [prefix]` uzskaita iestatījumus un slāni, no kura katrs nācis.
- `marelle config explain <key>` parāda katru slāni, kas uzstādīja atslēgu, no zemākās prioritātes uz augstāko.

| Atslēga | Noklusējums | Nozīme |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nav | Ar ko runāt. `chat.model` var būt `provider/model`. |
| `chat.system` | nav | Sistēmas uzvedne |
| `chat.max_tokens`, `chat.temperature` | nav | Nav uzstādīts — atstāts piegādātājam |
| `chat.stream` | `true` | Sodīša atbildes plūsma, kad tā ieslēgta; aģenta soļi ar rīkiem pašlaik izmanto pilnas atbildes, lai rīku izsaukumi paliktu veseli |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Pārdomas un tokenu skaits uz stderr |
| `agent.max_turns` | `20` | Maksimum rīku apmaiņu vienā lietotāja pieprasījumā |
| `agent.persist_sessions` | `true` | Saglabāt lokālas sarunas starp palaidieniem |
| `permissions.shell` | `ask` | Apvalka komandu izpilde: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Failu lasīšana: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Failu mainīšana: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL lejupielāde: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Atļautas saknes failu rīkiem |
| `permissions.denied_paths` | nav | Skaidri bloķēti failu ceļi |
| `network.connect_timeout_secs` | `15` | Laiks, atļauts savienojumam |
| `network.idle_timeout_secs` | `300` | Atļauta klusums, gaidot datus |
| `network.proxy` | `system` | `system`, `none` vai starpnieka adrese |
| `network.ca_bundle` | nav | Papildu fails ar uzticamiem sertifikācijas organismiem |
| `network.allow_insecure_http` | `false` | Atļaut atslēgu pār vienkāršu `http` uz svešu resursdatoru |
| `update.check` | `true` | Jautāt GitHub un GitLab par jaunāku versiju interaktīvas sesijas sākumā |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, no kura pārbaude jautā par jaunāko laidienu |
| `update.gitlab_repo` | nav | GitLab `owner/repo` tai pašai pārbaudei. Tukšs izslēdz GitLab |
| `update.timeout_secs` | `3` | Cik sekunžu pārbaude var pavadīt pie viena avota |

Marelle nes savu uzticamo sertifikācijas organismu sarakstu, tāpēc strādā arī tur, kur sistēmā to nav, kā Termux. `SSL_CERT_FILE` pievieno vēl.

Kad atrod jaunāku versiju, sesija to piemin vienreiz — versiju, to, kas jums ir, un kur to ņemt. Vispirms jautā laidienu; repozitorijs, kas savus iesniegumus tikai atzīmē ar birkām, dod jaunāko birku. Kas gan cietu, paliek kluss.

## Jūsu atslēgas paliek jūsu

- Iestatījumi nosauc mainīgo vai failu, kur atslēga guļ, nevis pašu atslēgu. Tāds iestatījums kā `providers.x.api_key` tiek noraidīts.
- Atslēgu `config show`, `config explain` vai `providers` neizdrukā, un tā tiek izgriezta no jebkura servera atnākuša kļūdu teksta.
- Tā sūtas tikai uz piegādātāja adresi, vienā galvenē, un nekad pār vienkāršu `http` uz citu mašīnu, ja jūs to neatļaujat.
- Pāradresācijas neseko, tāpēc atslēga nevar aizpeldēt citur.

## Valodas

Saskarne runā angļu, krievu, franču un ukraiņu valodā. Izvēlieties ar `--lang <code>` vai `MARELLE_LANG`; citādi izšķir `LC_ALL`, `LC_MESSAGES` un `LANG`. Katrs ziņojums dzīvo katalogā, un tests gūžas, ja kāda valoda vienu atstāj ārā. Mērķis ir visas Eirāzijas valodas, un tad vēl.

## Ceļa karte

1. **Runāt.** Adapteri, saderīgi ar OpenAI, un Anthropic, ar SSE plūsmu un pilnu atbilžu nolasīšanu. Aģenta soļi ar rīkiem pašlaik izmanto pilnas atbildes. *Gatavs.*
2. **Iestatījumi un izskats.** TOML iestatījumu faili, globāli un pa projektiem, ar `config set`, `get` un `unset` un aprakstu katrai atslēgai. Tematizēta saskarne [Hermes Agent](https://github.com/NousResearch/hermes-agent) garā: baneris, informācijas panelis, atbildes rāmī, virpulītis, pēc noklusējuma ceriņos, ar jebkuru krāsu kā HEX. *Gatavs.*
3. **Daudz modeļu.** Modeļu aizstājvārdi, vienlaikus jautājums vairākiem modeļiem un atbilžu salīdzināšana, rezerves ķēdes un tiesneša modelis, kas tos sapludina. *Gatavs.*
4. **Darīt.** Strukturēti rīku izsaukumi OpenAI saderīgiem un Anthropic piegādātājiem, lokāli failu, meklēšanas un termināla rīki, URL lejupielāde, lokālas prasmes un atļaujas allow/ask/deny; transkrīts rāda katru rīka izsaukumu un tā rezultātu. *Gatavs.*
5. **Atcerēties.** Nosauktas sarunas saglabājas lokāli starp palaidieniem. Ilgtermiņa semantiskā atmiņa vēl priekšā.
6. **Paplašināt.** MCP serveri, ārējie spraudņi, subaģenti un bagātāks prasmju atbalsts.
7. **Sasniegt.** Pārlūka automatizācija, ziņnešu vārti un plānoti uzdevumi.

Pārdomas aiz dizaina ir [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Zināmie ierobežojumi

- Piegādātāju un modeļu izvēlnes uzskaita uzstādītos piegādātājus un modeļu aizstājvārdus. Marelle vēl neatrod attālinātus modeļu katalogus.
- Izvēlnes atbalsta peles klikšķi fokusam un ripināšanu ar ritenīti; transkrīts un ievades redaktors paliek ar tastatūru vadāmi.
- Ctrl-C izej no interaktīvas sesijas. Vēl neatsaka tikai pašreizējo atbildi.
- Aģenta cikls, lokāli rīki, kas apzin atļaujas, un noturīgas lokālas sesijas ieviesti, bet Marelle paliek priekš-alfa prototips; par pašreizējo dizainu un robežām lasiet [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Attīstība

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testi nekad neaiztīk tīklu: tie runā ar maketa serveri `crates/marelle-testkit`. GitHub Actions arī būvē Android binārniekus un pārbauda deklarēto minimālo Rust versiju. GitLab spoguļim ir GitLab CI cauruļvads; skatiet [docs/GITLAB.md](docs/GITLAB.md).

## Licences

[AGPL-3.0-only](LICENSE). Trešo pušu paziņojumi ir [NOTICE](NOTICE). Laidienu arhīvos iekļauts `THIRD-PARTY-LICENSES.txt` Rust kastēm binārā iekšpusē.
