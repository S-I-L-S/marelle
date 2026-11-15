```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Un agente de IA configurable para tu terminal.**
Trae tus propias claves de API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, una Ollama local, o cualquier servidor compatible con OpenAI. Primero Linux y Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · Español · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
[Português](README.pt.md) · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pre-alfa, versión 0.1.0-alpha.1.** Implementado: interfaz TUI nativa, chat con varios proveedores, un bucle de agente consciente de permisos, herramientas locales y sesiones locales reanudables. Las API y los ajustes aún pueden cambiar. Esto es un prototipo, no una paridad de funciones con OpenCode ni Hermes.

*Marelle* es el nombre francés de la rayuela. Cada casilla es un paso.

## Instalación

**Termux en Android.** Descarga un binario ya construido desde la página de [lanzamientos](https://github.com/S-I-L-S/marelle/releases), o compílalo en el teléfono. Guía paso a paso: [docs/TERMUX.md](docs/TERMUX.md).

**Linux y dondequiera que funcione Rust.** Rust 1.85 o más nuevo:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

O clona el repositorio y ejecuta `cargo build --release`. El binario es `target/release/marelle`, unos 4 MiB.

## Primer chat

Guarda la clave en un archivo que solo tú lees, y dile a Marelle dónde está. Marelle nunca guarda una clave en sus ajustes.

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

Sustituye `MODEL` por un nombre de modelo de la lista de tu proveedor. Otras maneras de usarlo:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

La respuesta va a stdout. El razonamiento, los conteos de tokens, las notas y los errores van a stderr, así que `marelle chat ... > answer.txt` guarda solo la respuesta.

## Herramientas del agente

Marelle envía definiciones estructuradas de herramientas por las API compatibles con OpenAI y Anthropic. El juego de herramientas integrado:

| Herramienta | Propósito |
| --- | --- |
| `terminal` | Ejecutar un comando de shell desde el directorio de arranque |
| `read_file`, `write_file`, `edit_file` | Leer y modificar archivos de texto UTF-8 |
| `list_directory`, `search_text` | Inspeccionar y buscar en los archivos del proyecto |
| `fetch_url` | Traer una página pública HTTP(S) como texto (requiere `curl`) |
| `list_skills`, `load_skill` | Descubrir y cargar instrucciones locales `SKILL.md` |

El comando `/tools` muestra la lista de herramientas y `/skills` enumera las habilidades disponibles. La guía del proyecto se carga desde `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` y ciertos archivos `.marelle/` cuando están dentro de rutas permitidas.

Las herramientas de archivos están confinadas a `permissions.allowed_paths` (por defecto: el directorio actual del proyecto) y respetan `permissions.denied_paths`. La lectura está permitida por defecto; los cambios de archivos, los comandos de shell y las descargas de red preguntan antes. El permiso de shell aplica a comandos en el anfitrión y no es una caja de arena del sistema de archivos: tenlo en `ask`, a menos que confíes deliberadamente en el modelo para ejecutar comandos.

El agente se detiene tras `agent.max_turns` vueltas de herramientas. Las respuestas de herramientas se tratan como contenido no confiable. Las peticiones usan respuestas completas del modelo, de modo que los argumentos de herramientas no se arman de fragmentos incompletos del flujo.

## Sesiones

Las conversaciones se guardan localmente por defecto, incluido el historial de llamadas de herramientas y los ajustes del modelo y la sesión. En Unix, el directorio de sesiones tiene modo `0700` y el archivo JSON modo `0600`. El archivo se guarda en `$XDG_STATE_HOME/marelle/sessions.json`, o `~/.local/state/marelle/sessions.json` cuando `XDG_STATE_HOME` no está fijado. Las claves de API nunca se escriben en este archivo. Desactiva la persistencia con:

```sh
marelle config set agent.persist_sessions false
```

Códigos de salida: `0` éxito, `1` error de proveedor o de red, `2` error de uso o de configuración.

## Proveedores

| id | Protocolo | Variable de clave | |
| --- | --- | --- | --- |
| `zai` | compatible con OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internacional |
| `zai_coding` | compatible con OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, punto final compatible con Anthropic |
| `zhipu` | compatible con OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | compatible con OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatible con OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatible con OpenAI | `OPENROUTER_API_KEY` | muchos modelos tras una clave |
| `ollama` | compatible con OpenAI | ninguna | Ollama en esta máquina |

Los puntos finales de Z.ai y Zhipu provienen de documentación de terceros, no de las páginas del propio proveedor. Coteja con tu plan y sobrescribe cualquier campo si difieren.

Cada campo de un proveedor se puede cambiar, y un proveedor nuevo es solo un nombre con una dirección:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Otros campos del proveedor: `kind` (`openai-compatible` o `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` y `extra_body`. Si la variable nombrada por `api_key_env` está fijada, ella gana; si no, se lee el archivo.

## Ajustes

Los ajustes llegan en capas: valores predeterminados integrados, luego variables de entorno, luego banderas. Los archivos de ajustes se intercalan entre los valores predeterminados y el entorno.

- `MARELLE_CHAT__MODEL=zai/MODEL` fija `chat.model`. Un doble subrayado vale por un punto.
- `marelle config show [prefix]` enumera los ajustes y la capa de la que vino cada uno.
- `marelle config explain <key>` muestra cada capa que fijó una clave, de la prioridad más baja a la más alta.

| Clave | Predeterminado | Sentido |
| --- | --- | --- |
| `chat.provider`, `chat.model` | ninguno | Con quién hablar. `chat.model` puede ser `provider/model`. |
| `chat.system` | ninguno | Prompt de sistema |
| `chat.max_tokens`, `chat.temperature` | ninguno | Sin fijar — lo deja al proveedor |
| `chat.stream` | `true` | Flujo de la respuesta opcional del juez; las vueltas de agente con herramientas usan por ahora respuestas completas, para que las llamadas de herramientas queden íntegras |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Razonamiento y conteos de tokens en stderr |
| `agent.max_turns` | `20` | Máximo de vueltas de herramientas por petición de usuario |
| `agent.persist_sessions` | `true` | Guardar conversaciones locales entre ejecuciones |
| `permissions.shell` | `ask` | Ejecución de comandos de shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Lectura de archivos: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Cambio de archivos: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Descarga de URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Raíces permitidas para las herramientas de archivos |
| `permissions.denied_paths` | ninguno | Rutas de archivos expresamente bloqueadas |
| `network.connect_timeout_secs` | `15` | Tiempo permitido para conectar |
| `network.idle_timeout_secs` | `300` | Silencio permitido mientras esperan datos |
| `network.proxy` | `system` | `system`, `none`, o una dirección de proxy |
| `network.ca_bundle` | ninguno | Un archivo extra de autoridades de certificación de confianza |
| `network.allow_insecure_http` | `false` | Permitir una clave por `http` simple hacia un host ajeno |
| `update.check` | `true` | Preguntar a GitHub y GitLab por una versión más nueva al comienzo de una sesión interactiva |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` al que la comprobación pide el lanzamiento más nuevo |
| `update.gitlab_repo` | ninguno | GitLab `owner/repo` para la misma comprobación. Vacío desactiva GitLab |
| `update.timeout_secs` | `3` | Cuántos segundos puede gastar la comprobación en una fuente |

