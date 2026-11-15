```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigūruojamas DI agentas jūsų terminale.**
Atsineškite savus API raktus: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, vietinė Ollama arba bet kuris OpenAI suderinamas serveris. Pirmiausia Linux ir Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
Lietuvių · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Iki-alfa, versija 0.1.0-alpha.1.** Įgyvendinta: gimtoji TUI sąsaja, pokalbis su keliais tiekėjais, agento ciklas, atsižvelgiantis į leidimus, vietiniai įrankiai ir atnaujinamos vietinės sesijos. API ir nustatymai dar gali keistis. Tai prototipas, ne funkcijų paritetas su OpenCode ar Hermes.

*Marelle* – prancūziškas vaikų žaidimo „klasiukų" pavadinimas. Kiekvienas kvadratas – žingsnis.

## Įdiegimas

**Termux Android'e.** Atsisiųskite paruoštą dvejetainį failą iš [leidimų](https://github.com/S-I-L-S/marelle/releases) puslapio arba sukompiliuokite telefone. Žingsnis po žingsnio vadovas: [docs/TERMUX.md](docs/TERMUX.md).

**Linux ir visur, kur veikia Rust.** Reikia Rust 1.85 ar naujesnio:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Arba klonuokite saugyklą ir paleiskite `cargo build --release`. Dvejetainis failas – `target/release/marelle`, apie 4 MiB.

## Pirmasis pokalbis

Laikykite raktą faile, kurį skaito tik jūs, ir pasakykite Marelle, kur jis yra. Marelle niekada nesaugo rakto savo nustatymuose.

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

Pakeiskite `MODEL` modelio pavadinimu iš savo tiekėjo sąrašo. Kiti naudojimo būdai:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Atsakymas eina į stdout. Svarstymai, tokenų skaičiai, pastabos ir klaidos eina į stderr, taigi `marelle chat ... > answer.txt` išsaugo tik atsakymą.

## Agento įrankiai

Marelle siunčia struktūruotus įrankių apibrėžimus per OpenAI suderinamus ir Anthropic API. Integruotas įrankių rinkinys:

| Įrankis | Paskirtis |
| --- | --- |
| `terminal` | Paleisti apvalkalo komandą iš paleisties katalogo |
| `read_file`, `write_file`, `edit_file` | Skaityti ir keisti UTF-8 tekstinius failus |
| `list_directory`, `search_text` | Apžvelgti ir ieškoti projekto failuose |
| `fetch_url` | Atsisiųsti viešą HTTP(S) puslapį kaip tekstą (reikia `curl`) |
| `list_skills`, `load_skill` | Atrasti ir įkelti vietines `SKILL.md` instrukcijas |

Komanda `/tools` parodo įrankių sąrašą, `/skills` – turimas įgūdžius. Projekto nurodymai kraunami iš `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` ir pasirinktų `.marelle/` failų, jei jie yra leidžiamuose keliuose.

Failų įrankiai ribojami `permissions.allowed_paths` (numatyta: einantis projekto katalogas) ir gerbia `permissions.denied_paths`. Skaitymas pagal nutylėjimą leidžiamas; failų keitimas, apvalkalo komandos ir tinklo atsisiuntimai pirmiausia klausoja. Apvalkalo leidimas liečia komandas serveryje ir nėra failų sistemos smėlio dėžė: laikykite jį `ask`, nebent sąmoningai patikite modeliui komandų vykdymą.

Agentas sustoja po `agent.max_turns` įrankių apyrankių. Įrankių atsakymai laikomi nepatikimu turiniu. Užklausos naudoja pilnus modelio atsakymus, todėl įrankių argumentai nesusideda iš nepilnų srauto gabalų.

## Sesijos

Pokalbiai pagal numatytuosius nustatymus saugomi vietoje, įskaitant įrankių iškvietimų istoriją ir modelio bei sesijos nustatymus. Unix sesijų katalogas turi režimą `0700`, JSON failas – `0600`. Failas saugomas `$XDG_STATE_HOME/marelle/sessions.json`, arba `~/.local/state/marelle/sessions.json`, kai `XDG_STATE_HOME` nenustatytas. API raktai niekada neįrašomi į šį failą. Išjunkite išsaugojimą:

```sh
marelle config set agent.persist_sessions false
```

Išėjimo kodai: `0` sėkmė, `1` tiekėjo ar tinklo klaida, `2` naudojimo ar nustatymų klaida.

## Tiekėjai

| id | Protokolas | Rakto kintamasis | |
| --- | --- | --- | --- |
| `zai` | suderintas su OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), tarptautinis |
| `zai_coding` | suderintas su OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, su Anthropic suderinamas galinis taškas |
| `zhipu` | suderintas su OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kinija |
| `zhipu_coding` | suderintas su OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kinija |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | suderintas su OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | suderintas su OpenAI | `OPENROUTER_API_KEY` | daug modelių už vieną raktą |
| `ollama` | suderintas su OpenAI | nėra | Ollama šioje mašinoje |

Z.ai ir Zhipu galiniai taškai paimti iš trečiųjų šalių dokumentacijos, ne iš paties tiekėjo puslapių. Patikrinkite juos prieš savo planą ir perrašykite bet kurį lauką, jei jie skiriasi.

Kiekvienas tiekėjo laukas keičiamas, o naujas tiekėjas – tiesiog vardas su adresu:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Kiti tiekėjo laukai: `kind` (`openai-compatible` arba `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ir `extra_body`. Jei kintamasis, pavadintas `api_key_env`, nustatytas, jis laimi; kitaip skaitomas failas.

