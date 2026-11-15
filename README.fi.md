```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Asetettava tekoälyagentti terminaalillesi.**
Tuo omat API-avaimesi: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, paikallinen Ollama tai mikä tahansa OpenAI-yhteensopiva palvelin. Ensin Linux ja Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
Suomi · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Esi-alfa, versio 0.1.0-alpha.1.** Toteutettu: oma TUI-käyttöliittymä, monipalvelujen chat, lupatietoinen agenttisilmukka, paikalliset työkalut ja jatkettavat paikalliset istunnot. API:t ja asetukset voivat vielä muuttua. Tämä on prototyyppi, ei ominaisuuspariteetti OpenCode'n tai Hermeksen kanssa.

*Marelle* on ranskalaisen lasten hyppelyleikin nimi. Jokainen ruutu on askel.

## Asennus

**Termux Androidilla.** Lataa valmis binääri [julkaisujen](https://github.com/S-I-L-S/marelle/releases) sivulta tai rakenna se puhelimessa. Vaihe vaiheelta -opas: [docs/TERMUX.md](docs/TERMUX.md).

**Linux ja kaikkialla missä Rust toimii.** Tarvitaan Rust 1.85 tai uudempi:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Tai kloonaa tietovarasto ja aja `cargo build --release`. Binääri on `target/release/marelle`, noin 4 MiB.

## Ensimmäinen chat

Pidä avain tiedostossa, jonka vain sinä luet, ja kerro Marellelle missä se on. Marelle ei koskaan tallenna avainta asetuksiinsa.

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

Korvaa `MODEL` mallin nimellä palveluntarjoajasi listalta. Muita käyttötapoja:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Vastaus menee stdout'iin. Pohdinnat, tokenien määrät, muistiinpanot ja virheet menevät stderr'iin, joten `marelle chat ... > answer.txt` tallentaa vain vastauksen.

## Agentin työkalut

Marelle lähettää jäsennellyt työkalumäärittelyt OpenAI-yhteensopivien ja Anthropic API:en kautta. Sisäänrakennettu työkalusarja:

| Työkalu | Tarkoitus |
| --- | --- |
| `terminal` | Aja komentotulkki komento käynnistyskansiosta |
| `read_file`, `write_file`, `edit_file` | Lue ja muokkaa UTF-8-tekstitiedostoja |
| `list_directory`, `search_text` | Tarkastele ja etsi projektin tiedostoista |
| `fetch_url` | Nouda julkinen HTTP(S)-sivu tekstinä (vaatii `curl`) |
| `list_skills`, `load_skill` | Löydä ja lataa paikallisia `SKILL.md`-ohjeita |

Komento `/tools` näyttää työkalulistan ja `/skills` käytettävissä olevat taidot. Projektin ohjeet ladataan `AGENTS.md`-, `CLAUDE.md`-, `SOUL.md`-, `.cursorrules`- ja valituista `.marelle/`-tiedostoista, kun ne ovat sallituilla poluilla.

Tiedostotyökalut rajataan `permissions.allowed_paths`-iin (oletus: nykyinen projektikansio) ja kunnioittavat `permissions.denied_paths`-ia. Lukeminen on oletuksena sallittua; tiedostojen muutokset, komentotulkkikomennot ja verkkolataukset kysyvät ensin. Kuoren lupaa koskee isäntäkoneen komentoja, eikä se ole tiedostojärjestelmän hiekkalaatikko: pidä se `ask`-issa, ellei sinä tarkoituksella uskota mallille komentojen ajoa.

Agentti pysähtyy `agent.max_turns` työkalukierroksen jälkeen. Työkalujen vastauksia pidetään epäluotettavana sisältönä. Pyynnöt käyttävät mallin kokonaisia vastauksia, joten työkalujen argumentit eivät koota puutteellisista virtapaloista.

## Istunnot

Keskustelut tallennetaan oletuksena paikallisesti, mukaan lukien työkalukutsujen historia sekä mallin ja istunnon asetukset. Unixilla istuntokansiolla on tila `0700` ja JSON-tiedostolla `0600`. Tiedosto säilyy paikassa `$XDG_STATE_HOME/marelle/sessions.json`, tai `~/.local/state/marelle/sessions.json`, kun `XDG_STATE_HOME` ei ole asetettu. API-avaimia ei koskaan kirjoiteta tähän tiedostoon. Poista pysyvyys käytöstä:

```sh
marelle config set agent.persist_sessions false
```

Poistumiskoodit: `0` onnistuminen, `1` palveluntarjoajan tai verkon virhe, `2` käytön tai asetusten virhe.

## Palveluntarjoajat

| id | Protokolla | Avaimen muuttuja | |
| --- | --- | --- | --- |
| `zai` | OpenAI-yhteensopiva | `ZAI_API_KEY` | Z.ai (Zhipu GLM), kansainvälinen |
| `zai_coding` | OpenAI-yhteensopiva | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-yhteensopiva päätepiste |
| `zhipu` | OpenAI-yhteensopiva | `ZHIPUAI_API_KEY` | Zhipu BigModel, Kiina |
| `zhipu_coding` | OpenAI-yhteensopiva | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Kiina |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-yhteensopiva | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-yhteensopiva | `OPENROUTER_API_KEY` | monta mallia yhden avaimen takana |
| `ollama` | OpenAI-yhteensopiva | ei ole | Ollama täällä koneessa |

Z.ai:n ja Zhipun päätepisteet tulevat kolmansien osapuolten dokumentaatiosta, eivät toimittajan omilta sivuilta. Tarkista ne omaa pakettiasi vasten ja ylikirjoita mikä tahansa kenttä, jos ne eroavat.

Palveluntarjoajan jokainen kenttä on muutettavissa, ja uusi palveluntarjoaja on vain nimi osoitteella:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Muut palveluntarjoajan kentät: `kind` (`openai-compatible` tai `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ja `extra_body`. Jos `api_key_env` nimetty muuttuja on asetettu, se voittaa; muuten tiedosto luetaan.

