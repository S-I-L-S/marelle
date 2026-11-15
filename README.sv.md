```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**En konfigurerbar AI-agent för din terminal.**
Ta med dina egna API-nycklar: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, en lokal Ollama eller vilken OpenAI-kompatibel server som helst. Först Linux och Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · Svenska
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, version 0.1.0-alpha.1.** Implementerat: inhemkt TUI-gränssnitt, chatt med flera leverantörer, en behörighetsmedveten agentslinga, lokala verktyg och återupptagbara lokala sessioner. API:er och inställningar kan fortfarande ändras. Detta är en prototyp, inte funktionsparitet med OpenCode eller Hermes.

*Marelle* är det franska namnet på hage. Varje ruta är ett steg.

## Installation

**Termux på Android.** Ladda ner en färdig binärfil från [utgåvorna](https://github.com/S-I-L-S/marelle/releases), eller bygg den på telefonen. Steg för steg-guide: [docs/TERMUX.md](docs/TERMUX.md).

**Linux och vart annat Rust kör.** Rust 1.85 eller nyare krävs:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Eller klona repot och kör `cargo build --release`. Binären är `target/release/marelle`, omkring 4 MiB.

## Första chatten

Förvara nyckeln i en fil som bara du läser, och säg till Marelle var den är. Marelle lagrar aldrig en nyckel i sina inställningar.

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

Byt ut `MODEL` mot ett modellnamn från din leverantörs lista. Andra sätt att använda det:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Svaret går till stdout. Resonerande, tokenantal, anteckningar och fel går till stderr, så `marelle chat ... > answer.txt` sparar bara svaret.

## Agentens verktyg

Marelle skickar strukturerade verktygsdefinitioner via OpenAI-kompatibla och Anthropic API:er. Det inbyggda verktygssetet:

| Verktyg | Syfte |
| --- | --- |
| `terminal` | Köra ett skalkommando från startkatalogen |
| `read_file`, `write_file`, `edit_file` | Läsa och ändra UTF-8-textfiler |
| `list_directory`, `search_text` | Inspektera och söka i projektfiler |
| `fetch_url` | Hämta en offentlig HTTP(S)-sida som text (kräver `curl`) |
| `list_skills`, `load_skill` | Upptäcka och ladda lokala `SKILL.md`-instruktioner |

Kommandot `/tools` visar verktygslistan och `/skills` listar tillgängliga färdigheter. Projektvägledning laddas från `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` och valda `.marelle/`-filer när de ligger inom tillåtna vägar.

Filverktygen hålls inom `permissions.allowed_paths` (standard: den nuvarande projektkatalogen) och respekterar `permissions.denied_paths`. Läsning är tillåten som standard; filändringar, skalkommandon och nätverkshämtningar frågar först. Skalbehörigheten gäller kommandon på värden och är inte en filsystemsandlåda: håll den på `ask` om du inte medvetet litar på modellen för kommandokörning.

Agenten stannar efter `agent.max_turns` verktygomgångar. Verktygssvar behandlas som opålitligt innehåll. Anrop använder fullständiga modellsvar, så att verktygsargument inte sätts samman av ofullständiga strömbitar.

## Sessioner

Konversationer sparas lokalt som standard, inklusive verktygsanropens historia och modell- och sessioninställningar. På Unix har sessionskatalogen läge `0700` och JSON-filen läge `0600`. Filen lagras i `$XDG_STATE_HOME/marelle/sessions.json`, eller `~/.local/state/marelle/sessions.json` när `XDG_STATE_HOME` inte är satt. API-nycklar skrivs aldrig till denna fil. Stäng av beständighet med:

```sh
marelle config set agent.persist_sessions false
```

Utgångskoder: `0` framgång, `1` leverantörs- eller nätverksfel, `2` användnings- eller konfigurationsfel.

## Leverantörer

| id | Protokoll | Nyckelvariabel | |
| --- | --- | --- | --- |
| `zai` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internationell |
| `zai_coding` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-kompatibel slutpunkt |
| `zhipu` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kina |
| `zhipu_coding` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-kompatibel | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-kompatibel | `OPENROUTER_API_KEY` | många modeller bakom en nyckel |
| `ollama` | OpenAI-kompatibel | ingen | Ollama på denna maskin |

Slutpunkterna för Z.ai och Zhipu kommer från tredjepartsdokumentation, inte från leverantörens egna sidor. Kontrollera dem mot din plan och skriv över vilket fält som helst om de skiljer sig.

Varje leverantörsfält kan ändras, och en ny leverantör är bara ett namn med en adress:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Övriga leverantörsfält: `kind` (`openai-compatible` eller `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` och `extra_body`. Om variabeln namngiven av `api_key_env` är satt vinner den; annars läses filen.

## Inställningar

Inställningar kommer i lager: inbyggda standardvärden, sedan miljövariabler, sedan flaggor. Inställningsfiler lägger sig mellan standardvärdena och miljön.

- `MARELLE_CHAT__MODEL=zai/MODEL` sätter `chat.model`. Ett dubbelt understreck står för en punkt.
- `marelle config show [prefix]` listar inställningar och lagret varifrån varje kom.
- `marelle config explain <key>` visar varje lager som satte en nyckel, från lägst till högst prioritet.

| Nyckel | Standard | Betydelse |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ingen | Med vem att tala. `chat.model` kan vara `provider/model`. |
| `chat.system` | ingen | Systemprompt |
| `chat.max_tokens`, `chat.temperature` | ingen | Ossatt — lämnas till leverantören |
| `chat.stream` | `true` | Strömning av domarens valfria svar; agentturer med verktyg använder nuförtiden fullständiga svar, så att verktygsanrop förblir intakta |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Resonerande och tokenantal på stderr |
| `agent.max_turns` | `20` | Högst verktygomgångar per användaranrop |
| `agent.persist_sessions` | `true` | Spara lokala konversationer mellan körningar |
| `permissions.shell` | `ask` | Körning av skalkommandon: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Filläsning: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Filändring: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-hämtning: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Tillåtna rötter för filverktyg |
| `permissions.denied_paths` | ingen | Uttryckligen blockerade filsökvägar |
| `network.connect_timeout_secs` | `15` | Tid tillåten att ansluta |
| `network.idle_timeout_secs` | `300` | Tillåten tystnad i väntan på data |
| `network.proxy` | `system` | `system`, `none`, eller en proxyadress |
| `network.ca_bundle` | ingen | En extra fil med betrodda certifikatutfärdare |
| `network.allow_insecure_http` | `false` | Tillåta en nyckel över vanlig `http` till en främmande värd |
| `update.check` | `true` | Fråga GitHub och GitLab om en nyare version när en interaktiv session startar |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` som kontrollen frågar om den nyaste utgåvan |
| `update.gitlab_repo` | ingen | GitLab `owner/repo` för samma kontroll. Tom stänger av GitLab |
| `update.timeout_secs` | `3` | Hur många sekunder kontrollen får lägga på en källa |