## Nustatymai

Nustatymai ateina sluoksniais: įtaisytosios numatytosios reikšmės, tada aplinkos kintamieji, tada vėliavėlės. Nustatymų failai įsiterpia tarp numatytųjų reikšmių ir aplinkos.

- `MARELLE_CHAT__MODEL=zai/MODEL` nustato `chat.model`. Dvigubas apatinis brūkšnelis reiškia tašką.
- `marelle config show [prefix]` išvardija nustatymus ir sluoksnį, iš kurio kiekvienas atėjo.
- `marelle config explain <key>` rodo kiekvieną sluoksnį, nustatiusį raktą, nuo žemiausios pirmenybės iki aukščiausios.

| Raktas | Numatyta | Reikšmė |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nėra | Su kuo kalbėti. `chat.model` gali būti `provider/model`. |
| `chat.system` | nėra | Sistemos paragintas |
| `chat.max_tokens`, `chat.temperature` | nėra | Nenustatyta – paliekama tiekėjui |
| `chat.stream` | `true` | Teisėjo atsakymo srautinimas, kai jis įjungtas; agento ėjimai su įrankiais šiuo metu naudoja pilnus atsakymus, kad įrankių iškvietimai liktų sveiki |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Svarstymai ir tokenų skaičiai į stderr |
| `agent.max_turns` | `20` | Didžiausias įrankių apyrankių skaičius vienai vartotojo užklausai |
| `agent.persist_sessions` | `true` | Saugoti vietinius pokalbius tarp paleidimų |
| `permissions.shell` | `ask` | Apvalkalo komandų vykdymas: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Failų skaitymas: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Failų keitimas: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL atsisiuntimas: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Leidžiamos šaknys failų įrankiams |
| `permissions.denied_paths` | nėra | Aiškiai blokuoti failų keliai |
| `network.connect_timeout_secs` | `15` | Laikas, leidžiamas prisijungti |
| `network.idle_timeout_secs` | `300` | Leidžiama tyla laukiant duomenų |
| `network.proxy` | `system` | `system`, `none` arba įgaliotojo serverio adresas |
| `network.ca_bundle` | nėra | Papildomas failas su patikimais liudijimų centrais |
| `network.allow_insecure_http` | `false` | Leisti raktą paprastu `http` į kitą serverį |
| `update.check` | `true` | Klausti GitHub ir GitLab dėl naujesnės versijos interaktyvios sesijos pradžioje |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, iš kurio patikrinimas klausia naujausio leidimo |
| `update.gitlab_repo` | nėra | GitLab `owner/repo` tam pačiam patikrinimui. Tuščia išjungia GitLab |
| `update.timeout_secs` | `3` | Kiek sekundžių patikrinimas gali skirti vienam šaltiniui |

Marelle nešasi savąjį patikimų liudijimų centrų sąrašą, todėl veikia ir ten, kur sistemos jų nėra, kaip Termux. `SSL_CERT_FILE` prideda dar.

Radus naujesnę versiją, sesija ją paminės kartą – versiją, tai, ką turite, ir kur ją imti. Pirmiausia klausiama leidimo; saugykla, kuri savo įsipareigojimus žymi tik žymomis, duoda naujausią žymą. Kas tik žlugtų, lieka tyloje.

