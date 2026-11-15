```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Un agent AI configurabil pentru terminalul tău.**
Adu-ți propriile chei API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, un Ollama local sau orice server compatibil OpenAI. Întâi Linux și Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · Română · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, versiunea 0.1.0-alpha.1.** Implementat: interfață TUI nativă, chat cu mai mulți furnizori, bucla agentului conștientă de permisiuni, instrumente locale și sesiuni locale reluabile. API-urile și setările se pot schimba încă. Acesta este un prototip, nu o paritate de funcții cu OpenCode sau Hermes.

*Marelle* este denumirea franceză a jocului „țurca". Fiecare căsuță este un pas.

## Instalare

**Termux pe Android.** Descarcă un binar gata construit de pe pagina [versiunilor](https://github.com/S-I-L-S/marelle/releases) sau construiește-l pe telefon. Ghid pas cu pas: [docs/TERMUX.md](docs/TERMUX.md).

**Linux și oriunde rulează Rust.** Ai nevoie de Rust 1.85 sau mai nou:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Sau clonează depozitul și rulează `cargo build --release`. Binarul este `target/release/marelle`, aproximativ 4 MiB.

## Primul chat

Păstrează cheia într-un fișier pe care doar tu îl citești și spune-i Marelle unde este. Marelle nu salvează niciodată o cheie în setările sale.

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

Înlocuiește `MODEL` cu un nume de model din lista furnizorului tău. Alte moduri de folosire:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Răspunsul merge la stdout. Raționamentele, numărul de tokenuri, notele și erorile merg la stderr, astfel încât `marelle chat ... > answer.txt` salvează doar răspunsul.

## Instrumentele agentului

Marelle trimite definiții structurate de instrumente prin API-urile compatibile OpenAI și Anthropic. Setul de instrumente incluse:

| Instrument | Scop |
| --- | --- |
| `terminal` | Rularea unei comenzi shell din directorul de pornire |
| `read_file`, `write_file`, `edit_file` | Citirea și modificarea fișierelor text UTF-8 |
| `list_directory`, `search_text` | Inspectarea și căutarea în fișierele proiectului |
| `fetch_url` | Descărcarea unei pagini publice HTTP(S) ca text (necesită `curl`) |
| `list_skills`, `load_skill` | Descoperirea și încărcarea instrucțiunilor locale `SKILL.md` |

Comanda `/tools` arată lista instrumentelor, iar `/skills` listează abilitățile disponibile. Îndrumările proiectului se încarcă din `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` și fișierele alese `.marelle/`, dacă se află în căi permise.

Instrumentele pentru fișiere sunt mărginite la `permissions.allowed_paths` (implicit: directorul curent al proiectului) și respectă `permissions.denied_paths`. Citirea este permisă implicit; schimbările de fișiere, comenzile shell și descărcările din rețea întreb înainte. Permisiunea de shell se referă la comenzi pe gazdă și nu este o cușcă a sistemului de fișiere: ține-o pe `ask`, dacă nu ai încredere în modell pentru rularea comenzilor în mod deliberat.

Agentul se oprește după `agent.max_turns` ture cu instrumentele. Răspunsurile instrumentelor sunt tratate ca conținut de neîncredere. Cererile folosesc răspunsuri complete ale modelului, astfel încât argumentele instrumentelor nu se asamblează din fragmente incomplete de flux.

## Sesiuni

Conversațiile se salvează local implicit, inclusiv istoricul apelurilor de instrumente și setările modelului și sesiunii. Pe Unix directorul sesiunilor are modul `0700`, iar fișierul JSON modul `0600`. Fișierul se păstrează în `$XDG_STATE_HOME/marelle/sessions.json`, sau `~/.local/state/marelle/sessions.json` când `XDG_STATE_HOME` nu e stabilit. Cheile API nu se scriu niciodată în acest fișier. Dezactivează persistența cu:

```sh
marelle config set agent.persist_sessions false
```

Coduri de ieșire: `0` succes, `1` eroare de furnizor sau de rețea, `2` eroare de folosire ori de configurare.

## Furnizori

| id | Protocol | Variabila cheii | |
| --- | --- | --- | --- |
| `zai` | compatibil OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internațional |
| `zai_coding` | compatibil OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, punct final compatibil Anthropic |
| `zhipu` | compatibil OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | compatibil OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatibil OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatibil OpenAI | `OPENROUTER_API_KEY` | multe modele după o singură cheie |
| `ollama` | compatibil OpenAI | niciuna | Ollama pe această mașină |

Punctele finale Z.ai și Zhipu vin din documentația terților, nu de pe paginile furnizorului însuși. Verifică-le față de planul tău și suprascrie orice câmp dacă diferă.

Fiecare câmp al unui furnizor poate fi schimbat, iar un furnizor nou este doar un nume cu o adresă:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Alte câmpuri ale furnizorului: `kind` (`openai-compatible` sau `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` și `extra_body`. Dacă variabila numită de `api_key_env` e stabilită, ea câștigă; altfel se citește fișierul.

## Setări

Setările vin în straturi: valori implicite incluse, apoi variabile de mediu, apoi steaguri. Fișierele de setări se plasează între valorile implicite și mediu.

- `MARELLE_CHAT__MODEL=zai/MODEL` stabilește `chat.model`. Un dublu liniar inferior stă pentru un punct.
- `marelle config show [prefix]` listează setările și stratul din care fiecare a venit.
- `marelle config explain <key>` arată fiecare strat care a stabilit un cheie, de la prioritatea cea mai mică la cea mai mare.

| Cheie | Implicit | Sens |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nimic | Cu cine să vorbești. `chat.model` poate fi `provider/model`. |
| `chat.system` | nimic | Prompt de sistem |
| `chat.max_tokens`, `chat.temperature` | nimic | Nestabilite — lasate furnizorului |
| `chat.stream` | `true` | Fluxarea răspunsului opțional al judecătorului; turile agentului cu instrumente folosesc deocamdată răspunsuri complete, pentru ca apelurile de instrumente să rămână întregi |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Raționamente și număr de tokenuri pe stderr |
| `agent.max_turns` | `20` | Maximum de ture cu instrumente per cerere de utilizator |
| `agent.persist_sessions` | `true` | Salvează conversațiile locale între rulări |
| `permissions.shell` | `ask` | Rularea comenzilor shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Citirea fișierelor: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Schimbarea fișierelor: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Descărcarea URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Rădăcini permise pentru instrumentele de fișiere |
| `permissions.denied_paths` | nimic | Căi de fișiere explicit blocate |
| `network.connect_timeout_secs` | `15` | Timpul permis pentru conectare |
| `network.idle_timeout_secs` | `300` | Tăcerea permisă în așteptarea datelor |
| `network.proxy` | `system` | `system`, `none`, sau o adresă de proxy |
| `network.ca_bundle` | nimic | Un fișier suplimentar de autorități de certificare de încredere |
| `network.allow_insecure_http` | `false` | Permite o cheie pe `http` simplu către un host străin |
| `update.check` | `true` | Întreabă GitHub și GitLab de o versiune mai nouă la începutul unei sesiuni interactive |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` la care verificarea întreabă de cea mai nouă versiune |
| `update.gitlab_repo` | nimic | GitLab `owner/repo` pentru aceeași verificare. Gol dezactivează GitLab |
| `update.timeout_secs` | `3` | Câte secunde poate cheltui verificarea pe o sursă |

