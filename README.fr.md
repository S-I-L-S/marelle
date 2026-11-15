```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Un agent d'IA configurable pour votre terminal.**
Apportez vos propres clés d'API : Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, une Ollama locale, ou tout serveur compatible OpenAI. D'abord Linux et Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · Français · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pré-alpha, version 0.1.0-alpha.1.** Réalisé : une interface TUI native, un chat multi-fournisseurs, une boucle d'agent consciente des permissions, des outils locaux et des sessions locales reprenables. Les API et les réglages peuvent encore changer. Ceci est un prototype, pas une parité de fonctions avec OpenCode ou Hermes.

*Marelle* est le nom français du jeu de la marelle. Chaque case est un pas.

## Installation

**Termux sur Android.** Téléchargez un binaire prêt à l'emploi depuis la page des [publications](https://github.com/S-I-L-S/marelle/releases), ou compilez-le sur le téléphone. Guide pas à pas : [docs/TERMUX.md](docs/TERMUX.md).

**Linux, et partout où Rust fonctionne.** Rust 1.85 ou plus récent :

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ou cloner le dépôt et lancer `cargo build --release`. Le binaire est `target/release/marelle`, environ 4 Mio.

## Premier chat

Gardez la clé dans un fichier que vous seul lisez, et dites à Marelle où elle est. Marelle ne stocke jamais une clé dans ses réglages.

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

Remplacez `MODEL` par un nom de modèle de la liste de votre fournisseur. Autres façons de l'utiliser :

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

La réponse va sur stdout. Les raisonnements, les comptes de jetons, les notes et les erreurs vont sur stderr, de sorte que `marelle chat ... > answer.txt` n'enregistre que la réponse.

## Outils de l'agent

Marelle envoie des définitions d'outils structurées par les API compatibles OpenAI et Anthropic. L'ensemble d'outils intégré :

| Outil | But |
| --- | --- |
| `terminal` | Exécuter une commande de shell depuis le répertoire de lancement |
| `read_file`, `write_file`, `edit_file` | Lire et modifier des fichiers texte UTF-8 |
| `list_directory`, `search_text` | Inspecter et chercher dans les fichiers du projet |
| `fetch_url` | Récupérer une page HTTP(S) publique en texte (exige `curl`) |
| `list_skills`, `load_skill` | Découvrir et charger des instructions locales `SKILL.md` |

La commande `/tools` montre la liste des outils et `/skills` liste les compétences disponibles. Les directives du projet sont chargées depuis `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` et certains fichiers `.marelle/` lorsqu'ils se trouvent dans des chemins permis.

Les outils de fichiers sont confinés à `permissions.allowed_paths` (par défaut : le répertoire courant du projet) et respectent `permissions.denied_paths`. La lecture est permise par défaut ; les modifications de fichiers, les commandes de shell et les téléchargements réseau demandent avant. La permission de shell s'applique aux commandes sur l'hôte et n'est pas un bac à sable du système de fichiers : gardez-la à `ask`, sauf si vous confiez délibérément l'exécution de commandes au modèle.

L'agent s'arrête après `agent.max_turns` allers-retours d'outils. Les réponses d'outils sont traitées comme un contenu non fiable. Les requêtes utilisent des réponses complètes du modèle, de sorte que les arguments d'outils ne s'assemblent pas de fragments de flux incomplets.

## Sessions

Les conversations sont enregistrées localement par défaut, y compris l'historique des appels d'outils et les réglages du modèle et de la session. Sous Unix, le répertoire des sessions est en mode `0700` et le fichier JSON en mode `0600`. Le fichier est stocké dans `$XDG_STATE_HOME/marelle/sessions.json`, ou `~/.local/state/marelle/sessions.json` quand `XDG_STATE_HOME` n'est pas défini. Les clés d'API ne s'écrivent jamais dans ce fichier. Désactivez la persistance avec :

```sh
marelle config set agent.persist_sessions false
```

Codes de sortie : `0` succès, `1` erreur de fournisseur ou de réseau, `2` erreur d'usage ou de réglage.

## Fournisseurs

| id | Protocole | Variable de clé | |
| --- | --- | --- | --- |
| `zai` | compatible OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), international |
| `zai_coding` | compatible OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, point de terminaison compatible Anthropic |
| `zhipu` | compatible OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Chine |
| `zhipu_coding` | compatible OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Chine |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatible OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatible OpenAI | `OPENROUTER_API_KEY` | plusieurs modèles derrière une clé |
| `ollama` | compatible OpenAI | aucune | Ollama sur cette machine |

Les points de terminaison Z.ai et Zhipu proviennent d'une documentation tierce, non des pages du fournisseur lui-même. Vérifiez-les par rapport à votre offre et outrepassez tout champ si elles diffèrent.

Chaque champ d'un fournisseur est modifiable, et un nouveau fournisseur n'est qu'un nom avec une adresse :

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Autres champs de fournisseur : `kind` (`openai-compatible` ou `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` et `extra_body`. Si la variable nommée par `api_key_env` est définie, elle gagne ; sinon le fichier est lu.

## Réglages

Les réglages viennent en couches : valeurs par défaut intégrées, puis variables d'environnement, puis drapeaux. Les fichiers de réglages s'insèrent entre les valeurs par défaut et l'environnement.

- `MARELLE_CHAT__MODEL=zai/MODEL` fixe `chat.model`. Un double soulignement tient lieu de point.
- `marelle config show [prefix]` liste les réglages et la couche d'où chacun vient.
- `marelle config explain <key>` montre chaque couche qui a fixé une clé, de la priorité la plus basse à la plus haute.

| Clé | Défaut | Sens |
| --- | --- | --- |
| `chat.provider`, `chat.model` | aucun | Avec qui parler. `chat.model` peut être `provider/model`. |
| `chat.system` | aucun | Prompt système |
| `chat.max_tokens`, `chat.temperature` | aucun | Non fixé — laissé au fournisseur |
| `chat.stream` | `true` | Flux de la réponse facultative du juge ; les tours d'agent avec outils utilisent actuellement des réponses complètes, pour que les appels d'outils restent entiers |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Raisonnements et comptes de jetons sur stderr |
| `agent.max_turns` | `20` | Maximum d'allers-retours d'outils par requête d'utilisateur |
| `agent.persist_sessions` | `true` | Sauvegarder les conversations locales entre les lancements |
| `permissions.shell` | `ask` | Exécution de commandes de shell : `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Lecture de fichiers : `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Modification de fichiers : `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Téléchargement d'URL : `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Racines permises pour les outils de fichiers |
| `permissions.denied_paths` | aucun | Chemins de fichiers expressément bloqués |
| `network.connect_timeout_secs` | `15` | Temps permis pour se connecter |
| `network.idle_timeout_secs` | `300` | Silence permis en attendant des données |
| `network.proxy` | `system` | `system`, `none`, ou une adresse de proxy |
| `network.ca_bundle` | aucun | Un fichier supplémentaire d'autorités de certification de confiance |
| `network.allow_insecure_http` | `false` | Permettre une clé sur du `http` simple vers un hôte étranger |
| `update.check` | `true` | Demander à GitHub et GitLab une version plus récente au début d'une session interactive |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` auprès duquel la vérification demande la publication la plus récente |
| `update.gitlab_repo` | aucun | GitLab `owner/repo` pour la même vérification. Vide désactive GitLab |
| `update.timeout_secs` | `3` | Combien de secondes la vérification peut dépenser par source |

