```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**여러분의 터미널을 위한, 설정 가능한 AI 에이전트.**
API 키는 직접 가져오세요: Z.ai(Zhipu GLM), Anthropic, OpenAI, OpenRouter, 로컬 Ollama, 또는 OpenAI 호환 서버라면 무엇이든. 우선 Linux와 Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · 한국어 · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **프리-알파, 버전 0.1.0-alpha.1.** 구현됨: 네이티브 TUI 인터페이스, 다중 공급자 채팅, 권한을 인식하는 에이전트 루프, 로컬 도구, 그리고 재개할 수 있는 로컬 세션. API와 설정은 아직 바뀔 수 있습니다. 이것은 프로토타입이며, OpenCode나 Hermes와의 기능 동등이 아닙니다.

*Marelle*는 프랑스어로 아이들의 칸 뛰기 놀이 이름입니다. 한 칸이 곧 한 걸음입니다.

## 설치

**안드로이드의 Termux.** [릴리스 페이지](https://github.com/S-I-L-S/marelle/releases)에서 미리 빌드된 바이너리를 받거나, 휴대폰에서 직접 빌드하세요. 단계별 안내: [docs/TERMUX.md](docs/TERMUX.md).

**Linux, 그리고 Rust가 돌아가는 모든 곳.** Rust 1.85 이상이 필요합니다:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

또는 저장소를 클론하고 `cargo build --release`를 실행하세요. 바이너리는 `target/release/marelle`이며, 약 4 MiB입니다.

## 첫 채팅

키는 오직 자신만 읽는 파일에 보관하고, 그 위치를 Marelle에게 알려주세요. Marelle는 결코 키를 자신의 설정에 저장하지 않습니다.

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

`MODEL`을 여러분의 공급자 목록에 있는 모델 이름으로 바꾸세요. 그 밖의 사용법:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

답변은 stdout으로 나갑니다. 추론 과정, 토큰 수, 메모, 오류는 stderr로 나가므로, `marelle chat ... > answer.txt`는 답변만 저장합니다.

## 에이전트의 도구

Marelle은 OpenAI 호환 및 Anthropic API를 통해 구조화된 도구 정의를 보냅니다. 내장 도구 세트:

| 도구 | 용도 |
| --- | --- |
| `terminal` | 시작 디렉터리에서 셸 명령 실행 |
| `read_file`, `write_file`, `edit_file` | UTF-8 텍스트 파일 읽기와 수정 |
| `list_directory`, `search_text` | 프로젝트 파일 살펴보기와 검색 |
| `fetch_url` | 공개 HTTP(S) 페이지를 텍스트로 가져오기 (`curl` 필요) |
| `list_skills`, `load_skill` | 로컬 `SKILL.md` 지침 발견 및 불러오기 |

`/tools` 명령은 도구 목록을, `/skills`는 사용 가능한 스킬을 보여줍니다. 프로젝트 지침은 `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules`와 선택된 `.marelle/` 파일들에서, 허용된 경로 안에 있을 때 불러옵니다.

파일 도구는 `permissions.allowed_paths` 안으로 제한되며(기본값: 현재 프로젝트 디렉터리) `permissions.denied_paths`를 존중합니다. 읽기는 기본적으로 허용됩니다. 파일 변경, 셸 명령, 네트워크 가져오기는 실행 전에 물어봅니다. 셸 권한은 호스트 머신의 명령에 적용되며 파일 시스템 샌드박스가 아닙니다. 모델에 명령 실행을 의도적으로 맡길 생각이 아니라면 `ask`로 두세요.

에이전트는 `agent.max_turns`번의 도구 왕복 후 멈춥니다. 도구 응답은 신뢰할 수 없는 내용으로 취급됩니다. 요청은 완전한 모델 응답을 사용하므로, 도구 인수가 불완전한 스트림 조각들로부터 조립되는 일이 없습니다.

## 세션

대화는 기본적으로 로컬에 저장됩니다. 도구 호출 이력과 모델 및 세션 설정을 포함합니다. Unix에서 세션 디렉터리는 `0700` 모드이고, JSON 파일은 `0600` 모드입니다. 파일은 `$XDG_STATE_HOME/marelle/sessions.json`에 저장되고, `XDG_STATE_HOME`이 설정되지 않았다면 `~/.local/state/marelle/sessions.json`에 저장됩니다. API 키는 이 파일에 결코 기록되지 않습니다. 저장을 끄려면:

```sh
marelle config set agent.persist_sessions false
```

종료 코드: `0` 성공, `1` 공급자 또는 네트워크 오류, `2` 사용법 또는 구성 오류.

## 공급자

| id | 프로토콜 | 키 변수 | |
| --- | --- | --- | --- |
| `zai` | OpenAI 호환 | `ZAI_API_KEY` | Z.ai(Zhipu GLM), 국제 |
| `zai_coding` | OpenAI 호환 | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, Anthropic 호환 엔드포인트 |
| `zhipu` | OpenAI 호환 | `ZHIPUAI_API_KEY` | Zhipu BigModel, 중국 |
| `zhipu_coding` | OpenAI 호환 | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, 중국 |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | OpenAI 호환 | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | OpenAI 호환 | `OPENROUTER_API_KEY` | 하나의 키 뒤에 여러 모델 |
| `ollama` | OpenAI 호환 | 없음 | 이 머신의 Ollama |

Z.ai와 Zhipu의 엔드포인트는 제3자 문서에서 온 것이며, 공급자 자신의 페이지가 아닙니다. 자신의 요금제와 대조해 확인하고, 다르다면 어떤 필드든 덮어쓰세요.

공급자의 모든 필드는 바꿀 수 있으며, 새 공급자란 이름과 주소만 있으면 됩니다:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

공급자의 다른 필드들: `kind`(`openai-compatible` 또는 `anthropic`), `api_key_file`, `auth`(`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage`, `extra_body`. `api_key_env`가 가리키는 이름의 변수가 설정되어 있다면 그것이 이기고, 아니면 파일을 읽습니다.

## 설정

설정은 층으로 옵니다: 내장 기본값, 그다음 환경 변수, 그다음 플래그. 설정 파일은 기본값과 환경 사이에 끼어듭니다.

- `MARELLE_CHAT__MODEL=zai/MODEL`은 `chat.model`을 설정합니다. 이중 밑줄은 점을 대신합니다.
- `marelle config show [prefix]`는 설정과 각각이 어느 층에서 왔는지 나열합니다.
- `marelle config explain <key>`는 키를 설정한 모든 층을, 가장 낮은 우선순위부터 가장 높은 것까지 보여줍니다.

| 키 | 기본값 | 의미 |
| --- | --- | --- |
| `chat.provider`, `chat.model` | 없음 | 누구와 말할 것인가. `chat.model`은 `provider/model`일 수 있습니다. |
| `chat.system` | 없음 | 시스템 프롬프트 |
| `chat.max_tokens`, `chat.temperature` | 없음 | 설정되지 않으면 공급자에게 맡김 |
| `chat.stream` | `true` | 선택적인 심판 응답의 스트리밍. 도구를 쓰는 에이전트 턴은 현재 완전한 응답을 사용합니다. 도구 호출이 온전히 남도록 하기 위해서입니다 |
| `chat.show_reasoning`, `chat.show_usage` | `false` | 추론 과정과 토큰 수를 stderr에 표시 |
| `agent.max_turns` | `20` | 사용자 요청 하나당 도구 왕복의 상한 |
| `agent.persist_sessions` | `true` | 실행 사이에 로컬 대화를 저장 |
| `permissions.shell` | `ask` | 셸 명령 실행: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | 파일 읽기: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | 파일 수정: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | URL 가져오기: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | 파일 도구에 허용된 루트 |
| `permissions.denied_paths` | 없음 | 명시적으로 막힌 파일 경로 |
| `network.connect_timeout_secs` | `15` | 연결에 허용되는 시간 |
| `network.idle_timeout_secs` | `300` | 데이터를 기다리며 허용되는 침묵 |
| `network.proxy` | `system` | `system`, `none`, 또는 프록시 주소 |
| `network.ca_bundle` | 없음 | 신뢰하는 인증 기관을 추가하는 파일 |
| `network.allow_insecure_http` | `false` | 키가 날 `http`로 다른 호스트에 가는 것을 허용 |
| `update.check` | `true` | 대화형 세션이 시작될 때 GitHub와 GitLab에 더 새로운 버전을 물어봄 |
| `update.github_repo` | `S-I-L-S/marelle` | 점검이 최신 릴리스를 물어보는 GitHub `owner/repo` |
| `update.gitlab_repo` | 없음 | 같은 점검을 위한 GitLab `owner/repo`. 비워 두면 GitLab은 꺼짐 |
| `update.timeout_secs` | `3` | 점검이 한 출처에 쓸 수 있는 초 |

Marelle은 신뢰하는 인증 기관의 자체 목록을 갖고 다니므로, Termux처럼 시스템에 그런 목록이 없는 곳에서도 동작합니다. `SSL_CERT_FILE`로 더 추가할 수 있습니다.

더 새로운 버전이 발견되면, 세션은 그것을 한 번 언급합니다 — 버전, 여러분이 가진 것, 그리고 어디서 얻는지. 릴리스를 먼저 물어봅니다. 커밋을 태그로만 표시하는 저장소는 가장 최신 태그를 돌려줍니다. 무엇이든 실패하면 조용히 남습니다.

## 키는 여러분의 것으로 남습니다

- 설정은 키가 놓인 변수나 파일을 가리킬 뿐, 결코 키 그 자체를 가리키지 않습니다. `providers.x.api_key` 같은 설정은 거부됩니다.
- 키는 `config show`, `config explain`, `providers`에 의해 인쇄되지 않으며, 서버에서 돌아오는 모든 오류 텍스트에서 잘려나갑니다.
- 키는 공급자의 주소로만, 하나의 헤더 안으로 보내지며, 여러분이 허용하지 않는 한 날 `http`로 다른 머신에 가는 일이 결코 없습니다.
- 리디렉션을 따라가지 않으므로, 키가 다른 곳으로 전달될 수 없습니다.

## 언어

인터페이스는 영어, 러시아어, 프랑스어, 우크라이나어로 말합니다. `--lang <code>`나 `MARELLE_LANG`으로 고르세요. 아니면 `LC_ALL`, `LC_MESSAGES`, `LANG`이 결정합니다. 모든 메시지는 카탈로그 안에서 살아가고, 어떤 언어가 하나라도 빠뜨리면 테스트가 넘어집니다. 목표는 유라시아의 모든 언어, 그리고 그 뒤에 더.

## 로드맵

1. **말하기.** OpenAI 호환 및 Anthropic 어댑터. SSE 스트리밍과 완전 응답 해석을 모두 갖춤. 도구를 쓰는 에이전트 턴은 현재 완전한 응답을 사용합니다. *완료.*
2. **설정과 모습.** TOML 설정 파일. 전역과 프로젝트별. `config set`, `get`, `unset`과 함께, 모든 키에 대한 설명이 붙습니다. [Hermes Agent](https://github.com/NousResearch/hermes-agent)의 정신을 따르는 테마화된 인터페이스: 배너, 정보 패널, 테두리로 둘러싸인 답변, 회전 표시, 기본값은 자스민 꽃의 보랏빛, 모든 색은 HEX로 정할 수 있습니다. *완료.*
3. **여러 모델.** 모델 별명. 여러 모델에 동시에 물어 보고 응답을 비교하는 것. 대체 사슬. 그리고 그것들을 하나로 융합하는 심판 모델. *완료.*
4. **행동하기.** OpenAI 호환 및 Anthropic 공급자를 위한 구조화된 도구 호출. 로컬 파일·검색·터미널 도구. URL 가져오기. 로컬 스킬과 allow/ask/deny 권한. 대화 기록은 모든 도구 호출과 그 결과를 보여줍니다. *완료.*
5. **기억하기.** 이름 붙은 대화는 실행 사이에 로컬에 남습니다. 장기 의미 기억은 아직 앞에 있습니다.
6. **확장하기.** MCP 서버, 외부 플러그인, 하위 에이전트, 그리고 더 풍부한 스킬 지원.
7. **닿기.** 브라우저 자동화, 메신저 게이트웨이, 예약된 작업.

설계 뒤에 있는 사유는 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)에 있습니다.

## 알려진 한계

- 공급자와 모델 선택기는 설정된 공급자와 모델 별명을 나열합니다. Marelle은 아직 멀리 있는 모델 카탈로그를 발견하지 않습니다.
- 선택기는 초점을 위한 마우스 클릭과 휠 스크롤을 지원합니다. 대화 기록과 입력 편집기는 여전히 키보드로 구동됩니다.
- Ctrl-C는 대화형 세션을 나갑니다. 아직은 현재 응답만 취소하지는 못합니다.
- 에이전트 루프, 권한을 인식하는 로컬 도구, 지속되는 로컬 세션은 구현되었지만, Marelle은 여전히 프리-알파 프로토타입입니다. 현재 설계와 한계는 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)를 보세요.

## 개발

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

테스트는 결코 네트워크를 만지지 않습니다. `crates/marelle-testkit`의 모의 서버와 이야기합니다. GitHub Actions는 Android 바이너리도 빌드하고, 선언된 최소 Rust 버전을 검사합니다. GitLab 미러를 위한 GitLab CI 파이프라인도 있습니다. [docs/GITLAB.md](docs/GITLAB.md)를 보세요.

## 라이선스

[AGPL-3.0-only](LICENSE). 제3자 고지는 [NOTICE](NOTICE)에 있습니다. 릴리스 아카이브에는 바이너리 안의 Rust 상자들을 위한 `THIRD-PARTY-LICENSES.txt`가 포함됩니다.