Marelle lleva su propia lista de autoridades de certificación de confianza, así que funciona también donde el sistema no tiene, como en Termux. `SSL_CERT_FILE` añade más.

Cuando se encuentra una versión más nueva, la sesión la menciona una vez — la versión, lo que tienes, y dónde conseguirla. Se pide primero un lanzamiento; un repositorio que solo etiqueta sus commits da la etiqueta más nueva. Todo lo que falle permanece silencioso.

## Tus claves siguen siendo tuyas

- Los ajustes nombran una variable o un archivo que guarda la clave, nunca la clave misma. Un ajuste como `providers.x.api_key` se rechaza.
- La clave no la imprimen `config show`, `config explain` ni `providers`, y se recorta de todo texto de error que venga de un servidor.
- Va solo a la dirección del proveedor, en un único encabezado, y nunca por `http` simple hacia otra máquina, salvo que lo permitas.
- Las redirecciones no se siguen, de modo que una clave no puede enviarse a otro lado.

## Idiomas

La interfaz habla inglés, ruso, francés y ucraniano. Elige con `--lang <code>` o `MARELLE_LANG`; si no, deciden `LC_ALL`, `LC_MESSAGES` y `LANG`. Cada mensaje vive en un catálogo, y una prueba cae si un idioma deja pasar uno. La meta es cada idioma de Eurasia y luego más.

## Hoja de ruta

1. **Hablar.** Adaptadores compatibles con OpenAI y Anthropic, con flujo SSE y decodificación de respuestas completas. Las vueltas de agente con herramientas usan por ahora respuestas completas. *Hecho.*
2. **Ajustes y aspecto.** Archivos de ajustes TOML, globales y por proyecto, con `config set`, `get` y `unset` y una descripción para cada clave. Una interfaz tematizada al espíritu de [Hermes Agent](https://github.com/NousResearch/hermes-agent): pancarta, panel de información, respuestas enmarcadas, indicador giratorio, por defecto en el morado de la glicinia, con cualquier color fijable como HEX. *Hecho.*
3. **Muchos modelos.** Alias de modelos, preguntar a varios modelos a la vez y comparar las respuestas, cadenas de reserva, y un modelo juez que las funde. *Hecho.*
4. **Actuar.** Llamadas estructuradas de herramientas para proveedores compatibles con OpenAI y Anthropic, herramientas locales de archivos, búsqueda y terminal, descarga de URL, habilidades locales y permisos allow/ask/deny; la transcripción muestra cada llamada de herramienta y su resultado. *Hecho.*
5. **Recordar.** Las conversaciones con nombre persisten localmente entre ejecuciones. La memoria semántica de largo plazo aún queda adelante.
6. **Extender.** Servidores MCP, complementos externos, subagentes y un soporte más rico de habilidades.
7. **Alcanzar.** Automatización de navegador, puertas de mensajería y tareas programadas.

El razonamiento detrás del diseño está en [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Límites conocidos

- Los selectores de proveedores y modelos enumeran los proveedores configurados y los alias de modelos. Marelle aún no descubre catálogos de modelos remotos.
- Los selectores sostienen clic de ratón para el enfoque y desplazamiento con rueda; la transcripción y el editor de entrada siguen mandados desde el teclado.
- Ctrl-C sale de la sesión interactiva. Todavía no cancela solo la respuesta en curso.
- El bucle de agente, las herramientas locales conscientes de permisos y las sesiones locales persistentes están implementados, pero Marelle sigue siendo un prototipo pre-alfa; para el diseño actual y sus límites, lee [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Desarrollo

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

Las pruebas nunca tocan la red: hablan con un servidor simulado en `crates/marelle-testkit`. GitHub Actions también construye los binarios para Android y comprueba la versión mínima de Rust declarada. Para un espejo en GitLab hay un conducto GitLab CI; véase [docs/GITLAB.md](docs/GITLAB.md).

## Licencia

[AGPL-3.0-only](LICENSE). Los avisos de terceros están en [NOTICE](NOTICE). Los archivos de lanzamientos incluyen `THIRD-PARTY-LICENSES.txt` para las cajas de Rust dentro del binario.
