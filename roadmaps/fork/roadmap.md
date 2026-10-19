# Roadmap do fork (Enzo e sócio)

> Missão em `~/omni/PROMPT-OMNI-MISSAO.md`. Guia técnico em `docs/fork/CLAUDE.md`.
> Registrar aqui o andamento de cada fase (sessões podem ser substituídas; isto não).

## Repositório e upstream (2026-10-19)

- `origin` = fork público `EnzoViniski/OMNI-AGENTS` (criado pelo Enzo). `upstream` = Felipe.
- Fork no GitHub **não se atualiza sozinho**. Estratégia: `main` do `origin` = nosso produto;
  novidades do Felipe entram por `git fetch upstream && git merge upstream/main` (merge, não rebase:
  histórico compartilhado com o sócio), seguido da verificação completa.
- 0.7.2 do Felipe (agente Antigravity, perguntas/imagens do agente) já mesclada em `fase-1/macos`
  (conflito só em `crates/omni-core/src/lib.rs`, dois `pub mod` novos). Testes: Vitest 393, Rust 158,
  Tauri 25.
- Prompt de continuação para rodar dentro do OMNI: `~/omni/PROMPT-OMNI-CONTINUACAO.md`.

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
- [x] `macos/fonte-monoespacada`: terminal com Menlo/SF Mono (o genérico `monospace` do WebKit é
      Courier; Cascadia/Consolas não existem no Mac). `src/lib/fontes.ts`, teste `fontes.test.ts`.
- [x] Reinício do engine quando o binário muda (`engine_binario_trocado`) verificado no `.app`.
- [x] `macos/pastas-ocultas` (2026-10-19): log do `.app` local mostrou `forbidden path:
      ~/omni/.claude/skills`. Causa: no Unix o plugin fs exige ponto literal no glob
      (`require_literal_leading_dot` cai para `cfg!(unix)`), então o `**` do `fs:scope` não casava
      `.claude/skills`, `~/.claude` (skills do perfil Padrão), `.env`, `.github`. Correção:
      `plugins.fs.requireLiteralLeadingDot: false` em `tauri.conf.json`; teste
      `src-tauri/tests/escopo_de_arquivos.rs` (falha sem a correção). **Falta ver no `.app`** (exige
      recompilar o Local, o que derruba sessões rodando nele).
- [x] `correcao/fluxos-sem-tela` (2026-10-19, a partir do `main` do autor): CLI headless dos fluxos
      limpa `CLAUDE_CONFIG_DIR`/marcas herdadas de uma sessão do Claude e lê o stderr numa thread
      (guarda os últimos 16 KB). Testes `cli_sem_tela_nao_herda_a_conta_da_sessao_do_claude` e
      `stderr_grande_nao_trava_a_leitura_do_stdout`.
- [ ] 0.7.2 no Mac: o `.app` Local aberto ainda é 0.7.1 (compilado antes do merge). Código do
      Antigravity é portátil (`~/.gemini/antigravity-cli`, `agy` em `~/.local/bin` já está no PATH
      ampliado), mas `agy` não está instalado nesta máquina: verificar exige instalar (software
      global, perguntar) e login Google. Perguntas/imagens do agente: checar no `.app` recompilado.
- Atenção: sessões do Claude abertas no "OMNI AGENTS Local" morrem quando ele é recompilado/reaberto.

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

## Plano da Fase 2 — nosso produto (aguarda aprovação e as respostas acima)

Fatos que pesam na decisão (docs oficiais, 2026-10-19):
- Fork de repositório público é **sempre público**; não dá para mudar a visibilidade de um fork
  sozinho ([GitHub Docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-permissions-and-visibility-of-forks)).
  Para privado: repositório novo privado (`gh repo create --private`) recebendo nossas branches,
  com o autor só como remoto `upstream` (rebase periódico continua igual).
- Repo privado: o updater não baixa `latest.json` de release privada sem token. Opções:
  (a) repositório **público só de releases** (binários + `latest.json`), código privado;
  (b) atualização manual pelo `.dmg`. Recomendo (a).
- `gh` está logado na conta `EnzoViniski`.

Passos:
1. Repositório (fork público ou cópia privada) + `origin`; push das branches `fase-*` e `macos/*`
   (só depois do seu ok). Lembrete: avisar o Felipe do fork; MIT e créditos preservados.
