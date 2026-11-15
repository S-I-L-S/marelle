```text
 +---+---+---+---+---+---+---+
 | m | a | r | e | l | l | e |
 +---+---+---+---+---+---+---+
```

**Um agente de IA configurável para o seu terminal.**
Traga as suas próprias chaves de API: Z.ai (Zhipu GLM), Anthropic, OpenAI, OpenRouter, uma Ollama local, ou qualquer servidor compatível com OpenAI. Primeiro Linux e Termux.

[English](README.md) · [العربية](README.ar.md) · [Azərbaycanca](README.az.md) · [Беларуская](README.be.md) · [Български](README.bg.md) · [বাংলা](README.bn.md) · [Català](README.ca.md)
[Čeština](README.cs.md) · [Dansk](README.da.md) · [Deutsch](README.de.md) · [Ελληνικά](README.el.md) · [Español](README.es.md) · [Eesti](README.et.md) · [فارسی](README.fa.md)
[Suomi](README.fi.md) · [Français](README.fr.md) · [עברית](README.he.md) · [हिन्दी](README.hi.md) · [Hrvatski](README.hr.md) · [Magyar](README.hu.md) · [Հայերեն](README.hy.md)
[Bahasa Indonesia](README.id.md) · [Italiano](README.it.md) · [日本語](README.ja.md) · [ქართული](README.ka.md) · [Қазақша](README.kk.md) · [한국어](README.ko.md) · [Кыргызча](README.ky.md)
[Lietuvių](README.lt.md) · [Latviešu](README.lv.md) · [Монгол](README.mn.md) · [Bahasa Melayu](README.ms.md) · [Norsk bokmål](README.nb.md) · [Nederlands](README.nl.md) · [Polski](README.pl.md)
Português · [Română](README.ro.md) · [Русский](README.ru.md) · [Slovenčina](README.sk.md) · [Slovenščina](README.sl.md) · [Српски](README.sr.md) · [Svenska](README.sv.md)
[Тоҷикӣ](README.tg.md) · [ไทย](README.th.md) · [Türkmençe](README.tk.md) · [Türkçe](README.tr.md) · [Українська](README.uk.md) · [اردو](README.ur.md) · [Oʻzbekcha](README.uz.md)
[Tiếng Việt](README.vi.md) · [简体中文](README.zh.md) · [繁體中文](README.zh-Hant.md)

> **Pré-alfa, versão 0.1.0-alpha.1.** Implementado: interface TUI nativa, conversa com vários fornecedores, um ciclo de agente consciente de permissões, ferramentas locais e sessões locais retomáveis. As API e as configurações ainda podem mudar. Isto é um protótipo, não uma paridade de funções com OpenCode ou Hermes.

*Marelle* é o nome francês da amarelinha. Cada casa é um passo.

## Instalação

