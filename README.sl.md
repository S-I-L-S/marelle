```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Nastavljivi UI-agent za vaš terminal.**
Prinesite svoje API-ključe: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokalno Ollamo ali katerikoli strežnik, združljiv z OpenAI. Najprej Linux in Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · Slovenščina · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, različica 0.1.0-alpha.1.** Izvedeno: lastni vmesnik TUI, pogovor z več ponudniki, zanka agenta, ki poziva dovoljenja, lokalna orodja in obnovljive lokalne seje. API-ji in nastavitve se lahko še spremenijo. To je prototip, ne pariteta funkcij z OpenCode ali Hermes.

*Marelle* je francosko poimenovanje otroške igre s skakanjem po kvadratkih. Vsak kvadratek je korak.

## Namestitev

**Termux na Androidu.** Prenesite pripravljen binarni program s strani [izdaj](https://github.com/S-I-L-S/marelle/releases) ali ga zgradite na telefonu. Vodnik korak za korakom: [docs/TERMUX.md](docs/TERMUX.md).

**Linux in povsod, kjer teče Rust.** Potreben je Rust 1.85 ali novejši:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ali klonirajte repozitorij in zaženite `cargo build --release`. Binarka je `target/release/marelle`, približno 4 MiB.

## Prvi pogovor

Ključ hranite v datoteki, ki jo bere samo vi, in povejte Marelle, kje je. Marelle si ključa nikoli ne shrani v svoje nastavitve.

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

Zamenjajte `MODEL` z imenom modela s seznama vašega ponudnika. Drugi načini uporabe:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Odgovor gre na stdout. Premišljanja, številke žetonov, opombe in napake grejo na stderr, tako da `marelle chat ... > answer.txt` shrani samo odgovor.

## Orodja agenta

Marelle pošilja strukturirane definicije orodij prek API-jev, združljivih z OpenAI, in Anthropic. Vgrajen nabor orodij:

| Orodje | Namen |
| --- | --- |
| `terminal` | Zagon ukaza lupine iz zagonske mape |
| `read_file`, `write_file`, `edit_file` | Branje in spreminjanje besedilnih datotek UTF-8 |
| `list_directory`, `search_text` | Pregled in iskanje po projektnih datotekah |
| `fetch_url` | Prenos javne strani HTTP(S) kot besedilo (potreben `curl`) |
| `list_skills`, `load_skill` | Odkrivanje in nalaganje krajevnih navodil `SKILL.md` |

Ukaz `/tools` pokaže seznam orodij, `/skills` pa razpoložljive veščine. Vodila projekta se nalagajo iz `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` in izbranih datotek `.marelle/`, če so v dovoljenih poteh.

Datotečna orodja so omejena na `permissions.allowed_paths` (privzeto: trenutna mapa projekta) in upoštevajo `permissions.denied_paths`. Branje je privzeto dovoljeno; spremembe datotek, ukazi lupine in omrežni prenosi vprašajo naprej. Dovoljenje za lupino se nanaša na ukaze na gostitelju in ni peskovnik datotečnega sistema: držite ga na `ask`, razen če modelu namenoma zaupate izvajanje ukazov.

Agent se ustavi po `agent.max_turns` izmenjah z orodji. Odgovore orodij se šteje za nezaupanja vredno vsebino. Prošnje uporabljajo popolne odgovore modela, tako da argumenti orodij ne nastajajo iz nedokončanih delkov toka.

## Seje

Pogovori se privzeto shranjujejo krajevno, skupaj z zgodovino klicev orodij in nastavitvami modela in seje. Na Unixu ima mapa sej način `0700`, datoteka JSON pa `0600`. Datoteka se hrani v `$XDG_STATE_HOME/marelle/sessions.json`, ali `~/.local/state/marelle/sessions.json`, kadar `XDG_STATE_HOME` ni nastavljeno. API-ključi se v to datoteko nikoli ne zapisujejo. Trajnost izklopite z:

```sh
marelle config set agent.persist_sessions false
```

Izhodne kode: `0` uspeh, `1` napaka ponudnika ali omrežja, `2` napaka uporabe ali nastavitev.

## Ponudniki

| id | Protokol | Spremenljivka ključa | |
| --- | --- | --- | --- |
| `zai` | združljiv z OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), mednarodni |
| `zai_coding` | združljiv z OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, končna točka združljiva z Anthropic |
| `zhipu` | združljiv z OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kitajska |
| `zhipu_coding` | združljiv z OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kitajska |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | združljiv z OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | združljiv z OpenAI | `OPENROUTER_API_KEY` | mnogi modeli za enim ključem |
| `ollama` | združljiv z OpenAI | ni | Ollama na tem računalniku |

Končne točke Z.ai in Zhipu prihajajo iz dokumentacije tretjih stran, ne s strani ponudnika samega. Preverite jih glede na svoj načrt in prepišite katerokoli polje, če se razlikujejo.

Vsako polje ponudnika je mogoče spremeniti, nov ponudnik pa je zgolj ime z naslovom:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Preostala polja ponudnika: `kind` (`openai-compatible` ali `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` in `extra_body`. Če je spremenljivka, imenovana v `api_key_env`, nastavljena, ta zmaga; sicer se bere datoteka.

## Nastavitve

Nastavitve prihajajo v plasteh: vgrajene privzete vrednosti, potem okoljske spremenljivke, potem stikala. Datoteke nastavitev vstopajo med privzete vrednosti in okolje.

- `MARELLE_CHAT__MODEL=zai/MODEL` nastavi `chat.model`. Dvojni podčrtaj stoji za piko.
- `marelle config show [prefix]` izpiše nastavitve in plast, iz katere je vsaka prišla.
- `marelle config explain <key>` pokaže vsako plast, ki je nastavila ključ, od najnižje prednosti do najvišje.

| Ključ | Privzeto | Pomen |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nič | S kom govoriti. `chat.model` je lahko `provider/model`. |
| `chat.system` | nič | Sistemski poziv |
| `chat.max_tokens`, `chat.temperature` | nič | Nenastavljeno — prepuščeno ponudniku |
| `chat.stream` | `true` | Pretakanje odgovora sodnika, kadar je vključen; potezi agenta z orodji trenutno uporabljata popolne odgovore, da klici orodij ostanejo cel |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Premišljanja in število žetonov na stderr |
| `agent.max_turns` | `20` | Največ izmenj z orodji na eno prošnjo uporabnika |
| `agent.persist_sessions` | `true` | Shranjevanje krajevnih pogovorov med zagoni |
| `permissions.shell` | `ask` | Izvajanje ukazov lupine: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Branje datotek: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Spreminjanje datotek: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Prenos URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Dovoljeni koreni za datotečna orodja |
| `permissions.denied_paths` | nič | Izrecno blokirane poteh datotek |
| `network.connect_timeout_secs` | `15` | Čas, dovoljen za povezavo |
| `network.idle_timeout_secs` | `300` | Dovoljena tišina med čakanjem na podatke |
| `network.proxy` | `system` | `system`, `none` ali naslov vmesnika |
| `network.ca_bundle` | nič | Dodatna datoteka zaupanja vrednih certifikacijskih teles |
| `network.allow_insecure_http` | `false` | Dovoli ključ čez navaden `http` na tuj gostitelj |
| `update.check` | `true` | Vprašaj GitHub in GitLab po novejši različici ob začetku interaktivne seje |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, od katerega preverba vpraša po najnovejši izdaji |
| `update.gitlab_repo` | nič | GitLab `owner/repo` za isto preverbo. Prazno izklopi GitLab |
| `update.timeout_secs` | `3` | Koliko sekund preverba sme porabiti na enem viru |

Marelle nosi lasten seznam zaupanja vrednih certifikacijskih teles, zato deluje tudi tam, kjer jih sistem nima, kot v Termuxu. `SSL_CERT_FILE` dodaja še.

Ko se najde novejša različica, jo seja enkrat omeni — različico, tisto, kar imate, in kje jo dobiti. Najprej se vpraša izdaja; repozitorij, ki svoje zaveze le označuje z oznakami, da najnovejšo oznako. Kakršen koli neuspeh ostane tih.

## Vaši ključi ostanejo vaši

- Nastavitve poimenujejo spremenljivko ali datoteko, kjer leži ključ, nikoli sam ključ. Nastavitev kot `providers.x.api_key` se zavrne.
- Ključa `config show`, `config explain` in `providers` ne izpisujejo, in je izrezan iz vsakega besedila napake, ki pride od strežnika.
- Pošilja se le na naslov ponudnika, v enem glavju, in nikoli čez navaden `http` na drug stroj, če tega ne dovolite.
- Preusmeritve se ne sledijo, zato se ključ ne more poslati kam drugam.

## Jeziki

Vmesnik govori angleško, rusko, francosko in ukrajinsko. Izbirate prek `--lang <code>` ali `MARELLE_LANG`; sicer odločajo `LC_ALL`, `LC_MESSAGES` in `LANG`. Vsako sporočilo živi v katalogu, in test pade, če kateri jezik enega pusti ven. Cilj so vsi jeziki Evrazije in potem še.

## Načrt

1. **Govoriti.** Vmesniki, združljivi z OpenAI, in Anthropic, s pretakanjem SSE in razlaganjem popolnih odgovorov. Potezi agenta z orodji trenutno uporabljajo popolne odgovore. *Končano.*
2. **Nastavitve in podoba.** Datoteke nastavitev TOML, splošne in na projekt, z `config set`, `get` in `unset` in opisom vsakega ključa. Tematski vmesnik v duhu [Hermes Agent](https://github.com/NousResearch/hermes-agent): pasica, informacijska plošča, odgovori v okvirju, vrtiljak, privzeto v vijolični barve glicinije, s katerokoli barvo kot HEX. *Končano.*
3. **Mnogi modeli.** Vzdevki modelov, sočasno vprašanje več modelov in primerjava odgovorov, verige rezerv in model sodnik, ki jih zlije. *Končano.*
4. **Dejati.** Strukturirani klici orodij za ponudnike, združljive z OpenAI in Anthropic, krajevna orodja za datoteke, iskanje in terminal, prenos URL, krajevne veščine in dovoljenja allow/ask/deny; prepis pokaže vsak klic orodja in njegov rezultat. *Končano.*
5. **Zapomniti si.** Poimenovani pogovori ostanejo krajevno med zagoni. Dolgotrajna semantična pomnilnost še pred nami.
6. **Razširjati.** Strežniki MCP, zunanji vstavki, podagenti in bogatejša podpora veščinam.
7. **Dosezati.** Avtomatizacija brskalnika, vrata sporočevalcev in načrtovane naloge.

Razmišljanje za zasnovo je v [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Znane omejitve

- Izbirniki ponudnikov in modelov izpišejo nastavljene ponudnike in vzdevke modelov. Marelle še ne odkriva oddaljenih katalogov modelov.
- Izbirniki podpirajo klik z miško za poudarek in kotaljenje z kolescem; prepis in urejevalnik vnosa ostajata vodena s tipkovnice.
- Ctrl-C zapusti interaktivno sejo. Za zdaj ne prekine le trenutnega odgovora.
- Zanka agenta, krajevna orodja, ki pozivajo dovoljenja, in trajne krajevne seje so izvedeni, a Marelle ostaja prototip pre-alpha; o sedanji zasnovi in mejah berite [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Razvoj

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Preizkusi se nikoli ne dotaknejo omrežja: govorijo s posnetim strežnikom v `crates/marelle-testkit`. GitHub Actions gradi tudi androidske binarne datoteke in preveri naznanjeno najmanjšo različico Rust. Za ogledalo na GitLabu je cevovod GitLab CI; glej [docs/GITLAB.md](docs/GITLAB.md).

## Licenca

[AGPL-3.0-only](LICENSE). Obvestila tretjih so v [NOTICE](NOTICE). Arhivi izdaj vsebujejo `THIRD-PARTY-LICENSES.txt` za škatle Rust znotraj binarne datoteke.
