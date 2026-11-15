```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigurabilni AI agent za vaš terminal.**
Donesite vlastite API ključeve: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokalnu Ollamu ili bilo koji poslužitelj kompatibilan s OpenAI. Prvenstveno Linux i Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · Hrvatski · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, verzija 0.1.0-alpha.1.** Implementirano: izvorno sučelje TUI, čet s više pružatelja, petlja agenta svjesna ovlasti, lokalni alati i obnovivi lokalne sesije. API-ji i postavke mogu se još mijenjati. Ovo je prototip, a ne paritet funkcija s OpenCode ili Hermes.

*Marelle* je francuski naziv dječje igre školica. Svaki kvadratić je korak.

## Instalacija

**Termux na Androidu.** Preuzmite gotovu binarnu datoteku sa stranice [izdanja](https://github.com/S-I-L-S/marelle/releases) ili je izgradite na telefonu. Vodič korak po korak: [docs/TERMUX.md](docs/TERMUX.md).

**Linux i svugdje gdje radi Rust.** Treban je Rust 1.85 ili noviji:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ili klonirajte repozitorij i pokrenite `cargo build --release`. Binarnika je `target/release/marelle`, otprilike 4 MiB.

## Prvi čet

Čuvajte ključ u datoteci koju čitate samo vi, i recite Marelle gdje je. Marelle nikada ne sprema ključ u svoje postavke.

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

Zamijenite `MODEL` nazivom modela s popisa vašeg pružatelja. Drugi načini upotrebe:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Odgovor ide na stdout. Razmišljanja, broj tokena, napomene i pogreške idu na stderr, tako da `marelle chat ... > answer.txt` sprema samo odgovor.

## Alati agenta

Marelle šalje strukturirane definicije alata preko API-ja kompatibilnih s OpenAI i Anthropic. Ugrađeni skup alata:

| Alat | Svrha |
| --- | --- |
| `terminal` | Pokretanje naredbe školjke iz direktorija pokretanja |
| `read_file`, `write_file`, `edit_file` | Čitanje i mijenjanje tekstovnih datoteka UTF-8 |
| `list_directory`, `search_text` | Pregled i pretraživanje datoteka projekta |
| `fetch_url` | Preuzimanje javne stranice HTTP(S) kao teksta (treba `curl`) |
| `list_skills`, `load_skill` | Otkrivanje i učitavanje lokalnih uputa `SKILL.md` |

Naredba `/tools` prikazuje popis alata, a `/skills` dostupne vještine. Upute projekta učitavaju se iz `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` i odabranih datoteka `.marelle/`, ako su u dopuštenim putanjama.

Alati za datoteke ograničeni su na `permissions.allowed_paths` (zadano: trenutačni direktorij projekta) i poštuju `permissions.denied_paths`. Čitanje je zadano dopušteno; mijenjanje datoteka, naredbe školjke i mrežna preuzimanja prvo pitaju. Ovlast za školjku odnosi se na naredbe na računalo-domaćinu i nije pijesak sustava datoteka: držite ga na `ask`, osim ako namjerno ne povjerite modelu izvršavanje naredbi.

Agent se zaustavlja nakon `agent.max_turns` razmjena s alatima. Odgovori alata smatraju se nepouzdanom sadržajem. Zahtjevi koriste cele odgovore modela, tako da argumenti alata ne sklapaju se iz nepotpunih djelova toka.

## Sesije

Razgovori se zadano spremaju lokalno, uključujući povijest poziva alata i postavke modela i sesije. Na Unixu direktorij sesija ima način `0700`, a JSON datoteka `0600`. Datoteka se čuva u `$XDG_STATE_HOME/marelle/sessions.json`, ili `~/.local/state/marelle/sessions.json` kada `XDG_STATE_HOME` nije postavljeno. API ključevi se nikada ne upisuju u tu datoteku. Isključite trajnost s:

```sh
marelle config set agent.persist_sessions false
```

Kodovi izlaza: `0` uspjeh, `1` pogreška pružatelja ili mreže, `2` pogreška upotrebe ili postavki.

## Pružatelji

| id | Protokol | Varijabla ključa | |
| --- | --- | --- | --- |
| `zai` | kompatibilan s OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), međunarodni |
| `zai_coding` | kompatibilan s OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, završetak kompatibilan s Anthropic |
| `zhipu` | kompatibilan s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kina |
| `zhipu_coding` | kompatibilan s OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | kompatibilan s OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | kompatibilan s OpenAI | `OPENROUTER_API_KEY` | mnogo modela iza jednog ključa |
| `ollama` | kompatibilan s OpenAI | nema | Ollama na ovom stroju |

Završeci Z.ai i Zhipu dolaze iz dokumentacije trećih strana, ne sa stranica samoga pružatelja. Provjerite ih prema svom planu i prepišite bilo koje polje ako se razlikuju.

Svako polje pružatelja može se mijenjati, a novi pružatelj je jednostavno ime s adresom:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Ostala polja pružatelja: `kind` (`openai-compatible` ili `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` i `extra_body`. Ako je varijabla imenovana u `api_key_env` postavljena, ona pobjeđuje; inače se čita datoteka.

## Postavke

Postavke dolaze u slojevima: ugrađene zadane vrijednosti, zatim varijable okoliša, zatim zastavice. Datoteke postavki staju se između zadanih vrijednosti i okoliša.

- `MARELLE_CHAT__MODEL=zai/MODEL` postavlja `chat.model`. Dvostruko donje crtanje stoji za točku.
- `marelle config show [prefix]` ispisuje postavke i sloj iz kojeg je svako došlo.
- `marelle config explain <key>` prikazuje svaki sloj koji je postavio ključ, od najnižeg prioriteta do najvišeg.

| Ključ | Zadano | Smisao |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ništa | S kim razgovarati. `chat.model` može biti `provider/model`. |
| `chat.system` | ništa | Sistemski prompt |
| `chat.max_tokens`, `chat.temperature` | ništa | Nije postavljeno — ostavljeno pružatelju |
| `chat.stream` | `true` | Tok odgovora suca kad je uključen; potezi agenta s alatima trenutno koriste cele odgovore, da pozivi alata ostaju cijeli |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Razmišljanja i broj tokena na stderr |
| `agent.max_turns` | `20` | Maksimum razmjena s alatima na jedan zahtjev korisnika |
| `agent.persist_sessions` | `true` | Spremanje lokalnih razgovora između pokretanja |
| `permissions.shell` | `ask` | Izvršavanje naredbi školjke: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Čitanje datoteka: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Mijenjanje datoteka: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Preuzimanje URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Dopušteni korijeni za alate za datoteke |
| `permissions.denied_paths` | ništa | Izričito blokirane putanje datoteka |
| `network.connect_timeout_secs` | `15` | Vrijeme dopušteno za vezu |
| `network.idle_timeout_secs` | `300` | Dopuštena šutnja pri čekanju podataka |
| `network.proxy` | `system` | `system`, `none` ili adresa posrednika |
| `network.ca_bundle` | ništa | Dopunjska datoteka pouzdanih certifikacijskih tijela |
| `network.allow_insecure_http` | `false` | Dopustiti ključ preko običnog `http` na tuđ domaćin |
| `update.check` | `true` | Pitati GitHub i GitLab za noviju verziju pri početku interaktivne sesije |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` od kojega provjera pita za najnovije izdanje |
| `update.gitlab_repo` | ništa | GitLab `owner/repo` za istu provjeru. Prazno isključuje GitLab |
| `update.timeout_secs` | `3` | Koliko sekundi provjera može potrošiti na jedan izvor |