Marelle bär på en egen lista över betrodda certifikatutfärdare, så det fungerar även där systemet saknar dem, som i Termux. `SSL_CERT_FILE` lägger till fler.

När en nyare version hittas nämner sessionen den en gång — versionen, den du har, och var den fås. En utgåva frågas först; ett repo som bara taggar sina commits ger den nyaste taggen. Vad som helst som faller förblir tyst.

## Dina nycklar förblir dina

- Inställningar namnger en variabel eller en fil där nyckeln vilar, aldrig nyckeln själv. En inställning som `providers.x.api_key` avslås.
- Nyckeln skrivs inte ut av `config show`, `config explain` eller `providers`, och den klipps ur varje feltext som kommer från en server.
- Den skickas bara till leverantörens adress, i ett rubrikfält, och aldrig över vanlig `http` till annan maskin, om du inte tillåter det.
- Omdirigeringar följs inte, så en nyckel kan inte vidarebefordras någon annanstans.

## Språk

Gränssnittet talar engelska, ryska, franska och ukrainska. Välj med `--lang <code>` eller `MARELLE_LANG`; annars avgör `LC_ALL`, `LC_MESSAGES` och `LANG`. Varje meddelande lever i en katalog, och ett test faller om något språk utelämnar ett. Målet är vartjeurasiskt språk och sedan mera.

## Färdplan

1. **Tala.** OpenAI-kompatibla och Anthropic-adapterare, med både SSE-strömning och fullsvarsavkodning. Agentturer med verktyg använder nuförtiden fullständiga svar. *Klar.*
2. **Inställningar och utseende.** TOML-inställningsfiler, globala och per projekt, med `config set`, `get` och `unset` och en beskrivning för varje nyckel. Ett tematiserat gränssnitt i [Hermes Agent](https://github.com/NousResearch/hermes-agent) anda: banner, informationspanel, rammade svar, spinner, som standard i glicinias lila, med vilken färg som helst som HEX. *Klar.*
3. **Många modeller.** Modellalias, att fråga flera modeller på en gång och jämföra svaren, reservkedjor, och en domarmodell som fogar samman dem. *Klar.*
4. **Handla.** Strukturerade verktygsanrop för OpenAI-kompatibla och Anthropic-leverantörer, lokala fil-, sök- och terminalverktyg, URL-hämtning, lokala färdigheter och allow/ask/deny-behörigheter; transkriptet visar vartje verktygsanrop och dess resultat. *Klar.*
5. **Minnas.** Namngivna konversationer består lokalt mellan körningar. Långtidssemantiskt minne ligger ännu framför.
6. **Utöka.** MCP-servrar, yttre instick, underagenter och rikare färdighetsstöd.
7. **Räcka.** Webbläsarautomatisering, meddelandegateways och schemalagda uppgifter.

Resonerandet bakom formgivningen finns i [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Kända gränser

- Leverantörs- och modellväljare listar inställda leverantörer och modellalias. Marelle upptäcker ännu inte fjärrmodellkataloger.
- Väljare stöder musklick för fokus och rullning med hjulet; transkriptet och inmatningsredigeraren förblir tangentbordsdrivna.
- Ctrl-C lämnar den interaktiva sessionen. Den avbryter ännu inte bara det aktuella svaret.
- Agentslingan, behörighetsmedvetna lokala verktyg och beständiga lokala sessioner är implementerade, men Marelle förblir en pre-alfa-prototyp; se [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) för nuvarande formgivning och gränser.

## Utveckling

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testerna rör aldrig nätet: de talar med en skenserver i `crates/marelle-testkit`. GitHub Actions bygger också Android-binärerna och kontrollerar den deklarerade minimi-Rust-versionen. För en GitLab-spegel finns ett GitLab CI-rör; se [docs/GITLAB.md](docs/GITLAB.md).

## Licens

[AGPL-3.0-only](LICENSE). Tredjepartsnotiser finns i [NOTICE](NOTICE). Utgåvars arkiv inkluderar `THIRD-PARTY-LICENSES.txt` för Rust-lådorna inuti binären.
