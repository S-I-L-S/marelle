```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Agen AI yang dapat dikonfigurasi untuk terminal Anda.**
Bawa API key Anda sendiri: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, Ollama lokal, atau server apa pun yang kompatibel dengan OpenAI. Linux dan Termux lebih dulu.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
Bahasa Indonesia · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, versi 0.1.0-alpha.1.** TUI native, chat multi-provider, agent loop yang sadar izin, alat lokal, dan sesi lokal yang dapat dilanjutkan sudah diimplementasikan. API dan pengaturan masih bisa berubah. Ini adalah prototipe, bukan kesetaraan fitur dengan OpenCode atau Hermes.

*Marelle* adalah bahasa Prancis untuk engklek (hopscotch). Setiap petak adalah satu langkah.

## Instal

**Termux di Android.** Unduh binary yang sudah jadi dari [Releases](https://github.com/S-I-L-S/marelle/releases), atau build langsung di ponsel. Langkah demi langkah: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, dan di mana pun Rust bisa berjalan.** Rust 1.85 atau lebih baru:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Atau clone repositori lalu jalankan `cargo build --release`. Binary-nya adalah `target/release/marelle`, sekitar 4 MiB.

## Chat pertama

Simpan key dalam file yang hanya bisa Anda baca, dan beri tahu Marelle letaknya. Marelle tidak pernah menyimpan key di pengaturannya.

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

Ganti `MODEL` dengan nama model dari daftar provider Anda. Cara lain memakainya:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Jawaban keluar ke stdout. Reasoning, hitungan token, catatan, dan error keluar ke stderr, sehingga `marelle chat ... > answer.txt` hanya menyimpan jawabannya.

## Alat agen

Marelle mengirim definisi alat terstruktur melalui API yang kompatibel dengan OpenAI dan API Anthropic. Kumpulan alat bawaannya:

| Alat | Tujuan |
| --- | --- |
| `terminal` | Menjalankan perintah shell dari direktori peluncuran |
| `read_file`, `write_file`, `edit_file` | Membaca dan mengubah file teks UTF-8 |
| `list_directory`, `search_text` | Memeriksa dan mencari file proyek |
| `fetch_url` | Mengambil halaman HTTP(S) publik sebagai teks (perlu `curl`) |
| `list_skills`, `load_skill` | Menemukan dan memuat instruksi `SKILL.md` lokal |

Gunakan `/tools` untuk melihat daftar alat dan `/skills` untuk melihat daftar skill yang tersedia. Panduan proyek dimuat dari `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules`, dan file `.marelle/` terpilih selama berada di dalam path yang diizinkan.

Alat file terbatas pada `permissions.allowed_paths` (bawaan: direktori proyek saat ini) dan menghormati `permissions.denied_paths`. Membaca diizinkan secara bawaan; perubahan file, perintah shell, dan pengambilan jaringan akan bertanya sebelum dijalankan. Izin shell berlaku untuk perintah host dan bukan sandbox filesystem: biarkan di `ask` kecuali Anda sengaja memercayai model untuk menjalankan perintah.

Agen berhenti setelah `agent.max_turns` putaran tool. Respons tool diperlakukan sebagai konten yang tidak tepercaya. Permintaan menggunakan respons model yang lengkap sehingga argumen tool tidak dirakit dari potongan stream yang tidak utuh.

## Sesi

Percakapan disimpan secara lokal secara bawaan, termasuk riwayat pemanggilan tool dan pengaturan model/sesi. Di Unix, direktori sesi ber-mode `0700` dan file JSON ber-mode `0600`. File disimpan di `$XDG_STATE_HOME/marelle/sessions.json`, atau `~/.local/state/marelle/sessions.json` bila `XDG_STATE_HOME` tidak diatur. API key tidak pernah ditulis ke file ini. Nonaktifkan persistensi dengan:

```sh
marelle config set agent.persist_sessions false
```

Kode keluar: `0` sukses, `1` error provider atau jaringan, `2` error penggunaan atau konfigurasi.

## Providers

| id | Protokol | Variabel key | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internasional |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, endpoint kompatibel Anthropic |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel, Tiongkok |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Tiongkok |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | banyak model di balik satu key |
| `ollama` | OpenAI-compatible | none | Ollama di mesin ini |

Endpoint Z.ai dan Zhipu berasal dari dokumentasi pihak ketiga, bukan dari halaman vendor itu sendiri. Cocokkan dengan paket Anda, dan timpa field apa pun bila berbeda.

Setiap field provider bisa diubah, dan provider baru hanyalah sebuah nama dengan sebuah alamat:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Field provider lainnya: `kind` (`openai-compatible` atau `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage`, dan `extra_body`. Bila variabel yang dinamai oleh `api_key_env` terpasang, variabel itu yang menang; jika tidak, file dibaca.

## Pengaturan

Pengaturan datang berlapis: bawaan tertanam, lalu variabel lingkungan, lalu flag. File pengaturan menyisip di antara bawaan dan variabel lingkungan pada tahap berikutnya.

- `MARELLE_CHAT__MODEL=zai/MODEL` mengatur `chat.model`. Underscore ganda mewakili satu titik.
- `marelle config show [prefix]` mendaftar pengaturan beserta lapisan asal masing-masing.
- `marelle config explain <key>` menampilkan setiap lapisan yang mengatur sebuah key, dari prioritas terendah ke tertinggi.

| Key | Bawaan | Arti |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | Dengan siapa berbicara. `chat.model` bisa berupa `provider/model`. |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | Diserahkan ke provider bila tidak diatur |
| `chat.stream` | `true` | Stream respons judge opsional; giliran agen yang memakai tool saat ini menggunakan respons lengkap agar pemanggilan tool tetap utuh |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Reasoning dan hitungan token di stderr |
| `agent.max_turns` | `20` | Putaran tool maksimum per permintaan pengguna |
| `agent.persist_sessions` | `true` | Menyimpan percakapan lokal antar-run |
| `permissions.shell` | `ask` | Eksekusi perintah shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Membaca file: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Mengubah file: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Mengambil URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Akar yang diizinkan untuk alat file |
| `permissions.denied_paths` | none | Path file yang diblokir secara eksplisit |
| `network.connect_timeout_secs` | `15` | Waktu yang diizinkan untuk tersambung |
| `network.idle_timeout_secs` | `300` | Keheningan yang diizinkan saat menunggu data |
| `network.proxy` | `system` | `system`, `none`, atau URL proxy |
| `network.ca_bundle` | none | File tambahan berisi certificate authority tepercaya |
| `network.allow_insecure_http` | `false` | Mengizinkan key dikirim lewat `http` biasa ke host non-lokal |
| `update.check` | `true` | Menanyakan GitHub dan GitLab tentang versi lebih baru saat sesi interaktif dimulai |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` yang ditanya pemeriksaan tentang rilis terbaru |
| `update.gitlab_repo` | none | GitLab `owner/repo` yang juga ditanya pemeriksaan. Kosong menonaktifkan GitLab |
| `update.timeout_secs` | `3` | Detik yang boleh dipakai pemeriksaan untuk satu sumber |

