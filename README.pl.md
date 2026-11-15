```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Konfigurowalny agent AI dla twojego terminala.**
Przynieś własne klucze API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, lokalną Ollamę albo dowolny serwer zgodny z OpenAI. Najpierw Linux i Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · Polski
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alpha, wersja 0.1.0-alpha.1.** Zaimplementowano: natywny interfejs TUI, czat z wieloma dostawcami, pętlę agenta świadomą uprawnień, lokalne narzędzia i wznawialne lokalne sesje. API i ustawienia mogą się jeszcze zmienić. To prototyp, a nie parzystość funkcji z OpenCode czy Hermes.

*Marelle* to francuska nazwa gry w klasy. Każde pole to krok.

## Instalacja

**Termux na Androidzie.** Pobierz gotowy plik binarny ze strony [wydań](https://github.com/S-I-L-S/marelle/releases) albo zbuduj na telefonie. Instrukcja krok po kroku: [docs/TERMUX.md](docs/TERMUX.md).

**Linux i wszędzie tam, gdzie działa Rust.** Wymagany Rust 1.85 lub nowszy:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Albo sklonuj repozytorium i uruchom `cargo build --release`. Binarka to `target/release/marelle`, około 4 MiB.

## Pierwszy czat

Trzymaj klucz w pliku, który czytasz tylko ty, i powiedz Marelle, gdzie jest. Marelle nigdy nie zapisuje klucza w swoich ustawieniach.

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

Zamień `MODEL` na nazwę modelu z listy twojego dostawcy. Inne sposoby użycia:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

Odpowiedź trafia na stdout. Rozumowanie, liczba tokenów, notatki i błędy idą na stderr, więc `marelle chat ... > answer.txt` zapisuje tylko odpowiedź.

## Narzędzia agenta

Marelle wysyła ustrukturyzowane definicje narzędzi przez API zgodne z OpenAI oraz Anthropic. Wbudowany zestaw narzędzi:

| Narzędzie | Przeznaczenie |
| --- | --- |
| `terminal` | Uruchomienie polecenia powłoki z katalogu startowego |
| `read_file`, `write_file`, `edit_file` | Czytanie i modyfikacja plików tekstowych UTF-8 |
| `list_directory`, `search_text` | Przegląd i wyszukiwanie w plikach projektu |
| `fetch_url` | Pobranie publicznej strony HTTP(S) jako tekstu (wymaga `curl`) |
| `list_skills`, `load_skill` | Odnajdywanie i ładowanie lokalnych instrukcji `SKILL.md` |

Poleceniem `/tools` obejrzysz listę narzędzi, a `/skills` wypisze dostępne umiejętności. Wskazówki projektu są ładowane z `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` i wybranych plików `.marelle/`, jeśli leżą w dozwolonych ścieżkach.

Narzędzia plikowe są ograniczone do `permissions.allowed_paths` (domyślnie: bieżący katalog projektu) i respektują `permissions.denied_paths`. Czytanie jest domyślnie dozwolone; zmiany plików, polecenia powłoki i pobieranie z sieci pytają przed wykonaniem. Uprawnienie do powłoki dotyczy poleceń na hoście i nie jest piaskownicą systemu plików: trzymaj je na `ask`, chyba że świadomie ufasz modeli wykonaniu poleceń.

Agent zatrzymuje się po `agent.max_turns` wymianach z narzędziami. Odpowiedzi narzędzi traktowane są jako niezaufana treść. Żądania korzystają z pełnych odpowiedzi modelu, więc argumenty narzędzi nie są składane z niekompletnych fragmentów strumienia.

## Sesje

Rozmowy są domyślnie zapisywane lokalnie, łącznie z historią wywołań narzędzi i ustawieniami modeli oraz sesji. Na Uniksie katalog sesji ma tryb `0700`, a plik JSON tryb `0600`. Plik zapisywany jest w `$XDG_STATE_HOME/marelle/sessions.json`, albo `~/.local/state/marelle/sessions.json`, gdy `XDG_STATE_HOME` nie jest ustawione. Klucze API nigdy nie trafiają do tego pliku. Wyłącz zapisywanie poleceniem:

```sh
marelle config set agent.persist_sessions false
```

Kody wyjścia: `0` sukces, `1` błąd dostawcy lub sieci, `2` błąd użycia albo konfiguracji.

## Dostawcy

| id | Protokół | Zmienna klucza | |
| --- | --- | --- | --- |
| `zai` | zgodny z OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), międzynarodowy |
| `zai_coding` | zgodny z OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, punkt końcowy zgodny z Anthropic |
| `zhipu` | zgodny z OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, Chiny |
| `zhipu_coding` | zgodny z OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, Chiny |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | zgodny z OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | zgodny z OpenAI | `OPENROUTER_API_KEY` | wiele modeli za jednym kluczem |
| `ollama` | zgodny z OpenAI | brak | Ollama na tej maszynie |

Punkty końcowe Z.ai i Zhipu pochodzą z dokumentacji stron trzecich, nie ze stron samego dostawcy. Sprawdź je względem swojego planu i nadpisz dowolne pole, jeśli się różnią.

Każde pole dostawcy można zmienić, a nowy dostawca to po prostu nazwa z adresem:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Pozostałe pola dostawcy: `kind` (`openai-compatible` lub `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` i `extra_body`. Jeśli zmienna wskazana przez `api_key_env` jest ustawiona, wygrywa; w przeciwnym razie czytany jest plik.

## Ustawienia

Ustawienia przychodzą warstwami: wbudowane domyślne wartości, potem zmienne środowiskowe, potem flagi. Pliki ustawień wypadają między wartościami domyślnymi a środowiskiem.

- `MARELLE_CHAT__MODEL=zai/MODEL` ustawia `chat.model`. Podwójne podkreślenie oznacza kropkę.
- `marelle config show [prefix]` wypisuje ustawienia i warstwę, z której każde przyszło.
- `marelle config explain <key>` pokazuje każdą warstwę, która ustawiła klucz, od najniższego priorytetu do najwyższego.

| Klucz | Domyślnie | Znaczenie |
| --- | --- | --- |
| `chat.provider`, `chat.model` | brak | Z kim rozmawiać. `chat.model` może być `provider/model`. |
| `chat.system` | brak | Prompt systemowy |
| `chat.max_tokens`, `chat.temperature` | brak | Nieustawione — decyduje dostawca |
| `chat.stream` | `true` | Strumieniowanie odpowiedzi sędziego, gdy jest włączony; tury agenta z narzędziami używają obecnie pełnych odpowiedzi, by wywołania narzędzi pozostawały nietknięte |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Rozumowanie i liczba tokenów na stderr |
| `agent.max_turns` | `20` | Maksimum wymian z narzędziami na jedno żądanie użytkownika |
| `agent.persist_sessions` | `true` | Zapisywanie lokalnych rozmów między uruchomieniami |
| `permissions.shell` | `ask` | Wykonywanie poleceń powłoki: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Czytanie plików: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Zmiana plików: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Pobieranie URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Dozwolone korzenie dla narzędzi plikowych |
| `permissions.denied_paths` | brak | Wymownie zablokowane ścieżki plików |
| `network.connect_timeout_secs` | `15` | Czas dozwolony na połączenie |
| `network.idle_timeout_secs` | `300` | Dozwolona cisza przy oczekiwaniu na dane |
| `network.proxy` | `system` | `system`, `none` albo adres proxy |
| `network.ca_bundle` | brak | Dodatkowy plik zaufanych urzędów certyfikacji |
| `network.allow_insecure_http` | `false` | Pozwolić na klucz przez zwykły `http` do obcego hosta |
| `update.check` | `true` | Pytać GitHub i GitLab o nowszą wersję przy starcie sesji interaktywnej |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo`, u którego kontrola pyta o najnowsze wydanie |
| `update.gitlab_repo` | brak | GitLab `owner/repo` dla tej samej kontroli. Puste wyłącza GitLab |
| `update.timeout_secs` | `3` | Ile sekund kontrola może poświęcić na jedno źródło |