## Jūsų raktai lieka jūsų

- Nustatymai pavadina kintamąjį arba failą, kuriame gulėra raktas, niekada patį raktą. Toks nustatymas kaip `providers.x.api_key` atmetamas.
- Rakto `config show`, `config explain` arba `providers` nespausdina, ir jis išpjaunamas iš bet kokio iš serverio atėjusio klaidos teksto.
- Jis siunčiamas tik į tiekėjo adresą, vienoje antraštėje, ir niekada paprastu `http` į kitą mašiną, jei to neleidžiate.
- Nukreipimai nesekami, todėl raktas negali nukeliauti kitur.

## Kalbos

Sąsaja kalba anglų, rusų, prancūzų ir ukrainiečių kalbomis. Rinkitės per `--lang <code>` arba `MARELLE_LANG`; kitaip sprendžia `LC_ALL`, `LC_MESSAGES` ir `LANG`. Kiekvienas pranešimas gyvena kataloge, ir testas krenta, jei kuri kalba vieną palieka. Tikslas – visos Eurgazijos kalbos, o paskui dar daugiau.

## Kelio žemėlapis

1. **Kalbėti.** Adapteriai, suderinti su OpenAI ir Anthropic, su SSE srautavimu ir pilnų atsakymų skaitymu. Agento ėjimai su įrankiais šiuo metu naudoja pilnus atsakymus. *Atlikta.*
2. **Nustatymai ir išvaizda.** TOML nustatymų failai, globalūs ir pagal projektą, su `config set`, `get` ir `unset` bei kiekvieno rakto aprašymu. Temuota sąsaja [Hermes Agent](https://github.com/NousResearch/hermes-agent) dvasia: juosta, informacijos skydas, atsakymai rėmelyje, sukinys, pagal nutylėjimą alyvų spalvos glicinijos, bet kokia spalva kaip HEX. *Atlikta.*
3. **Daug modelių.** Modelių slapyvardžiai, kelių modelių vienu metu klausimas ir atsakymų lyginimas, atsargos grandinės ir teisėjo modelis, kuris juos sujungia. *Atlikta.*
4. **Veikti.** Struktūruoti įrankių iškvietimai OpenAI suderinamiems ir Anthropic tiekėjams, vietiniai failų, paieškos ir terminalo įrankiai, URL atsiuntimas, vietinės įgūdžių ir leidimai allow/ask/deny; nuorašas rodo kiekvieną įrankio iškvietimą ir jo rezultatą. *Atlikta.*
5. **Prisiminti.** Pavadinti pokalbiai išlieka vietoje tarp paleidimų. Ilgalaikė semantinė atmintis dar priešais.
6. **Plėsti.** MCP serveriai, išoriniai įskiepiai, poagentai ir turtingesnė įgūdžių palaikymas.
7. **Pasiekti.** Naršyklės automatizavimas, pasiuntimų vartai ir suplanuoti darbai.

Dizaino priežastys – [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Žinomi apribojimai

- Tiekėjų ir modelių pasirinkėjai išvardija sukonfigūruotus tiekėjus ir modelių slapyvardžius. Marelle dar neatranda nutolusių modelių katalogų.
- Pasirinkėjai palaiko pelės spragtelėjimą fokusui ir sukimąsi ritele; nuorašas ir įvesties redaktorius lieka klaviatūros valdomi.
- Ctrl-C išeina iš interaktyvios sesijos. Dar nutraukia ne tik einantį atsakymą.
- Agento ciklas, leidimus atsižvelgiantys vietiniai įrankiai ir nuolatinės vietinės sesijos įgyvendintos, bet Marelle lieka iki-alfa prototipu; apie einantį dizainą ir ribas skaitykite [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Plėtra

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testai niekada neliečia tinklo: jie kalba su imituotu serveriu `crates/marelle-testkit`. GitHub Actions taip pat stato Android dvejetainius failus ir patikrina deklaruotą mažiausią Rust versiją. GitLab veidrodžiui yra GitLab CI vamzdynas; žiūrėkite [docs/GITLAB.md](docs/GITLAB.md).

## Licencija

[AGPL-3.0-only](LICENSE). Trečiųjų šalių pranešimai yra [NOTICE](NOTICE). Leidimų archyvuose yra `THIRD-PARTY-LICENSES.txt` Rust dėžėms dvejetainio viduje.