Marelle porte sa propre liste d'autorités de certification de confiance, donc il marche aussi là où le système n'en a pas, comme dans Termux. `SSL_CERT_FILE` en ajoute.

Quand une version plus récente est trouvée, la session la mentionne une fois — la version, celle que vous avez, et où la trouver. Une publication est demandée d'abord ; un dépôt qui ne fait que taguer ses commits renvoie le tag le plus récent. Tout échec reste silencieux.

## Vos clés restent les vôtres

- Les réglages nomment une variable ou un fichier qui garde la clé, jamais la clé elle-même. Un réglage tel que `providers.x.api_key` est refusé.
- La clé n'est pas imprimée par `config show`, `config explain` ni `providers`, et elle est retirée de tout texte d'erreur revenant d'un serveur.
- Elle ne part que vers l'adresse du fournisseur, dans un seul en-tête, et jamais sur du `http` simple vers une autre machine, sauf si vous le permettez.
- Les redirections ne sont pas suivies, si bien qu'une clé ne peut être relayée ailleurs.

## Langues

L'interface parle anglais, russe, français et ukrainien. Choisissez avec `--lang <code>` ou `MARELLE_LANG` ; sinon `LC_ALL`, `LC_MESSAGES` et `LANG` décident. Chaque message vit dans un catalogue, et un test échoue si une langue en laisse passer un. L'objectif : chaque langue d'Eurasie, puis davantage.