Marelle niesie własną listę zaufanych urzędów certyfikacji, więc działa też tam, gdzie system ich nie ma, jak w Termuxie. `SSL_CERT_FILE` dodaje kolejne.

Gdy zostanie znaleziona nowsza wersja, sesja wspomni o niej raz — wersję, to, co masz, i gdzie ją wziąć. Najpierw pytane jest wydanie; repozytorium, które tylko taguje commity, oddaje najnowszy tag. Każda porażka pozostaje cicha.

## Twoje klucze pozostają twoje

- Ustawienia nazywają zmienną albo plik, gdzie leży klucz, nigdy sam klucz. Ustawienie takie jak `providers.x.api_key` jest odrzucane.
- Klucz nie jest drukowany przez `config show`, `config explain` ani `providers` i jest wycinany z każdego tekstu błędu, który wraca od serwera.
- Jest wysyłany tylko na adres dostawcy, w jednym nagłówku, i nigdy zwykłym `http` do innej maszyny, chyba że na to pozwolisz.
- Przekierowania nie są śledzone, więc klucz nie może polecieć gdzieś indziej.

## Języki

Interfejs mówi po angielsku, rosyjsku, francusku i ukraińsku. Wybierz przez `--lang <code>` albo `MARELLE_LANG`; inaczej decydują `LC_ALL`, `LC_MESSAGES` i `LANG`. Każdy komunikat żyje w katalogu, a test pada, gdy jakiś język jeden pomija. Celem są wszystkie języki Eurazji, a potem więcej.

