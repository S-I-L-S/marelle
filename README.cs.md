```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigurovatelný AI agent pro váš terminál.**
Přineste si vlastní API klíče: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokální Ollama nebo jakýkoli server kompatibilní s OpenAI. Především Linux a Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
Čeština · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, verze 0.1.0-alpha.1.** Implementováno: nativní TUI rozhraní, čet s více poskytovateli, agent smyčka vědomá oprávnění, lokální nástroje a obnovitelné lokální relace. API a nastavení se mohou ještě změnit. Toto je prototyp, nikoli parita funkcí s OpenCode či Hermes.

*Marelle* je francouzské pojmenování hry panák. Každé políčko je krok.

## Instalace

**Termux na Androidu.** Stáhněte hotový binární soubor ze stránky [vydání](https://github.com/S-I-L-S/marelle/releases), nebo si ho sestavte přímo na telefonu. Podrobný postup: [docs/TERMUX.md](docs/TERMUX.md).

**Linux a všude tam, kde běží Rust.** Potřebujete Rust 1.85 nebo novější:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Nebo klonujte repozitář a spusťte `cargo build --release`. Binárka je `target/release/marelle`, zhruba 4 MiB.

## První čet

Uchovávejte klíč v souboru, který čtete jen vy, a řekněte Marelle, kde je. Marelle si klíč nikdy neukládá do svých nastavení.

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

Nahraďte `MODEL` názvem modelu ze seznamu vašeho poskytovatele. Další způsoby použití:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Odpověď jde na stdout. Uvažování, počty tokenů, poznámky a chyby jdou na stderr, takže `marelle chat ... > answer.txt` uloží pouze odpověď.

## Nástroje agenta

Marelle posílá strukturované definice nástrojů přes API kompatibilní s OpenAI a přes Anthropic. Vestavěná sada nástrojů:

| Nástroj | Účel |
| --- | --- |
| `terminal` | Spustit příkaz shellu z adresáře spuštění |
| `read_file`, `write_file`, `edit_file` | Číst a upravovat textové soubory UTF-8 |
| `list_directory`, `search_text` | Prohlížet a prohledávat soubory projektu |
| `fetch_url` | Stáhnout veřejnou stránku HTTP(S) jako text (vyžaduje `curl`) |
| `list_skills`, `load_skill` | Objevovat a načítat lokální instrukce `SKILL.md` |

Příkaz `/tools` ukáže seznam nástrojů a `/skills` vypíše dostupné dovednosti. Vedení projektu se načítá z `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` a vybraných souborů `.marelle/`, pokud leží v povolených cestách.

Souborové nástroje jsou omezeny na `permissions.allowed_paths` (výchozí: aktuální adresář projektu) a respektují `permissions.denied_paths`. Čtení je ve výchozím stavu povoleno; změny souborů, příkazy shellu a stahování ze sítě se předem ptají. Oprávnění k shellu se týká příkazů na hostiteli a není to pískoviště systému souborů: nechávejte ho na `ask`, pokud záměrně nesvěříte modeli spouštění příkazů.

Agent se zastaví po `agent.max_turns` kol s nástroji. Odpovědi nástrojů jsou považovány za nedůvěryhodný obsah. Požadavky používají celé odpovědi modelu, takže argumenty nástrojů se neskládají z neúplných fragmentů proudu.

## Relace

Konverzace se ve výchozím stavu ukládají lokálně, včetně historie volání nástrojů a nastavení modelu a relace. Na Unixu má adresář relací režim `0700` a JSON soubor režim `0600`. Soubor je uložen v `$XDG_STATE_HOME/marelle/sessions.json`, případně `~/.local/state/marelle/sessions.json`, když `XDG_STATE_HOME` není nastaveno. API klíče se do tohoto souboru nikdy nezapisují. Trvalost vypněte příkazem:

```sh
marelle config set agent.persist_sessions false
```

Návratové kódy: `0` úspěch, `1` chyba poskytovatele nebo sítě, `2` chyba použití či nastavení.

## Poskytovatelé

| id | Protokol | Proměnná klíče | |
| --- | --- | --- | --- |
| `zai` | kompatibilní s OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), mezinárodní |
| `zai_coding` | kompatibilní s OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, koncový bod kompatibilní s Anthropic |
| `zhipu` | kompatibilní s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Čína |
| `zhipu_coding` | kompatibilní s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Čína |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | kompatibilní s OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | kompatibilní s OpenAI | `OPENROUTER_API_KEY` | mnoho modelů za jeden klíč |
| `ollama` | kompatibilní s OpenAI | žádný | Ollama na tomto stroji |

Koncové body Z.ai a Zhipu pocházejí z dokumentace třetích stran, nikoli ze stránek samotného dodavatele. Prověřte je vůči svému plánu a přepište jakékoli pole, pokud se liší.

Každé pole poskytovatele lze změnit a nový poskytovatel je prostě název s adresou:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Ostatní pole poskytovatele: `kind` (`openai-compatible` nebo `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` a `extra_body`. Pokud je nastavena proměnná pojmenovaná v `api_key_env`, vítězí; jinak se čte soubor.

## Nastavení

Nastavení přicházejí ve vrstvách: vestavěné výchozí hodnoty, potom proměnné prostředí, potom přepínače. Soubory nastavení se vkládají mezi výchozí hodnoty a prostředí.

- `MARELLE_CHAT__MODEL=zai/MODEL` nastavuje `chat.model`. Dvojité podtržítko zastupuje tečku.
- `marelle config show [prefix]` vypisuje nastavení a vrstvu, ze které každé přišlo.
- `marelle config explain <key>` ukazuje každou vrstvu, která nastavila klíč, od nejnižší priority k nejvyšší.

| Klíč | Výchozí | Význam |
| --- | --- | --- |
| `chat.provider`, `chat.model` | žádné | S kým mluvit. `chat.model` může být `provider/model`. |
| `chat.system` | žádný | Systémový prompt |
| `chat.max_tokens`, `chat.temperature` | žádné | Nenastaveno — nechává se na poskytovateli |
| `chat.stream` | `true` | Streamování volitelné odpovědi sudičky; tahy agenta s nástroji zatím používají celé odpovědi, aby volání nástrojů zůstala neporušená |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Uvažování a počty tokenů na stderr |
| `agent.max_turns` | `20` | Maximum kol s nástroji na jeden požadavek uživatele |
| `agent.persist_sessions` | `true` | Ukládat lokální konverzace mezi spuštěními |
| `permissions.shell` | `ask` | Spouštění příkazů shellu: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Čtení souborů: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Změny souborů: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Stahování URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Povolené kořeny pro souborové nástroje |
| `permissions.denied_paths` | žádné | Výslovně blokované cesty k souborům |
| `network.connect_timeout_secs` | `15` | Čas povolený na spojení |
| `network.idle_timeout_secs` | `300` | Povolené mlčení při čekání na data |
| `network.proxy` | `system` | `system`, `none`, nebo adresa proxy |
| `network.ca_bundle` | žádný | Doplněk: soubor důvěryhodných certifikačních autorit |
| `network.allow_insecure_http` | `false` | Povolit klíč přes prostý `http` na cizí host |
| `update.check` | `true` | Ptát se GitHubu a GitLabu na novější verzi při startu interaktivní relace |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, u kterého kontrola ptá po nejnovějším vydání |
| `update.gitlab_repo` | žádné | GitLab `owner/repo` pro tutéž kontrolu. Prázdné vypíná GitLab |
| `update.timeout_secs` | `3` | Kolik sekund smí kontrola strávit na jednom zdroji |

Marelle nese vlastní seznam důvěryhodných certifikačních autorit, takže funguje i tam, kde je systém nemá, jako v Termuxu. `SSL_CERT_FILE` přidává další.

Když se najde novější verze, relace o ní zmíní jednou — verzi, co máte a kde ji vzít. Ptá se nejdříve po vydání; repozitář, který commity jen taguje, dodá nejnovější tag. Cokoli selže, zůstane mlčenlivé.

## Vaše klíče zůstávají vaše

- Nastavení pojmenovávají proměnnou nebo soubor, kde klíč leží, nikdy samotný klíč. Nastavení typu `providers.x.api_key` je odmítnuto.
- Klíč se netiskne v `config show`, `config explain` ani `providers` a je vystřižen z každého textu chyby vráceného od serveru.
- Posílá se jen na adresu poskytovatele, v jedné hlavičce, a nikdy přes prostý `http` na jiný stroj, pokud to nedovolíte.
- Přesměrování se nesledují, takže klíč nemůže odtáhnout jinam.

## Jazyky

Rozhraní mluví anglicky, rusky, francouzsky a ukrajinsky. Volí se přes `--lang <code>` nebo `MARELLE_LANG`; jinak rozhodují `LC_ALL`, `LC_MESSAGES` a `LANG`. Každá zpráva bydlí v katalogu a test padne, když nějaký jazyk jednu opomene. Cílem jsou všechny jazyky Eurasie a pak víc.

## Plán cesty

1. **Mluvit.** Adaptéry kompatibilní s OpenAI a Anthropic, se streamováním SSE i rozbiorem celých odpovědí. Tahy agenta s nástroji zatím používají celé odpovědi. *Hotovo.*
2. **Nastavení a vzhled.** Soubory nastavení TOML, globální i na projekt, s `config set`, `get` a `unset` a popisem každého klíče. Tematické rozhraní v duchu [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, informační panel, odpovědi v rámečku, spinner, výchozí ve fialové barvě glicýnie, s libovolnou barvou jako HEX. *Hotovo.*
3. **Mnoho modelů.** Aliasy modelů, současné dotázání několika modelů a porovnání odpovědí, řetězce fallback a model rozhodčí, který je skloubí. *Hotovo.*
4. **Jednat.** Strukturovaná volání nástrojů pro poskytovatele kompatibilní s OpenAI a Anthropic, lokální nástroje pro soubory, hledání a terminál, stahování URL, lokální dovednosti a oprávnění allow/ask/deny; transkript ukazuje každé volání nástroje i jeho výsledek. *Hotovo.*
5. **Pamatovat.** Pojmenované konverzace přetrvávají lokálně mezi spuštěními. Dlouhodobá sémantická paměť ještě před námi.
6. **Rozšiřovat.** Servery MCP, vnější zásuvné moduly, podagenti a bohatší podpora dovedností.
7. **Dosažit.** Automatizace prohlížeče, brány messengerů a naplánované úlohy.

Uvažování stojící za návrhem je v [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Známé limity

- Voliče poskytovatelů a modelů vypisují nastavené poskytovate a aliasy modelů. Marelle zatím neobjevuje vzdálené katalogy modelů.
- Voliče podporují klik myší pro zaměření a posouvání kolečkem; transkript a editor vstupu zůstávají ovládané klávesnicí.
- Ctrl-C opouští interaktivní relaci. Zatím neruší jen aktuální odpověď.
- Smyčka agenta, lokální nástroje vědomé oprávnění a trvalé lokální relace jsou implementovány, ale Marelle zůstává prototypem pre-alpha; o současném návrhu a mezích čtěte [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Vývoj

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testy se sítě nikdy nedotýkají: baví se s atrapovým serverem v `crates/marelle-testkit`. GitHub Actions sestavuje také binárky pro Android a ověřuje deklarovanou minimální verzi Rust. Pro zrcadlo na GitLabu je tu roura GitLab CI; viz [docs/GITLAB.md](docs/GITLAB.md).

## Licence

[AGPL-3.0-only](LICENSE). Oznámení třetích stran jsou v [NOTICE](NOTICE). Archivy vydání obsahují `THIRD-PARTY-LICENSES.txt` pro bedny Rust uvnitř binárky.
