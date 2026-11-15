```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Ein konfigurierbarer KI-Agent für dein Terminal.**
Bring deine eigenen API-Schlüssel mit: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, ein lokales Ollama oder jeder beliebige OpenAI-kompatible Server. Zuerst Linux und Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · Deutsch · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-Alpha, Version 0.1.0-alpha.1.** Implementiert: einheimische TUI-Oberfläche, Chat mit mehreren Anbietern, eine bewilligungsbewusste Agentenschleife, lokale Werkzeuge und wiederaufnehmbare lokale Sitzungen. APIs und Einstellungen können sich noch ändern. Dies ist ein Prototyp, keine Funktionsparität mit OpenCode oder Hermes.

*Marelle* ist das französische Wort für „Himmel und Hölle". Jedes Feld ist ein Schritt.

## Installation

**Termux auf Android.** Lade eine fertige Binärdatei von der [Veröffentlichungsseite](https://github.com/S-I-L-S/marelle/releases) herunter oder baue sie auf dem Handy. Schritt für Schritt: [docs/TERMUX.md](docs/TERMUX.md).

**Linux und überall dort, wo Rust läuft.** Rust 1.85 oder neuer:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Oder klone das Repository und führe `cargo build --release` aus. Die Binärdatei ist `target/release/marelle`, rund 4 MiB.

## Erster Chat

Bewahre den Schlüssel in einer Datei auf, die nur du liest, und sage Marelle, wo er liegt. Marelle speichert nie einen Schlüssel in seinen Einstellungen.

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

Ersetze `MODEL` durch einen Modellnamen aus der Liste deines Anbieters. Weitere Verwendungsarten:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Die Antwort geht auf stdout. Überlegungen, Tokenzählungen, Notizen und Fehler gehen auf stderr, sodass `marelle chat ... > answer.txt` nur die Antwort speichert.

## Werkzeuge des Agenten

Marelle sendet strukturierte Werkzeugdefinitionen über OpenAI-kompatible und Anthropic-APIs. Der eingebaute Werkzeugsatz:

| Werkzeug | Zweck |
| --- | --- |
| `terminal` | Einen Shell-Befehl aus dem Startverzeichnis ausführen |
| `read_file`, `write_file`, `edit_file` | UTF-8-Textdateien lesen und ändern |
| `list_directory`, `search_text` | Projektdateien ansehen und durchsuchen |
| `fetch_url` | Eine öffentliche HTTP(S)-Seite als Text holen (braucht `curl`) |
| `list_skills`, `load_skill` | Lokale `SKILL.md`-Anweisungen finden und laden |

Mit `/tools` siehst du die Werkzeugliste und mit `/skills` die verfügbaren Fertigkeiten. Projektleitung wird aus `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` und ausgewählten `.marelle/`-Dateien geladen, wenn sie innerhalb erlaubter Pfade liegen.

Dateiwerkzeuge sind auf `permissions.allowed_paths` beschränkt (Vorgabe: das aktuelle Projektverzeichnis) und beachten `permissions.denied_paths`. Lesen ist standardmäßig erlaubt; Dateiänderungen, Shell-Befehle und Netzabrufe fragen vorher. Die Shell-Bewilligung gilt für Befehle auf dem Gastgeber und ist keine Dateisystem-Sandbox: halte sie auf `ask`, außer du traust dem Modell die Befehlsausführung bewusst zu.

Der Agent hält nach `agent.max_turns` Werkzeugrunden an. Werkzeugantworten gelten als nicht vertrauenswürdiger Inhalt. Anfragen verwenden vollständige Modellantworten, sodass Werkzeugargumente nicht aus unvollständigen Strombruchstücken zusammengesetzt werden.

## Sitzungen

Unterhaltungen werden standardmäßig lokal gespeichert, einschließlich Werkzeugaufrufhistorie und Modell- und Sitzungseinstellungen. Auf Unix hat der Sitzungsordner Modus `0700` und die JSON-Datei Modus `0600`. Die Datei liegt in `$XDG_STATE_HOME/marelle/sessions.json`, oder `~/.local/state/marelle/sessions.json`, wenn `XDG_STATE_HOME` nicht gesetzt ist. API-Schlüssel werden nie in diese Datei geschrieben. Beharrlichkeit ausschalten mit:

```sh
marelle config set agent.persist_sessions false
```

Ausgangscodes: `0` Erfolg, `1` Anbieter- oder Netzfehler, `2` Verwendungs- oder Konfigurationsfehler.

## Anbieter

| id | Protokoll | Schlüsselvariable | |
| --- | --- | --- | --- |
| `zai` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai (Zhipu GLM), international |
| `zai_coding` | OpenAI-kompatibel | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-kompatibler Endpunkt |
| `zhipu` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | OpenAI-kompatibel | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-kompatibel | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-kompatibel | `OPENROUTER_API_KEY` | viele Modelle hinter einem Schlüssel |
| `ollama` | OpenAI-kompatibel | keine | Ollama auf dieser Maschine |

Die Endpunkte von Z.ai und Zhipu stammen aus Fremddokumentation, nicht von den Seiten des Anbieters selbst. Prüfe sie gegen deinen Tarif und überschreibe jedes Feld, wenn sie abweichen.

Jedes Feld eines Anbieters ist änderbar, und ein neuer Anbieter ist einfach ein Name mit einer Adresse:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Weitere Anbieterfelder: `kind` (`openai-compatible` oder `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` und `extra_body`. Ist die von `api_key_env` benannte Variable gesetzt, gewinnt sie; sonst wird die Datei gelesen.

## Einstellungen

Einstellungen kommen in Schichten: eingebaute Vorgabewerte, dann Umgebungsvariablen, dann Schalter. Einstellungsdateien schieben sich zwischen die Vorgaben und die Umgebung.

- `MARELLE_CHAT__MODEL=zai/MODEL` setzt `chat.model`. Ein doppelter Unterstrich steht für einen Punkt.
- `marelle config show [prefix]` listet Einstellungen und die Schicht auf, aus der jede kam.
- `marelle config explain <key>` zeigt jede Schicht, die einen Schlüssel setzte, von niedrigster zu höchster Priorität.

| Schlüssel | Vorgabe | Bedeutung |
| --- | --- | --- |
| `chat.provider`, `chat.model` | keine | Mit wem sprechen. `chat.model` kann `provider/model` sein. |
| `chat.system` | keine | Systemprompt |
| `chat.max_tokens`, `chat.temperature` | keine | Ungesetzt — dem Anbieter überlassen |
| `chat.stream` | `true` | Streamen der optionalen Richterantwort; werkzeugführende Agentenzüge nutzen derzeit vollständige Antworten, damit Werkzeugaufrufe ganz bleiben |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Überlegungen und Tokenzählungen auf stderr |
| `agent.max_turns` | `20` | Höchstzahl an Werkzeugrunden pro Benutzeranfrage |
| `agent.persist_sessions` | `true` | Lokale Unterhaltungen zwischen Läufen speichern |
| `permissions.shell` | `ask` | Ausführung von Shell-Befehlen: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Dateilesen: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Dateiändern: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-Abholen: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Erlaubte Wurzeln für Dateiwerkzeuge |
| `permissions.denied_paths` | keine | Ausdrücklich gesperrte Dateipfade |
| `network.connect_timeout_secs` | `15` | Für die Verbindung erlaubte Zeit |
| `network.idle_timeout_secs` | `300` | Erlaubtes Schweigen beim Warten auf Daten |
| `network.proxy` | `system` | `system`, `none`, oder eine Proxy-Adresse |
| `network.ca_bundle` | keine | Eine Zusatzdatei vertrauenswürdiger Zertifizierungsstellen |
| `network.allow_insecure_http` | `false` | Einen Schlüssel über blosses `http` zu einem fremden Host erlauben |
| `update.check` | `true` | GitHub und GitLab nach einer neueren Version fragen, wenn eine interaktive Sitzung beginnt |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, von dem die Prüfung die neueste Veröffentlichung erfragt |
| `update.gitlab_repo` | keine | GitLab `owner/repo` für dieselbe Prüfung. Leer schaltet GitLab ab |
| `update.timeout_secs` | `3` | Wie viele Sekunden die Prüfung auf eine Quelle verwenden darf |

Marelle trägt eine eigene Liste vertrauenswürdiger Zertifizierungsstellen mit sich, darum funktioniert es auch dort, wo das System keine hat, wie in Termux. `SSL_CERT_FILE` fügt mehr hinzu.

Wird eine neuere Version gefunden, erwähnt die Sitzung sie einmal — die Version, was du hast, und wo man sie bekommt. Zuerst wird eine Veröffentlichung erfragt; ein Repository, das seine Commits nur taggt, liefert den neuesten Tag. Was auch immer scheitert, bleibt still.

## Deine Schlüssel bleiben deine

- Einstellungen benennen eine Variable oder eine Datei, die den Schlüssel hält, nie den Schlüssel selbst. Eine Einstellung wie `providers.x.api_key` wird abgelehnt.
- Den Schlüssel drucken `config show`, `config explain` oder `providers` nicht, und er wird aus jedem Fehlertext herausgeschnitten, der von einem Server zurückkommt.
- Er wird nur an die Adresse des Anbieters geschickt, in einer einzigen Kopfzeile, und nie über blosses `http` zu einer anderen Maschine, außer du erlaubst es.
- Umleitungen werden nicht gefolgt, sodass ein Schlüssel nicht woanders hingelangen kann.

## Sprachen

Die Oberfläche spricht Englisch, Russisch, Französisch und Ukrainisch. Wähle mit `--lang <code>` oder `MARELLE_LANG`; sonst entscheiden `LC_ALL`, `LC_MESSAGES` und `LANG`. Jede Nachricht lebt in einem Katalog, und ein Test fällt, wenn eine Sprache einen auslässt. Das Ziel ist jede Sprache Eurasiens und dann mehr.

## Fahrplan

1. **Sprechen.** OpenAI-kompatible und Anthropic-Adapter, mit SSE-Stromung und vollständiger Antwortdekodierung. Werkzeugführende Agentenzüge nutzen derzeit vollständige Antworten. *Fertig.*
2. **Einstellungen und Aussehen.** TOML-Einstellungsdateien, global und pro Projekt, mit `config set`, `get` und `unset` und einer Beschreibung für jeden Schlüssel. Eine thematisierte Oberfläche im Geist von [Hermes Agent](https://github.com/NousResearch/hermes-agent): Banner, Informationsfeld, gerahmte Antworten, Spinner, als Vorgabe in Glyzinien-Violett, mit jeder Farbe als HEX einstellbar. *Fertig.*
3. **Viele Modelle.** Modell-Aliasse, mehrere Modelle zugleich fragen und die Antworten vergleichen, Rückfallketten, und ein Richtermodell, das sie zusammenschmilzt. *Fertig.*
4. **Handeln.** Strukturierte Werkzeugaufrufe für OpenAI-kompatible und Anthropic-Anbieter, lokale Datei-, Such- und Terminalwerkzeuge, URL-Abholen, lokale Fertigkeiten und allow/ask/deny-Bewilligungen; das Protokoll zeigt jeden Werkzeugaufruf und sein Ergebnis. *Fertig.*
5. **Erinnern.** Benannte Unterhaltungen halten lokal zwischen Läufen. Langfristiges semantisches Gedächtnis liegt noch vor uns.
6. **Erweitern.** MCP-Server, äußere Plugins, Unteragenten und reichere Fertigkeitsunterstützung.
7. **Reichen.** Browserautomatisierung, Botengänge und geplante Aufgaben.

Das Nachdenken hinter dem Entwurf steht in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Bekannte Grenzen

- Anbieter- und Modellauswähler listen eingestellte Anbieter und Modell-Aliasse auf. Marelle entdeckt noch keine fernen Modellkataloge.
- Auswähler unterstützen Mausklick für Fokus und Rollen mit dem Rad; das Protokoll und der Eingabegeber bleiben tastaturgetrieben.
- Ctrl-C verlässt die interaktive Sitzung. Es bricht noch nicht nur die laufende Antwort ab.
- Agentenschleife, bewilligungsbewusste lokale Werkzeuge und beständige lokale Sitzungen sind verwirklicht, aber Marelle bleibt ein Pre-Alpha-Prototyp; für den gegenwärtigen Entwurf und die Grenzen lies [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Entwicklung

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Prüfungen berühren nie das Netz: sie sprechen mit einem Scheinserver in `crates/marelle-testkit`. GitHub Actions baut auch die Android-Binärdateien und prüft die erklärte kleinstmögliche Rust-Version. Für einen GitLab-Spiegel gibt es eine GitLab-CI-Leitung; siehe [docs/GITLAB.md](docs/GITLAB.md).

## Lizenz

[AGPL-3.0-only](LICENSE). Drittanbieter-Hinweise stehen in [NOTICE](NOTICE). Veröffentlichungsarchive enthalten `THIRD-PARTY-LICENSES.txt` für die Rust-Kisten in der Binärdatei.