**Termux no Android.** Baixe um binário pronto da página de [lançamentos](https://github.com/S-I-L-S/marelle/releases), ou compile no telefone. Guia passo a passo: [docs/TERMUX.md](docs/TERMUX.md).

**Linux e em todo lugar onde o Rust funciona.** Rust 1.85 ou mais novo:

```sh
cargo install --locked --git https://github.com/S-I-L-S/marelle marelle-cli
```

Ou clone o repositório e execute `cargo build --release`. O binário é `target/release/marelle`, cerca de 4 MiB.

## Primeira conversa

Guarde a chave em um arquivo que só você lê, e diga à Marelle onde está. A Marelle nunca guarda uma chave nas suas configurações.

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

Substitua `MODEL` por um nome de modelo da lista do seu fornecedor. Outras maneiras de usar:

```sh
marelle                                                # interactive TUI (when chat.model is configured)
marelle --tui                                           # explicit TUI launch
marelle --continue                                      # resume the most recent saved conversation
marelle chat -q "Explain this repository"               # one prompt, then exit
marelle chat -m zai/MODEL                               # interactive TUI with a one-time model override
git diff | marelle chat -m zai/MODEL "Review this:" -   # "-" stands for what was piped in
marelle chat -p anthropic -m MODEL --show-usage "Why is the sky blue?"
```

A resposta vai para o stdout. O raciocínio, as contagens de tokens, as notas e os erros vão para o stderr, de modo que `marelle chat ... > answer.txt` salva apenas a resposta.

## Ferramentas do agente

A Marelle envia definições estruturadas de ferramentas pelas API compatíveis com OpenAI e Anthropic. O conjunto de ferramentas integrado:

| Ferramenta | Propósito |
| --- | --- |
| `terminal` | Executar um comando de shell do diretório de partida |
| `read_file`, `write_file`, `edit_file` | Ler e modificar arquivos de texto UTF-8 |
| `list_directory`, `search_text` | Inspecionar e procurar nos arquivos do projeto |
| `fetch_url` | Trazer uma página pública HTTP(S) como texto (requer `curl`) |
| `list_skills`, `load_skill` | Descobrir e carregar instruções locais `SKILL.md` |

O comando `/tools` mostra a lista de ferramentas e `/skills` lista as habilidades disponíveis. A orientação do projeto carrega-se de `AGENTS.md`, `CLAUDE.md`, `SOUL.md`, `.cursorrules` e certos arquivos `.marelle/` quando estão dentro de caminhos permitidos.

As ferramentas de arquivos ficam confinadas a `permissions.allowed_paths` (por defeito: o diretório atual do projeto) e respeitam `permissions.denied_paths`. A leitura é permitida por defeito; mudanças de arquivos, comandos de shell e descargas de rede perguntam antes. A permissão de shell aplica-se a comandos no anfitrião e não é uma caixa de areia do sistema de arquivos: mantenha-a em `ask`, a menos que confie deliberadamente no modelo a execução de comandos.

O agente para depois de `agent.max_turns` rodadas de ferramentas. As respostas das ferramentas tratam-se como conteúdo não confiável. Os pedidos usam respostas completas do modelo, de modo que os argumentos das ferramentas não se montem de pedaços incompletos do fluxo.

## Sessões

As conversas salvam-se localmente por defeito, incluindo o histórico de chamadas de ferramentas e as configurações de modelo e sessão. Em Unix o diretório de sessões tem modo `0700` e o arquivo JSON modo `0600`. O arquivo guarda-se em `$XDG_STATE_HOME/marelle/sessions.json`, ou `~/.local/state/marelle/sessions.json` quando `XDG_STATE_HOME` não está fixado. Chaves de API nunca se escrevem neste arquivo. Desligue a persistência com:

```sh
marelle config set agent.persist_sessions false
```

Códigos de saída: `0` sucesso, `1` erro de fornecedor ou de rede, `2` erro de uso ou de configuração.

## Fornecedores

| id | Protocolo | Variável da chave | |
| --- | --- | --- | --- |
| `zai` | compatível com OpenAI | `ZAI_API_KEY` | Z.ai (Zhipu GLM), internacional |
| `zai_coding` | compatível com OpenAI | `ZAI_API_KEY` | Z.ai GLM Coding Plan |
| `zai_anthropic` | Anthropic | `ZAI_API_KEY` | Z.ai, ponto final compatível com Anthropic |
| `zhipu` | compatível com OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel, China |
| `zhipu_coding` | compatível com OpenAI | `ZHIPUAI_API_KEY` | Zhipu BigModel Coding Plan, China |
| `anthropic` | Anthropic | `ANTHROPIC_API_KEY` | Claude |
| `openai` | compatível com OpenAI | `OPENAI_API_KEY` | OpenAI |
| `openrouter` | compatível com OpenAI | `OPENROUTER_API_KEY` | muitos modelos atrás de uma chave |
| `ollama` | compatível com OpenAI | nenhuma | Ollama nesta máquina |

Os pontos finais da Z.ai e da Zhipu vêm de documentação de terceiros, não das páginas do próprio fornecedor. Confira-os contra o seu plano e sobrescreva qualquer campo se diferirem.

Cada campo de um fornecedor pode mudar, e um fornecedor novo é apenas um nome com um endereço:

```sh
export MARELLE_PROVIDERS__GATEWAY__BASE_URL=https://llm.example.com/v1
export MARELLE_PROVIDERS__GATEWAY__API_KEY_ENV=GATEWAY_KEY
marelle chat -p gateway -m some-model "hello"
```

Outros campos do fornecedor: `kind` (`openai-compatible` ou `anthropic`), `api_key_file`, `auth` (`bearer`, `x-api-key`, `api-key`, `none`), `max_tokens_field`, `stream_usage` e `extra_body`. Se a variável nomeada por `api_key_env` está fixada, ela ganha; senão o arquivo lê-se.

## Configurações

As configurações vêm em camadas: valores padrão integrados, depois variáveis de ambiente, depois bandeiras. Arquivos de configurações encaixam-se entre os valores padrão e o ambiente.

- `MARELLE_CHAT__MODEL=zai/MODEL` fixa `chat.model`. Um sublinhado duplo vale por um ponto.
- `marelle config show [prefix]` lista as configurações e a camada de onde cada uma veio.
- `marelle config explain <key>` mostra cada camada que fixou uma chave, da prioridade mais baixa à mais alta.

| Chave | Padrão | Sentido |
| --- | --- | --- |
| `chat.provider`, `chat.model` | nenhuma | Com quem falar. `chat.model` pode ser `provider/model`. |
| `chat.system` | nenhuma | Prompt de sistema |
| `chat.max_tokens`, `chat.temperature` | nenhuma | Não fixado — deixado ao fornecedor |
| `chat.stream` | `true` | Fluxo da resposta opcional do juiz; as rodadas de agente com ferramentas usam por agora respostas completas, para que as chamadas de ferramentas fiquem inteiras |
| `chat.show_reasoning`, `chat.show_usage` | `false` | Raciocínio e contagens de tokens no stderr |
| `agent.max_turns` | `20` | Máximo de rodadas de ferramentas por pedido de usuário |
| `agent.persist_sessions` | `true` | Guardar conversas locais entre execuções |
| `permissions.shell` | `ask` | Execução de comandos de shell: `allow`, `ask`, `deny` |
| `permissions.files_read` | `allow` | Leitura de arquivos: `allow`, `ask`, `deny` |
| `permissions.files_write` | `ask` | Mudança de arquivos: `allow`, `ask`, `deny` |
| `permissions.network` | `ask` | Descarga de URL: `allow`, `ask`, `deny` |
| `permissions.allowed_paths` | `["."]` | Raízes permitidas para as ferramentas de arquivos |
| `permissions.denied_paths` | nenhuma | Caminhos de arquivos expressamente bloqueados |
| `network.connect_timeout_secs` | `15` | Tempo permitido para conectar |
| `network.idle_timeout_secs` | `300` | Silêncio permitido à espera de dados |
| `network.proxy` | `system` | `system`, `none`, ou um endereço de proxy |
| `network.ca_bundle` | nenhuma | Um arquivo extra de autoridades de certificação de confiança |
| `network.allow_insecure_http` | `false` | Permitir uma chave por `http` simples para uma máquina estranha |
| `update.check` | `true` | Perguntar a GitHub e GitLab por uma versão mais nova ao começo de uma sessão interativa |
| `update.github_repo` | `S-I-L-S/marelle` | GitHub `owner/repo` a quem a verificação pede o lançamento mais novo |
| `update.gitlab_repo` | nenhuma | GitLab `owner/repo` para a mesma verificação. Vazio desliga o GitLab |
| `update.timeout_secs` | `3` | Quantos segundos a verificação pode gastar numa fonte |

A Marelle traz consigo a própria lista de autoridades de certificação de confiança, por isso funciona também onde o sistema não tem, como no Termux. `SSL_CERT_FILE` acrescenta mais.

Quando se acha uma versão mais nova, a sessão menciona-a uma vez — a versão, a que você tem, e onde tomá-la. Pede-se primeiro um lançamento; um repositório que só marca seus commits com etiquetas dá a etiqueta mais nova. O que quer que falhe fica silencioso.

## As suas chaves continuam suas

- As configurações nomeiam uma variável ou um arquivo que guarda a chave, nunca a chave mesma. Uma configuração como `providers.x.api_key` recusa-se.
- A chave não a imprimem `config show`, `config explain` nem `providers`, e corta-se de todo texto de erro que volta de um servidor.
- Ela vai apenas ao endereço do fornecedor, num único cabeçalho, e nunca por `http` simples para outra máquina, salvo que você permita.
- Os redirecionamentos não se seguem, de modo que uma chave não pode seguir para outro lado.

## Línguas

A interface fala inglês, russo, francês e ucraniano. Escolha com `--lang <code>` ou `MARELLE_LANG`; senão decidem `LC_ALL`, `LC_MESSAGES` e `LANG`. Cada mensagem vive num catálogo, e um teste cai se alguma língua deixar passar uma. A meta é cada língua da Eurásia e depois mais.

## Roteiro

1. **Falar.** Adaptadores compatíveis com OpenAI e Anthropic, com fluxo SSE e decodificação de respostas completas. As rodadas de agente com ferramentas usam por agora respostas completas. *Feito.*
2. **Configurações e aspecto.** Arquivos de configurações TOML, globais e por projeto, com `config set`, `get` e `unset` e uma descrição para cada chave. Uma interface tematizada no espírito de [Hermes Agent](https://github.com/NousResearch/hermes-agent): faixa, painel de informações, respostas emolduradas, girador, por defeito no roxo da glicínia, qualquer cor ajustável como HEX. *Feito.*
3. **Muitos modelos.** Apelidos de modelos, perguntar a vários modelos de uma vez e comparar as respostas, cadeias de reserva, e um modelo juiz que os funde. *Feito.*
4. **Agir.** Chamadas estruturadas de ferramentas para fornecedores compatíveis com OpenAI e Anthropic, ferramentas locais de arquivos, busca e terminal, descarga de URL, habilidades locais e permissões allow/ask/deny; a transcrição mostra cada chamada de ferramenta e o seu resultado. *Feito.*
5. **Lembrar.** As conversas nomeadas persistem localmente entre execuções. A memória semântica de longo prazo ainda fica à frente.
6. **Estender.** Servidores MCP, plugues externos, subagentes e um apoio mais rico de habilidades.
7. **Alcançar.** Automação de navegador, portas de mensageiros e tarefas agendadas.

O raciocínio atrás do desenho está em [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Limites conhecidos

- Os seletores de fornecedores e modelos listam os fornecedores configurados e os apelidos de modelos. A Marelle ainda não descobre catálogos de modelos remotos.
- Os seletores sustentam clique de rato para o foco e rolamento com a rodinha; a transcrição e o editor de entrada ficam mandados do teclado.
- Ctrl-C sai da sessão interativa. Ainda não cancela apenas a resposta em curso.
- O ciclo de agente, as ferramentas locais conscientes de permissões e as sessões locais persistentes estão implementados, mas a Marelle fica um protótipo pré-alfa; para o desenho atual e os seus limites, leia [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Desenvolvimento

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
```

As provas nunca tocam a rede: falam com um servidor de simulação em `crates/marelle-testkit`. O GitHub Actions também constrói os binários para Android e verifica a versão mínima de Rust declarada. Para um espelho no GitLab há um conduto GitLab CI; veja [docs/GITLAB.md](docs/GITLAB.md).

## Licença

[AGPL-3.0-only](LICENSE). Os avisos de terceiros estão em [NOTICE](NOTICE). Os arquivos de lançamentos incluem `THIRD-PARTY-LICENSES.txt` para as caixas de Rust dentro do binário.
