```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Testreszabható MI-ügynök a terminálodhoz.**
Hozd a saját API-kulcsaidat: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, helyi Ollama vagy bármely OpenAI-kompatibilis szerver. Először Linux és Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · Magyar · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, 0.1.0-alpha.1 verzió.** Megvalósítva: natív TUI felület, több szolgáltatós csevegés, engedélytudatos ügynökciklus, helyi eszközök és folytatható helyi munkamenetek. Az API-k és a beállítások még változhatnak. Ez egy prototípus, nem funkcióparitás az OpenCode-tal vagy a Hermesszel.

A *Marelle* francia szó, a klasszikus ugrós gyerekjáték (hopscotch) neve. Minden négyzet egy lépés.

## Telepítés

**Termux Androidon.** Töltsd le az előre elkészített binárist a [kiadások](https://github.com/S-I-L-S/marelle/releases) oldaláról, vagy építsd meg magad a telefonon. Lépésről lépésre: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, és mindenhol, ahol a Rust fut.** Rust 1.85 vagy újabb kell:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Vagy klónozd a repót, és futtasd a `cargo build --release`-t. A bináris a `target/release/marelle`, kb. 4 MiB.

## Első csevegés

A kulcsot olyan fájlban tartsd, amit csak te olvasol, és mondd meg a Marelle-nek, hol van. A Marelle soha nem tárol kulcsot a beállításaiban.

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

A `MODEL` helyére a szolgáltatód modelllistájából való nevet írj. További használati módok:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

A válasz a stdout-ra megy. A gondolkodás, a token-számok, a jegyzetek és a hibák a stderr-re, így a `marelle chat ... > answer.txt` csak a választ menti el.

## Az ügynök eszközei

A Marelle strukturált eszközdefiníciókat küld az OpenAI-kompatibilis és az Anthropic API-n keresztül. A beépített eszközkészlet:

| Eszköz | Rendeltetés |
| --- | --- |
| `terminal` | Shellparancs futtatása az indítási könyvtárból |
| `read_file`, `write_file`, `edit_file` | UTF-8 szövegfájlok olvasása és módosítása |
| `list_directory`, `search_text` | Projektfájlok átnézése és keresése |
| `fetch_url` | Nyilvános HTTP(S) oldal letöltése szövegként (`curl` kell) |
| `list_skills`, `load_skill` | Helyi `SKILL.md` utasítások felderítése és betöltése |

A `/tools` kilistázza az eszközöket, a `/skills` az elérhető készségeket. A projektvezetés a `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` és kijelölt `.marelle/` fájlokból töltődik be, ha engedélyezett utakon vannak.

A fáyleszközök a `permissions.allowed_paths` közé vannak zárva (alapértelmezés: az aktuális projektkönyvtár), és tisztelik a `permissions.denied_paths`-t. Az olvasás alapból engedélyezett; fájl módosítás, shellparancs és hálózati letöltés előbb kérdez. A shell-engedély a gépen futó parancsokra vonatkozik, és nem fájlrendszer-homokozó: tartsd `ask`-on, hacsak nem szándékosan bízod a modellre a parancsfuttatást.

Az ügynök `agent.max_turns` eszközkör után megáll. Az eszközválaszokat nem megbízható tartalomnak kezeli. A kérések teljes modellválaszokat használnak, így az eszközargumentumok nem hiányos adatdarabokból állnak össze.

## Munkamenetek

A beszélgetések alapból helyben mentődnek, az eszközhívások előzményeivel és a modell- és munkamenet-beállításokkal együtt. Unixon a munkamenet-könyvtár módja `0700`, a JSON-fájlé `0600`. A fájl a `$XDG_STATE_HOME/marelle/sessions.json` helyen van, vagy `~/.local/state/marelle/sessions.json`, ha a `XDG_STATE_HOME` nincs beállítva. API-kulcsok soha nem íródnak ebbe a fájlba. A perzisztenciát így kapcsold ki:

```sh
marelle config set agent.persist_sessions false
```

Kilépési kódok: `0` siker, `1` szolgáltatói vagy hálózati hiba, `2` használati vagy beállítási hiba.

## Szolgáltatók

| id | Protokoll | Kulcs-változó | |
| --- | --- | --- | --- |
| `zai` | OpenAI-kompatibilis | `ZAI_API_KEY` | Z.ai (Zhipu GLM), nemzetközi |
| `zai_coding` | OpenAI-kompatibilis | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-kompatibilis végpont |
| `zhipu` | OpenAI-kompatibilis | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kína |
| `zhipu_coding` | OpenAI-kompatibilis | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kína |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-kompatibilis | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-kompatibilis | `OPENROUTER_API_KEY` | sok modell egy kulcs mögött |
| `ollama` | OpenAI-kompatibilis | nincs | Ollama ezen a gépen |

A Z.ai és Zhipu végpontok harmadik felek dokumentációjából valók, nem a szolgáltató saját oldalairól. Ellenőrizd őket a csomagodhoz, és írd felül bármelyik mezőt, ha eltérnek.

A szolgáltató minden mezője megváltoztatható, és egy új szolgáltató csupán egy név egy címmel:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

További szolgáltatói mezők: `kind` (`openai-compatible` vagy `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` és `extra_body`. Ha az `api_key_env` által nevezett változó be van állítva, ő nyer; különben a fájl olvasódik.

## Beállítások

A beállítások rétegekben jönnek: beépített alapértékek, aztán környezeti változók, aztán kapcsolók. A beállítófájlok az alapértékek és a környezet közé ékelődnek.

- A `MARELLE_CHAT__MODEL=zai/MODEL` beállítja a `chat.model`-t. A kettős aláhúzás pontot jelent.
- A `marelle config show [prefix]` kilistázza a beállításokat és a réteget, ahonnan mindegyik jött.
- A `marelle config explain <key>` megmutat minden réteget, amely a kulcsot beállította, a legalacsonyabb prioritiástól a legmagasabbig.

| Kulcs | Alapérték | Jelentés |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nincs | Kivel beszéljünk. A `chat.model` lehet `provider/model`. |
| `chat.system` | nincs | Rendszerprompt |
| `chat.max_tokens`, `chat.temperature` | nincs | Ha nincs beállítva, a szolgáltató dönt |
| `chat.stream` | `true` | A választó bíráló válaszának streamelése; az eszközhasználatos ügynökkörök jelenleg teljes válaszokat használnak, hogy az eszközhívások épek maradjanak |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Gondolkodás és token-számok a stderr-en |
| `agent.max_turns` | `20` | Maximum eszközkör felhasználónkénti kérésenként |
| `agent.persist_sessions` | `true` | Helyi beszélgetések mentése futtatások között |
| `permissions.shell` | `ask` | Shellparancs futtatás: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Fájlolvasás: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Fájlmódosítás: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-letöltés: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Engedélyezett gyökerek a fáyleszközöknek |
| `permissions.denied_paths` | nincs | Kifejezetten tiltott fájlutak |
| `network.connect_timeout_secs` | `15` | A kapcsolódásra hagyott idő |
| `network.idle_timeout_secs` | `300` | Megengedett csend adatvárakozás közben |
| `network.proxy` | `system` | `system`, `none`, vagy proxy-cím |
| `network.ca_bundle` | nincs | Extra fájl megbízható tanúsítványkiadókkal |
| `network.allow_insecure_http` | `false` | Kulcs engedélyezése sima `http`-n távoli gépre |
| `update.check` | `true` | GitHub és GitLab megkérdezése újabb verzióról interaktív munkamenet indításakor |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, amelytől az ellenőrzés a legfrissebb kiadást kérdezi |
| `update.gitlab_repo` | nincs | GitLab `owner/repo` ugyanarra az ellenőrzésre. Üresen kikapcsolja a GitLabot |
| `update.timeout_secs` | `3` | Hány másodpercet tölthet az ellenőrzés egy forráson |

A Marelle saját megbízható tanúsítványkiadói listát hordoz, így ott is működik, ahol a rendszernek nincs, mint a Termux. Az `SSL_CERT_FILE` továbbiakat ad.

Újabb verzió találásakor a munkamenet egyszer említi — a verziót, azt, amit birtokolsz, és hol szerezheted. Először a kiadást kérdezi; az olyan repó, amely csak tagekkel jelöli a commitjeit, a legfrissebb taget adja. Bármi hiba elhallgatva marad.

## A kulcsaid a tiéid maradnak

- A beállítások változót vagy fájlt neveznek meg, amely a kulcsot tartja, sosem magát a kulcsot. A `providers.x.api_key` szerű beállítást visszautasítja.
- A kulcsot a `config show`, a `config explain` és a `providers` nem írja ki, és kivágódik minden szerverről visszatérő hibaszövegből.
- Csak a szolgáltató címére megy, egyetlen fejlécben, és soha sem sima `http`-n másik gépre, hacsak nem engedélyezed.
- Átirányításokat nem követ, így a kulcs nem képzethet máshova kerülni.

## Nyelvek

A felület angolul, oroszul, franciául és ukránul beszél. Válaszd így: `--lang <code>` vagy `MARELLE_LANG`; egyébként a `LC_ALL`, az `LC_MESSAGES` és a `LANG` dönt. Minden üzenet katalógusban él, és egy teszt elbukik, ha egy nyelv egyet kihagy. A cél Eurasia minden nyelve, és aztán több.

## Ütemterv

1. **Beszélni.** OpenAI-kompatibilis és Anthropic adaptérek, SSE-streameléssel és teljes-válasz dekódolással. Az eszközhasználatos ügynökkörök jelenleg teljes válaszokat használnak. *Kész.*
2. **Beállítások és külcsím.** TOML beállítófájlok, globálisan és projektenként, `config set`, `get` és `unset`-tal, minden kulcs leírásával. Tematizált felület a [Hermes Agent](https://github.com/NousResearch/hermes-agent) szellemében: banner, információs panel, keretezett válaszok, spinner, alapból glycínia-lilában, bármely szín HEX-ként állítható. *Kész.*
3. **Sok modell.** Modell-áliasok, több modell egyidejű megkérdezése és a válaszok összehasonlítása, fallback-láncok, és bíráló modell, amely egybeolvasztja őket. *Kész.*
4. **Cselekedni.** Strukturált eszközhívások OpenAI-kompatibilis és Anthropic szolgáltatókhoz, helyi fájl-, kereső- és termináleszközök, URL-letöltés, helyi készségek és allow/ask/deny engedélyek; a transcript minden eszközhívást és eredményét megmutatja. *Kész.*
5. **Emlékezni.** Nevesített beszélgetések futtatások között helyben megmaradnak. A hosszú távú szemantikai memória még előttünk.
6. **Bővíteni.** MCP szerverek, külső pluginek, alügynökök és gazdagabb készségtámogatás.
7. **Elérni.** Böngészőautomatizálás, üzenetküldő-átjárók és ütemezett feladatok.

A tervezés mögötti gondolatmenet a [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) fájlban.

## Ismert korlátok

- A szolgáltató- és modellválasztók a beállított szolgáltatókat és modell-áliasokat sorolják fel. A Marelle távoli modellkatalógusokat még nem deríti fel.
- A választók támogatják az egérkattintást a fókuszhoz és a görgős görgetést; a transcript és a bemenetszerkesztő billentyűzetvezérelt marad.
- A Ctrl-C kilép az interaktív munkamenetből. Egyelőre nem csak az aktuális választ szakítja meg.
- Az ügynökciklus, az engedélytudatos helyi eszközök és a perzisztens helyi munkamenetek megvalósultak, de a Marelle pre-alpha prototípus marad; a jelenlegi tervet és határokat a [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) írja.

## Fejlesztés

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

A tesztek soha nem érintik a hálózatot: egy álserverrel beszélnek a `crates/marelle-testkit`-ben. A GitHub Actions az Android-binárisokat is építi, és ellenőrzi a deklarált minimális Rust-verziót. GitLab-tükörhöz GitLab CI csővezeték jár; lásd [docs/GITLAB.md](docs/GITLAB.md).

## Licenc

[AGPL-3.0-only](LICENSE). A harmadik felek megjegyzései a [NOTICE](NOTICE) fájlban. A kiadási archívumok `THIRD-PARTY-LICENSES.txt`-t tartalmaznak a binárisban élő Rust crate-ekhez.
