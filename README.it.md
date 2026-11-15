```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Un agente IA configurabile per il tuo terminale.**
Porta le tue chiavi API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, una Ollama locale, oppure qualsiasi server compatibile con OpenAI. Prima Linux e Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · Italiano · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, versione 0.1.0-alpha.1.** Implementato: interfaccia TUI nativa, chat con più fornitori, un ciclo dell'agente consapevole dei permessi, strumenti locali e sessioni locali riprendibili. Le API e le impostazioni possono ancora cambiare. Questo è un prototipo, non una parità di funzioni con OpenCode o Hermes.

*Marelle* è il nome francese del gioco della campana. Ogni casella è un passo.

## Installazione

**Termux su Android.** Scarica un binario già pronto dalla pagina dei [rilasci](https://github.com/S-I-L-S/marelle/releases), oppure compila sul telefono. Guida passo a passo: [docs/TERMUX.md](docs/TERMUX.md).

**Linux e ovunque funzioni Rust.** Serve Rust 1.85 o più recente:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Oppure clona il deposito ed esegui `cargo build --release`. Il binario è `target/release/marelle`, circa 4 MiB.

## Prima chat

Tieni la chiave in un file che legge solo tu, e dì a Marelle dov'è. Marelle non conserva mai una chiave nelle sue impostazioni.

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

Sostituisci `MODEL` con un nome di modello dalla lista del tuo fornitore. Altri modi d'uso:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

La risposta va su stdout. Il ragionamento, i conteggi di token, le note e gli errori vanno su stderr, così `marelle chat ... > answer.txt` salva solo la risposta.

## Strumenti dell'agente

Marelle invia definizioni strutturate di strumenti tramite le API compatibili OpenAI e Anthropic. Il corredo di strumenti integrato:

| Strumento | Scopo |
| --- | --- |
| `terminal` | Eseguire un comando di shell dalla directory di avvio |
| `read_file`, `write_file`, `edit_file` | Leggere e modificare file di testo UTF-8 |
| `list_directory`, `search_text` | Ispezionare e cercare nei file del progetto |
| `fetch_url` | Scaricare una pagina pubblica HTTP(S) come testo (richiede `curl`) |
| `list_skills`, `load_skill` | Scoprire e caricare istruzioni locali `SKILL.md` |

Il comando `/tools` mostra la lista degli strumenti e `/skills` elenca le competenze disponibili. Le direttive del progetto si caricano da `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` e certi file `.marelle/`, quando giacciono in percorsi consentiti.

Gli strumenti per file sono confinati a `permissions.allowed_paths` (predefinito: la directory corrente del progetto) e rispettano `permissions.denied_paths`. La lettura è permessa per impostazione predefinita; le modifiche di file, i comandi di shell e gli scaricamenti di rete chiedono prima. Il permesso di shell vale per i comandi sull'ospite e non è una vasca di sabbia del sistema di file: tienilo su `ask`, a meno che non affidi di proposito l'esecuzione di comandi al modello.

L'agente si ferma dopo `agent.max_turns` giri di strumenti. Le risposte degli strumenti si trattano come contenuto non affidabile. Le richieste usano risposte complete del modello, così gli argomenti degli strumenti non si assemblano da frammenti incompleti di flusso.

## Sessioni

Le conversazioni si salvano localmente per impostazione predefinita, incluso il registro delle chiamate di strumenti e le impostazioni di modello e sessione. Su Unix la directory delle sessioni è in modo `0700` e il file JSON in modo `0600`. Il file si conserva in `$XDG_STATE_HOME/marelle/sessions.json`, oppure `~/.local/state/marelle/sessions.json` quando `XDG_STATE_HOME` non è fissato. Le chiavi API non si scrivono mai in questo file. Disattiva la persistenza con:

```sh
marelle config set agent.persist_sessions false
```

Codici d'uscita: `0` successo, `1` errore di fornitore o di rete, `2` errore d'uso o di configurazione.

## Fornitori

| id | Protocollo | Variabile della chiave | |
| --- | --- | --- | --- |
| `zai` | compatibile OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internazionale |
| `zai_coding` | compatibile OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, punto terminale compatibile Anthropic |
| `zhipu` | compatibile OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Cina |
| `zhipu_coding` | compatibile OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Cina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatibile OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatibile OpenAI | `OPENROUTER_API_KEY` | molti modelli dietro una chiave |
| `ollama` | compatibile OpenAI | nessuna | Ollama su questa macchina |

I punti terminali di Z.ai e Zhipu provengono da documentazione di terzi, non dalle pagine del fornitore stesso. Verificali contro il tuo piano e sovrascrivi qualsiasi campo se differiscono.

Ogni campo di un fornitore è mutabile, e un nuovo fornitore è semplicemente un nome con un indirizzo:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Altri campi del fornitore: `kind` (`openai-compatible` o `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` e `extra_body`. Se la variabile nominata da `api_key_env` è fissata, vince lei; altrimenti si legge il file.

## Impostazioni

Le impostazioni arrivano a strati: valori predefiniti integrati, poi variabili d'ambiente, poi bandiere. I file di impostazioni si incastrano tra i valori predefiniti e l'ambiente.

- `MARELLE_CHAT__MODEL=zai/MODEL` fissa `chat.model`. Un doppio sottolineato sta per un punto.
- `marelle config show [prefix]` elenca le impostazioni e lo strato da cui ciascuna venne.
- `marelle config explain <key>` mostra ogni strato che fissò una chiave, dalla priorità più bassa alla più alta.

| Chiave | Predefinito | Senso |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nessuno | Con chi parlare. `chat.model` può essere `provider/model`. |
| `chat.system` | nessuno | Prompt di sistema |
| `chat.max_tokens`, `chat.temperature` | nessuno | Non fissato — lasciato al fornitore |
| `chat.stream` | `true` | Flusso della risposta facoltativa del giudice; i turni d'agente con strumenti usano al presente risposte complete, perché le chiamate di strumenti restino intatte |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Ragionamento e conteggi di token su stderr |
| `agent.max_turns` | `20` | Massimo di giri di strumenti per richiesta di utente |
| `agent.persist_sessions` | `true` | Salvare le conversazioni locali tra gli avvii |
| `permissions.shell` | `ask` | Esecuzione di comandi di shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Lettura di file: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Modifica di file: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Scaricamento di URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Radici consentite per gli strumenti di file |
| `permissions.denied_paths` | nessuno | Percorsi di file espressamente bloccati |
| `network.connect_timeout_secs` | `15` | Tempo concesso per connettersi |
| `network.idle_timeout_secs` | `300` | Silenzio concesso in attesa di dati |
| `network.proxy` | `system` | `system`, `none`, o un indirizzo di proxy |
| `network.ca_bundle` | nessuno | Un file extra di autorità di certificazione fidate |
| `network.allow_insecure_http` | `false` | Permettere una chiave su `http` nudo verso un ospite estraneo |
| `update.check` | `true` | Chiedere a GitHub e GitLab una versione più nuova all'inizio di una sessione interattiva |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` dal quale la verifica chiede il rilascio più recente |
| `update.gitlab_repo` | nessuno | GitLab `owner/repo` per la stessa verifica. Vuoto spegne GitLab |
| `update.timeout_secs` | `3` | Quanti secondi la verifica può spendere su una fonte |

