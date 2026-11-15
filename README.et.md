```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Seadistatav tehisintellekti agent teie terminalile.**
Tooge oma API-võtmed: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, kohalik Ollama või ükskõik milline OpenAI-ga ühilduv server. Kõigepealt Linux ja Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · Eesti · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Eel-alfa, versioon 0.1.0-alpha.1.** Rakendatud: oma TUI-liides, vestlus mitme pakkuja kaudu, õigustest teadlik agendi tsükkel, kohalikud tööriistad ja taastatavad kohalikud sessioonid. API-d ja seaded võivad veel muutuda. See on prototüüp, mitte funktsioonide võrdsus OpenCode või Hermesega.

*Marelle* on prantsuse laste hüplemängu nimi. Iga ruut on üks samm.

## Paigaldamine

**Termux Androidil.** Laadige valmis binaarfail alla [väljalasete](https://github.com/S-I-L-S/marelle/releases) lehelt või ehitage see telefonis. Samm-sammult juhend: [docs/TERMUX.md](docs/TERMUX.md).

**Linux ja kõikjal, kus Rust töötab.** Vaja on Rust 1.85 või uuem:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Või kloonige repositoorium ja käivitage `cargo build --release`. Binaar on `target/release/marelle`, umbes 4 MiB.

## Esimene vestlus

Hoidke võtit failis, mida loete ainult teie, ja öelge Marelle'ile, kus ta on. Marelle ei salvesta võtit kunagi oma seadetesse.

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

Asendage `MODEL` mudeli nimega teie pakkuja loendist. Teised kasutusviisid:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Vastus läheb stdout'ile. Mõtisklused, tokenite arv, märkused ja vead lähevad stderr'ile, nii et `marelle chat ... > answer.txt` salvestab ainult vastuse.

## Agendi tööriistad

Marelle saadab struktureeritud tööriistade kirjeldusi läbi OpenAI-ga ühilduvate ja Anthropic API-de. Sisseehitatud tööriistade kogum:

| Tööriist | Eesmärk |
| --- | --- |
| `terminal` | Käivitada shelli käsk käivituskataloogist |
| `read_file`, `write_file`, `edit_file` | Lugeda ja muuta UTF-8 tekstifaile |
| `list_directory`, `search_text` | Vaadata ja otsida projekti failidest |
| `fetch_url` | Laadida avalik HTTP(S)-lehekülg tekstina (vaja `curl`) |
| `list_skills`, `load_skill` | Leida ja laadida kohalikke `SKILL.md` juhiseid |

Käsk `/tools` näitab tööriistade loendit ja `/skills` saadaolevaid oskuseid. Projekti juhised laaditakse `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` ja valitud `.marelle/` failidest, kui nad on lubatud radadel.

Failitööriistad on piiratud `permissions.allowed_paths`-ga (vaikimisi: praegune projekti kataloog) ja arvestavad `permissions.denied_paths`-t. Lugemine on vaikimisi lubatud; failide muutmine, shelli käsud ja võrgust laadimine küsivad enne. Shelli õigus puudutab käske hostil ega ole failisüsteemi liivakast: hoige seda `ask`-il, kui te teadlikult ei usalda mudelile käskude käivitamist.

Agent peatub pärast `agent.max_turns` tööriistaringe. Tööriistade vastuseid käsitletakse kui ebausaldusväärset sisu. Päringud kasutavad mudeli terviklikke vastuseid, nii et tööriistade argumendid ei kogune mittetäielikest voogude killukestest.

## Sessioonid

Vestlused salvestatakse vaikimisi kohalikult, sealhulgas tööriistakutsete ajalugu ning mudeli ja sessiooni seaded. Unixil on sessioonide kataloogil režiim `0700` ja JSON-failil `0600`. Fail hoitakse asukohas `$XDG_STATE_HOME/marelle/sessions.json`, või `~/.local/state/marelle/sessions.json`, kui `XDG_STATE_HOME` pole määratud. API-võtmeid ei kirjutata sellesse faili kunagi. Väljalülitage salvestamine käsuga:

```sh
marelle config set agent.persist_sessions false
```

Väljumiskoodid: `0` edu, `1` pakkuja või võrgu viga, `2` kasutamise või seadete viga.

## Pakkujad

| id | Protokoll | Võtme muutuja | |
| --- | --- | --- | --- |
| `zai` | ühilduv OpenAI-ga | `ZAI_API_KEY` | Z.ai (Zhipu GLM), rahvusvaheline |
| `zai_coding` | ühilduv OpenAI-ga | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic'iga ühilduv lõpp-punkt |
| `zhipu` | ühilduv OpenAI-ga | `ZHIPUAI_API_KEY` | Zhipu BigModel, Hiina |
| `zhipu_coding` | ühilduv OpenAI-ga | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Hiina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | ühilduv OpenAI-ga | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | ühilduv OpenAI-ga | `OPENROUTER_API_KEY` | palju mudeleid ühe võtme taga |
| `ollama` | ühilduv OpenAI-ga | puudub | Ollama selles masinas |

Z.ai ja Zhipu lõpp-punktid pärinevad kolmandate osapoolte dokumentatsioonist, mitte tarnija enda lehekülgedelt. Kontrollige neid oma plaani suhtes ja tõstke ükskõik milline väli üle, kui nad erinevad.

Iga pakkuja väli on muudetav ja uus pakkuja on lihtsalt nimi aadressiga:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Teised pakkuja väljad: `kind` (`openai-compatible` või `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ja `extra_body`. Kui `api_key_env` poolt nimetatud muutuja on määratud, võidab ta; muidu loetakse faili.

