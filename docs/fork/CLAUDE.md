# Guia do fork (Enzo e sócio)

Este clone é a base do nosso produto, derivado do OMNI AGENTS (upstream
`https://github.com/FelipeCamposM/OMNI-AGENTS`, MIT). O `CLAUDE.md` da raiz é do autor e está
desatualizado (fala de CAMPS-UTILS e Windows); **este arquivo manda** onde os dois divergirem.
O brief da missão está em `~/omni/PROMPT-OMNI-MISSAO.md` (fases 0 a 5). Responder em pt-BR,
curto e direto.

## Estado e onde registrar

- Andamento das fases do fork: `roadmaps/fork/roadmap.md` (fonte de verdade entre sessões).
- Inventário de contas: `roadmaps/contas/inventario.md` (sem e-mails, tokens ou ids de conta).
- Convenção do autor: roadmap por tema em `roadmaps/<tema>/roadmap.md` + uma linha em
  `roadmaps/omni-agents/roadmap.md`.
- Remotos: `upstream` = autor. `origin` só depois que o Enzo confirmar conta/nome/visibilidade.
  Uma branch por fase (`fase-0/...`, `fase-1/...`), commits pequenos. Correções genéricas de macOS
  em branches pequenas e independentes, para poder oferecer ao upstream.

## Comandos (verificados no macOS arm64, 2026-10-19)

```bash
npm install
npm run typecheck                 # tsc --noEmit
npm test                          # vitest (jsdom), ~390 testes
npm run build:engine              # gera dist-mobile + engine/shim debug em src-tauri/binaries
cargo test --no-fail-fast -p omni-engine -p omni-core -p omni-protocol
cd src-tauri && cargo check       # app Tauri
npm run dev:vite                  # só a interface (invoke() falha; há mock em alguns serviços)
npm run dev                       # app completo; herda o ambiente do terminal (esconde bugs do Finder)
node scripts/build-mac-local.mjs  # .app local isolado do app instalado (ver abaixo)
```

- `cargo test` do engine precisa do `dist-mobile/` (o `mobile.rs` embute com `include_dir!`):
  rode `npm run build:engine` (ou `npm run build:mobile`) antes, senão o teste `/pc` dá 404.
- Rust vem do Homebrew (sem `rustup`): só o alvo nativo `aarch64-apple-darwin`.

## Isolamento (app instalado × dev × build local)

| | porta do engine | pasta de dados | bundle id / webview |
|---|---|---|---|
| App instalado (release do autor) | 47321 | `~/Library/Application Support/com.omni.agents` | `com.omni.agents` |
| `npm run dev` (debug) | 47341 | `com.omni.agents.dev` | `com.omni.agents` |
| `scripts/build-mac-local.mjs` | 47381 | `com.omni.agents.local` | `com.omni.agents.local` |
| Medza Agents (release nossa, `identidade.json`) | 47421 | `ai.medza.agents` | `ai.medza.agents` |
| Medza Agents dev | 47441 | `ai.medza.agents.dev` | — |

Identidade do produto num lugar só: `identidade.json` (ver `scripts/identidade.mjs`). O build local
deriva dela (`<bundleId>.local`, porta + 60) salvo `OMNI_PASTA_DE_DADOS`/`OMNI_PORTA_DO_ENGINE`.

- Porta e pasta de release são constantes de `crates/omni-protocol` sobrescrevíveis **na
  compilação** por `OMNI_PORTA_DO_ENGINE` e `OMNI_PASTA_DE_DADOS` (app aberto pelo Finder não recebe
  env). Em tempo de execução, `OMNI_ENGINE_PORT` e `OMNI_DATA_DIR` ainda vencem.
- `ensure_engine` derruba o engine que responder na porta se o binário dele "mudou" (compara
  caminho e data). Por isso **nunca** rode um build de release na porta 47321 com o app instalado.
- Perfis extras ficam em `~/Library/Application Support/OMNI-AGENTS/profiles/<provider>/<nome>`,
  compartilhados entre todas as variantes; o registro deles (`profiles.json`) é por pasta de dados.
