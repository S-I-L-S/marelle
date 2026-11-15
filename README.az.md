```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Terminalınız üçün konfiqurasiya olunan Süni İntellekt agenti.**
Öz API açarlarınızı gətirin: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, yerli Ollama və ya OpenAI ilə uyumlu istənilən server. İlk növbədə Linux və Termux.

[English](README.md) · [العربية](README.ar.md) · Azərbaycanca · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Ön-alfa, versiya 0.1.0-alpha.1.** İcraya verilib: yerli TUI interfeysi, bir neçə təchizatçı ilə söhbət, icazələrdən xəbərdar ajan dövrü, yerli alətlər və davam etdirilə bilən yerli sessiyalar. API-lər və parametrlər hələ dəyişə bilər. Bu prototipdir, OpenCode və ya Hermes ilə funksiyaların bərabərliyi deyil.

*Marelle* — uşaq oyununun fransızca adıdır. Hər kvadrat bir addımdır.

## Quraşdırma

**Android-da Termux.** Hazır ikili faylı [buraxılışlar](https://github.com/S-I-L-S/marelle/releases) səhifəsindən endirin, ya da telefon tərtib edin. Addım-addım bələdçi: [docs/TERMUX.md](docs/TERMUX.md).

**Linux və Rust-un işlədiyi hər yerdə.** Rust 1.85 və ya daha yenisi lazımdır:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ya da anbarı klonlayın və `cargo build --release` işə salın. İkili fayl `target/release/marelle`-dir, təxminən 4 MiB.

## İlk söhbət

Açarı yalnız sizin oxuduğunuz bir faylda saxlayın və Marelle-ə harada olduğunu bildirin. Marelle açarı heç vaxt öz parametrlərində saxlamır.

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

`MODEL` yerinə təchizatçınızın siyahısından model adı qoyun. İstifadənin digər yolları:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Cavab stdout-a gedir. Düşüncələr, jeton sayları, qeydlər və xətalar stderr-ə gedir; beləcə `marelle chat ... > answer.txt` yalnız cavabı saxlayır.

## Ajanın alətləri

Marelle OpenAI ilə uyumlu və Anthropic API-ləri vasitəsilə strukturlaşdırılmış alət tərifləri göndərir. Daxili alət dəsti:

| Alət | Məqsəd |
| --- | --- |
| `terminal` | Başlanğıc qovluğundan qabıq əmrini icra etmək |
| `read_file`, `write_file`, `edit_file` | UTF-8 mətn fayllarını oxumaq və dəyişmək |
| `list_directory`, `search_text` | Layihə fayllarını gözdən keçirmək və axtarmaq |
| `fetch_url` | İctimai HTTP(S) səhifəsini mətn kimi gətirmək (`curl` tələb olunur) |
| `list_skills`, `load_skill` | Yerli `SKILL.md` təlimatlarını tapmaq və yükləmək |

`/tools` əmri alət siyahısını, `/skills` mövcud bacarıqları göstərir. Layihə rəhbərliyi `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` və seçilmiş `.marelle/` fayllarından, icazəli yollardadırlarsa, yüklənir.

Fayl alətləri `permissions.allowed_paths` ilə məhdudlaşır (susmaya görə: layihənin cari qovluğu) və `permissions.denied_paths`-i gözləyir. Oxumaq susmaya görə icazəlidir; fayl dəyişmələri, qabıq əmrləri və şəbəkə yükləmələri əvvəl soruşur. Qabıq icazəsi məxsusi maşındakı əmrlərə aiddir və fayl sisteminin qum qabı deyil: modellərə əmr icrasını bilərələnə vicdan etsəmirsinizsə `ask`-da saxlayın.

Ajan `agent.max_turns` alət turundan sonra dayanır. Alət cavabları etibarsız məzmun sayılır. Sorğular modelin tam cavablarından istifadə edir; beləcə alət arqumentləri yarım axın parçalarından yığılmır.

## Sessiyalar

Söhbətlər susmaya görə yerli yazılır; alət çağırış tarixçəsi ilə model və sessiya parametrləri də daxil. Unix-də sessiya qovluğunun rejimi `0700`, JSON faylınınkı `0600`-dür. Fayl `$XDG_STATE_HOME/marelle/sessions.json`-da, ya da `XDG_STATE_HOME` qoyulmadıqda `~/.local/state/marelle/sessions.json`-da saxlanılır. API açarları bu fayla heç vaxt yazılmır. Qalıcılığı bununla söndürün:

```sh
marelle config set agent.persist_sessions false
```

Çıxış kodları: `0` müvəffəqiyyət, `1` təchizatçı və ya şəbəkə xətası, `2` istifadə və ya tənzimləmə xətası.

## Təchizatçılar

| id | Protokol | Açar dəyişəni | |
| --- | --- | --- | --- |
| `zai` | OpenAI ilə uyumlu | `ZAI_API_KEY` | Z.ai (Zhipu GLM), beynəlxalq |
| `zai_coding` | OpenAI ilə uyumlu | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic ilə uyumlu son nöqtə |
| `zhipu` | OpenAI ilə uyumlu | `ZHIPUAI_API_KEY` | Zhipu BigModel, Çin |
| `zhipu_coding` | OpenAI ilə uyumlu | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Çin |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI ilə uyumlu | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI ilə uyumlu | `OPENROUTER_API_KEY` | bir açar arxasında çoxlu model |
| `ollama` | OpenAI ilə uyumlu | yoxdur | Bu maşında Ollama |

Z.ai və Zhipu son nöqtələri satıcının öz səhifələrindən yox, üçüncü tərəf sənədlərindən gəlir. Bunları tarifinizlə müqayisə edin və fərqlənirsə istənilən sahəni üstünə yazın.

Təchizatçının hər sahəsi dəyişə bilər; yeni təchizatçı isə sadəcə bir ad və ünvandır:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Təchizatçının öteki sahələri: `kind` (`openai-compatible` və ya `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` və `extra_body`. `api_key_env`-in adını daşıyan dəyişən qoyulubsa, o qalib gəlir; yoxsa fayl oxunur.

## Parametrlər

Parametrlər təbəqələrlə gəlir: daxili susma dəyərləri, sonra mühit dəyişənləri, sonra bayraqlar. Parametr faylları susma dəyərləri ilə mühitin arasına girir.

- `MARELLE_CHAT__MODEL=zai/MODEL` `chat.model`-i müəyyən edir. Cüt altdan xətt nöqtənin yerinə durur.
- `marelle config show [prefix]` parametrləri və hər birinin gəldiyi təbəqəni sıralayır.
- `marelle config explain <key>` açarı qoyan hər təbəqəni, ən altdan ən üstə göstərir.

| Açar | Susma | Mənası |
| --- | --- | --- |
| `chat.provider`, `chat.model` | yoxdur | Kiminlə danışmaq. `chat.model` `provider/model` ola bilər. |
| `chat.system` | yoxdur | Sistem istemi |
| `chat.max_tokens`, `chat.temperature` | yoxdur | Qoyulmayıb — təchizatçıya buraxılır |
| `chat.stream` | `true` | Hakimin qeyri-müəyyən cavabının axması; alətli ajan turları hələlik tam cavablardan istifadə edir, ki alət çağırışları sağ qalsın |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Düşüncələr və jeton sayları stderr-də |
| `agent.max_turns` | `20` | İstifadəçi sorğusu başına maksimum alət turu |
| `agent.persist_sessions` | `true` | Yerli söhbətləri işə salmalar arasında saxlamaq |
| `permissions.shell` | `ask` | Qabıq əmrlərinin icrası: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Fayl oxumaq: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Fayl dəyişmək: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL endirmək: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Fayl alətləri üçün icazəli köklər |
| `permissions.denied_paths` | yoxdur | Açıqcasına bloklanmış fayl yolları |
| `network.connect_timeout_secs` | `15` | Qoşulmaq üçün verilən vaxt |
| `network.idle_timeout_secs` | `300` | Məlumat gözləyərkən verilən sükut |
| `network.proxy` | `system` | `system`, `none`, və ya proksi ünvanı |
| `network.ca_bundle` | yoxdur | Etibarlı sertifikat mərkəzlərinin əlavə faylı |
| `network.allow_insecure_http` | `false` | Açarın quru `http` üzərindən yad bir hosta yol verilməsi |
| `update.check` | `true` | İnteraktiv sessiya başlayanda GitHub və GitLab-a daha yeni versiya barədə soruşmaq |
| `update.github_repo` | `S-I-L-S/marelle` | Yoxlamanın ən yeni buraxılışı soruşduğu GitHub `owner/repo` |
| `update.gitlab_repo` | yoxdur | Eyni yoxlama üçün GitLab `owner/repo`. Boş olsa GitLab söndürülür |
| `update.timeout_secs` | `3` | Yoxlamanın bir mənbəyə nə qədər saniyə xərcləyə biləcəyi |

Marelle öz etibarlı sertifikat mərkəzləri siyahısını daşıyır; onun üçün sistemdə belə bir siyahı olmadıqda da, Termux-da da işləyir. `SSL_CERT_FILE` əlavə edər.

Daha yeni versiya tapılanda sessiya onu bir dəfə xatırladacaq — versiyanı, sizdə olanı və haradan alınacağını. Əvvəlcə buraxılış soruşulur; komitlərini yalnız etiketləyən anbar ən yeni etiketi verir. Nə səhv etsin, susaraq qalır.

## Açarlarınız sizin olaraq qalır

- Parametrlər açarı daşıyan dəyişəni və ya faylı adlandırır, asla açarın özünü yox. `providers.x.api_key` kimi parametr rədd edilir.
- Açarı `config show`, `config explain` və ya `providers` basmır, və o, serverdən gələn hər xəta mətnindən kəsilir.
- O yalnız təchizatçının ünvanına, bir başlıqda göndərilir və sizə icazə vermədiyinizcə asla quru `http` üzərindən başqa maşına getmir.
- Yönəltmələr izlənmir; beləcə açar başqa yerə çatdırıla bilmir.

## Dillər

İnterfeys İngilis, Rus, Fransız və Ukrayna dillərində danışır. `--lang <code>` və ya `MARELLE_LANG` ilə seçin; yoxsa `LC_ALL`, `LC_MESSAGES` və `LANG` qərar verir. Hər mesaj kataloqda yaşayır və bir dil birini qoyub getsə, sınaq düşər. Məqsəd Avrasiyanın hər dili, sonra artıq.

## Yol xəritəsi

1. **Danışmaq.** OpenAI ilə uyumlu və Anthropic adapterləri, həm SSE axını, həm də tam cavabların oxunması ilə. Alətli ajan turları hələlik tam cavablardan istifadə edir. *Hazırdır.*
2. **Parametrlər və görüntü.** TOML parametr faylları, ümumi və layihə üzrə, `config set`, `get` və `unset` ilə, hər açarın təsviri ilə. [Hermes Agent](https://github.com/NousResearch/hermes-agent) ruhunda mövzulu interfeys: banner, məlumat paneli, çərçivəli cavablar, fırlanan, susmaya görə glisinya morunda, istənilən rəng HEX kimi. *Hazırdır.*
3. **Çoxlu model.** Model ləqəbləri, bir neçə modele eyni vaxtda sual vermək və cavabları müqayisə etmək, ehtiyat zəncirləri, və onları birləşdirən hakim bir model. *Hazırdır.*
4. **Hərəkət etmək.** OpenAI ilə uyumlu və Anthropic təchizatçıları üçün strukturlaşdırılmış alət çağırışları, yerli fayl/axtarış/terminal alətləri, URL endirmə, yerli bacarıqlar və allow/ask/deny icazələri; yazı hər alət çağırışını və onun nəticəsini göstərir. *Hazırdır.*
5. **Xatırlamaq.** Adlandırılmış söhbətlər işə salmalar arasında yerli qalır. Uzunmüddətli mənaca yaddaş hələ öndədir.
6. **Genişləndirmək.** MCP serverləri, xarici qoşmalar, alt-ajanlar və bacarıqlara daha zəngin dəstək.
7. **Çatmaq.** Brauzerin avtomatlaşdırılması, mesajlaşma darvazaları və planlaşdırılan tapşırıqlar.

Dizayn arxasındakı düşüncə [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-dadır.

## Məlum məhdudluqlar

- Təchizatçı və model seçiciləri qoyulmuş təchizatçıları və model ləqəblərini sadalayır. Marelle hələ uzaqdakı model kataloqlarını aşkar etmir.
- Seçicilər fokus üçün siçan klikini və çarxla fırlatmayı dəstəkləyir; yazı və girişredaktoru klaviaturayla idarə olunur.
- Ctrl-C interaktiv sessiyadan çıxır. Hələ yalnız cari cavabı kəsmir.
- Ajan dövrü, icazələrdən xəbərdar yerli alətlər və davamlı yerli sessiyalar həyata keçirilib; amma Marelle ön-alfa prototip olaraq qalır. Hazırkı dizayn və sərhədlər üçün [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)-i oxuyun.

## İnkişaf

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Sınaqlar şəbəkəyə heç vaxt toxunmur: onlar `crates/marelle-testkit`-dəki saxta serverlə danışırlar. GitHub Actions həmçinin Android ikililərini tərtib edir və bildirilən ən aşağı Rust versiyasını yoxlayır. GitLab güzgüsü üçün GitLab CI xətti var; baxınız [docs/GITLAB.md](docs/GITLAB.md).

## Lisenziya

[AGPL-3.0-only](LICENSE). Üçüncü tərəf bildirişləri [NOTICE](NOTICE)-dadır. Buraxılış arxivləri ikilinin içindəki Rust qutuları üçün `THIRD-PARTY-LICENSES.txt` ehtiva edir.
