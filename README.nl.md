```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Een configureerbare AI-agent voor jouw terminal.**
Breng je eigen API-sleutels mee: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, een lokale Ollama of eender welke OpenAI-compatibele server. Eerst Linux en Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · Nederlands · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, versie 0.1.0-alpha.1.** Geïmplementeerd: inheemse TUI-interface, chatten met meerdere providers, een toestemmingsbewuste agents-lus, lokale tools en hervatbare lokale sessies. API's en instellingen kunnen nog wijzigen. Dit is een prototype, geen functiepariteit met OpenCode of Hermes.

*Marelle* is het Franse woord voor hinkelen. Elk hokje is een stap.

## Installatie

**Termux op Android.** Download een kant-en-klare binair van de [uitgavenpagina](https://github.com/S-I-L-S/marelle/releases), of bou hem op de telefoon. Stap voor stap: [docs/TERMUX.md](docs/TERMUX.md).

**Linux en overal waar Rust draait.** Rust 1.85 of nieuwer:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Of kloon de repository en draai `cargo build --release`. De binair is `target/release/marelle`, zo'n 4 MiB.

## Eerste chat

Bewaar de sleutel in een bestand dat alleen jij leest, en vertel Marelle waar hij ligt. Marelle slaat een sleutel nooit op in zijn instellingen.

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

Vervang `MODEL` door een modelnaam uit de lijst van je provider. Andere manieren van gebruik:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Het antwoord gaat naar stdout. Redeneringen, tokentellingen, notities en fouten gaan naar stderr, dus `marelle chat ... > answer.txt` slaat alleen het antwoord op.

## Agent-tools

Marelle stuurt gestructureerde tooldefinities via OpenAI-compatibele en Anthropic API's. De ingebouwde toolset:

| Tool | Doel |
| --- | --- |
| `terminal` | Een shellopdracht uitvoeren vanuit de startmap |
| `read_file`, `write_file`, `edit_file` | UTF-8-tekstbestanden lezen en wijzigen |
| `list_directory`, `search_text` | Projectbestanden bekijken en doorzoeken |
| `fetch_url` | Een publieke HTTP(S)-pagina ophalen als tekst (vereist `curl`) |
| `list_skills`, `load_skill` | Lokale `SKILL.md`-instructies vinden en laden |

Met `/tools` zie je de toollist en `/skills` de beschikbare vaardigheden. Projectbegeleiding wordt geladen uit `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` en gekozen `.marelle/`-bestanden, mits ze binnen toegestane paden liggen.

Bestandstools zijn beperkt tot `permissions.allowed_paths` (standaard: de huidige projectmap) en respecteren `permissions.denied_paths`. Lezen is standaard toegestaan; bestandswijzigingen, shell-opdrachten en netwerkopvragingen vragen eerst. De shelltoestemming geldt voor opdrachten op de gastheer en is geen bestandssysteem-zandbak: houd hem op `ask`, tenzij je de model bewust vertrouwt met het uitvoeren van opdrachten.

De agent stopt na `agent.max_turns` toolrondes. Toolantwoorden worden als onbetrouwbaar weergegeven. Aanvragen gebruiken volledige modelantwoorden, zodat toolargumenten niet uit onvolledige stroomfragmenten worden samengesteld.

## Sessies

Gesprekken worden standaard lokaal opgeslagen, inclusief tooloproepgeschiedenis en model- en sessie-instellingen. Op Unix heeft de sessiemap modus `0700` en het JSON-bestand modus `0600`. Het bestand staat in `$XDG_STATE_HOME/marelle/sessions.json`, of `~/.local/state/marelle/sessions.json` als `XDG_STATE_HOME` niet is gezet. API-sleutels worden nooit in dit bestand geschreven. Zet het bewaren uit met:

```sh
marelle config set agent.persist_sessions false
```

Exitcodes: `0` succes, `1` provider- of netwerkfout, `2` gebruiks- of instellingsfout.

## Providers

| id | Protocol | Sleutelvariabele | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatibel | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internationaal |
| `zai_coding` | OpenAI-compatibel | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-compatibel eindpunt |
| `zhipu` | OpenAI-compatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | OpenAI-compatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatibel | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatibel | `OPENROUTER_API_KEY` | veel modellen achter één sleutel |
| `ollama` | OpenAI-compatibel | geen | Ollama op deze machine |

De eindpunten van Z.ai en Zhipu komen uit derdedocumentatie, niet van de pagina's van de provider zelf. Controleer ze tegen je abonnement en herschrijf elk veld als ze afwijken.

Elk provider-veld is te veranderen, en een nieuwe provider is gewoon een naam met een adres:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Overige provider-velden: `kind` (`openai-compatible` of `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` en `extra_body`. Als de door `api_key_env` benoemde variabele is gezet, wint die; anders wordt het bestand gelezen.

## Instellingen

Instellingen komen in lagen: ingebouwde standaarden, dan omgevingsvariabelen, dan vlaggen. Instellingsbestanden schuiven tussen de standaarden en het milieu.

- `MARELLE_CHAT__MODEL=zai/MODEL` zet `chat.model`. Een dubbel onderstrepingsteken staat voor een punt.
- `marelle config show [prefix]` zet instellingen en de laag op een rij waaruit elk kwam.
- `marelle config explain <key>` toont elke laag die een sleutel zette, van laagste naar hoogste prioriteit.

| Sleutel | Standaard | Betekenis |
| --- | --- | --- |
| `chat.provider`, `chat.model` | geen | Met wie praten. `chat.model` kan `provider/model` zijn. |
| `chat.system` | geen | Systeemprompt |
| `chat.max_tokens`, `chat.temperature` | geen | Niet gezet — aan de provider overgelaten |
| `chat.stream` | `true` | Streamen van het antwoord van de optionele rechter; agentbeurten met tools gebruiken thans volledige antwoorden, zodat tooloproepen heel blijven |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Redeneringen en tokentellingen op stderr |
| `agent.max_turns` | `20` | Maximum toolrondes per gebruikersaanvraag |
| `agent.persist_sessions` | `true` | Lokale gesprekken bewaren tussen starts |
| `permissions.shell` | `ask` | Uitvoeren van shellopdrachten: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Bestanden lezen: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Bestanden wijzigen: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL ophalen: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Toegestane wortels voor bestandstools |
| `permissions.denied_paths` | geen | Uitdrukkelijk geblokkeerde bestandspaden |
| `network.connect_timeout_secs` | `15` | Tijd toegestaan om te verbinden |
| `network.idle_timeout_secs` | `300` | Toegestane stilte bij wachten op data |
| `network.proxy` | `system` | `system`, `none`, of een proxyadres |
| `network.ca_bundle` | geen | Een extra bestand met vertrouwde certificeringsinstanties |
| `network.allow_insecure_http` | `false` | Een sleutel toelaten over kale `http` naar een vreemde host |
| `update.check` | `true` | GitHub en GitLab vragen om een nieuwere versie bij de start van een interactieve sessie |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` waaraan de check om de nieuwste uitgave vraagt |
| `update.gitlab_repo` | geen | GitLab `owner/repo` voor dezelfde check. Leeg schakelt GitLab uit |
| `update.timeout_secs` | `3` | Hoeveel seconden de check op één bron mag besteden |