2. Identidade num lugar só: nome do produto, bundle id novo (= pasta de dados nova), **porta padrão
   própria** (as constantes de `omni-protocol` já aceitam isso), título da janela, nome no código
   como constante. Os dados do app do autor ficam intactos; importação opcional de perfis/workspace
   com backup antes.
3. Updater próprio: `tauri signer generate` (a chave privada você cadastra nos secrets do GitHub;
   nunca passa pelo chat), pubkey e endpoint novos em `tauri.conf.json`, workflow de release
   adaptado (só macOS arm64/universal + o que vocês usarem).
4. Ícones novos (`tauri icon` a partir de uma imagem-base) e 2–3 direções visuais em mockup para
   você escolher; implementação pela camada de tokens (variáveis CSS + `tailwind.config.ts`).
5. README/manual/screenshots no fim.

## Andamento da Fase 2 (branch `fase-2/identidade`, 2026-10-19)

- [x] Identidade num lugar só: `identidade.json` (nome, nome curto, bundle id = pasta de dados,
      portas do engine/dev/celular, pasta de perfis, repositório, updater). Rust lê por
      `crates/omni-protocol/build.rs` (`OMNI_ID_*` → `NOME_DO_PRODUTO`, `DATA_DIR`, portas,
      `PASTA_DE_PERFIS`); interface por `src/lib/identidade.ts`; Tauri por sobreposição gerada em
      `scripts/identidade.mjs` (`tauri build --config ...`), sem tocar no `tauri.conf.json` do autor.
      Build local deriva (`+ " Local"`, `.local`, porta + 60). Valores ainda = os do autor.
- [ ] Trocar valores quando o Enzo der o nome (só `identidade.json`). `pastaDePerfis` fica
      `OMNI-AGENTS` de propósito: mantém os logins dos perfis extras já feitos.
- [ ] `release.yml` (tauri-action) ainda não passa a sobreposição nem lê o nome: fazer junto do updater.
- [ ] Updater: o Enzo roda `tauri signer generate` e cadastra a privada nos secrets; a pubkey vai
      em `identidade.json` → `updater.pubkey`.
- [ ] Ícones (`tauri icon`) depois da direção visual.
- [x] 3 direções visuais em `roadmaps/fork/visual/direcoes.html` (Grafite, Sinal, Barro) +
      `tokens-*.css` (mesmos nomes de variável do `src/index.css`). Para aplicar: paleta padrão em
      `src/lib/palettes.ts`, fonte base do `body`, cores fixas do xterm em `TerminalPane.tsx`,
      `--c-on-accent` no `.btn-primary`, fontes empacotadas (app offline).

## Interface (branch `fase-2/interface`, 2026-10-19) — decisão do Enzo

Manter a identidade visual do OMNI; trocar só a logo pela da Medza. Direções Grafite/Sinal/Barro
ficam arquivadas.
- [x] Logo Medza: `src/assets/marca/`, componente `MarcaDoProduto` no lugar do `OmniLogo`
      (barra, web do celular, pareamento). Ícones do app gerados por `tauri icon` a partir de
      `roadmaps/fork/marca/icone-do-app-1024.png`. Ícone da janela não é mais repintado com a paleta.
- [x] Semáforos nativos do macOS: `src-tauri/tauri.macos.conf.json` (`titleBarStyle: Overlay`,
      `trafficLightPosition` 14/20); `TitleBar` esconde os botões próprios no Mac (`src/lib/plataforma.ts`).
      **Ver no `.app`**: posição vertical dos semáforos e arrasto da janela.
- [x] Projetos como pastas com as conversas dentro, várias abertas, cor por agente
      (`src/features/projects/PastasDeProjetos.tsx`, `src/lib/coresDosAgentes.ts`); seção CHATS saiu
      da lateral. Clique numa conversa de outro projeto usa `FOCUS_SESSION`.
- [x] Gaveta "Uso" na barra de título, entra da direita (`GavetaDeUso.tsx`), reaproveita a lista do
      popover (`ListaDeUsoDasContas`, agora com Antigravity).
- CSS do fork em `src/styles/fork.css` (fora do `index.css` do upstream).
- [x] Nome: **Medza Agents** (2026-10-19). `identidade.json`: bundle id `ai.medza.agents`, portas
      47421 (engine) / 47422 (celular) / 47441 (dev), repositório `EnzoViniski/OMNI-AGENTS`.
      Updater aponta para o nosso `latest.json`, mas sem pubkey própria ainda (release não configurada).