## Feuille de route

1. **Parler.** Adaptateurs compatibles OpenAI et Anthropic, avec flux SSE et décodage de réponses complètes. Les tours d'agent avec outils utilisent actuellement des réponses complètes. *Fait.*
2. **Réglages et allure.** Fichiers de réglages TOML, globaux et par projet, avec `config set`, `get` et `unset` et une description pour chaque clé. Une interface thématisée dans l'esprit de [Hermes Agent](https://github.com/NousResearch/hermes-agent) : bandeau, panneau d'information, réponses encadrées, spinner, en violet glycine par défaut, toute couleur réglable en HEX. *Fait.*
3. **Plusieurs modèles.** Alias de modèles, question simultanée de plusieurs modèles et comparaison des réponses, chaînes de repli, et un modèle juge qui les fond ensemble. *Fait.*
4. **Agir.** Appels d'outils structurés pour les fournisseurs compatibles OpenAI et Anthropic, outils locaux de fichiers, de recherche et de terminal, téléchargement d'URL, compétences locales et permissions allow/ask/deny ; le transcript montre chaque appel d'outil et son résultat. *Fait.*
5. **Se souvenir.** Les conversations nommées persistent localement entre les lancements. La mémoire sémantique de long terme est encore devant.
6. **Étendre.** Serveurs MCP, greffons externes, sous-agents et une prise en charge plus riche des compétences.
7. **Atteindre.** Automatisation du navigateur, passerelles de messagerie et tâches planifiées.

Le raisonnement derrière la conception se trouve dans [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Limites connues

- Les sélecteurs de fournisseurs et de modèles listent les fournisseurs configurés et les alias de modèles. Marelle ne découvre pas encore de catalogues de modèles distants.
- Les sélecteurs soutiennent le clic de souris pour la mise au point et le défilement à la molette ; le transcript et l'éditeur d'entrée restent menés au clavier.
- Ctrl-C quitte la session interactive. Il n'annule pas encore uniquement la réponse en cours.
- La boucle d'agent, les outils locaux conscients des permissions et les sessions locales persistantes sont réalisés, mais Marelle reste un prototype pré-alpha ; pour la conception actuelle et ses limites, voir [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Développement

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Les tests ne touchent jamais le réseau : ils parlent à un serveur factice dans `crates/marelle-testkit`. GitHub Actions construit aussi les binaires Android et vérifie la version minimale de Rust déclarée. Pour un miroir GitLab, un pipeline GitLab CI est fourni ; voir [docs/GITLAB.md](docs/GITLAB.md).

## Licence

[AGPL-3.0-only](LICENSE). Les mentions de tiers sont dans [NOTICE](NOTICE). Les archives de publication incluent `THIRD-PARTY-LICENSES.txt` pour les caisses Rust à l'intérieur du binaire.