Marelle draagt een eigen lijst vertrouwde certificeringsinstanties met zich mee, dus het werkt ook daar waar het systeem ze niet heeft, zoals in Termux. `SSL_CERT_FILE` voegt meer toe.

Wanneer een nieuwere versie gevonden wordt, vermeldt de sessie haar één keer — de versie, wat je hebt, en waar het te halen valt. Eerst wordt een uitgave gevraagd; een repo die zijn commits alleen tagt, geeft de nieuwste tag. Alles wat faalt blijft stil.

## Jouw sleutels blijven jouw

- Instellingen noemen een variabele of bestand waar de sleutel ligt, nooit de sleutel zelf. Een instelling als `providers.x.api_key` wordt geweigerd.
- De sleutel wordt niet gedrukt door `config show`, `config explain` of `providers`, en wordt uit elke fouttekst geknipt die van een server komt.
- Hij gaat alleen naar het adres van de provider, in één kop, en nooit over kale `http` naar een andere machine, tenzij je het toelaat.
- Omleidingen worden niet gevolgd, zodat een sleutel niet ergens anders kan belanden.

## Talen

De interface spreekt Engels, Russisch, Frans en Oekraïens. Kies met `--lang <code>` of `MARELLE_LANG`; anders beslissen `LC_ALL`, `LC_MESSAGES` en `LANG`. Elke boodschap leeft in een catalogus, en een test valt als een taal er één mist. Het doel is elke taal van Eurazië en dan meer.