Marelle nosi vlastit popis pouzdanih certifikacijskih tijela, tako da radi i tamo gdje ih sustav nema, kao u Termuxu. `SSL_CERT_FILE` dodaje još.

Kad se nađe novija verzija, sesija će je spomenuti jednom — verziju, ono što imate i gdje je uzeti. Prvo se pita izdanje; repozitorij koji samo tagira svoje commite daje najnoviji tag. Što god zakazala, ostaje tiho.

## Vaši ključevi ostaju vaši

- Postavke imenuju varijablu ili datoteku gdje leži ključ, nikada sam ključ. Postavka kao `providers.x.api_key` se odbija.
- Ključ se ne ispisuje u `config show`, `config explain` ili `providers` i se iščupava iz svakog teksta pogreške od poslužitelja.
- On se šalje samo na adresu pružatelja, u jednom zaglavlju, i nikada preko običnog `http` na drugo računalo, ako to ne dopustite.
- Preusmjeravanja se ne prate, tako da ključ ne može poći negdje drugdje.

## Jezici

Sučelje govori engleski, ruski, francuski i ukrajinski. Birajte preko `--lang <code>` ili `MARELLE_LANG`; inače odlučuju `LC_ALL`, `LC_MESSAGES` i `LANG`. Svaka poruka živi u katalogu, i test pada ako neki jezik propusti jednu. Cilj su svi jezici Euroazije, a onda još.