## Asetukset

Asetukset tulevat kerroksina: sisäänrakennetut oletusarvot, sitten ympäristömuuttujat, sitten liput. Asetustiedostot asettuvat oletusarvojen ja ympäristön väliin.

- `MARELLE_CHAT__MODEL=zai/MODEL` asettaa `chat.model`-in. Kaksoisalaviiva tarkoittaa pistettä.
- `marelle config show [prefix]` luettelee asetukset ja kerroksen, josta kukin tuli.
- `marelle config explain <key>` näyttää jokaisen kerroksen, joka asetti avaimen, matalimmasta prioriteetista korkeimpaan.

| Avain | Oletus | Merkitys |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ei ole | Kanssa kuka puhua. `chat.model` voi olla `provider/model`. |
| `chat.system` | ei ole | Järjestelmäkehote |
| `chat.max_tokens`, `chat.temperature` | ei ole | Ei asetettu — jätetään palveluntarjoajalle |
| `chat.stream` | `true` | Tuomarin vastauksen striimaus, kun hän on päällä; työkalukäyttöiset agenttikierrokset käyttävät nyt kokonaisia vastauksia, jotta työkalukutsut säilyvät ehyinä |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Pohdinnat ja tokenimäärät stderr'issä |
| `agent.max_turns` | `20` | Enimmäismäärä työkalukierroksia yhtä käyttäjän pyyntöä kohden |
| `agent.persist_sessions` | `true` | Paikallisten keskustelujen tallennus ajkojen välillä |
| `permissions.shell` | `ask` | Komentotulkkikomentojen ajo: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Tiedostojen luku: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Tiedostojen muutos: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL-lataus: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Sallitut juuret tiedostotyökaluille |
| `permissions.denied_paths` | ei ole | Selkeesti estetyt tiedostopolut |
| `network.connect_timeout_secs` | `15` | Yhdistämiseen sallittu aika |
| `network.idle_timeout_secs` | `300` | Sallittu hiljaisuus datan odotuksessa |
| `network.proxy` | `system` | `system`, `none`, tai proxy-osoite |
| `network.ca_bundle` | ei ole | Lisätiedosto luotettavista varmenteen myöntäjistä |
| `network.allow_insecure_http` | `false` | Salli avain tavallisella `http`-llä vieräälle isäntälle |
| `update.check` | `true` | Kysy GitHub'iltä ja GitLab'iltä uudempaa versiota interaktiivisen istunnon alussa |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, jolta tarkistus kysyy tuoreinta julkaisua |
| `update.gitlab_repo` | ei ole | GitLab `owner/repo` samalle tarkistukselle. Tyhjä poistaa GitLabin käytöstä |
| `update.timeout_secs` | `3` | Kuinka monta sekuntia tarkistus saa käyttää yhteen lähteeseen |

Marelle kantaa mukanaan omia luotettavien varmenteen myöntäjiä, joten se toimii myös siellä, missä järjestelmällä niitä ei ole, kuten Termuxissa. `SSL_CERT_FILE` lisää lisää.

