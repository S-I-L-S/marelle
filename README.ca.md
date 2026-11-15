```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Un agent d'IA configurable per al teu terminal.**
Traeu-hi les vostres claus d'API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, una Ollama local, o qualsevol servidor compatible amb OpenAI. Primer Linux i Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · Català
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, versió 0.1.0-alpha.1.** Implementat: interfície TUI nativa, xat amb diversos proveïdors, un bucle d'agent conscient dels permisos, eines locals i sessions locals que es poden reprendre. Les API i la configuració encara poden canviar. Això és un prototip, no una paritat de funcions amb OpenCode ni Hermes.

*Marelle* és el nom francès del joc de la xarranca. Cada casella és un pas.

## Instal·lació

**Termux a l'Android.** Baixa un binari ja construït de la pàgina de [llançaments](https://github.com/S-I-L-S/marelle/releases), o compila'l al telèfon. Guia pas a pas: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, i en qualsevol lloc on Rust funcioni.** Rust 1.85 o més nou:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

O clona el repositori i executa `cargo build --release`. El binari és `target/release/marelle`, uns 4 MiB.

## Primer xat

Guarda la clau en un fitxer que només llegeixis tu, i digues a Marelle on és. Marelle mai guarda una clau dins la seva configuració.

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

Substitueix `MODEL` per un nom de model de la llista del teu proveïdor. Altres maneres d'usar-lo:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

La resposta va al stdout. El raonaments, els comptes de tokens, les notes i els errors van al stderr, de manera que `marelle chat ... > answer.txt` només desa la resposta.

## Eines de l'agent

Marelle envia definicions estructurades d'eines per les API compatibles amb OpenAI i Anthropic. El joc d'eines integrat:

| Eina | Propòsit |
| --- | --- |
| `terminal` | Executar una ordre de shell des del directori d'arrencada |
| `read_file`, `write_file`, `edit_file` | Llegir i modificar fitxers de text UTF-8 |
| `list_directory`, `search_text` | Inspeccionar i cercar en els fitxers del projecte |
| `fetch_url` | Obtenir una pàgina pública HTTP(S) com a text (requereix `curl`) |
| `list_skills`, `load_skill` | Descobrir i carregar instruccions locals `SKILL.md` |

L'ordre `/tools` mostra la llista d'eines i `/skills` enumera les habilitats disponibles. Les pautes del projecte es carreguen des de `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` i certs fitxers `.marelle/` quan són dins de camins permesos.

Les eines de fitxers es confinen a `permissions.allowed_paths` (per defecte: el directori actual del projecte) i respecten `permissions.denied_paths`. La lectura és permesa per defecte; els canvis de fitxers, les ordres de shell i les descàrregues de xarxa pregunten abans. El permís de shell aplica a ordres a l'amfitrió i no és una caixa de sorra del sistema de fitxers: mantén-lo a `ask`, tret que confiïs deliberadament al model l'execució d'ordres.

L'agent s'atura després de `agent.max_turns` rondes d'eines. Les respostes de les eines es tracten com a contingut no fiable. Les peticions usen respostes completes del model, de manera que els arguments de les eines no es muntin de fragments incomplets del flux.

## Sessions

Les converses es desen localment per defecte, incloent-hi la història de crides d'eines i la configuració del model i de la sessió. A Unix el directori de sessions té mode `0700` i el fitxer JSON mode `0600`. El fitxer es guarda a `$XDG_STATE_HOME/marelle/sessions.json`, o a `~/.local/state/marelle/sessions.json` quan `XDG_STATE_HOME` no és establert. Les claus d'API no s'escriuen mai en aquest fitxer. Desactiva la persistència amb:

```sh
marelle config set agent.persist_sessions false
```

Codis de sortida: `0` èxit, `1` error de proveïdor o de xarxa, `2` error d'ús o de configuració.

## Proveïdors

| id | Protocol | Variable de la clau | |
| --- | --- | --- | --- |
| `zai` | compatible amb OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internacional |
| `zai_coding` | compatible amb OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, punt final compatible amb Anthropic |
| `zhipu` | compatible amb OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Xina |
| `zhipu_coding` | compatible amb OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Xina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatible amb OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatible amb OpenAI | `OPENROUTER_API_KEY` | molts models darrere d'una clau |
| `ollama` | compatible amb OpenAI | cap | Ollama en aquesta màquina |

Els punts finals de Z.ai i Zhipu provenen de documentació de tercers, no de les pàgines del proveïdor mateix. Comprova-los davant el teu pla i sobrescriu qualsevol camp si difereixen.

Cada camp d'un proveïdor és canviable, i un proveïdor nou és només un nom amb una adreça:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Altres camps del proveïdor: `kind` (`openai-compatible` o `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` i `extra_body`. Si la variable anomenada per `api_key_env` és establerta, ella guanya; si no, es llegeix el fitxer.

## Configuració

La configuració ve en capes: valors per defecte integrats, després variables d'entorn, després banderes. Els fitxers de configuració s'insereixen entre els valors per defecte i l'entorn.

- `MARELLE_CHAT__MODEL=zai/MODEL` fixa `chat.model`. Un doble subratllat fa de punt.
- `marelle config show [prefix]` enumera la configuració i la capa d'on cadascuna ve.
- `marelle config explain <key>` mostra cada capa que ha fixat una clau, de la prioritat més baixa a la més alta.

| Clau | Per defecte | Sentit |
| --- | --- | --- |
| `chat.provider`, `chat.model` | cap | Amb qui parlar. `chat.model` pot ser `provider/model`. |
| `chat.system` | cap | Prompt de sistema |
| `chat.max_tokens`, `chat.temperature` | cap | Sense fixar — es deixa al proveïdor |
| `chat.stream` | `true` | Flux de la resposta opcional del jutge; les rondes d'agent amb eines usen ara respostes completes, perquè les crides d'eines quedin senceres |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Raonaments i comptes de tokens al stderr |
| `agent.max_turns` | `20` | Màxim de rondes d'eines per petició d'usuari |
| `agent.persist_sessions` | `true` | Desar converses locals entre execucions |
| `permissions.shell` | `ask` | Execució d'ordres de shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Lectura de fitxers: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Canvi de fitxers: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Descàrrega d'URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Arrels permeses per a les eines de fitxers |
| `permissions.denied_paths` | cap | Camins de fitxers expressament blocats |
| `network.connect_timeout_secs` | `15` | Temps permès per connectar |
| `network.idle_timeout_secs` | `300` | Silenci permès mentre s'esperen dades |
| `network.proxy` | `system` | `system`, `none`, o una adreça de proxy |
| `network.ca_bundle` | cap | Un fitxer extra d'autoritats de certificació de confiança |
| `network.allow_insecure_http` | `false` | Permetre una clau per `http` nu cap a un hoste aliè |
| `update.check` | `true` | Preguntar a GitHub i GitLab per una versió més nova a l'inici d'una sessió interactiva |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` al qual la comprovació demana el llançament més nou |
| `update.gitlab_repo` | cap | GitLab `owner/repo` per a la mateixa comprovació. Buit apaga el GitLab |
| `update.timeout_secs` | `3` | Quants segons la comprovació pot gastar en una font |

