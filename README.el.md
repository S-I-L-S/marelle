```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Παραγώγιμος πράκτορας τεχνητής νοημοσύνης για το τερματικό σας.**
Φέρτε τα δικά σας κλειδιά API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, τοπικό Ollama, ή οποιονδήποτε διακοβιστή συμβατό με OpenAI. Πρώτα Linux και Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · Ελληνικά · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Προ-άλφα, έκδοση 0.1.0-alpha.1.** Υλοποιημένα: γηγενές περιβάλλον TUI, συζήτηση με πολλούς παρόχους, βρόχος πράκτορα ενήμερος για τα δικαιώματα, τοπικά εργαλεία και επαναφέρεται-τοπικές συνεδρίες. Τα API και οι ρυθμίσεις μπορούν ακόμη να αλλάξουν. Αυτό είναι ένα πρωτότυπο, και όχι αντιστοιχία λειτουργιών με το OpenCode ή το Hermes.

*Marelle* είναι η γαλλική ονομασία του παιδικού παιχνιδιού με τα άλματα σε τετράγωνα. Κάθε τετράγωνο είναι ένα βήμα.

## Εγκατάσταση

**Termux σε Android.** Κατεβάστε έτοιμο εκτελέσιμο από τη σελίδα των [κυκλοφοριών](https://github.com/S-I-L-S/marelle/releases), ή κτίστε το στο τηλέφωνο. Οδηγίες βήμα-βήμα: [docs/TERMUX.md](docs/TERMUX.md).

**Linux και οπουδήποτε τρέχει Rust.** Χρειάζεται Rust 1.85 ή νεότερο:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ή κλωνοποιήστε το αποθετήριο και εκτελέστε `cargo build --release`. Το εκτελέσιμο είναι το `target/release/marelle`, περίπου 4 MiB.

## Πρώτη συνομιλία

Κρατήστε το κλειδί σε αρχείο που διαβάζει μόνο εσείς, και πείτε στη Marelle πού βρίσκεται. Η Marelle δεν αποθηκεύει ποτέ κλειδί στις ρυθμίσεις της.

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

Αντικαταστήστε το `MODEL` με όνομα μοντέλου από τον κατάλογο του παρόχου σας. Άλλοι τρόποι χρήσης:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Η απάντηση πάει στο stdout. Οι συλλογισμοί, οι μετρήσεις τουκεν, οι σημειώσεις και τα σφάλματα πάουν στο stderr, έτσι το `marelle chat ... > answer.txt` αποθηκεύει μόνο την απάντηση.

## Εργαλεία του πράκτορα

Η Marelle στέλνει δομημένες ορισμούς εργαλείων μέσω API συμβατών με OpenAI και Anthropic. Το ενσωματωμένο σετ εργαλείων:

| Εργαλείο | Σκοπός |
| --- | --- |
| `terminal` | Εκτέλεση εντολής κελύφους από τον φάκελο εκκίνησης |
| `read_file`, `write_file`, `edit_file` | Ανάγνωση και τροποποίηση αρχείων κειμένου UTF-8 |
| `list_directory`, `search_text` | Επισκόπηση και αναζήτηση στα αρχεία του έργου |
| `fetch_url` | Λήψη δημόσιας σελίδας HTTP(S) ως κείμενο (απαιτεί `curl`) |
| `list_skills`, `load_skill` | Ανακάλυψη και φόρτωση τοπικών οδηγιών `SKILL.md` |

Η εντολή `/tools` δείχνει τη λίστα εργαλείων και η `/skills` τις διαθέσιμες δεξιότητες. Οι οδηγίες του έργου φορτώνονται από `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` και επιλεγμένα αρχεία `.marelle/`, όταν βρίσκονται μέσα σε επιτρεπόμενες διαδρομές.

Τα εργαλεία αρχείων περιέχονται μέσα στο `permissions.allowed_paths` (από προεπιλογή: ο τρέχων φάκελος του έργου) και σέβονται το `permissions.denied_paths`. Η ανάγνωση επιτρέπεται από προεπιλογή· οι αλλαγές αρχείων, οι εντολές κελύφους και οι λήψεις δικτύου ρωτούν πρώτα. Η άδεια κελύφους ισχύει για εντολές στον οικοδεσπότη και δεν είναι άμμος του συστήματος αρχείων: κρατήστε την σε `ask`, εκτός αν εμπιστεύεστε σκόπιμα το μοντέλο την εκτέλεση εντολών.

Ο πράκτορας σταματά μετά από `agent.max_turns` γύρους με εργαλεία. Οι απαντήσεις των εργαλείων θεωρούνται μη αξιόπιστο περιεχόμενο. Τα αιτήματα χρησιμοποιούν πλήρεις απαντήσεις του μοντέλου, ώστε τα ορίσματα των εργαλείων να μη συναρμολογούνται από ατελή θραύσματα ροής.

## Συνεδρίες

Οι συνομιλίες αποθηκεύονται τοπικά από προεπιλογή, μαζί με το ιστορικό κλήσεων εργαλείων και τις ρυθμίσεις μοντέλου και συνεδρίας. Στο Unix ο φάκελος συνεδριών έχει λειτουργία `0700` και το αρχείο JSON λειτουργία `0600`. Το αρχείο αποθηκεύεται στο `$XDG_STATE_HOME/marelle/sessions.json`, ή στο `~/.local/state/marelle/sessions.json` όταν το `XDG_STATE_HOME` δεν έχει οριστεί. Κλειδιά API δεν γράφονται ποτέ σε αυτό το αρχείο. Απενεργοποιήστε την αποθήκευση με:

```sh
marelle config set agent.persist_sessions false
```

Κωδικοί εξόδου: `0` επιτυχία, `1` σφάλμα παρόχου ή δικτύου, `2` σφάλμα χρήσης ή ρύθμισης.

## Πάροχοι

| id | Πρωτόκολλο | Μεταβλητή κλειδιού | |
| --- | --- | --- | --- |
| `zai` | συμβατό με OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), διεθνής |
| `zai_coding` | συμβατό με OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, τελικός σταθμός συμβατός με Anthropic |
| `zhipu` | συμβατό με OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Κίνα |
| `zhipu_coding` | συμβατό με OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Κίνα |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | συμβατό με OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | συμβατό με OpenAI | `OPENROUTER_API_KEY` | πολλά μοντέλα πίσω από ένα κλειδί |
| `ollama` | συμβατό με OpenAI | κανένα | Ollama σε αυτή τη μηχανή |

Οι τελικοί σταθμοί των Z.ai και Zhipu προέρχονται από τεκμηρίωση τρίτων, όχι από τις σελίδες του ίδιου του πάροχου. Ελέγξτε τους σε σχέση με το πρόγραμμά σας και υπερισχύστε οποιοδήποτε πεδίο αν διαφέρουν.

Κάθε πεδίο ενός παρόχου είναι μεταβλητό, και ένας νέος πάροχος είναι απλώς όνομα με διεύθυνση:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Άλλα πεδία παρόχου: `kind` (`openai-compatible` ή `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` και `extra_body`. Αν η μεταβλητή που ονομάζεται από το `api_key_env` έχει οριστεί, εκείνη νικά· αλλιώς διαβάζεται το αρχείο.

## Ρυθμίσεις

Οι ρυθμίσεις έρχονται σε στρώματα: ενσωματωμένες προεπιλεγμένες τιμές, μετά μεταβλητές περιβάλλοντος, μετά σημαίες. Τα αρχεία ρυθμίσεων τοποθετούνται μεταξύ των προεπιλογών και του περιβάλλοντος.

- `MARELLE_CHAT__MODEL=zai/MODEL` θέτει το `chat.model`. Διπλή κάτω παύλα αντιπροσωπεύει τελεία.
- `marelle config show [prefix]` απαριθμεί τις ρυθμίσεις και το στρώμα από το οποίο ήρθε καθεμία.
- `marelle config explain <key>` δείχνει κάθε στρώμα που θέτει κλειδί, από τη χαμηλότερη προτεραιότητα στην υψηλότερη.

| Κλειδί | Προεπιλογή | Νόημα |
| --- | --- | --- |
| `chat.provider`, `chat.model` | κανένα | Με ποιον να μιλήσετε. Το `chat.model` μπορεί να είναι `provider/model`. |
| `chat.system` | κανένα | Προτροπή συστήματος |
| `chat.max_tokens`, `chat.temperature` | κανένα | Μη ορισμένο — αφήνεται στον πάροχο |
| `chat.stream` | `true` | Ροή της προαιρετικής απάντησης του κριτή· οι γύροι πράκτορα με εργαλεία χρησιμοποιούν τώρα πλήρεις απαντήσεις, ώστε οι κλήσεις εργαλείων να μένουν άθικτες |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Συλλογισμοί και μετρήσεις τουκεν στο stderr |
| `agent.max_turns` | `20` | Μέγιστο γύρων εργαλείων ανά αίτημα χρήστη |
| `agent.persist_sessions` | `true` | Αποθήκευση τοπικών συνομιλιών μεταξύ εκτελέσεων |
| `permissions.shell` | `ask` | Εκτέλεση εντολών κελύφους: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Ανάγνωση αρχείων: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Τροποποίηση αρχείων: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Λήψη URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Επιτρεπόμενες ρίζες για τα εργαλεία αρχείων |
| `permissions.denied_paths` | κανένα | Ρητά φραγμένες διαδρομές αρχείων |
| `network.connect_timeout_secs` | `15` | Χρόνος επιτρεπτός για σύνδεση |
| `network.idle_timeout_secs` | `300` | Επιτρεπτή σιωπή κατά την αναμονή δεδομένων |
| `network.proxy` | `system` | `system`, `none`, ή διεύθυνση μεσολαβητή |
| `network.ca_bundle` | κανένα | Πρόσθετο αρχείο έμπιστων αρχών πιστοποίησης |
| `network.allow_insecure_http` | `false` | Επιτρέψτε κλειδί πάνω από γυμνό `http` προς ξένο υπολογιστή |
| `update.check` | `true` | Ρωτήστε GitHub και GitLab για νεότερη έκδοση στην αρχή μιας διαδραστικής συνεδρίας |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` από τον οποίο ο έλεγχος ρωτά την νεότερη κυκλοφορία |
| `update.gitlab_repo` | κανένα | GitLab `owner/repo` για τον ίδιο έλεγχο. Κενό σβήνει το GitLab |
| `update.timeout_secs` | `3` | Πόσες δευτερόλεπτες μπορεί να ξοδέψει ο έλεγχος σε μία πηγή |