## Seaded

Seaded tulevad kihtidena: sisseehitatud vaikeväärtused, siis keskkonnamuutujad, siis lipud. Seadete failid seavad end vaikeväärtuste ja keskkonna vahele.

- `MARELLE_CHAT__MODEL=zai/MODEL` määrab `chat.model`. Korduv alakriips tähistab punkti.
- `marelle config show [prefix]` loetleb seaded ja kihi, kust iga tuli.
- `marelle config explain <key>` näitab iga kihti, mis kõnealust võtit määras, madalaimast prioriteedist kõrgeimani.

| Võti | Vaikeväärtus | Tähendus |
| --- | --- | --- |
| `chat.provider`, `chat.model` | puudub | Kellega rääkida. `chat.model` võib olla `provider/model`. |
| `chat.system` | puudub | Süsteemne viip |
| `chat.max_tokens`, `chat.temperature` | puudub | Määramata — jäetakse pakkujale |
| `chat.stream` | `true` | Valikulise kohtuniku vastuse voogimine; tööriistadega agendi sammud kasutavad praegu terviklikke vastuseid, et tööriistade kutsed jääksid terviks |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Mõtisklused ja tokenite arv stderr'il |
| `agent.max_turns` | `20` | Maksimaalsed tööriistaringed ühe kasutaja päringu kohta |
| `agent.persist_sessions` | `true` | Kohalike vestluste salvestamine käikude vahel |
| `permissions.shell` | `ask` | Shelli käskude käivitamine: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Failide lugemine: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Failide muutmine: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-de laadimine: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Lubatud juured failitööriistadele |
| `permissions.denied_paths` | puudub | Selgelt blokeeritud failiteed |
| `network.connect_timeout_secs` | `15` | Ühendamiseks lubatud aeg |
| `network.idle_timeout_secs` | `300` | Lubatud vaikus andmete ootel |
| `network.proxy` | `system` | `system`, `none` või proksi aadress |
| `network.ca_bundle` | puudub | Lisafail usaldusväärsete sertifitseerimisasutustega |
| `network.allow_insecure_http` | `false` | Lubada võti lihtsa `http` kaudu võõrale hostile |
| `update.check` | `true` | Küsida GitHub'ilt ja GitLab'ilt uuemat versiooni interaktiivse sessiooni alguses |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, kust kontroll küsib kõige uuemat väljalaset |
| `update.gitlab_repo` | puudub | GitLab `owner/repo` sama kontrolli jaoks. Tühjaks jättes on GitLab väljas |
| `update.timeout_secs` | `3` | Mitu sekundit kontroll tohib ühele allikale kulutada |

Marelle kannab endaga kaasas usaldusväärsete sertifitseerimisasutuste oma loendit, nii et see töötab ka seal, kus süsteemil neid pole, nagu Termuxis. `SSL_CERT_FILE` lisab veel.

