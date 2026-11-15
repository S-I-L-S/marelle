```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**En konfigurerbar AI-agent til din terminal.**
Medbring dine egne API-nøgler: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, en lokal Ollama, eller enhver OpenAI-kompatibel server. Først Linux og Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · Dansk · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, version 0.1.0-alpha.1.** Implementeret: indfødt TUI-grænseflade, chat med flere udbydere, en tilladelsesbevidst agentløkke, lokale værktøjer og genoptagelige lokale sessioner. API'er og indstillinger kan stadig ændre sig. Dette er en prototype, ikke funktionsparitet med OpenCode eller Hermes.

*Marelle* er det franske navn for hinkespillet. Hver firkant er et skridt.

## Installation

**Termux på Android.** Hent en færdigbygget binærfil fra [udgivelsessiden](https://github.com/S-I-L-S/marelle/releases), eller byg den på telefonen. Trin for trin-guide: [docs/TERMUX.md](docs/TERMUX.md).

**Linux og hvor Rust ellers kører.** Rust 1.85 eller nyere:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Eller klon depotet og kør `cargo build --release`. Binærfilen er `target/release/marelle`, omkring 4 MiB.

## Første chat

Opbevar nøglen i en fil, som kun du læser, og sig til Marelle, hvor den er. Marelle gemmer aldrig en nøgle i sine indstillinger.

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

Erstat `MODEL` med et modelnavn fra din udbyders liste. Andre måder at bruge det på:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Svaret går til stdout. Ræsonnementer, tællinger af tokens, bemærkninger og fejl går til stderr, så `marelle chat ... > answer.txt` gemmer kun svaret.

## Agentens værktøjer

Marelle sender strukturerede værktøjsdefinitioner gennem OpenAI-kompatible og Anthropic-API'er. Det indbyggede værktøjssæt:

| Værktøj | Formål |
| --- | --- |
| `terminal` | Køre en shell-kommando fra startmappen |
| `read_file`, `write_file`, `edit_file` | Læse og ændre UTF-8-tekstfiler |
| `list_directory`, `search_text` | Se og søge i projektets filer |
| `fetch_url` | Hente en offentlig HTTP(S)-side som tekst (kræver `curl`) |
| `list_skills`, `load_skill` | Finde og indlæse lokale `SKILL.md`-instruktioner |

Kommandoen `/tools` viser værktøjslisten og `/skills` de tilgængelige færdigheder. Projektvejledning indlæses fra `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` og udvalgte `.marelle/`-filer, når de ligger i tilladte stier.

Filværktøjerne holdes inden for `permissions.allowed_paths` (standard: den nuværende projektmappe) og respekterer `permissions.denied_paths`. Læsning er tilladt som standard; filændringer, shell-kommandoer og netværkshentninger spørger først. Skaltilladelsen gælder kommandoer på værten og er ikke en filsystem-sandkasse: hold den på `ask`, medmindre du bevidst stoler på modellen til kommandoudørelse.

Agenten standser efter `agent.max_turns` værktøjsgange. Værktøjssvar behandles som utroværdigt indhold. Anmodninger bruger fulde modelsvar, så værktøjsargumenter ikke sammensættes af ufuldstændige strømbrudstykker.

## Sessioner

Samtaler gemmes lokalt som standard, inklusive værktøjskaldshistorik og model- og sessionsindstillinger. På Unix har sessionsmappen tilstand `0700`, og JSON-filen tilstand `0600`. Filen gemmes i `$XDG_STATE_HOME/marelle/sessions.json`, eller `~/.local/state/marelle/sessions.json`, når `XDG_STATE_HOME` ikke er sat. API-nøgler skrives aldrig til denne fil. Slå gemmen fra med:

```sh
marelle config set agent.persist_sessions false
```

Udgangskoder: `0` succes, `1` udbyder- eller netværksfejl, `2` brugs- eller konfigurationsfejl.

## Udbydere

| id | Protokol | Nøglevariabel | |
| --- | --- | --- | --- |
| `zai` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai (Zhipu GLM), international |
| `zai_coding` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-kompatibelt endepunkt |
| `zhipu` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kina |
| `zhipu_coding` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-kompatibel | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-kompatibel | `OPENROUTER_API_KEY` | mange modeller bag én nøgle |
| `ollama` | OpenAI-kompatibel | ingen | Ollama på denne maskine |

Endepunkterne for Z.ai og Zhipu kommer fra tredjepartsdokumentation, ikke fra udbyderens egne sider. Sammenlign dem med din plan og overskriv hvert felt, hvis de afviger.

Hvert udbyderfelt kan ændres, og en ny udbyder er bare et navn med en adresse:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Andre udbyderfelter: `kind` (`openai-compatible` eller `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` og `extra_body`. Hvis variablen, der nævnes af `api_key_env`, er sat, vinder den; ellers læses filen.

## Indstillinger

Indstillinger kommer i lag: indbyggede standardværdier, derefter miljøvariabler, derefter flag. Indstillingsfiler skubber sig mellem standardværdierne og miljøet.

- `MARELLE_CHAT__MODEL=zai/MODEL` sætter `chat.model`. Et dobbelt understregning står for et punktum.
- `marelle config show [prefix]` lister indstillinger og laget, hvorfra hver kom.
- `marelle config explain <key>` viser hvert lag, der satte en nøgle, fra laveste til højeste prioritet.

| Nøgle | Standard | Betydning |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ingen | Med hvem at tale. `chat.model` kan være `provider/model`. |
| `chat.system` | ingen | Systemprompt |
| `chat.max_tokens`, `chat.temperature` | ingen | Usat — overlades til udbyderen |
| `chat.stream` | `true` | Strømning af dommerens valgfrie svar; agentgange med værktøjer bruger for tiden fulde svar, så værktøjskald forbliver hele |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Ræsonnementer og tokental på stderr |
| `agent.max_turns` | `20` | Højst værktøjsgange pr. brugeranmodning |
| `agent.persist_sessions` | `true` | Gemme lokale samtaler mellem kørsler |
| `permissions.shell` | `ask` | Udførelse af shell-kommandoer: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Fillæsning: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Filændring: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-hentning: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Tilladte rødder til filværktøjer |
| `permissions.denied_paths` | ingen | Udtrykkeligt blokerede filstier |
| `network.connect_timeout_secs` | `15` | Tid tilladt at forbinde |
| `network.idle_timeout_secs` | `300` | Tilladt stilhed, mens der ventes på data |
| `network.proxy` | `system` | `system`, `none`, eller en proxyadresse |
| `network.ca_bundle` | ingen | En ekstra fil med betroede certifikatautoriteter |
| `network.allow_insecure_http` | `false` | Tillade en nøgle over simpel `http` til en fremmed vært |
| `update.check` | `true` | Spørge GitHub og GitLab om en nyere version, når en interaktiv session begynder |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, som kontrollen spørger om den nyeste udgivelse |
| `update.gitlab_repo` | ingen | GitLab `owner/repo` til samme kontrol. Tomt slukker GitLab |
| `update.timeout_secs` | `3` | Hvor mange sekunder kontrollen må bruge på én kilde |

Marelle bærer en egen liste over betroede certifikatautoriteter, så den virker også dér, hvor systemet ikke har nogen, som i Termux. `SSL_CERT_FILE` tilføjer flere.

Når en nyere version findes, nævner sessionen den én gang — versionen, den du har, og hvor den tages. En udgivelse spørges først; et depot, der blot tagger sine commits, giver den nyeste tag. Hvad end det fejer, forbliver stille.

## Dine nøgler forbliver dine

- Indstillinger nævner en variabel eller en fil, hvor nøglen ligger, aldrig selve nøglen. En indstilling som `providers.x.api_key` afvises.
- Nøglen udskrives ikke af `config show`, `config explain` eller `providers`, og den skæres ud af hver fejltekst, der kommer fra en server.
- Den sendes kun til udbyderens adresse, i én overskrift, og aldrig over simpel `http` til en anden maskine, medmindre du tillader det.
- Omdirigeringer følges ikke, så en nøgle kan ikke sendes videre et andet sted hen.

## Sprog

Grænsefladen taler engelsk, russisk, fransk og ukrainsk. Vælg med `--lang <code>` eller `MARELLE_LANG`; ellers afgør `LC_ALL`, `LC_MESSAGES` og `LANG`. Hver meddelelse lever i en katalog, og en test falder, hvis et sprog udelader én. Målet er hvert eurasiske sprog og derefter mere.

## Køreplan

1. **Tale.** OpenAI-kompatible og Anthropic-adaptere, med både SSE-strømning og fuldsvarsafkodning. Agentgange med værktøjer bruger for tiden fulde svar. *Færdig.*
2. **Indstillinger og udseende.** TOML-indstillingsfiler, globale og pr. projekt, med `config set`, `get` og `unset` og en beskrivelse for hver nøgle. Et tematiseret grænseflade i [Hermes Agent](https://github.com/NousResearch/hermes-agent) ånd: banner, informationspanel, indrammede svar, spinner, som standard i glyciniers lilla, med hver farve sat som HEX. *Færdig.*
3. **Mange modeller.** Modelaliaser, at spørge flere modeller på én gang og sammenligne svarene, faldtilstandskæder, og en dommermodel, der smelter dem sammen. *Færdig.*
4. **Handle.** Strukturerede værktøjskald til OpenAI-kompatible og Anthropic-udbydere, lokale fil-, søge- og terminalværktøjer, URL-hentning, lokale færdigheder og allow/ask/deny-tilladelser; protokollen viser hvert værktøjskald og dets resultat. *Færdig.*
5. **Huske.** Navngivne samtaler består lokalt mellem kørsler. Langtidssemantisk hukommelse ligger endnu forude.
6. **Udvide.** MCP-servere, ydre indstiksmoduler, underagenter og rigere færdighedsstøtte.
7. **Række.** Browserautomatisering, budbringereporte og planlagte opgaver.

Tænkningen bag formgivningen ligger i [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Kendte grænser

- Udbyder- og modelvælgere lister satte udbydere og modelaliaser. Marelle opdager endnu ikke fjerne modelkataloger.
- Vælgere understøtter museklik for fokus og rulning med hjulet; protokollen og indgangsredaktøren forbliver tastaturdrevne.
- Ctrl-C forlader den interaktive session. Den afbryder endnu ikke kun det igangværende svar.
- Agentløkken, tilladelsesbevidste lokale værktøjer og bestandige lokale sessioner er gennemført, men Marelle forbliver en pre-alfa-prototype; for den nuværende formgivning og grænser, læs [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Udvikling

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testene rører aldrig nettet: de taler med en skueserver i `crates/marelle-testkit`. GitHub Actions bygger også Android-binærfilerne og tjekker den erklærede minimale Rust-version. Til et GitLab-spejl findes et GitLab CI-rør; se [docs/GITLAB.md](docs/GITLAB.md).

## Licens

[AGPL-3.0-only](LICENSE). Tredjepartsnotitser ligger i [NOTICE](NOTICE). Udgivelsesarkiver inkluderer `THIRD-PARTY-LICENSES.txt` til Rust-kasserne inde i binærfilen.