Marelle porta con sé un proprio elenco di autorità di certificazione fidate, così funziona anche là dove il sistema non ne ha, come in Termux. `SSL_CERT_FILE` ne aggiunge.

Quando si trova una versione più nuova, la sessione la menziona una volta — la versione, quella che hai, e dove prenderla. Si chiede prima un rilascio; un deposito che i suoi commit solo etichetta rende l'etichetta più recente. Qualunque cosa fallisca resta silenziosa.

## Le tue chiavi restano tue

- Le impostazioni nominano una variabile o un file che custodisce la chiave, mai la chiave stessa. Un'impostazione come `providers.x.api_key` si respinge.
- La chiave non la stampano `config show`, `config explain` né `providers`, e si taglia via da ogni testo d'errore che torna da un server.
- Va soltanto all'indirizzo del fornitore, in un solo intestazione, e mai su `http` nudo verso un'altra macchina, salvo che tu lo permetta.
- I redirigimenti non si seguono, così una chiave non può esser sospinta altrove.

## Lingue

L'interfaccia parla inglese, russo, francese e ucraino. Scegli con `--lang <code>` o `MARELLE_LANG`; altrimenti decidono `LC_ALL`, `LC_MESSAGES` e `LANG`. Ogni messaggio vive in un catalogo, e un test cade se una lingua ne lascia fuori uno. La meta è ogni lingua dell'Eurasia e poi di più.

## Programma

1. **Parlare.** Adattatori compatibili OpenAI e Anthropic, con flusso SSE e decodifica di risposte complete. I turni d'agente con strumenti usano al presente risposte complete. *Fatto.*
2. **Impostazioni e apparenza.** File d'impostazioni TOML, globali e per progetto, con `config set`, `get` e `unset` e una descrizione per ogni chiave. Un'interfaccia tematica nello spirito di [Hermes Agent](https://github.com/NousResearch/hermes-agent): striscione, pannello d'informazioni, risposte incorniciate, filatore, per difetto nel viola della glicine, ogni colore fissabile come HEX. *Fatto.*
3. **Molti modelli.** Soprannomi di modelli, chiedere più modelli insieme e confrontare le risposte, catene di ripiego, e un modello giudice che li fonde. *Fatto.*
4. **Agire.** Chiamate strutturate di strumenti per fornitori compatibili OpenAI e Anthropic, strumenti locali di file, ricerca e terminale, scaricamento di URL, competenze locali e permessi allow/ask/deny; il trascritto mostra ogni chiamata di strumento e il suo risultato. *Fatto.*
5. **Ricordare.** Le conversazioni nominate persistono localmente tra gli avvii. La memoria semantica di lungo corso è ancora davanti.
6. **Estendere.** Server MCP, esterni innesti, sotto-agenti e un sostegno più ricco delle competenze.
7. **Raggiungere.** Automazione del navigatore, portali di messaggeria e compiti programmati.

Il ragionamento dietro la progettazione è in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Limiti noti

- I selettori di fornitori e modelli elencano i fornitori configurati e i soprannomi di modelli. Marelle non scopre ancora cataloghi di modelli remoti.
- I selettori sostengono il clic del mouse per il fuoco e lo scorrimento con la rotella; il trascritto e l'editor di entrata restano guidati da tastiera.
- Ctrl-C esce dalla sessione interattiva. Non interrompe ancora solo la risposta corrente.
- Il ciclo dell'agente, gli strumenti locali consapevoli dei permessi e le sessioni locali persistenti sono realizzati, ma Marelle resta un prototipo pre-alfa; per la progettazione attuale e i suoi confini leggi [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Sviluppo

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Le prove non toccano mai la rete: parlano con un server fittizio in `crates/marelle-testkit`. GitHub Actions compila anche i binari per Android e verifica la versione minima di Rust dichiarata. Per uno specchio su GitLab c'è un condotto GitLab CI; vedi [docs/GITLAB.md](docs/GITLAB.md).

## Licenza

[AGPL-3.0-only](LICENSE). Le note di terzi sono in [NOTICE](NOTICE). Gli archivi dei rilasci includono `THIRD-PARTY-LICENSES.txt` per le casse Rust dentro il binario.