Marelle membawa daftar certificate authority tepercayanya sendiri, sehingga tetap bekerja di tempat yang tidak memilikinya, seperti Termux. `SSL_CERT_FILE` menambah lagi.

Ketika versi lebih baru ditemukan, sesi menyebutnya sekali — versinya, yang Anda punya, dan di mana mendapatkannya. Rilis ditanya lebih dulu; repositori yang hanya menandai commit-nya dengan tag beralih ke tag terbarunya. Apa pun yang bermasalah tetap diam.

## Key Anda tetap milik Anda

- Pengaturan hanya menyebut nama variabel atau file yang menyimpan key, tidak pernah key-nya. Pengaturan seperti `providers.x.api_key` ditolak.
- Key tidak dicetak oleh `config show`, `config explain`, maupun `providers`, dan dibuang dari teks error apa pun yang kembali dari server.
- Key hanya dikirim ke alamat provider, dalam satu header, dan tidak pernah lewat `http` biasa ke mesin lain kecuali Anda mengizinkannya.
- Redirect tidak diikuti, sehingga key tidak bisa diteruskan ke tempat lain.

## Bahasa

Antarmuka berbicara bahasa Inggris, Rusia, Prancis, dan Ukraina. Pilih dengan `--lang <code>` atau `MARELLE_LANG`; jika tidak, `LC_ALL`, `LC_MESSAGES`, dan `LANG` yang menentukan. Setiap pesan tinggal dalam satu catalog, dan sebuah test akan gagal bila satu bahasa kehilangan satu pesan. Tujuannya adalah setiap bahasa di Eurasia, lalu lebih banyak lagi.

