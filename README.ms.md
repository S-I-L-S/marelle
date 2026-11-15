```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Agen AI yang boleh dikonfigurasi untuk terminal anda.**
Bawa API key anda sendiri: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, Ollama setempat, atau mana-mana pelayan yang serasi dengan OpenAI. Linux dan Termux didahulukan.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · Bahasa Melayu · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pra-alpha, versi 0.1.0-alpha.1.** TUI natif, chat multi-provider, agent loop yang peka kebenaran (permission-aware), alatan setempat dan sesi setempat yang boleh disambung semula telah dilaksanakan. API dan tetapan masih boleh berubah. Ini adalah prototaip, bukan kesamarataan ciri dengan OpenCode atau Hermes.

*Marelle* ialah perkataan bahasa Perancis untuk hopscotch, iaitu permainan melompat petak. Setiap petak ialah satu langkah.

## Pemasangan

**Termux pada Android.** Muat turun binary yang sedia dibina daripada [Releases](https://github.com/S-I-L-S/marelle/releases), atau bina pada telefon itu sendiri. Langkah demi langkah: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, dan di mana-mana sahaja Rust berjalan.** Rust 1.85 atau lebih baharu:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Atau klon repositori dan jalankan `cargo build --release`. Binary ialah `target/release/marelle`, kira-kira 4 MiB.

## Chat pertama

Simpan key dalam fail yang hanya anda boleh baca, dan beritahu Marelle di mana ia berada. Marelle tidak pernah menyimpan key dalam tetapannya.

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

Gantikan `MODEL` dengan nama model daripada senarai provider anda. Cara lain menggunakannya:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Jawaban keluar ke stdout. Reasoning, kiraan token, nota dan ralat keluar ke stderr, jadi `marelle chat ... > answer.txt` hanya menyimpan jawaban.

## Alatan agen

Marelle menghantar takrif alat berstruktur melalui API serasi OpenAI dan API Anthropic. Set alat terbina dalam ialah:

| Alat | Tujuan |
| --- | --- |
| `terminal` | Menjalankan arahan shell daripada direktori pelancaran |
| `read_file`, `write_file`, `edit_file` | Membaca dan mengubah fail teks UTF-8 |
| `list_directory`, `search_text` | Memeriksa dan mencari fail projek |
| `fetch_url` | Mendapatkan halaman HTTP(S) awam sebagai teks (memerlukan `curl`) |
| `list_skills`, `load_skill` | Menemui dan memuatkan arahan `SKILL.md` setempat |

Gunakan `/tools` untuk menyemak senarai alat dan `/skills` untuk menyenaraikan skill yang tersedia. Panduan projek dimuatkan daripada `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` dan fail `.marelle/` terpilih selagi ia berada dalam laluan yang dibenarkan.

Alatan fail terhad kepada `permissions.allowed_paths` (lalai: direktori projek semasa) dan menghormati `permissions.denied_paths`. Membaca dibenarkan secara lalai; perubahan fail, arahan shell dan pengambilan rangkaian akan bertanya sebelum dijalankan. Kebenaran shell terpakai pada arahan hos dan bukan sandbox sistem fail: kekalkan pada `ask` melainkan anda sengaja mempercayai model untuk menjalankan arahan.

Agen berhenti selepas `agent.max_turns` round-trip alat. Respons alat dianggap sebagai kandungan yang tidak dipercayai. Permintaan menggunakan respons model yang lengkap supaya argumen alat tidak dibina daripada serpihan strim yang tidak lengkap.

## Sesi

Perbualan disimpan secara setempat secara lalai, termasuk sejarah panggilan alat dan tetapan model/sesi. Pada Unix, direktori sesi dengan mod `0700` dan fail JSON dengan mod `0600`. Fail disimpan di `$XDG_STATE_HOME/marelle/sessions.json`, atau `~/.local/state/marelle/sessions.json` apabila `XDG_STATE_HOME` tidak ditetapkan. API key tidak pernah ditulis ke fail ini. Matikan persistensi dengan:

```sh
marelle config set agent.persist_sessions false
```

Kod keluar: `0` berjaya, `1` ralat provider atau rangkaian, `2` ralat penggunaan atau konfigurasi.

## Providers

| id | Protokol | Pemboleh ubah key | |
| --- | --- | --- | --- |
| `zai` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai (Zhipu GLM), antarabangsa |
| `zai_coding` | OpenAI-compatible | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, titik akhir serasi Anthropic |
| `zhipu` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | OpenAI-compatible | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI-compatible | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI-compatible | `OPENROUTER_API_KEY` | banyak model di sebalik satu key |
| `ollama` | OpenAI-compatible | none | Ollama pada mesin ini |

Titik akhir Z.ai dan Zhipu datang daripada dokumentasi pihak ketiga, bukan daripada halaman vendor sendiri. Semaknya dengan pelan anda, dan override mana-mana medan jika ia berbeza.

Setiap medan provider boleh diubah, dan provider baharu hanyalah satu nama dengan satu alamat:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Medan provider lain: `kind` (`openai-compatible` atau `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` dan `extra_body`. Jika pemboleh ubah yang dinamakan oleh `api_key_env` ditetapkan, ia yang menang; jika tidak, fail dibaca.

## Tetapan

Tetapan datang berlapis: lalai terbina dalam, kemudian pemboleh ubah persekitaran, kemudian flag. Fail tetapan pula akan menyelit di antara lalai dan pemboleh ubah persekitaran pada langkah berikutnya.

- `MARELLE_CHAT__MODEL=zai/MODEL` menetapkan `chat.model`. Dua underscore berturut-turut mewakili satu titik.
- `marelle config show [prefix]` menyenaraikan tetapan dan lapisan asal setiap satu.
- `marelle config explain <key>` menunjukkan setiap lapisan yang menetapkan sesuatu key, daripada keutamaan paling rendah hingga paling tinggi.

| Key | Lalai | Maksud |
| --- | --- | --- |
| `chat.provider`, `chat.model` | none | Dengan siapa untuk berbual. `chat.model` boleh jadi `provider/model`. |
| `chat.system` | none | System prompt |
| `chat.max_tokens`, `chat.temperature` | none | Diserahkan kepada provider jika tidak ditetapkan |
| `chat.stream` | `true` | Streaming respons judge pilihan; giliran agen yang memakai alat buat masa ini menggunakan respons lengkap supaya panggilan alat kekal utuh |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Reasoning dan kiraan token pada stderr |
| `agent.max_turns` | `20` | Round-trip alat maksimum bagi setiap permintaan pengguna |
| `agent.persist_sessions` | `true` | Menyimpan perbualan setempat antara run |
| `permissions.shell` | `ask` | Pelaksanaan arahan shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Pembacaan fail: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Perubahan fail: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Pengambilan URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Akar yang dibenarkan untuk alatan fail |
| `permissions.denied_paths` | none | Laluan fail yang disekat secara jelas |
| `network.connect_timeout_secs` | `15` | Masa yang dibenarkan untuk bersambung |
| `network.idle_timeout_secs` | `300` | Senyap yang dibenarkan sambil menunggu data |
| `network.proxy` | `system` | `system`, `none`, atau URL proksi |
| `network.ca_bundle` | none | Satu fail tambahan pihak berkuasa sijil yang dipercayai |
| `network.allow_insecure_http` | `false` | Membenarkan key melalui `http` biasa ke hos bukan setempat |
| `update.check` | `true` | Bertanya kepada GitHub dan GitLab tentang versi lebih baharu apabila sesi interaktif bermula |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` yang semakan bertanya keluaran terbaharu |
| `update.gitlab_repo` | none | GitLab `owner/repo` yang semakan turut bertanya. Kosong mematikan GitLab |
| `update.timeout_secs` | `3` | Bilangan saat yang boleh digunakan semakan bagi satu sumber |