Marelle își poartă propria listă de autorități de certificare de încredere, deci funcționează și unde sistemul nu are, ca în Termux. `SSL_CERT_FILE` adaugă altele.

Când se găsește o versiune mai nouă, sesiunea o menționează o dată — versiunea, ce ai tu și unde s-o iei. Se întreabă întâi de versiune; un depozit care își marchează commiturile doar cu taguri dă cel mai nou tag. Oricice eșuează rămâne tăcut.

## Cheile tale rămân ale tale

- Setările numesc o variabilă sau un fișier care ține cheia, niciodată cheia însăși. O setare precum `providers.x.api_key` este refuzată.
- Cheia nu e tipărită de `config show`, `config explain` sau `providers` și e tăiată din orice text de eroare care vine de la un server.
- Se trimite doar la adresa furnizorului, într-un singur antet, și niciodată pe `http` simplu către altă mașină, dacă nu permiți.
- Redirecționările nu sunt urmărite, astfel o cheie nu poate fi trimisă aiurea.

## Limbi

Interfața vorbește engleză, rusă, franceză și ucraineană. Alege cu `--lang <code>` sau `MARELLE_LANG`; altfel decid `LC_ALL`, `LC_MESSAGES` și `LANG`. Fiecare mesaj trăiește într-un catalog, iar un test cade dacă o limbă îi lasă unul afară. Ținta este fiecare limbă a Eurasiei și apoi mai mult.

