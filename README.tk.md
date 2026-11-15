```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Terminalyňyz üçin sazlanýan HI-agent.**
Öz API açarlaryňyzy alyp geliň: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, ýerli Ollama ýa-da OpenAI-bilen gabalýan islendik serwer. Ilki Linux we Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · Türkmençe · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Öňki-alfa, wersiýa 0.1.0-alpha.1.** Işlenip düzüldi: özbaşdak TUI interfeýs, köp üpjünçiler bilen gepleşik, rugsatlary bilýän agent aýlanmasy, ýerli guralar we dowam edilmegi mümkin ýerli seanslar. API-ler we sazlamalar entek üýtgedilip bilner. Bu prototip, OpenCode ýa-da Hermes bilen mümkinçilikleri deň däl.

*Marelle* — çagalaryň owuş ýaryş oyunynyň fransuz ady. Her kwadrat — bir ädim.

## Gurmak

**Android-daky Termux.** Taýyn binar faýly [goýumlar](https://github.com/S-I-L-S/marelle/releases) sahypasından göçürip aliň, ýa-da telefondada ýygaryň. Ädim-ädimden gollanma: [docs/TERMUX.md](docs/TERMUX.md).

**Linux we Rust işleýän äýýer ýerde.** Rust 1.85 ýa-da täze rk gerek:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ýa-da hansy repozitoriýany klonlaň da, `cargo build --release` işlediň. Binar faýl — `target/release/marelle`, takmyn 4 MiB.

## Ilkinji gepleşik

Açary diňe siz okaýan faýlda saklaň da, Marelle-ä nirä-de ýerleşýändigini aýdyň. Marelle açary hiç wagt öz sazlamalarynda saklamaýar.

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

`MODEL` ornuna üpjünçiniň sanawyndan modeliň adyny goýuň. Ulanmagyň beýleki ýollary:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Jogap stdout-a barýar. Oýlanmalar, token sanlary, bellikler we ýalňyşlar stderr-a barýar; şeýlelikde `marelle chat ... > answer.txt` diňe jogaby saklaýar.

## Agent gurallary

Marelle OpenAI-bilen gabalýan we Anthropic API arkaly düzümlenen gural kesgitlemelerini ýollaýar. Içerli gural toplumy:

| Gural | Maksat |
| --- | --- |
| `terminal` | Başlanýan papkadan çalşy buýrugyny ýerine yetirmek |
| `read_file`, `write_file`, `edit_file` | UTF-8 tekst faýllaryny okamak we üýtgetmek |
| `list_directory`, `search_text` | Taslamanyň faýllaryny barlamak we gözlemek |
| `fetch_url` | Açyk HTTP(S) sahypasyny tekst hökmünde almak (`curl` gerek) |
| `list_skills`, `load_skill` | Ýerli `SKILL.md` görkezmelerini tapmak we ýüklemek |

`/tools` buýrugy gural sanawyny, `/skills` — elýeterli uýkunlyklary görkezýär. Taslama görkezmeleri `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` we saýlanan `.marelle/` faýllaryndan, rugsat berilen ýollardalary bolanda, ýüklenýär.

Faýl gurallary `permissions.allowed_paths` bilen çäkli (öntümle: taslamanyň häzirki papkasy) we `permissions.denied_paths`-e seretýär. Okamak öntümle rugsatly; faýllary üýtgetmek, çalşy buýruklary we ýol ýüklemeleri öňden sorayar. Çalşy rugsady gostunjyň buýruklaryna degişli we faýl ulgamyň kum gutasy däl: modele buýruklary ýerine yetirmäge bil erkli ynanmaýan bolsaňyz `ask`-da tutuň.

Agent `agent.max_turns` gural turundan soň durýar. Gural jogaplary ýanamaýan mazmun hasaplanýar. Soraglar modeliň doly jogaplaryny ulanýar; şeýlelikde gural argumentleri akymyň bitmedik böleklerinden ýygnanylmaýar.

## Seanslar

Gepleşikler adat boýunça ýerli ýazylýar; gural çagyryşlarynyň taryhy we model- seans sazlamalary bilen. Unix-de seans papkasynyň tertibi `0700`, JSON faýlyňky `0600`. Faýl `$XDG_STATE_HOME/marelle/sessions.json` ýerinde, ýa-da `XDG_STATE_HOME` goýulmadyk bolsa `~/.local/state/marelle/sessions.json` ýerinde saklanýar. API açarlary bu faýla hiç wagt ýazylmaýar. Saklamany şu bilen öçüriň:

```sh
marelle config set agent.persist_sessions false
```

Çykyş kodlary: `0` üstünlik, `1` üpjünçiniň ýa-da ýolyň ýalňyşy, `2` ulanmakyň ýa-da sazlamalaryň ýalňyşy.

## Üpjünçiler

| id | Protokol | Açar değişkeni | |
| --- | --- | --- | --- |
| `zai` | OpenAI-bilen gabalýan | `ZAI_API_KEY` | Z.ai (Zhipu GLM), halkara |
| `zai_coding` | OpenAI-bilen gabalýan | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic-bilen gabalýan son nokady |
| `zhipu` | OpenAI-bilen gabalýan | `ZHIPUAI_API_KEY` | Zhipu BigModel, Hytaý |
| `zhipu_coding` | OpenAI-bilen gabalýan | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Hytaý |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-bilen gabalýan | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-bilen gabalýan | `OPENROUTER_API_KEY` | bir açar arkasynda köp model |
| `ollama` | OpenAI-bilen gabalýan | ýok | Bu maşynda Ollama |

Z.ai we Zhipu son nokatlary üçünji tarap resminamalaryndan gelýär, üpjünçiniň öz sahypalaryndan däl. Olary öz tarifiňiz bilen deňeşdiriň, tapawutlanyp bolsa her harydy üstünden ýazyň.

Üpjünçiniň her harisi üýtgedilip bilner; täze üpjünçi — diňe bir at we salgy:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Üpjünçiniň galan haralary: `kind` (`openai-compatible` ýa-da `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` we `extra_body`. `api_key_env` atalan değişken goýlan bolsa, ol ýeňijä; bolmasa faýl okaýar.

## Sazlamalar

Sazlamalar gatlaklar bilen gelýär: gurluşykdaky öntümle bahalar, soň üpjünçileriň üýtgeýjileri, soň toglar. Sazlama faýllary öntümleleriň we üpjünçileriň arasynda durýar.

- `MARELLE_CHAT__MODEL=zai/MODEL` — `chat.model` goýýar. Goşanýgy ýokary çyzyk nokadyň ornunda durýar.
- `marelle config show [prefix]` — sazlamalary we her biriniň gaýsy gatlakdan gelendigini sanap görkezer.
- `marelle config explain <key>` — açary goýan her gatlagy, iň aşakdan iň ýokary mürtaçalyga çenli görkezer.

| Açar | Öntümle | Manysy |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ýok | Bilin kim bilen gepleşmelidir. `chat.model` `provider/model` bolup bilýär. |
| `chat.system` | ýok | Ulgam teklipi |
| `chat.max_tokens`, `chat.temperature` | ýok | Goýlan ýok — üpjünçi bilen galdyrylýar |
| `chat.stream` | `true` | Hakim modelin üstünlikli jogabynyň akymy; guraly agent ädimleri häzirki wagtda doly jogaplary ulanýar, gural çagyryşlary bitig gelmeli bolşy üçin |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Oýlanmalar we token sanlary stderr-de |
| `agent.max_turns` | `20` | Ulanyjynyň bir soraýyşyna gural aýlanmalarynyň iň köpü |
| `agent.persist_sessions` | `true` | Ýerli gepleşikleri işledişler aralygynda saklamak |
| `permissions.shell` | `ask` | Çalşy buýruklaryny ýerine yetirmek: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Faýl okamak: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Faýl üýtgetmek: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL göçürip almak: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Faýl gurallar üçin rugsat berilen kökler |
| `permissions.denied_paths` | ýok | Aç-açan bloklanan faýl ýollary |
| `network.connect_timeout_secs` | `15` | Birikmek üçin berilýän wagt |
| `network.idle_timeout_secs` | `300` | Maglumat garaşanda rugsat berilýän däbe |
| `network.proxy` | `system` | `system`, `none`, ýa-da proksi salgysy |
| `network.ca_bundle` | ýok | Ynançly sertifikat beriji guramalaryň goşmaça faýly |
| `network.allow_insecure_http` | `false` | Açaryň adaty `http` arkaly beýleki hosta gitmegine rugsat |
| `update.check` | `true` | Interaktiw seans başlanda GitHub we GitLab-dan täze wersiýany sormak |
| `update.github_repo` | `S-I-L-S/marelle` | Barlag iň soňky goýumy soraýan GitHub `owner/repo` |
| `update.gitlab_repo` | ýok | Şol bir barlag üçin GitLab `owner/repo`. Boş bolsa GitLab söndürilir |
| `update.timeout_secs` | `3` | Barlag bir çeşmä ýana näçe sekunt harjlayar |

Marelle öz ynançly sertifikat beriji guramalary sanawyny alyp ýöre; şonuň üçin ulgamda bolmadyk ýerde-de, Termux-daky ýaly işleýär. `SSL_CERT_FILE` goşulýar.

Täze wersiýa tapsa, seans ony bir gezek belläp geçer — wersiýany, sizdäki goýlany we niräden almalyny. Ilkinji goýum soralar; komitlerini diňe teglär bilen bellän depo iň täze tegi berer. Näme şuňar düşse, sessiz galýar.

## Açarlaryňyz sizniki bolýar

- Sazlamalar açar ýatan değişkeni ýa-da faýly adlandyryr, hiç wagt açaryň özüni däl. `providers.x.api_key` ýaly sazlama ret edilýär.
- Açary `config show`, `config explain` ýa-da `providers` basyp çykarmaýar, we ol serwerden gaýdan her ýalňyş tekstden kesilip alynýar.
- Ol diňe üpjünçiniň salgysyna, bir seri tepinde ugradylýar, we rugsat bermediňizçe hiç wagt ýalňyz `http` arkaly başga maşyna barmaz.
- Ýöneldişlere ýörütdiler gelmeýär; şeýlelikde açar başga ýere ugratylmaýar.

## Diller

Interfeýs iňlis, rus, fransuz we ukrain dillerinde gepleýär. `--lang <code>` ýa-da `MARELLE_LANG` bilen saýlaň; bolmasa `LC_ALL`, `LC_MESSAGES` we `LANG` karar berer. Her habar katalogda ýaşaýar, we bir dil birini goýbalaňda synag ýagşaýar. Maxsat — Ýewraziýanyň her dili, we soňra ýene köp.

## Ýol karta

1. **Gepleşmek.** OpenAI-bilen gabalýan we Anthropic ölçegçileri, hem SSE akymy, hem doly jogaplary okamak bilen. Guraly agent ädimleri häzirki wagtda doly jogaplary ulanýar. *Ýerine ýetirildi.*
2. **Sazlamalar we görnüş.** TOML sazlama faýllary, umumy we taslama boýunça, `config set`, `get` we `unset` bilen, her açaryň düşündirilişi bilen. [Hermes Agent](https://github.com/NousResearch/hermes-agent) ruhyndaky temaly interfeýs: banniere, maglumat paneli, çergeçedäki jogaplar, aylanýan, öntümlede glisiniýa megeden, her reňk HEX bolup goýulýan. *Ýerine ýetirildi.*
3. **Köp model.** Modeliň lakamly atlary, birnäçe modele bir wagtda soraýş we jogaplary deňeşdirmek, ehtiyat zynjyrlar, we olary birleşdirýän hakim model. *Ýerine ýetirildi.*
4. **Häräket etmek.** OpenAI-bilen gabalýan we Anthropic üpjünçiler üçin düzümlenen gural çagyryşlary, ýerli faýl/gözleg/terminal gurallary, URL göçürip almak, ýerli uýkunlyklar we allow/ask/deny rugsatlary; ýazgy her gural çagyryşyny we onuň netijesini görkezer. *Ýerine ýetirildi.*
5. **Ýatlamak.** Atalan gepleşikler işledişler aralygynda ýerli galýar. Uzun möhletli manyly ýat ýene öňünde.
6. **Giňeltmek.** MCP serwerleri, daşary goşmaçalar, kiçi agentler we uýkunlyklara baýrak goldaw.
7. **Ýetmek.** Brauzer awtomatlaşdyrmagy, habarlaşmak kapygylary we gözlenen işler.

Dizayn arkasyndaky oýlanma [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-de.

## Bellenen çäkler

- Üpjünçi we model saýlaýjylary goýulan üpjünçileri we model lakamlaryny sanap görkezer. Marelle entek alystaky model kataloglaryny tapmaýar.
- Saýlaýjylar fokus üçin süýjeni basmagy we çarpyz bilen aýlanmagy goldaýar; ýazgy we girişredaktöri klawiatura arkaly idare edilýär.
- Ctrl-C interaktiw seansdan çykýar. Ýaňy diňe dowam edýän jogaby kesdär.
- Agent aýlanmasy, rugsatlary bilýän ýerli gurallar we berk ýerli seanslar ýerine ýetirildi; emma Marelle öňki-alfa prototip bolup galýar. Häzirki dizaýny we çäklerini okamak üçin [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) okuň.

## Ösüş

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Barlaglar ýoly hiç wagt daşap bilmez: olar `crates/marelle-testkit`-däki ýalan serwer bilen gepleşýärler. GitHub Actions hem Android binarlaryny ýygnar we mälimlenen iň pes Rust wersiýasyny barlaýar. GitLab aýnasy üçin GitLab CI liniýasy bar; seret [docs/GITLAB.md](docs/GITLAB.md).

## Liçõesnya

[AGPL-3.0-only](LICENSE). Üçünji tarap bellikleri [NOTICE](NOTICE)-de. Goýum arhiwleri binar içindeki Rust gutulary üçin `THIRD-PARTY-LICENSES.txt` içine alýar.