Marelle membawa senarai pihak berkuasa sijil dipercayainya sendiri, maka ia juga berfungsi di mana sistem tiada satu pun, seperti dalam Termux. `SSL_CERT_FILE` menambah lagi.

Apabila versi lebih baharu ditemui, sesi menyebutnya sekali — versinya, apa yang anda ada, dan di mana mendapatkannya. Keluaran (release) ditanya dahulu; repositori yang hanya menanda commit dengan tag kembali ke tag terbaharunya. Apa-apa yang bermasalah kekal senyap.

## Key anda kekal milik anda

- Tetapan hanya menamakan pemboleh ubah atau fail yang menyimpan key, bukan key itu sendiri. Tetapan seperti `providers.x.api_key` ditolak.
- Key tidak dicetak oleh `config show`, `config explain` atau `providers`, dan dibuang daripada sebarang teks ralat yang kembali dari pelayan.
- Ia dihantar hanya ke alamat provider, dalam satu header, dan tidak pernah melalui `http` biasa ke mesin lain melainkan anda membenarkannya.
- Redirect tidak diikuti, maka key tidak boleh diteruskan ke tempat lain.

## Bahasa

Antara muka bertutur dalam bahasa Inggeris, Rusia, Perancis dan Ukraine. Pilih dengan `--lang <code>` atau `MARELLE_LANG`; jika tidak, `LC_ALL`, `LC_MESSAGES` dan `LANG` yang menentukan. Setiap mesej tinggal dalam satu katalog, dan satu ujian akan gagal jika sesuatu bahasa terlepas satu mesej. Matlamatnya ialah setiap bahasa di Eurasia dan kemudian lebih banyak lagi.