Η Marelle κουβαλά τον δικό της κατάλογο έμπιστων αρχών πιστοποίησης, γι' αυτό δουλεύει και εκεί όπου το σύστημα δεν έχει, όπως στο Termux. Το `SSL_CERT_FILE` προσθέτει κι άλλα.

Όταν βρίσκεται νεότερη έκδοση, η συνεδρία το αναφέρει μια φορά — την έκδοση, αυτή που έχετε, και πού θα την πάρετε. Ρωτιέται πρώτα η κυκλοφορία· αποθετήριο που σηματοδοτεί τα commits του μόνο με ετικέτες δίνει την νεότερη ετικέτα. Ό,τι αποτύχει μένει σιωπηλό.

## Τα κλειδιά σας μένουν δικά σας

- Οι ρυθμίσεις ονομάζουν μεταβλητή ή αρχείο όπου κείτεται το κλειδί, ποτέ το ίδιο το κλειδί. Ρύθμιση όπως `providers.x.api_key` απορρίπτεται.
- Το κλειδί δεν το τυπώνουν το `config show`, το `config explain` ή το `providers`, και κόβεται από κάθε κείμενο σφάλματος που έρχεται από διακοβιστή.
- Πάει μόνο στη διεύθυνση του παρόχου, σε μία μόνο επικεφαλίδα, και ποτέ πάνω από γυμνό `http` προς άλλη μηχανή, αν δεν το επιτρέψετε.
- Οι ανακατευθύνσεις δεν ακολουθούνται, έτσι το κλειδί δεν μπορεί να πάει αλλού.

