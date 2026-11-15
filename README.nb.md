```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**En konfigurerbar KI-agent for terminalen din.**
Ta med dine egne API-nøkler: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, en lokal Ollama, eller en hvilken som helst OpenAI-kompatibel server. Først Linux og Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · Norsk bokmål · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, versjon 0.1.0-alpha.1.** Implementert: innfødt TUI-grensesnitt, prat med flere leverandører, en tillitsmedveten agentløkke, lokale verktøy og gjenopptakbare lokale økter. API-er og innstillinger kan fortsatt endres. Dette er en prototype, ikke funksjonsparitet med OpenCode eller Hermes.

*Marelle* er det franske navnet på et hinkespill. Hver rute er et steg.

## Installasjon

**Termux på Android.** Last ned en ferdigbygd binærfil fra [utgivelsessiden](https://github.com/S-I-L-S/marelle/releases), eller bygg den på telefonen. Trinn for trinn-veiledning: [docs/TERMUX.md](docs/TERMUX.md).

**Linux og hvor Rust ellers går.** Rust 1.85 eller nyere:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Eller klon depotet og kjør `cargo build --release`. Binærfilen er `target/release/marelle`, rundt 4 MiB.

## Første prat

Oppbevar nøkkelen i en fil som bare du leser, og si til Marelle hvor den er. Marelle lagrer aldri en nøkkel i sine innstillinger.

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

Erstatt `MODEL` med et modellnavn fra leverandørens liste. Andre måter å bruke det på:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Svaret går til stdout. Resonneringer, tokentellinger, merknader og feil går til stderr, så `marelle chat ... > answer.txt` lagrer bare svaret.

## Agentens verktøy

Marelle sender strukturerte verktøydefinisjoner gjennom OpenAI-kompatible og Anthropic API-er. Det innebygde verktøysettet:

| Verktøy | Hensikt |
| --- | --- |
| `terminal` | Kjøre en skalkommando fra startmappa |
| `read_file`, `write_file`, `edit_file` | Lese og endre UTF-8-tekstfiler |
| `list_directory`, `search_text` | Se gjennom og søke i prosjektets filer |
| `fetch_url` | Hente en offentlig HTTP(S)-side som tekst (krever `curl`) |
| `list_skills`, `load_skill` | Finne og laste lokale `SKILL.md`-instruksjoner |

Kommandoen `/tools` viser verktøylista og `/skills` de tilgjengelige ferdighetene. Prosjektveiledning lastes fra `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` og utvalgte `.marelle/`-filer når de ligger innen tillatte stier.

Fil-verktøyene holdes innenfor `permissions.allowed_paths` (misligholde: den nåværende prosjektmappe) og respekterer `permissions.denied_paths`. Lesning er tillatt som misligholde; filendringer, skalkommandoer og nettverksnedlastinger spør først. Skaltillatelsen gjelder kommandoer på verten og er ikke en filsystem-sandkasse: hold den på `ask`, med mindre du bevisst stoler på modellen for kommandokjøring.

Agenten stopper etter `agent.max_turns` verktøyrunder. Verktøysvar behandles som uvederig innhold. Forespørsler bruker fullstendige modelsvar, slik at verktøyargumenter ikke settes sammen av ufullstendige strømbiter.

## Økter

Samtaler lagres lokalt som misligholde, inkludert verktøykallshistorikk og modell- og øktinnstillinger. På Unix har øktmapen modus `0700` og JSON-fila modus `0600`. Fila lagres i `$XDG_STATE_HOME/marelle/sessions.json`, eller `~/.local/state/marelle/sessions.json` når `XDG_STATE_HOME` ikke er satt. API-nøkler skrives aldri til denne fila. Slå av lagringen med:

```sh
marelle config set agent.persist_sessions false
```

Utgangskoder: `0` suksess, `1` leverandør- eller nettverksfeil, `2` bruks- eller oppsettsfeil.

## Leverandører

| id | Protokoll | Nøkkelvariabel | |
| --- | --- | --- | --- |
| `zai` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internasjonal |
| `zai_coding` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, endepunkt kompatibel med Anthropic |
| `zhipu` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kina |
| `zhipu_coding` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-kompatibel | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-kompatibel | `OPENROUTER_API_KEY` | mange modeller bak én nøkkel |
| `ollama` | OpenAI-kompatibel | ingen | Ollama på denne maskinen |

Endepunktene for Z.ai og Zhipu kommer fra tredjepartsdokumentasjon, ikke fra leverandørens egne sider. Sammenlign dem med din plan og skriv over hvert felt om de skiller seg.

Hvert leverandørfelt kan endres, og en ny leverandør er bare et navn med en adresse:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Andre leverandørfelter: `kind` (`openai-compatible` eller `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` og `extra_body`. Hvis variabelen, som nevnes av `api_key_env`, er satt, vinner den; ellers leses fila.

## Innstillinger

Innstillinger kommer i lag: innebygde misligholdsværdier, deretter miljøvariabler, deretter flagg. Innstillingsfiler legger seg mellom misligholdene og miljøet.

- `MARELLE_CHAT__MODEL=zai/MODEL` setter `chat.model`. Et dobbelt understrek står for et punktum.
- `marelle config show [prefix]` lister innstillinger og laget hver kom fra.
- `marelle config explain <key>` viser hvert lag som satte en nøkkel, fra laveste til høyeste prioritet.

| Nøkkel | Misligholde | Betydning |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ingen | Med hvem å snakke. `chat.model` kan være `provider/model`. |
| `chat.system` | ingen | Systemprompt |
| `chat.max_tokens`, `chat.temperature` | ingen | Ikke satt — overlatt til leverandøren |
| `chat.stream` | `true` | Strømning av dommerens valgfrie svar; agentrunder med verktøy bruker nå fulle svar, slik at verktøykall forblir hele |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Resonneringer og tokentall på stderr |
| `agent.max_turns` | `20` | Høyest verktøyrunder per brukerforespørsel |
| `agent.persist_sessions` | `true` | Lagre lokale samtaler mellom kjøringer |
| `permissions.shell` | `ask` | Kjøring av skalkommandoer: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Fillæsning: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Filendring: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-nedlasting: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Tillatte røtter for filverktøy |
| `permissions.denied_paths` | ingen | Uttrykkelig blokkerte filstier |
| `network.connect_timeout_secs` | `15` | Tid tillatt å koble til |
| `network.idle_timeout_secs` | `300` | Tillatt stillhet mens man venter på data |
| `network.proxy` | `system` | `system`, `none`, eller en proxy-adresse |
| `network.ca_bundle` | ingen | En tilleggsfil med tilitsverdige sertifikatsmyndigheter |
| `network.allow_insecure_http` | `false` | Tillate en nøkkel over vanlig `http` til en fremmed vert |
| `update.check` | `true` | Spørre GitHub og GitLab om en nyere versjon når en interaktiv økt begynner |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` som kontrollen spør om nyeste utgivelse |
| `update.gitlab_repo` | ingen | GitLab `owner/repo` for samme kontroll. Tom slår av GitLab |
| `update.timeout_secs` | `3` | Hvor mange sekunder kontrollen kan bruke på én kilde |