## Peta jalan

1. **Bercakap.** Adapter serasi OpenAI dan Anthropic, dengan kedua-dua streaming SSE dan penyahkodan respons lengkap. Giliran agen yang memakai alat buat masa ini menggunakan respons lengkap. *Siap.*
2. **Tetapan dan rupa.** Fail tetapan TOML, global dan setiap projek, dengan `config set`, `get`, `unset` dan huraian bagi setiap key. Antara muka bertema dalam semangat [Hermes Agent](https://github.com/NousResearch/hermes-agent): banner, panel maklumat, jawaban berbingkai, spinner, ungu wisteria secara lalai dengan warna apa pun boleh ditetapkan sebagai HEX. *Siap.*
3. **Banyak model.** Alias model, menanya beberapa model sekaligus dan membandingkan jawapan, rantaian fallback, dan model hakim yang menggabungkannya. *Siap.*
4. **Bertindak.** Panggilan alat berstruktur untuk provider serasi OpenAI dan Anthropic, alatan fail/carian/terminal setempat, pengambilan URL, skill setempat dan kebenaran allow/ask/deny, dengan transkrip menunjukkan setiap panggilan alat dan hasilnya. *Siap.*
5. **Mengingat.** Perbualan bernama kekal secara setempat antara run. Ingatan semantik jangka panjang masih di hadapan.
6. **Mengembang.** Pelayan MCP, pemalam luaran, subagen dan sokongan skill yang lebih kaya.
7. **Menjangkau.** Automasi pelayar, gerbang pemesejan dan tugasan berjadual.

Alasan di sebalik reka bentuk ada di [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Had yang diketahui

- Pemilih provider dan model menyenaraikan provider yang ditetapkan dan alias model. Marelle belum lagi menemui katalog model jarak jauh.
- Pemilih menyokong klik tetikus untuk fokus dan tatalan roda; transkrip dan editor input kekal dikendalikan papan kekunci.
- Ctrl-C keluar dari sesi interaktif. Ia belum lagi boleh membatalkan respons semasa sahaja.
- Agent loop, alatan setempat yang peka kebenaran dan sesi setempat berterusan telah dilaksanakan, tetapi Marelle kekal sebagai prototaip pra-alpha; lihat [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) untuk reka bentuk dan sempadan semasa.

## Pembangunan

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Ujian tidak pernah menyentuh rangkaian: ia berbual dengan mock server dalam `crates/marelle-testkit`. GitHub Actions juga membina binary Android dan menyemak versi Rust minimum yang dinyatakan. Pipeline GitLab CI disediakan untuk cermin GitLab; lihat [docs/GITLAB.md](docs/GITLAB.md).

## Lesen

[AGPL-3.0-only](LICENSE). Notis pihak ketiga ada dalam [NOTICE](NOTICE). Arkib keluaran menyertakan `THIRD-PARTY-LICENSES.txt` untuk crate Rust di dalam binary.