## Routekaart

1. **Spreken.** OpenAI-compatibele en Anthropic-adapters, met zowel SSE-stromen als volledige-antwoorddecoding. Agentbeurten met tools gebruiken thans volledige antwoorden. *Klaar.*
2. **Instellingen en uiterlijk.** TOML-instellingsbestanden, globaal en per project, met `config set`, `get` en `unset` en een beschrijving voor elke sleutel. Een gethematiseerde interface in de geest van [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, informatiepaneel, omrande antwoorden, spinner, standaard in wisteria-paars, met elke kleur als HEX instelbaar. *Klaar.*
3. **Veel modellen.** Modelaliasnamen, meerdere modellen tegelijk bevragen en de antwoorden vergelijken, valketstelsels, en een rechtersmodel dat ze samensmelt. *Klaar.*
4. **Handelen.** Gestructureerde tooloproepen voor OpenAI-compatibele en Anthropic-providers, lokale bestand-, zoek- en terminaltools, URL ophalen, lokale vaardigheden en allow/ask/deny-toestemmingen; het transcript toont elke tooloproep en zijn resultaat. *Klaar.*
5. **Herinneren.** Genoemde gesprekken blijven lokaal bestaan tussen starts. Langetermijn-semantisch geheugen ligt nog voor ons.
6. **Uitbreiden.** MCP-servers, externe plug-ins, subagenten en rijkere vaardigheidsondersteuning.
7. **Reiken.** Browserautomatisering, boodschapperspoorten en geplande taken.

Het nadenken achter het ontwerp staat in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Bekende grenzen

- Provider- en modelkiezers tonen ingestelde providers en modelaliasnamen. Marelle ontdekt nog geen verre modelkatalogen.
- Kiezers ondersteunen muisklik voor focus en scrollen met het wieltje; het transcript en de invoereditor blijven toetsenbordgedreven.
- Ctrl-C verlaat de interactieve sessie. Het onderbreekt nog niet alleen het lopende antwoord.
- De agents-lus, toestemmingsbewuste lokale tools en blijvende lokale sessies zijn gerealiseerd, maar Marelle blijft een pre-alpha-prototype; voor het huidige ontwerp en de grenzen lees [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Ontwikkeling

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Tests raken het net nooit: ze spreken met een schijnserver in `crates/marelle-testkit`. GitHub Actions bouwt ook de Android-binaire bestanden en controleert de verklaarde minimale Rust-versie. Voor een GitLab-spiegel is er een GitLab CI-pijplijn; zie [docs/GITLAB.md](docs/GITLAB.md).

## Licentie

[AGPL-3.0-only](LICENSE). Mededelingen van derden staan in [NOTICE](NOTICE). Uitgavearchieven bevatten `THIRD-PARTY-LICENSES.txt` voor de Rust-kisten in de binair.
