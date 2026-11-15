```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigurovateľný AI agent pre váš terminál.**
Prineste si vlastné API kľúče: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokálnu Ollamu alebo akýkoľvek server kompatibilný s OpenAI. Predovšetkým Linux a Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · Slovenčina · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, verzia 0.1.0-alpha.1.** Implementované: natívne rozhranie TUI, čet s viacerými poskytovateľmi, agenta slučka vedúca oprávnenia, lokálne nástroje a obnoviteľné lokálne relácie. API a nastavenia sa môžu ešte zmeniť. Toto je prototyp, nie parita funkcií s OpenCode či Hermes.

*Marelle* je francúzske pomenovanie detskej skákacej hry panák. Každé políčko je jeden krok.

## Inštalácia

**Termux na Androide.** Stiahnite hotový binárny súbor zo stránky [vydaní](https://github.com/S-I-L-S/marelle/releases), alebo si ho zostavte priamo na telefóne. Podrobný postup: [docs/TERMUX.md](docs/TERMUX.md).

**Linux a všade tam, kde beží Rust.** Potrebujete Rust 1.85 alebo novší:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Alebo klonujte repozitár a spustite `cargo build --release`. Binárka je `target/release/marelle`, zhruba 4 MiB.

## Prvý čet

Uchovávajte kľúč v súbore, ktorý čítate len vy, a povedzte Marelle, kde je. Marelle si kľúč nikdy neukladá do svojich nastavení.

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

Nahraďte `MODEL` názvom modelu zo zoznamu vášho poskytovateľa. Ďalšie spôsoby použitia:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Odpoveď ide na stdout. Uvažovanie, počty tokenov, poznámky a chyby idú na stderr, takže `marelle chat ... > answer.txt` uloží len odpoveď.

## Nástroje agenta

Marelle posiela štruktúrované definície nástrojov cez API kompatibilné s OpenAI a cez Anthropic. Vstavaná sada nástrojov:

| Nástroj | Účel |
| --- | --- |
| `terminal` | Spustiť príkaz shellu z adresára spustenia |
| `read_file`, `write_file`, `edit_file` | Čítať a upravovať textové súbory UTF-8 |
| `list_directory`, `search_text` | Prehliadať a prehľadávať súbory projektu |
| `fetch_url` | Stiahnuť verejnú stránku HTTP(S) ako text (vyžaduje `curl`) |
| `list_skills`, `load_skill` | Objavovať a načítať lokálne inštrukcie `SKILL.md` |

Príkaz `/tools` ukáže zoznam nástrojov a `/skills` vypíše dostupné zručnosti. Vedenie projektu sa načítava z `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` a vybraných súborov `.marelle/`, pokiaľ ležia v povolených cestách.

Súborové nástroje sú obmedzené na `permissions.allowed_paths` (predvolené: aktuálny adresár projektu) a rešpektujú `permissions.denied_paths`. Čítanie je v predvolenom stave povolené; zmeny súborov, príkazy shellu a sťahovanie zo siete sa vopred pýtajú. Oprávnenie k shellu sa týka príkazov na hostiteľovi a nie je to pieskovisko systému súborov: držte ho na `ask`, pokiaľ zámerne nesveríte modeli spúšťanie príkazov.

Agent sa zastaví po `agent.max_turns` kolách s nástrojmi. Odpovede nástrojov sa považujú za nedôveryhodný obsah. Požiadavky používajú celé odpovede modelu, takže argumenty nástrojov sa neskladajú z neúplných úsekov prúdu.

## Relácie

Konverzácie sa v predvolenom stave ukladajú lokálne, vrátane histórie volaní nástrojov a nastavení modelu a relácie. Na Unixe má adresár relácií režim `0700` a JSON súbor režim `0600`. Súbor je uložený v `$XDG_STATE_HOME/marelle/sessions.json`, prípadne `~/.local/state/marelle/sessions.json`, keď `XDG_STATE_HOME` nie je nastavené. API kľúče sa do tohto súboru nikdy nezapisujú. Trvalosť vypnite príkazom:

```sh
marelle config set agent.persist_sessions false
```

Návratové kódy: `0` úspech, `1` chyba poskytovateľa alebo siete, `2` chyba použitia či nastavení.

## Poskytovatelia

| id | Protokol | Premenná kľúča | |
| --- | --- | --- | --- |
| `zai` | kompatibilný s OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), medzinárodný |
| `zai_coding` | kompatibilný s OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, koncový bod kompatibilný s Anthropic |
| `zhipu` | kompatibilný s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Čína |
| `zhipu_coding` | kompatibilný s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Čína |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | kompatibilný s OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | kompatibilný s OpenAI | `OPENROUTER_API_KEY` | mnoho modelov za jeden kľúč |
| `ollama` | kompatibilný s OpenAI | žiadny | Ollama na tomto stroji |

Koncové body Z.ai a Zhipu pochádzajú z dokumentácie tretích strán, nie zo stránok samotného dodávateľa. Preverte ich voči svojmu plánu a prepíšte ľubovoľné pole, ak sa líšia.

Každé pole poskytovateľa možno zmeniť a nový poskytovateľ je jednoducho názv s adresou:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Ostatné polia poskytovateľa: `kind` (`openai-compatible` alebo `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` a `extra_body`. Ak je nastavená premenná pomenovaná v `api_key_env`, víťazí; inak sa číta súbor.

## Nastavenia

Nastavenia prichádzajú vo vrstvách: vstavané predvolené hodnoty, potom premenné prostredia, potom prepínače. Súbory nastavení sa vkladajú medzi predvolené hodnoty a prostredie.

- `MARELLE_CHAT__MODEL=zai/MODEL` nastavuje `chat.model`. Dvojité podčiarknutie zastupuje bodku.
- `marelle config show [prefix]` vypisuje nastavenia a vrstvu, z ktorej každé prišlo.
- `marelle config explain <key>` ukazuje každú vrstvu, ktorá nastavila kľúč, od najnižšej priority po najvyššiu.

| Kľúč | Predvolené | Význam |
| --- | --- | --- |
| `chat.provider`, `chat.model` | žiadne | S kým hovoriť. `chat.model` môže byť `provider/model`. |
| `chat.system` | žiadny | Systémový prompt |
| `chat.max_tokens`, `chat.temperature` | žiadne | Nenastavené — necháva sa na poskytovateľovi |
| `chat.stream` | `true` | Streamovanie voliteľnej odpovede sudkyne; ťahy agenta s nástrojmi zatiaľ používajú celé odpovede, aby volania nástrojov zostali neporušené |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Uvažovanie a počty tokenov na stderr |
| `agent.max_turns` | `20` | Maximumký kol s nástrojmi na jednu požiadavku používateľa |
| `agent.persist_sessions` | `true` | Ukladať lokálne konverzácie medzi spusteniami |
| `permissions.shell` | `ask` | Spúšťanie príkazov shellu: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Čítanie súborov: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Zmeny súborov: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Sťahovanie URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Povolené korene pre súborové nástroje |
| `permissions.denied_paths` | žiadne | Výslovne blokované cesty k súborom |
| `network.connect_timeout_secs` | `15` | Čas povolený na spojenie |
| `network.idle_timeout_secs` | `300` | Povolené mlčanie pri čakaní na dáta |
| `network.proxy` | `system` | `system`, `none`, alebo adresa proxy |
| `network.ca_bundle` | žiadny | Doplnok: súbor dôveryhodných certifikačných autorít |
| `network.allow_insecure_http` | `false` | Povoliť kľúč cez prostý `http` na cudzí host |
| `update.check` | `true` | Pýtať sa GitHubu a GitLabu na novšiu verziu pri štarte interaktívnej relácie |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, u ktorého kontrola pýta po najnovšom vydaní |
| `update.gitlab_repo` | žiadne | GitLab `owner/repo` pre tú istú kontrolu. Prázdne vypína GitLab |
| `update.timeout_secs` | `3` | Koľko sekúnd smie kontrola stráviť na jednom zdroji |

Marelle nesie vlastný zoznam dôveryhodných certifikačných autorít, takže funguje aj tam, kde ich systém nemá, ako v Termuxe. `SSL_CERT_FILE` pridáva ďalšie.

Keď sa nájde novšia verzia, relácia o nej spomenie raz — verziu, čo máte a kde ju vziať. Pýta sa najprv po vydaní; repozitár, ktorý commity len taguje, dodá najnovší tag. Čokoľvek zlyhá, zostane mlčanlivé.

## Vaše kľúče zostávajú vaše

- Nastavenia pomenúvajú premennú alebo súbor, kde kľúč leží, nikdy samotný kľúč. Nastavenie typu `providers.x.api_key` je odmietnuté.
- Kľúč sa netlačí v `config show`, `config explain` ani `providers` a je vystrihnutý z každého textu chyby vráteného od servera.
- Posiela sa len na adresu poskytovateľa, v jednej hlavičke, a nikdy cez prostý `http` na iný stroj, pokiaľ to nedovolíte.
- Presmerovania sa nesledujú, takže kľúč nemôže odtiahnuť inam.

## Jazyky

Rozhranie hovorí po anglicky, rusky, francúzsky a ukrajinsky. Volí sa cez `--lang <code>` alebo `MARELLE_LANG`; inak rozhodujú `LC_ALL`, `LC_MESSAGES` a `LANG`. Každá správa býva v katalógu a test padne, keď nejaký jazyk jednu opomenie. Cieľom sú všetky jazyky Eurasie a potom viac.

## Plán cesty

1. **Hovoriť.** Adaptéry kompatibilné s OpenAI a Anthropic, so streamovaním SSE i rozborom celých odpovedí. Ťahy agenta s nástrojmi zatiaľ používajú celé odpovede. *Hotovo.*
2. **Nastavenia a vzhľad.** Súbory nastavení TOML, globálne i na projekt, s `config set`, `get` a `unset` a popisom každého kľúča. Tematické rozhranie v duchu [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, informačný panel, odpovede v rámčeku, spinner, predvolené vo fialovej farbe glicínie, s ľubovoľnou farbou ako HEX. *Hotovo.*
3. **Mnoho modelov.** Aliasy modelov, súčasné opýtanie niekoľkých modelov a porovnanie odpovedí, reťaze fallback a model rozhodca, ktorý ich skĺbi. *Hotovo.*
4. **Jednať.** Štruktúrované volania nástrojov pre poskytovateľov kompatibilných s OpenAI a Anthropic, lokálne nástroje pre súbory, hľadanie a terminál, sťahovanie URL, lokálne zručnosti a oprávnenia allow/ask/deny; transkript ukazuje každé volanie nástroja a jeho výsledok. *Hotovo.*
5. **Pamätať.** Pomenované konverzácie pretrvávajú lokálne medzi spusteniami. Dlhodobá sémantická pamäť ešte pred nami.
6. **Rozširovať.** Servery MCP, vonkajšie zásuvné moduly, podagenti a bohatšia podpora zručností.
7. **Dosažať.** Automatizácia prehliadača, brány messengerov a naplánované úlohy.

Uvažovanie stojace za návrhom je v [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Známe limity

- Voliče poskytovateľov a modelov vypisujú nastavených poskytovateľov a aliasy modelov. Marelle zatiaľ neobjavuje vzdialené katalógy modelov.
- Voliče podporujú klik myšou na zameranie a posúvanie kolieskom; transkript a editor vstupu zostávajú ovládané klávesnicou.
- Ctrl-C opúšťa interaktívnu reláciu. Zatiaľ neruší len aktuálnu odpoveď.
- Slučka agenta, lokálne nástroje vedúce oprávnenia a trvalé lokálne relácie sú implementované, ale Marelle zostáva prototypom pre-alpha; o súčasnom návrhu a medziach čítajte [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Vývoj

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testy sa so sieťou nikdy nedotýkajú: bavia sa s atrapovým serverom v `crates/marelle-testkit`. GitHub Actions zostavuje aj binárky pre Android a overuje deklarovanú minimálnu verziu Rust. Pre zrkadlo na GitLabe je tu rúra GitLab CI; viď [docs/GITLAB.md](docs/GITLAB.md).

## Licencia

[AGPL-3.0-only](LICENSE). Oznámenia tretích strán sú v [NOTICE](NOTICE). Archívy vydaní obsahujú `THIRD-PARTY-LICENSES.txt` pre bedne Rust vnútri binárky.