## Γλώσσες

Το περιβάλλον μιλά αγγλικά, ρωσικά, γαλλικά και ουκρανικά. Επιλέξτε με `--lang <code>` ή `MARELLE_LANG`· αλλιώς αποφασίζουν τα `LC_ALL`, `LC_MESSAGES` και `LANG`. Κάθε μήνυμα ζει σε κατάλογο, και ένας έλεγχος πέφτει αν κάποια γλώσσα αφήσει ένα έξω. Ο στόχος: κάθε γλώσσα της Ευρασίας, και μετά περισσότερα.

## Χάρτης πορείας

1. **Μιλάει.** Προσαρμογείς συμβατοί με OpenAI και Anthropic, με ροή SSE και ανάγνωση πλήρων απαντήσεων. Οι γύροι πράκτορα με εργαλεία χρησιμοποιούν τώρα πλήρεις απαντήσεις. *Έτοιμο.*
2. **Ρυθμίσεις και όψη.** Αρχεία ρυθμίσεων TOML, συνολικά και ανά έργο, με `config set`, `get` και `unset` και περιγραφή για κάθε κλειδί. Θεματοποιημένο περιβάλλον στο πνεύμα του [Hermes Agent](https://github.com/NousResearch/hermes-agent): πανό, πίνακας πληροφοριών, απαντήσεις σε πλαίσιο, περιστρεφόμενος δείκτης, από προεπιλογή στο βιολετί της γλυτσίνιας, κάθε χρώμα ρυθμίσιμο ως HEX. *Έτοιμο.*
3. **Πολλά μοντέλα.** Επώνυμα μοντέλα, ταυτόχρονη ερώτηση πολλών μοντέλων και σύγκριση απαντήσεων, αλυσίδες εφεδρείας, και μοντέλο κριτής που τα λιώνει μαζί. *Έτοιμο.*
4. **Δρα.** Δομημένες κλήσεις εργαλείων για παρόχους συμβατούς με OpenAI και Anthropic, τοπικά εργαλεία αρχείων, αναζήτησης και τερματικού, λήψη URL, τοπικές δεξιότητες και άδειες allow/ask/deny· η απομαγνητοφωνία δείχνει κάθε κλήση εργαλείου και το αποτέλεσμα της. *Έτοιμο.*
5. **Θυμάται.** Ονομασμένες συνομιλίες αντέχουν τοπικά μεταξύ εκτελέσεων. Η μακροπρόθεσμη σημασιολογική μνήμη είναι ακόμη μπροστά.
6. **Επεκτείνει.** Διακοβιστές MCP, εξωτερικά πρόσθετα, υποπράκτορες και πλουσιότερη υποστήριξη δεξιοτήτων.
7. **Φτάνει.** Αυτοματοποίηση περιηγητή, πύλες ανταλλαγής μηνυμάτων και προγραμματισμένες εργασίες.

Ο συλλογισμός πίσω από τον σχεδιασμό είναι στο [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Γνωστά όρια

- Οι επιλογείς παρόχων και μοντέλων απαριθμούν ρυθμισμένους παρόχους και ψευδώνυμα μοντέλων. Η Marelle δεν ανακαλύπτει ακόμη απομακρυσμένους καταλόγους μοντέλων.
- Οι επιλογείς υποστηρίζουν κλικ ποντικιού για εστίαση και κύλιση με τροχό· η απομαγνητοφωνία και ο συντάκτης εισόδου μένουν οδηγούμενα από πληκτρολόγιο.
- Το Ctrl-C βγαίνει από τη διαδραστική συνεδρία. Δεν ακυρώνει ακόμη μόνο την τρέχουσα απάντηση.
- Ο βρόχος πράκτορα, τα τοπικά εργαλεία ενήμερα για άδειες και οι διαρκείς τοπικές συνεδρίες υλοποιημένα· αλλά η Marelle μένει πρωτότυπο προ-άλφα. Για τον τρέχοντα σχεδιασμό και τα όρια διαβάστε [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Ανάπτυξη

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Οι δοκιμές δεν αγγίζουν ποτέ το δίκτυο: μιλούν σε δοκιμαστικό διακοβιστή στο `crates/marelle-testkit`. Το GitHub Actions κτίζει και τα εκτελέσιμα Android και ελέγχει τη δηλωμένη ελάχιστη έκδοση Rust. Για καθρέφτη στο GitLab υπάρχει αγωγός GitLab CI· δείτε [docs/GITLAB.md](docs/GITLAB.md).

## Άδεια

[AGPL-3.0-only](LICENSE). Οι ανακοινώσεις τρίτων είναι στο [NOTICE](NOTICE). Τα αρχεία κυκλοφοριών περιλαμβάνουν `THIRD-PARTY-LICENSES.txt` για τα κιβώτια Rust μέσα στο εκτελέσιμο.
