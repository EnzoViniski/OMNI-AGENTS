# Roadmap do fork (Enzo e sócio)

> Missão em `~/omni/PROMPT-OMNI-MISSAO.md`. Guia técnico em `docs/fork/CLAUDE.md`.
> Registrar aqui o andamento de cada fase (sessões podem ser substituídas; isto não).

## Decisões pendentes do Enzo

- [ ] Conta do GitHub e nome do repositório; público ou privado; nome do produto (Fase 2).
- [ ] Tipo do benefício Claude for Startups (assentos Team ou crédito de API) — colar `/status`
      e `/usage` de uma sessão com o perfil "Enzo Medza".
- [ ] Plano da conta Padrão (`~/.claude`), plano do Codex/ChatGPT, Google Pro pessoal ou Workspace.
- [ ] Projetos confidenciais que não podem sair da conta Team.
- [ ] Qual perfil usar quando for preciso rodar CLI real com prompt.

## Fase 0 — preparação (branch `fase-0/base`)

- [x] Clone em `~/omni/OMNI-AGENTS`, remoto `upstream` (2026-10-19, `main` em `569c7f3`, v0.7.1).
- [x] Backup de `~/Library/Application Support/com.omni.agents` em
      `~/omni-backups/com.omni.agents-2026-10-19.tgz` (permissão 600).
- [x] Isolamento: dev já tinha porta 47341 e pasta `.dev`. Release ganhou porta/pasta
      sobrescrevíveis na compilação (`OMNI_PORTA_DO_ENGINE`, `OMNI_PASTA_DE_DADOS`,
      `crates/omni-protocol/src/lib.rs`) e `scripts/build-mac-local.mjs` (porta 47381, pasta e bundle
      id `com.omni.agents.local`, sem updater).
- [x] Linha de base: typecheck limpo; Vitest 390/390; Rust 141 ok (o teste `/pc` exige
      `dist-mobile`, gerado por `npm run build:engine`).
- [x] Log em disco só com metadados: `crates/omni-core/src/registro.rs` (4 testes), engine
      (`main.rs`: subida, porta ocupada, sessão aberta/terminou, pedido com erro), app
      (`lib.rs`/`engine_client.rs`: subida, PATH do shell, ensure_engine, engine indisponível) e UI
      (`src/lib/registroDeErros.ts` → comando `registrar_evento_da_ui`).
- [x] `.app` local compilado (`node scripts/build-mac-local.mjs`, ~5 min) e aberto pelo
      LaunchServices (`open`, mesmo ambiente do Finder). Log confirmou `term=nao lang=nao` no app e no
      engine; engine instalado (47321) intocado. Bundle ad-hoc + hardened runtime, como o do autor.
- [x] Inventário de contas inicial (`roadmaps/contas/inventario.md`), com planos e termos
      consultados na documentação oficial; campos `?` aguardam o Enzo.
- [ ] Ver a janela: computer use não está disponível nesta sessão; a checagem visual fica com o Enzo.

## Fase 1 — macOS (branch de integração `fase-1/macos`)

Correções em branches pequenas a partir do `main` do autor (dá para oferecer ao upstream):

- [x] `macos/ambiente-do-pty`: `TERM=xterm-256color`, `COLORTERM=truecolor` e `LC_CTYPE` (só se
      nada de locale vier) no ambiente dos terminais (`terminal_e_locale_padrao`,
      `crates/omni-engine/src/main.rs`); teste `terminal_aberto_pelo_finder_ganha_term_e_locale`.
      Verificado no `.app` local: shell com `TERM=xterm-256color`, TUI do `claude` com cor
      (0 → 22 sequências SGR na abertura).
- [x] `macos/copia-do-engine`: `build-engine.mjs` apaga antes de copiar o binário (sobrescrever um
      executável em uso faz o macOS matar o processo).
- [x] Reinício do engine quando o binário muda (`engine_binario_trocado`) verificado no `.app`.

Pendente (precisa dos olhos do Enzo no app, roteiro abaixo): conversa com streaming, troca de
modelo no selo, retomar sessão, dois perfis Claude com contas separadas, fechar/reabrir janela.

### Roteiro para o Enzo (5 min)

1. Abrir `~/omni/OMNI-AGENTS/target/release/bundle/macos/OMNI AGENTS Local.app` pelo Finder
   (convive com o instalado: porta, dados e janela próprios).
2. Adicionar um projeto, abrir terminal (cores e edição de linha normais?), Novo agente Claude e
   Codex, mandar um "oi" curto, abrir a aba Conversa, trocar o modelo, fechar e reabrir a janela.
3. Print do que falhar + `~/Library/Logs/com.omni.agents.local/app.log` e `engine.log` (só
   metadados, pode colar).
4. Dizer o que exatamente "não funciona" no app instalado (tela em branco? terminal sem prompt?
   agente não responde? conversa vazia?).

Achados até agora (reprodução com o engine de dev num ambiente igual ao do Finder):

- **`TERM` ausente no PTY** (corrigido em `macos/ambiente-do-pty`). App aberto pelo Finder herda o ambiente do launchd (sem `TERM`,
  `COLORTERM`), e nem o engine nem o `portable-pty` definem. Efeito: zsh em modo degradado de
  edição, `claude` sem nenhuma cor. `npm run dev` herda do terminal e esconde. Correção prevista:
  `TERM=xterm-256color`, `COLORTERM=truecolor` no `ambiente_do_terminal` quando ausentes.
- PATH: `import_login_shell_path` funcionou (≈1 s) e achou `claude`/`codex`; o shell da PTY é
  interativo (tem tty), então lê `.zshrc` mesmo com `-l`.
- `LANG` chegou como `C.UTF-8` e `locale charmap` = UTF-8 nesta máquina.
- Flags usadas pelo código existem no `claude` 2.1.302 e no `codex` 0.163.0.
- Slug de `projects/<slug>`: bate com o disco; `/tmp`→`/private/tmp` (symlink) ainda não tratado.
- `scripts/build-engine.mjs` só sabe derrubar o engine de dev no Windows (`LOCALAPPDATA`).
- O app instalado já está na 0.7.1 (atualizou sozinho); a UI estava fechada e o engine dele
  continua vivo desde 17/10 19:20, com dois `claude` rodando há ~9 h — o spawn funciona lá; o
  defeito relatado deve estar na interação/tela.
- Transcript do Claude 2.1.302: `assistant.message.model`, `effort` no envelope e
  `permission-mode` batem com o que `crates/omni-core/src/runtime.rs` lê.
- Perfil Padrão não define `CLAUDE_CONFIG_DIR` (Keychain da conta nativa); extras definem (Keychain
  próprio por pasta), o isolamento previsto.
