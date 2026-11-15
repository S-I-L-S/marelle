```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Terminaliniz için yapılandırılabilir bir YZ ajanı.**
Kendi API anahtarlarınızı getirin: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, yerel bir Ollama ya da herhangi bir OpenAI uyumlu sunucu. Önce Linux ve Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · Türkçe · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Ön-alfa, sürüm 0.1.0-alpha.1.** Uygulandı: yerel TUI arayüzü, çoklu sağlayıcı sohbeti, izin duyarlı ajan döngüsü, yerel araçlar ve kaldığı yerden devam edilebilir yerel oturumlar. API'ler ve ayarlar henüz değişebilir. Bu bir prototiptir, OpenCode ya da Hermes ile özellik eşitliği değildir.

*Marelle*, seksek oyununun Fransızca adıdır. Her kare bir adımdır.

## Kurulum

**Android'da Termux.** Hazır ikili dosyayı [sürümler](https://github.com/S-I-L-S/marelle/releases) sayfasından indirin ya da telefonda derleyin. Adım adım kılavuz: [docs/TERMUX.md](docs/TERMUX.md).

**Linux ve Rust'un çalıştığı her yer.** Rust 1.85 ya da daha yenisi:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ya da depoyu klonlayın ve `cargo build --release` komutunu çalıştırın. İkili dosya `target/release/marelle`, yaklaşık 4 MiB.

## İlk sohbet

Anahtarı yalnızca sizin okuduğunuz bir dosyada tutun ve Marelle'e nerede olduğunu söyleyin. Marelle bir anahtarı asla kendi ayarlarında saklamaz.

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

`MODEL` yerine sağlayıcınızın listesinden bir model adı koyun. Kullanımın diğer yolları:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Yanıt stdout'a gider. Düşünmeler, jeton sayıları, notlar ve hatalar stderr'e gider; böylece `marelle chat ... > answer.txt` yalnızca yanıtı kaydeder.

## Ajanın araçları

Marelle, OpenAI uyumlu ve Anthropic API'leri üzerinden yapılandırılmış araç tanımları gönderir. Dahili araç takımı:

| Araç | Amaç |
| --- | --- |
| `terminal` | Başlatma dizininden bir kabuk komutu çalıştırmak |
| `read_file`, `write_file`, `edit_file` | UTF-8 metin dosyalarını okumak ve değiştirmek |
| `list_directory`, `search_text` | Proje dosyalarını incelemek ve aramak |
| `fetch_url` | Halka açık bir HTTP(S) sayfasını metin olarak getirmek (`curl` gerekir) |
| `list_skills`, `load_skill` | Yerel `SKILL.md` talimatlarını bulmak ve yüklemek |

`/tools` komutu araç listesini, `/skills` kullanılabilir becerileri gösterir. Proje yönergeleri `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` ve seçili `.marelle/` dosyalarından, izin verilen yollardaysalar, yüklenir.

Dosya araçları `permissions.allowed_paths` ile sınırlıdır (öntanımlı: geçerli proje dizini) ve `permissions.denied_paths`'i gözetir. Okuma öntanımlı olarak izinlidir; dosya değişiklikleri, kabuk komutları ve ağ indirmeleri önce sorar. Kabuk izni ana makinedeki komutlara uygulanır ve bir dosya sistemi kumu havuzu değildir: komut yürütmeyi modellere bilinçli olarak emanet etmediyseniz `ask`'ta tutun.

Ajan `agent.max_turns` araç turundan sonra durur. Araç yanıtları güvenilmez içerik sayılır. İstemler tüm model yanıtlarını kullanır; böylece araç argümanları yarım akış parçalarından birleştirilmez.

## Oturumlar

Konuşmalar öntanımlı olarak yerel kaydedilir; araç çağrısı geçmişi ile model ve oturum ayarları dahil. Unix'te oturum klasörünün kipi `0700`, JSON dosyasınınki `0600`'dür. Dosya `$XDG_STATE_HOME/marelle/sessions.json` içinde, ya da `XDG_STATE_HOME` atanmamışsa `~/.local/state/marelle/sessions.json` içinde saklanır. API anahtarları bu dosyaya asla yazılmaz. Kalıcılığı şununla kapatın:

```sh
marelle config set agent.persist_sessions false
```

Çıkış kodları: `0` başarı, `1` sağlayıcı ya da ağ hatası, `2` kullanım ya da yapılandırma hatası.

## Sağlayıcılar

| id | Protokol | Anahtar değişkeni | |
| --- | --- | --- | --- |
| `zai` | OpenAI uyumlu | `ZAI_API_KEY` | Z.ai (Zhipu GLM), uluslararası |
| `zai_coding` | OpenAI uyumlu | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic uyumlu uç nokta |
| `zhipu` | OpenAI uyumlu | `ZHIPUAI_API_KEY` | Zhipu BigModel, Çin |
| `zhipu_coding` | OpenAI uyumlu | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Çin |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI uyumlu | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI uyumlu | `OPENROUTER_API_KEY` | tek anahtar ardında birçok model |
| `ollama` | OpenAI uyumlu | yok | Bu makinede Ollama |

Z.ai ve Zhipu uç noktaları satıcının kendi sayfalarından değil, üçüncü parti belgelerinden gelir. Bunları tarifenizle karşılaştırın ve farklılık gösterirlerse herhangi bir alanı üzerine yazın.

Sağlayıcının her alanı değiştirilebilir; yeni bir sağlayıcı yalnızca bir adla bir adrestir:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Öteki sağlayıcı alanları: `kind` (`openai-compatible` ya da `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` ve `extra_body`. `api_key_env`'in adını taşıyan değişken atanmışsa o kazanır; yoksa dosya okunur.

## Ayarlar

Ayarlar katmanlar halinde gelir: yerleşik öntanımlılar, sonra ortam değişkenleri, sonra bayraklar. Ayar dosyaları öntanımlılar ile ortamın arasına girer.

- `MARELLE_CHAT__MODEL=zai/MODEL`, `chat.model`'i belirler. Çift alt çizgi nokta yerine geçer.
- `marelle config show [prefix]` ayarları ve her birinin geldiği katmanı sıralar.
- `marelle config explain <key>` bir anahtarı belirleyen her katmanı, en alt öncelikten en üste gösterir.

| Anahtar | Öntanımlı | Anlam |
| --- | --- | --- |
| `chat.provider`, `chat.model` | yok | Kiminle konuşulacak. `chat.model` `provider/model` olabilir. |
| `chat.system` | yok | Sistem istemi |
| `chat.max_tokens`, `chat.temperature` | yok | Atanmamış — sağlayıcıya bırakılır |
| `chat.stream` | `true` | Hakem yanıtının akıtılması; araçlı ajan turları şimdilik tüm yanıtları kullanır, böylece araç çağrıları bozulmadan kalır |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Düşünmeler ve jeton sayıları stderr'de |
| `agent.max_turns` | `20` | Kullanıcı isteği başına en fazla araç turu |
| `agent.persist_sessions` | `true` | Yerel konuşmaları çalıştırmalar arasında kaydet |
| `permissions.shell` | `ask` | Kabuk komutu yürütme: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Dosya okuma: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Dosya değiştirme: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL indirme: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Dosya araçları için izinli kökler |
| `permissions.denied_paths` | yok | Açıkça engellenmiş dosya yolları |
| `network.connect_timeout_secs` | `15` | Bağlanmak için tanınan süre |
| `network.idle_timeout_secs` | `300` | Veri beklerken tanınan sessizlik |
| `network.proxy` | `system` | `system`, `none`, ya da bir vekil adresi |
| `network.ca_bundle` | yok | Güvenilir sertifika kurumlarının ek bir dosyası |
| `network.allow_insecure_http` | `false` | Bir anahtarın çıplak `http` üzerinden yabancı bir sunucuya gitmesine izin ver |
| `update.check` | `true` | Etkileşimli bir oturum başlarken GitHub ve GitLab'a daha yeni bir sürümü sormak |
| `update.github_repo` | `S-I-L-S/marelle` | Denetimin en yeni sürümü sorduğu GitHub `owner/repo` |
| `update.gitlab_repo` | yok | Aynı denetim için GitLab `owner/repo`. Boşsa GitLab kapatılır |
| `update.timeout_secs` | `3` | Denetimin tek bir kaynağa harcayabileceği saniye sayısı |

Marelle kendi güvenilir sertifika kurumu listesini taşır; bu yüzden sistemin böyle bir listeye sahip olmadığı yerlerde, Termux'ta bile çalışır. `SSL_CERT_FILE` daha fazlasını ekler.

Daha yeni bir sürüm bulunduğunda oturum onu bir kez anar — sürümü, sizde olanı ve nereden alınacağını. Önce bir yayın sorulur; commit'lerini yalnızca etiketleyen bir depo en yeni etiketini verir. Ne yanlış giderse sessizce kalır.

## Anahtarlarınız sizde kalır

- Ayarlar anahtarı taşıyan bir değişkeni ya da dosyayı adlandırır, asla anahtarın kendisini değil. `providers.x.api_key` gibi bir ayar reddedilir.
- Anahtar `config show`, `config explain` ya da `providers` tarafından basılmaz ve bir sunucudan gelen her hata metninden ayıklanır.
- Yalnızca sağlayıcının adresine, tek bir başlıkta gönderilir ve izin vermediğiniz sürece asla çıplak `http` üzerinden başka bir makineye gitmez.
- Yönlendirmeler izlenmez; böylece bir anahtar başka bir yere iletilemez.

## Diller

Arayüz İngilizce, Rusça, Fransızca ve Ukraynaca konuşur. `--lang <code>` ya da `MARELLE_LANG` ile seçin; yoksa `LC_ALL`, `LC_MESSAGES` ve `LANG` karar verir. Her ileti bir katalogda yaşar ve bir dil birini atarsa bir test düşer. Hedef, Avrasya'nın her dili ve sonra fazlası.

## Yol haritası

1. **Konuşmak.** OpenAI uyumlu ve Anthropic uyarlayıcıları, hem SSE akıtma hem tam yanıt çözümü ile. Araçlı ajan turları şimdilik tüm yanıtları kullanır. *Tamam.*
2. **Ayarlar ve görünüm.** TOML ayar dosyaları, genel ve proje başına, `config set`, `get` ve `unset` ile her anahtarın tanımıyla. [Hermes Agent](https://github.com/NousResearch/hermes-agent) ruhunda temalı bir arayüz: pankart, bilgi paneli, çerçeveli yanıtlar, çevirgeç, öntanımlı olarak glisinye morunda, herhangi bir renk HEX olarak atanabilir. *Tamam.*
3. **Çok model.** Model lakapları, birkaç modele aynı anda sorup yanıtları karşılaştırmak, yedek zincirleri ve bunları birleştiren hakem bir model. *Tamam.*
4. **Eylemek.** OpenAI uyumlu ve Anthropic sağlayıcılar için yapılandırılmış araç çağrıları, yerel dosya/arama/terminal araçları, URL indirme, yerel beceriler ve allow/ask/deny izinleri; döküm her araç çağrısını ve sonucunu gösterir. *Tamam.*
5. **Hatırlamak.** Adlı konuşmalar çalıştırmalar arasında yerelde kalır. Uzun soluklu anlamsal bellek henüz önde.
6. **Genişletmek.** MCP sunucuları, dış eklentiler, alt-ajanlar ve zengin beceri desteği.
7. **Ulaşmak.** Tarayıcı otomasyonu, mesajlaşma kapıları ve zamanlanmış görevler.

Tasarımın ardındaki düşünce [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) içinde.

## Bilinen sınırlar

- Sağlayıcı ve model seçicileri atanmış sağlayıcıları ve model lakaplarını sıralar. Marelle henüz uzaktaki model kataloglarını keşfetmez.
- Seçiciler odak için fare tıklamasını ve tekerlekle kaydırmayı destekler; döküm ve girişte yazı kılavyeyle sürülür.
- Ctrl-C etkileşimli oturumdan çıkar. Henüz yalnızca geçerli yanıtı kesmez.
- Ajan döngüsü, izin duyarlı yerel araçlar ve kalıcı yerel oturumlar uygulanmıştır; ama Marelle bir ön-alfa prototipi olarak kalır. Şimdiki tasarım ve sınırlar için [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)'i okuyun.

## Geliştirme

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Sınamalar ağa hiç dokunmaz: `crates/marelle-testkit` içindeki sahte bir sunucuyla konuşurlar. GitHub Actions ayrıca Android ikililerini derler ve bildirilen en düşük Rust sürümünü denetler. GitLab aynası için bir GitLab CI hattı vardır; bakınız [docs/GITLAB.md](docs/GITLAB.md).

## Lisans

[AGPL-3.0-only](LICENSE). Üçüncü parti bildirimleri [NOTICE](NOTICE) içinde. Sürüm arşivleri ikilinin içindeki Rust sandıkları için `THIRD-PARTY-LICENSES.txt` içerir.