## Mapa drogowa

1. **Rozmawiać.** Adaptery zgodne z OpenAI oraz Anthropic, ze strumieniowaniem SSE i rozbiorem pełnych odpowiedzi. Tury agenta z narzędziami używają obecnie pełnych odpowiedzi. *Gotowe.*
2. **Ustawienia i wygląd.** Pliki ustawień TOML, globalne i per projekt, z `config set`, `get` i `unset` oraz opisem każdego klucza. Klamrowany interfejs w duchu [Hermes Agent](https://github.com/NousResearch/hermes-agent): baner, panel informacyjny, odpowiedzi w ramce, spinner, domyślnie w fiolecie glicynii, z dowolnym kolorem jako HEX. *Gotowe.*
3. **Wiele modeli.** Aliasy modeli, jednoczesne pytanie kilku modeli i porównywanie odpowiedzi, łańcuchy fallback oraz model sędzia, który je scala. *Gotowe.*
4. **Działać.** Ustrukturyzowane wywołania narzędzi dla dostawców zgodnych z OpenAI oraz Anthropic, lokalne narzędzia plików, wyszukiwania i terminala, pobieranie URL, lokalne umiejętności i uprawnienia allow/ask/deny; transkrypt pokazuje każde wywołanie narzędzia i jego wynik. *Gotowe.*
5. **Pamiętać.** Nazwane rozmowy trwają lokalnie między uruchomieniami. Długotrwała pamięć semantyczna wciąż przed nami.
6. **Rozszerzać.** Serwery MCP, zewnętrzne wtyczki, subagenci i bogatsza obsługa umiejętności.
7. **Sięgać.** Automatyzacja przeglądarki, bramy komunikatorów i zadania planowane.

Rozumowanie stojące za projektem jest w [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Znane ograniczenia

- Selektory dostawców i modeli wypisują skonfigurowanych dostawców i aliasy modeli. Marelle nie odkrywa jeszcze zdalnych katalogów modeli.
- Selektory wspierają kliknięcie myszą dla fokusu i przewijanie kółkiem; transkrypt i edytor wejścia pozostają sterowane klawiaturą.
- Ctrl-C kończy sesję interaktywną. Nie anuluje jeszcze tylko bieżącej odpowiedzi.
- Pętla agenta, lokalne narzędzia świadome uprawnień i trwałe lokalne sesje są zaimplementowane, ale Marelle pozostaje prototypem pre-alpha; o obecnym projekcie i granicach czytaj [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Rozwój

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Testy nigdy nie dotykają sieci: rozmawiają z serwerem atrapą w `crates/marelle-testkit`. GitHub Actions buduje też binarki Androida i sprawdza zadeklarowaną minimalną wersję Rust. Dla lustra na GitLabie jest potok GitLab CI; zobacz [docs/GITLAB.md](docs/GITLAB.md).

## Licencja

[AGPL-3.0-only](LICENSE). Noty stron trzecich są w [NOTICE](NOTICE). Archiwa wydań zawierają `THIRD-PARTY-LICENSES.txt` dla skrzynek Rust wewnątrz binarki.