Marelle porta la seva pròpia llista d'autoritats de certificació de confiança, així que funciona també allà on el sistema no en té, com a Termux. `SSL_CERT_FILE` en afegeix més.

Quan es troba una versió més nova, la sessió la mencionarà un cop — la versió, la que tens, i d'on agafar-la. Es demana primer un llançament; un repositori que només etiqueta els seus commits dona l'etiqueta més nova. Tot allò que falli roman silenciós.

## Les teves claus segueixen sent teves

- La configuració anomena una variable o un fitxer que guarda la clau, mai la clau mateixa. Una configuració com `providers.x.api_key` es refusa.
- La clau no l'imprimeixen `config show`, `config explain` ni `providers`, i es retalla de tot text d'error que torna d'un servidor.
- Va només a l'adreça del proveïdor, en una sola capçalera, i mai per `http` nu cap a una altra màquina, tret que ho permetis.
- Les redireccions no es segueixen, de manera que una clau no pot anar enlloc més.

## Llengües

La interfície parla anglès, rus, francès i ucraïnès. Tria amb `--lang <code>` o `MARELLE_LANG`; si no, decideixen `LC_ALL`, `LC_MESSAGES` i `LANG`. Cada missatge viu en un catàleg, i un test cau si alguna llengua n'omet una. La meta és cada llengua d'Euràsia i després més.

## Full de ruta

1. **Parlar.** Adaptadors compatibles amb OpenAI i Anthropic, amb flux SSE i descodificació de respostes completes. Les rondes d'agent amb eines usen ara respostes completes. *Fet.*
2. **Configuració i aspecte.** Fitxers de configuració TOML, globals i per projecte, amb `config set`, `get` i `unset` i una descripció per a cada clau. Una interfície tematitzada a l'esperit de [Hermes Agent](https://github.com/NousResearch/hermes-agent): bàner, plafó d'informació, respostes emmarcades, filador, per defecte en el lila de la glicínia, qualsevol color posable com a HEX. *Fet.*
3. **Molts models.** Sobrenoms de models, preguntar a diversos models de cop i comparar les respostes, cadenes de reserva, i un model jutge que els fon. *Fet.*
4. **Actuar.** Crides estructurades d'eines per a proveïdors compatibles amb OpenAI i Anthropic, eines locals de fitxers, cerca i terminal, descàrrega d'URL, habilitats locals i permisos allow/ask/deny; la transcripció mostra cada crida d'eina i el seu resultat. *Fet.*
5. **Recordar.** Les converses anomenades persisteixen localment entre execucions. La memòria semàntica a llarg termini encara és al davant.
6. **Ampliar.** Servidors MCP, endolls externs, subagents i un suport més ric d'habilitats.
7. **Arribar.** Automatització de navegador, portes de missatgeria i tasques programades.

El raonament darrere el disseny és a [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Límits coneguts

- Els selectors de proveïdors i models enumeren els proveïdors configurats i els sobrenoms de models. Marelle encara no descobreix catàlegs de models remots.
- Els selectors sostenen el clic de ratolí per al focus i el desplaçament amb rodetes; la transcripció i l'editor d'entrada romanen dirigits per teclat.
- Ctrl-C surt de la sessió interactiva. Encara no cancel·la només la resposta en curs.
- El bucle d'agent, les eines locals conscients dels permisos i les sessions locals persistents són implementats, però Marelle roman un prototip pre-alfa; per al disseny actual i els seus límits, llegeix [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Desenvolupament

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Les proves mai toquen la xarxa: parlen amb un servidor simulat a `crates/marelle-testkit`. GitHub Actions també construeix els binaris per a Android i comprova la versió mínima de Rust declarada. Per a un mirall a GitLab hi ha un conducte GitLab CI; vegeu [docs/GITLAB.md](docs/GITLAB.md).

## Llicència

[AGPL-3.0-only](LICENSE). Els avisos de tercers són a [NOTICE](NOTICE). Els arxius de llançaments inclouen `THIRD-PARTY-LICENSES.txt` per a les caixes de Rust dins del binari.