## Putna karta

1. **Pričati.** Adapteri kompatibilni s OpenAI i Anthropic, s SSE-tokom i razbirom celih odgovora. Potezi agenta s alatima trenutno koriste cele odgovore. *Gotovo.*
2. **Postavke i izgled.** TOML datoteke postavki, globalne i po projektu, s `config set`, `get` i `unset` i opisom svakog ključa. Tematizirano sučelje u duhu [Hermes Agent](https://github.com/NousResearch/hermes-agent): baner, informativna ploča, odgovori u okviru, vrtuljak, zadano u ljubičastoj glicinije, s bilo kojom bojom kao HEX. *Gotovo.*
3. **Mnogo modela.** Aliasi modela, istovremeno pitanje nekoliko modela i usporedba odgovora, lanci rezerve i model-sudac koji ih sažima. *Gotovo.*
4. **Djelovati.** Strukturirani pozivi alata za pružatelje kompatibilne s OpenAI i Anthropic, lokalni alati za datoteke, pretraživanje i terminál, preuzimanje URL, lokalne vještine i ovlasti allow/ask/deny; transkript prikazuje svaki poziv alata i njegov rezultat. *Gotovo.*
5. **Sjećati se.** Imenovani razgovori traju lokalno između pokretanja. Dugotrajna semantička memorija još je pred nama.
6. **Proširivati.** MCP poslužitelji, vanjski dodaci, podagenti i bogatija podrška vještina.
7. **Dosezati.** Automatizacija preglednika, prolazi glasnika i zakazani poslovi.

Razmišljanje iza dizajna je u [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Poznata ograničenja

- Biraci pružatelja i modela ispisuju postavljene pružatelje i aliase modela. Marelle još ne otkriva udaljene kataloge modela.
- Biraci podržavaju klik mišom za fokus i kotrljanje točkića; transkript i uređivač unosa ostaju vođeni tipkovnicom.
- Ctrl-C izlazi iz interaktivne sesije. Još ne prekida samo tekući odgovor.
- Petlja agenta, lokalni alati svjesni ovlasti i trajne lokalne sesije provedene su, ali Marelle ostaje pre-alpha prototip; o sadašnjem dizajnu i granicama čitajte [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Razvoj

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Probе nikada ne diraju mrežu: govore s maketom poslužitelja u `crates/marelle-testkit`. GitHub Actions također gradi Android binarne datoteke i provjerava deklariranu minimalnu verziju Rusta. Za ogledalo na GitLabu postoji cjevovod GitLab CI; vidjeti [docs/GITLAB.md](docs/GITLAB.md).

## Licenca

[AGPL-3.0-only](LICENSE). Obavijesti trećih strana su u [NOTICE](NOTICE). Arhivi izdanja uključuju `THIRD-PARTY-LICENSES.txt` za Rust sandučiće unutar binarnika.