- O build Local de teste foi compilado com `OMNI_PASTA_DE_DADOS=com.omni.agents.local` e
  `OMNI_PORTA_DO_ENGINE=47381` para manter os dados atuais do Enzo (`Medza Agents Local.app`).

## Fase 3 — andamento

- [x] `crates/omni-core/src/roteamento.rs` (branch `fase-3/roteamento`, a partir da `fase-1/macos`):
      modelo de conta, `Politica` (`routing.json`), filtros duros, modos fila/auto,
      `proxima_apos_falha` só headless, explicação com fonte/idade; `Cota::de_uso` reaproveita
      `usage.rs`. 31 casos de teste em tabela. Falta: rotas no engine, tela, Novo agente, nó de fluxo.

## Plano da Fase 3 — fila de contas (aguarda a Fase 2 e o tipo da conta Team)

1. `crates/omni-core/src/roteamento.rs` (novo): modelo de conta (`tipo`, `plano`, `janelas`,
   `uso_extra`, `orcamento`, `expira_em`, `max_concorrencia`, `apelido`), fila de etapas
   (`routing.json` no diretório do engine), filtros duros (política de dados do projeto, orçamento,
   uso extra, concorrência, limite estourado) e os modos `fila` e `auto`. Função de escolha pura,
   com testes em tabela (cada filtro, cada modo, conta repetida, reset de janela) e a explicação
   "por que esta conta".
2. Fonte de cota: o que o OMNI já lê (`/usage` do Claude, `rate_limits` do Codex); quando não houver,
   "desconhecido" + fonte + idade do dado. Preços por modelo em arquivo editável, nunca no código.
3. Rotas no engine (arquivo novo, chamado por uma linha no `mobile.rs`), tela
   `src/features/roteamento/` em Configurações → "Fila de contas", e uso pelo "Novo agente", pelo nó
   de fluxo (fecha "Escolher conta com uso disponível" em `roadmaps/orquestracao/roadmap.md`) e
   depois pelo Copiloto.
4. Queda automática para a próxima conta só em execução headless; sessão interativa recebe aviso.

## Pendências técnicas anotadas (não urgentes)

- [x] `fluxos.rs`: `CLAUDE_CONFIG_DIR` herdado no headless e stderr em thread (ver Fase 1,
  `correcao/fluxos-sem-tela`).
- Slug do transcript não trata `/tmp` → `/private/tmp` (symlink do macOS); só afeta projeto em `/tmp`.
- Atalhos usam Ctrl no macOS (pendência antiga do autor); Ctrl+C com seleção copia em vez de SIGINT.
- Convenção: o andamento do fork fica em `roadmaps/fork/` e não ganha linha no
  `roadmaps/omni-agents/roadmap.md` do autor, para o rebase não conflitar a cada release dele.

## Persistência e Antigravity (2026-10-19)

- [x] Antigravity "travado" para trocar modelo: o alvo `safari13` do Vite fazia o esbuild apagar a
      declaração de variável do `requestMode` do xterm 6 → `ReferenceError: t` quando a CLI consulta um
      modo (DECRQM) e a resposta nunca chegava. Alvo agora `safari14` (`vite.config.ts`; o macOS 11
      mínimo já tem Safari 14). Erro aparecia no `app.log` como `ui erro_nao_tratado`.
- [x] O "Novo agente" lembra o último agente e a conta usada em cada um; o modelo/esforço aplicado
      por último em cada agente entra nas conversas novas (`src/lib/escolhasSalvas.ts`, chave
      `medza-escolhas` no localStorage).
- Por que "tudo sumia" hoje: cada build trocou de identidade (OMNI AGENTS Local → Medza com pasta
  `com.omni.agents.local` → Medza com `ai.medza.agents.local`). localStorage é por bundle id e
  sessões/contas são por pasta de dados. Daqui em diante, manter `ai.medza.agents.local` (build
  local sem `OMNI_PASTA_DE_DADOS`). O estado antigo segue em `com.omni.agents.local` (não migrado).
- Pendente: modo de permissão não é guardado (só shift+tab na CLI); abas em segundo plano só
  revivem ao serem selecionadas; Codex reaberto pelo terminal começa conversa nova.