- Backup da pasta do instalado: `~/omni-backups/` (faça outro antes de mudar formato de estado).

## Log em disco (só metadados)

`crates/omni-core/src/registro.rs`. macOS: `~/Library/Logs/<pasta de dados>/{engine,app}.log`,
girado em 2 MB, 3 antigos. Eventos: `engine_iniciou` (porta, exe, TERM/LANG presentes),
`engine_porta_ocupada`, `sessao_aberta`/`sessao_terminou`, `pedido_falhou` (tipo + código),
`app_iniciou` (exe, resultado do PATH do shell, claude/codex achados), `engine_aberto`,
`engine_nao_abriu`, `engine_sem_resposta`, `engine_binario_trocado`, `ui` (exceções da interface,
via `registrar_evento_da_ui`), `panico`. `limpar()` corta trechos entre aspas (mensagens de parser
citam o valor) e limita tamanho. `OMNI_LOG_DEBUG=1` liga o nível `depurar`. `OMNI_LOG_DIR` troca a pasta.

## Arquitetura (lida no código)

- `crates/omni-engine`: processo separado, dono das PTYs (`portable-pty`), sessões, scrollback,
  conversas, servidor do celular (`mobile.rs`, axum, também atende a visão "Conversa" do desktop
  via `EngineRequest::Api`), executor de fluxos (`fluxos.rs`), detecção de estado/aprovação
  (`interaction.rs`). IPC: JSON por linha em TCP loopback com token (`engine.token`, ADR-004).
  Fechar a janela não mata o engine (ADR-003).
- `crates/omni-protocol`: `EngineRequest`/`EngineResponse`, constantes de porta e pasta.
- `crates/omni-core`: domínio sem I/O de rede: `cli_path` (amplia PATH com locais conhecidos),
  `usage`, `runtime` (modelo/esforço lidos do transcript da CLI), `conversations`, `agentes`,
  `fluxos`, `compartilhar`/`extensoes`, `relatorio`, `registro`.
- `crates/omni-shim`: só para projetos em WSL/SSH; vira `CLAUDE_CODE_SHELL_PREFIX` e desvia os
  comandos do agente para o alvo. Projeto local não usa.
- `src-tauri/src`: casca. `engine_client.rs` (spawn/ping do engine, `import_login_shell_path`,
  comandos de terminal), `profiles.rs` (um config dir por perfil: `CLAUDE_CONFIG_DIR`,
  `CODEX_HOME`), `account_usage.rs`, `conversations.rs`, `git_client.rs`, `agentes.rs`, `fluxos.rs`.
- `src/`: React + Vite + Tailwind, UI em pt-BR. `src/features/*` por tela; `src/mobile` é a web do
  celular, reaproveitada como chat da aba Conversa (`features/conversa/transporteDoEngine.ts`).
- Agente é **digitado no shell** da PTY (`initial_command`), não lançado direto: o PATH e o
  ambiente do shell de login valem para a CLI.

## Regras duras (do brief)

- Nunca ler/imprimir/logar/commitar: `engine.token`, `mobile.json`, chave privada do updater,
  `.credentials.json`, `.claude.json`, Keychain. Testes com fixtures sintéticas.
- O app só dirige as CLIs oficiais; nunca extrai nem reutiliza token OAuth. Assentos são pessoais.
- Log e telemetria: só metadados, nunca conteúdo de conversa.
- Perguntar antes de: criar repo/fork, push, release, secrets do GitHub, instalar software
  global, encerrar o app instalado, apagar/migrar dados de `com.omni.agents`. Nada para o upstream
  sem pedido. Nunca `npm audit fix --force`.
- Testes automatizados com CLIs falsas; CLI real só com perfil confirmado pelo Enzo, prompt curto,
  menor modelo, sem laço longo/paralelo.
- Preços, IDs de modelo e limites: conferir na documentação oficial na hora, nunca de memória.
- Código novo em arquivos novos; mudança em arquivo grande do autor pequena, localizada e comentada.