Kui leitakse uuem versioon, mainib sessioon seda korra — versiooni, seda, mis teil käes, ja kust seda võtta. Küsitakse kõigepealt väljalaset; repositoorium, kes oma sissesid vaid sildistab, annab kõige uuema sildi. Mis iganes nurjub, jääb vaikselt.

## Teie võtmed jäävad teie omaks

- Seaded nimetavad muutuja või faili, kus võti elab, mitte võtit ennast. Selline seade nagu `providers.x.api_key` lükatakse tagasi.
- Võtit ei trüki `config show`, `config explain` ega `providers`, ja ta lõigatakse välja igast serverilt tulnud veatekstist.
- Ta saadetakse ainult pakkuja aadressile, ühes päises, ja mitte kunagi lihtsa `http` kaudu teise masinasse, kui te seda ei luba.
- Ümbersuunamisi ei järgita, nii et võti ei saa mujale saata.

## Keeled

Liides räägib inglise, vene, prantsuse ja ukraina keelt. Valige `--lang <code>` või `MARELLE_LANG` kaudu; muidu otsustavad `LC_ALL`, `LC_MESSAGES` ja `LANG`. Iga teade elab kataloogis, ja test kukub, kui mõni keel ühe maha jätab. Eesmärk on iga Eurasia keel ja siis rohkem.

## Teekond

1. **Rääkida.** OpenAI-ga ühilduvad ja Anthropic'i adapterid, nii SSE-voolavusega kui terviklike vastuste lugemisega. Tööriistadega agendi sammud kasutavad praegu terviklikke vastuseid. *Valmis.*
2. **Seaded ja ilme.** TOML seadefailid, globaalsed ja projekti kaupa, koos `config set`, `get` ja `unset` ning igae kötvi kirjeldusega. Teemaline liides [Hermes Agent](https://github.com/NousResearch/hermes-agent) vaimus: bänner, teaberiba, vastused raamis, keerleja, vaikimisi jerevannas, iga värv HEX-i kujul seadistatav. *Valmis.*
3. **Palju mudeleid.** Mudelite varjunimed, mitme mudeli üheaegne küsimine ja vastuste võrdlemine, varuketjad ja kohtunik-mudel, kes need kokku sulab. *Valmis.*
4. **Käituda.** Struktureeritud tööriistakutsed OpenAI-ga ühilduvatele ja Anthropic'i pakkujatele, kohalikud faili-, otsingu- ja terminalitööriistad, URL'i laadimine, kohalikud oskused ja õigused allow/ask/deny; transkript näitab iga tööriistakutset ja selle tulemust. *Valmis.*
5. **Meeles pidada.** Nimetatud vestlused jäävad käikude vahel kohalikult püsima. Pikaajaline semantiline mälu on veel ees.
6. **Laiendada.** MCP-serverid, välised lisad, alamagendid ja rikkam oskuste tugi.
7. **Kätta.** Brauseri automatiseerimine, sõnumiväravate ja ajastatud ülesanded.

Disaini taga olev mõtlemine on [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-is.

## Teadaolevad piirid

- Pakkuja- ja mudelivalijad loetlevad seadistatud pakkujaid ja mudelite varjunimesid. Marelle ei avasta veel eemalseisvaid mudelkatalooge.
- Valijad toetavad hiireklõpsu fookuseks ja kerimist rattaga; transkript ja sisestaja jäävad klaviatuuril juhitavaks.
- Ctrl-C lahkub interaktiivsest sessioonist. Ta ei katkesta veel üksnes praegust vastust.
- Agendi tsükkel, õigustest teadlikud kohalikud tööriistad ja püsivad kohalikud sessioonid on rakendatud, kuid Marelle jääb eel-alfa prototüübiks; praeguse kujunduse ja piiride kohta lugege [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Arendus

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testid ei puutu võrku kunagi: nad räägivad võltsserveriga `crates/marelle-testkit`-is. GitHub Actions ehitab ka Androidi binaarid ja kontrollib deklareeritud minimaalset Rust'i versiooni. GitLab'i peegli jaoks on GitLab CI rada; vaadake [docs/GITLAB.md](docs/GITLAB.md).

## Litsents

[AGPL-3.0-only](LICENSE). Kolmandate osapoolte teated on [NOTICE](NOTICE)-is. Väljalasete arhiivid hõlmavad `THIRD-PARTY-LICENSES.txt` binaaris olevate Rust'ikastide kohta.