## Peta jalan

1. **Bicara.** Adapter kompatibel OpenAI dan Anthropic, dengan streaming SSE maupun dekode respons lengkap. Giliran agen yang memakai tool saat ini menggunakan respons lengkap. *Selesai.*
2. **Pengaturan dan tampilan.** File pengaturan TOML, global dan per proyek, dengan `config set`, `get`, `unset` dan deskripsi untuk setiap key. Antarmuka bertema dalam semangat [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, panel informasi, jawaban berbingkai, spinner, ungu wisteria secara bawaan dengan warna apa pun dapat diatur sebagai HEX. *Selesai.*
3. **Banyak model.** Alias model, bertanya ke beberapa model sekaligus dan membandingkan jawabannya, rantai fallback, dan sebuah model judge yang menggabungkannya. *Selesai.*
4. **Bertindak.** Pemanggilan tool terstruktur untuk provider kompatibel OpenAI dan Anthropic, alat file/pencarian/terminal lokal, pengambilan URL, skill lokal, dan izin allow/ask/deny, dengan transkrip yang menampilkan setiap pemanggilan tool beserta hasilnya. *Selesai.*
5. **Mengingat.** Percakapan bernama bertahan secara lokal antar-run. Memori semantik jangka panjang masih di depan.
6. **Memperluas.** Server MCP, plugin eksternal, subagen, dan dukungan skill yang lebih kaya.
7. **Menjangkau.** Otomasi browser, gateway pesan, dan tugas terjadwal.

Alasan di balik desain ada di [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Keterbatasan yang diketahui

- Pemilih provider dan model menampilkan provider terkonfigurasi dan alias model. Marelle belum dapat menemukan katalog model jarak jauh.
- Pemilih mendukung klik mouse untuk fokus dan pengguliran roda; transkrip dan editor masukan tetap dikendalikan keyboard.
- Ctrl-C keluar dari sesi interaktif. Ia belum bisa membatalkan hanya respons saat ini.
- Agent loop, alat lokal yang sadar izin, dan sesi lokal yang bertahan sudah diimplementasikan, tetapi Marelle tetap prototipe pre-alpha; lihat [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) untuk desain dan batas saat ini.

## Pengembangan

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Test tidak pernah menyentuh jaringan: mereka berbicara dengan mock server di `crates/marelle-testkit`. GitHub Actions juga membangun binary Android dan memeriksa versi Rust minimum yang dinyatakan. Pipeline GitLab CI disediakan untuk mirror GitLab; lihat [docs/GITLAB.md](docs/GITLAB.md).

## Lisensi

[AGPL-3.0-only](LICENSE). Pemberitahuan pihak ketiga ada di [NOTICE](NOTICE). Arsip rilis menyertakan `THIRD-PARTY-LICENSES.txt` untuk crate Rust di dalam binary.