## Planul

1. **Vorbi.** Adaptor compatibil OpenAI și Anthropic, cu fluxare SSE și decodare de răspunsuri complete. Turile agentului cu instrumente folosesc deocamdată răspunsuri complete. *Gata.*
2. **Setări și înfățișare.** Fișiere de setări TOML, globale și per proiect, cu `config set`, `get` și `unset` și o descriere pentru fiecare cheie. O interfață temată în spiritul [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, panou de informații, răspunsuri în chenar, spinner, implicit în violet de glicinie, cu orice culoare stabilită ca HEX. *Gata.*
3. **Multe modele.** Aliasuri de modele, întrebarea mai multor modele deodată și compararea răspunsurilor, lanțuri de rezervă și un model judecător care le unește. *Gata.*
4. **Acționa.** Apeluri structurate de instrumente pentru furnizori compatibili OpenAI și Anthropic, instrumente locale de fișiere/căutare/terminal, descărcare URL, abilități locale și permisiuni allow/ask/deny; transcribul arată fiecare apel de instrument și rezultatul lui. *Gata.*
5. **A ține minte.** Conversațiile numite persistă local între rulări. Memoria semantică pe termen lung mai așteaptă.
6. **A extinde.** Servere MCP, pluginuri externe, subagenți și o susținere mai bogată a abilităților.
7. **A ajunge.** Automatizare de browser, porți de mesagerie și sarcini programate.

Raționamentul din spatele designului e în [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Limite cunoscute

- Selectorii de furnizori și modele listează furnizorii configurați și aliasurile de modele. Marelle nu descoperă încă cataloage de modele la distanță.
- Selectorii susțin clic de mouse pentru focalizare și rostovire cu rotița; transcribul și editorul de intrare rămân conduse din tastatură.
- Ctrl-C ieșe din sesiunea interactivă. Încă nu anulează doar răspunsul curent.
- Bucla agentului, instrumentele locale conștiente de permisiuni și sesiunile locale persistente sunt implementate, dar Marelle rămâne un prototip pre-alpha; vezi [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) pentru designul actual și limitele lui.

## Dezvoltare

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testele nu ating niciodată rețeaua: vorbesc cu un server-fantomă în `crates/marelle-testkit`. GitHub Actions construiește și binarele pentru Android și verifică versiunea minimă de Rust declarată. Pentru o oglindă pe GitLab există o conductă GitLab CI; vezi [docs/GITLAB.md](docs/GITLAB.md).

## Licență

[AGPL-3.0-only](LICENSE). Mențiunile terților sunt în [NOTICE](NOTICE). Arhivele versiunilor includ `THIRD-PARTY-LICENSES.txt` pentru crate-urile Rust din interiorul binarului.