Kun uudempi versio löytyy, istunto mainitsee sen kerran — version, sen, mitä sinulla on, ja mistä sen saa. Julkaisua kysytään ensin; tietovarasto, joka committejaan vain tägeillä merkitsee, antaa tuoreimman tägin. Mikä epäonnistuu, jää hiljaiseksi.

## Avaimesi pysyvät sinun

- Asetukset nimeävät muuttujan tai tiedoston, jossa avain makaa, ei koskaan itse avainta. Sellainen asetus kuin `providers.x.api_key` hylätään.
- Avainta `config show`, `config explain` tai `providers` ei tulosta, ja se leikataan pois jokaisesta palvelimelta palanneesta virhetekstistä.
- Se lähetetään vain palveluntarjoajan osoitteeseen, yhdessä otsikkoon, ja ei koskaan tavallisella `http`-llä toiselle koneelle, jos et sitä salli.
- Uudelleenohjauksia ei seurata, joten avain ei voi lähteä muualle.

## Kielet

Käyttöliittymä puhuu englantia, venäjää, ranskaa ja ukrainaa. Valitse `--lang <code>`-llä tai `MARELLE_LANG`-illa; muuten `LC_ALL`, `LC_MESSAGES` ja `LANG` päättävät. Jokainen viesti elää katalogissa, ja testi kaatuu, jos jokin kieli jättää yhden pois. Tavoite on joka Eurasian kieli ja sitten lisää.

## Tie

1. **Puhua.** OpenAI-yhteensopivat ja Anthropic-adapterit, sekä SSE-virransiirrolla että kokonaisten vastausten lukijalla. Työkalukäyttöiset agenttikierrokset käyttävät nyt kokonaisia vastauksia. *Valmis.*
2. **Asetukset ja ilme.** TOML-asetustiedostot, yleiset ja projektikohtaiset, `config set`-llä, `get`-llä ja `unset`-illä, ja jokaisen avaimen kuvauksella. Teemoitettu käyttöliittymä [Hermes Agent](https://github.com/NousResearch/hermes-agent) hengessä: banneri, tietopaneeli, kehyksiset vastaukset, pyörähtäjä, oletuksena glysiinin violetissa, mikä tahansa väri HEX-inä. *Valmis.*
3. **Monta mallia.** Mallien lempinimet, useiden mallien yhtäaikainen kysyminen ja vastausten vertailu, varaketjut, ja tuomarimalli, joka ne yhteen sulaa. *Valmis.*
4. **Toimia.** Jäsennellyt työkalukutsut OpenAI-yhteensopiville ja Anthropic-palveluntarjoajille, paikalliset tiedosto-, haku- ja terminaalityökalut, URL-lataus, paikalliset taidot ja allow/ask/deny-luvat; transkripti näyttää jokaisen työkalukutsun ja sen tuloksen. *Valmis.*
5. **Muistaa.** Nimetyt keskustelut säilyvät paikallisesti ajkojen välillä. Pitkäaikainen semanttinen muisti vielä edessä.
6. **Laajentaa.** MCP-palvelimet, ulkoiset lisäosat, ala-agentit ja rikkaampi taitotuki.
7. **Ulottaa.** Selainautomaatio, viestintäportit ja ajastetut tehtävät.

Suunnittelun takainen ajattelu on [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-ssä.

## Tunnetut rajat

- Palveluntarjoaja- ja mallivalitsimet luettelevat asetetut palveluntarjoajat ja mallien lempinimet. Marelle ei vielä löydä etänä olevia mallikatalogeja.
- Valitsimet tukevat hiiren klikkiä kohdistukseen ja rullan vieritystä; transkripti ja syötteen muokkaaja pysyvät näppäimistöllä ajettuina.
- Ctrl-C poistuu interaktiivisesta istunnosta. Se ei vielä keskeytä vain nykyistä vastausta.
- Agenttisilmukka, lupatietoiset paikalliset työkalut ja pysyvät paikalliset istunnot on toteutettu, mutta Marelle pysyy esi-alfa prototyyppinä; nykyisestä suunnittelusta ja rajoista lue [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Kehitys

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testit eivät koske verkkoon: ne puhuvat valepalvelimelle `crates/marelle-testkit`-ssä. GitHub Actions rakentaa myös Android-binäärit ja tarkistaa ilmoitetun minimi-Rust-version. GitLab-peilille on GitLab CI-putki; katso [docs/GITLAB.md](docs/GITLAB.md).

## Lisenssi

[AGPL-3.0-only](LICENSE). Kolmansien osapuolten ilmoitukset ovat [NOTICE](NOTICE)-ssä. Julkaisuarkistot sisältävät `THIRD-PARTY-LICENSES.txt` binäärin sisällä oleville Rust-laatikoille.