Marelle bærer en egen liste over tilitsverdige sertifikatsmyndigheter, så den virker også der systemet ikke har noen, som i Termux. `SSL_CERT_FILE` legger til flere.

Når en nyere versjon finnes, nevner økten den én gang — versjonen, den du har, og hvor den hentes. En utgivelse spørres først; et depot som bare merker sine commits med tags, gir den nyeste taggen. Hva enn svikter, forblir stille.

## Nøklene dine forblir dine

- Innstillinger navngir en variabel eller en fil der nøkkelen ligger, aldri selve nøkkelen. En innstilling som `providers.x.api_key` avslås.
- Nøkkelen skrives ikke ut av `config show`, `config explain` eller `providers`, og den klippes ut av hver feiltekst som kommer fra en server.
- Den sendes bare til leverandørens adresse, i ett hodefelt, og aldri over vanlig `http` til en annen maskin, med mindre du tillater det.
- Omdirigeringer følges ikke, slik at en nøkkel ikke kan sendes videre et annet sted.

## Språk

Grensesnet taler engelsk, russisk, fransk og ukrainsk. Velg med `--lang <code>` eller `MARELLE_LANG`; ellers bestemmer `LC_ALL`, `LC_MESSAGES` og `LANG`. Hver melding lever i en katalog, og en test faller om et språk utelater én. Målet er hvert eurasiske språk og så mer.

## Veikart

1. **Tale.** OpenAI-kompatible og Anthropic-adaptere, med både SSE-strømning og fullsvarsdekoding. Agentrunder med verktøy bruker nå fulle svar. *Ferdig.*
2. **Innstillinger og utseende.** TOML-innstillingsfiler, globale og per prosjekt, med `config set`, `get` og `unset` og en beskrivelse for hver nøkkel. Et tematisert grensesnett i [Hermes Agent](https://github.com/NousResearch/hermes-agent) ånd: bannere, informasjonspanel, innrammede svar, spinnere, som misligholde i glysinens lilla, med hver farge sett som HEX. *Ferdig.*
3. **Mange modeller.** Modellaliaser, å spørre flere modeller på én gang og sammenligne svarene, reserveløfter, og en dommermodell som smelter dem sammen. *Ferdig.*
4. **Handle.** Strukturerte verktøykall for OpenAI-kompatible og Anthropic-leverandører, lokale fil-, søke- og terminalverktøy, nedlasting av URL, lokale ferdigheter og allow/ask/deny-tilatelser; transkribeten viser hvert verktøykall og dets resultat. *Ferdig.*
5. **Huske.** Navngitte samtaler består lokalt mellom kjøringer. Langtidssemantisk hukommelse ligger ennå foran oss.
6. **Utvide.** MCP-servere, ytre innstikksmoduler, underagenter og rikere ferdighetsstøtte.
7. **Rekke.** Nettleserautomatisering, budbærerporter og planlagte oppgaver.

Tenkningen bak formgivningen ligger i [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Kjente grenser

- Leverandør- og modellvelgere lister satte leverandører og modellaliaser. Marelle oppdager ennå ikke fjerne modellkataloger.
- Velgere støtter museklikk for fokus og rulling med hjulet; transkriben og inngangsredaktøren forblir tastaturdrevne.
- Ctrl-C forlater den interaktive økten. Den avbryter ennå ikke bare det løpende svaret.
- Agentløkka, tilitsmedvetne lokale verktøy og varige lokale økter er gjennomført, men Marelle forblir en pre-alfa-prototype; for den nåværende formgivningen og grensene, les [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Utvikling

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Prøvene berører aldri nettet: de taler med en skueserver i `crates/marelle-testkit`. GitHub Actions bygger også Android-binærfilene og sjekker den erklærte minste Rust-versjonen. For et GitLab-speil finnes et GitLab CI-rør; se [docs/GITLAB.md](docs/GITLAB.md).

## Lisens

[AGPL-3.0-only](LICENSE). Tredjepartsnotiser ligger i [NOTICE](NOTICE). Utgivelsesarkiver inkluderer `THIRD-PARTY-LICENSES.txt` for Rust-kassene inne i binærfilen.
