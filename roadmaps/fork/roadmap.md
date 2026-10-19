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

## Voz no chat e "Reiniciar para atualizar" (2026-10-19)

- [x] Microfone no chat (só no app de desktop): `src-tauri/src/voz.rs` grava com `cpal` e transcreve
      com `whisper-rs` 0.16 (whisper.cpp, Metal no Mac), pt-BR, tudo local. Modelo
      `ggml-large-v3-turbo-q5_0.bin` (574 MB, SHA256 conferido) baixado no primeiro uso pelo `curl`
      para `<Application Support>/OMNI-AGENTS/modelos`, compartilhado entre variantes. Interface em
      `src/mobile/BotaoDeVoz.tsx` (clique grava, clique transcreve e o texto entra no campo, sem enviar).
      Permissão: `src-tauri/Info.plist` (`NSMicrophoneUsageDescription`). Build exige `cmake` (Homebrew).
      Log só metadados (`voz_modelo_baixado`, `voz_transcreveu`: duração, ms, nº de caracteres).
- [x] Botão "Reiniciar para atualizar" na barra (`src/components/AvisoDeAtualizacao.tsx` +
      `src-tauri/src/atualizacao.rs`): compara a data dos executáveis do app/engine com a da abertura;
      se o engine mudou, pede confirmação (reabrir reinicia os agentes).
- [x] Link do Metal: `src-tauri/build.rs` linka `libclang_rt.osx.a` (`__isPlatformVersionAtLeast` do
      `@available` do whisper.cpp). `cargo check` não pega; só o build release.
- [x] Áudio mudo (sem permissão de microfone) não vai mais ao whisper, que inventava "E aí": erro
      claro na tela + `voz_audio_mudo` no log com o pico.
- [x] Build com o app aberto não derruba mais os agentes: `ensure_engine` só troca o engine na
      primeira conferência da janela (`ENGINE_CONFERIDO` em `engine_client.rs`); depois disso vira o
      botão. Antes, o app reiniciava o engine sozinho no meio do trabalho.
- Pendente: confirmar a transcrição com permissão dada (build ad-hoc pode pedir de novo a cada
      assinatura); sessão Claude revivida que o chat não reconhece como pronta ("Entrada do CLI não
      reconhecida"); atalho de teclado para ditar; o CI e o sócio precisam de `cmake`. Nada commitado.
- [x] Microfone mudo era o hardened runtime (`codesign` mostra `flags=adhoc,runtime`) sem
      `com.apple.security.device.audio-input`: o macOS nem pergunta. `src-tauri/Entitlements.plist` +
      `bundle.macOS.entitlements` no `tauri.conf.json`.
- [x] Prévia ao vivo da fala: `voz_previa` transcreve os últimos 20 s sem parar a gravação (a
      interface pede uma por vez, a cada ~0,6 s); ao parar, `voz_transcrever` relê o áudio inteiro e é
      esse texto que entra no campo. Callback do microfone usa `lock` para a cópia da prévia não abrir
      buraco no áudio.

## Tela cheia e otimização (2026-10-19)

- [ ] Tela cheia no Mac como no Chrome (3ª versão, aguarda teste do Enzo): `src-tauri/src/tela_cheia.rs`
      (ligado no `setup` de `lib.rs`; deps `objc2`/`block2`/`objc2-foundation`/`objc2-app-kit` só no
      macOS). Ao entrar em tela cheia pendura um `NSTitlebarAccessoryViewController` de 1 pt que
      recebe o `setRevealAmount:` privado (gancho do Chromium). O deslocamento é só a barra de menus
      que entra na janela × `revealAmount` (log `tela_cheia_barra`: 30 px no ultrawide do Enzo, sem
      notch), em `--medza-barra-revelada`; `src/styles/fork.css` desce só a TitleBar com `top` (não
      `transform`: transform virava bloco de contenção da gaveta de Uso, `position: fixed`, que ficava
      encavalada) e desce a gaveta junto.
      Histórico: v1 deslocava o `#root` inteiro; v2 escondia a `NSToolbarFullScreenWindow` com
      `alphaValue` 0 + semáforos próprios — não pegou no macOS 26 (faixa cinza continuou; o AppKit
      anima o alfa) e foi removida (`SemaforosDaTelaCheia.tsx` e a permissão `allow-set-fullscreen`
      saíram). v3: a janela da faixa fica transparente (`opaque` NO, fundo `clearColor`, fundos
      `*VisualEffect*`/`*BackgroundView*` sem botão dentro escondidos) e os semáforos nativos ficam
      por cima da TitleBar, que já reserva 78 px. Log `tela_cheia_faixa janela=<classe>
      fundos=<Classe:escondido|mantido,...>` na 1ª revelação: se a faixa ainda aparecer, é por ele
      que se vê qual vista desenha o cinza. Fora da tela cheia nada muda.
- [x] Monaco tardio: `src/features/files/FilePaneTardio.tsx` (lazy + Suspense), usado por
      `PaneView.tsx` e `MarkdownPane.tsx`. React Flow tardio: `src/features/fluxos/FluxosViewTardio.tsx`
      (usado no `App.tsx`). Chunk principal 5.371 kB → 1.160 kB (gzip 1.454 → 351 kB); CSS principal
      373 → 195 kB.
- [x] `useTerminalSessions` (poll de 1 s que alimenta o App inteiro) devolve a mesma referência quando
      a lista não mudou: com tudo parado o App não re-renderiza mais a cada segundo. Teste
      `src/test/useTerminalSessions.test.tsx`.
- [x] `TerminalPane`: com a janela oculta (`document.hidden`) o snapshot cai de 100 ms para 1 s
      (estado/aviso/failover seguem vivos) e volta a ler na hora no `visibilitychange`.
- [x] `GitPanel`, `GitBranchStatus` (git status a cada 3 s, cada um) e `DockerPanel` (docker ps a cada
      5 s) não rodam com a janela oculta.
- Medido e não mexido: `path_ms` 680–1.150 ms no `app_iniciou` = `zsh -ilc` síncrono antes da
  janela abrir (`import_login_shell_path`). Proposta: guardar o último PATH em disco, usar na hora e
  conferir em segundo plano antes do spawn do engine (cuidado: `set_var` com threads vivas). CPU com o
  app em uso (este terminal rodando um agente): WebContent ~23%, GPU ~10%, app 4%, engine 2–4% —
  confundido pela própria sessão; medir de novo com o app parado.
- Propostas (não feitas): store único de `git status` por projeto (hoje painel e rodapé leem em
  dobro); worker `ts.worker` do Monaco (7 MB) só se usar TS/JS; gsap (230 kB) e qrcode (73 kB) fora do
  chunk principal; pausar fundos WebGL com a janela oculta (o rAF do WebKit já para com a página
  oculta, conferir); `ensureEngine` a cada 1 s junto do `listTerminalSessions` poderia virar só no erro.

## Sessões: diagnóstico e correções (2026-10-19)

Fonte: `~/Library/Logs/ai.medza.agents.local/{engine,app}.log` (só metadados) + `sessions.json`/
`conversations.json` (só ids, cwd, comando) + nomes/datas de arquivo em `~/.claude/projects`.

- Reinícios de hoje: 6 `engine_iniciou` com `sessoes_salvas>0` vindos de `engine_binario_trocado`
  (builds) e `app_reiniciou_para_atualizar`. Cada um derruba todas as PTYs; as abas revivem com o
  mesmo id quando selecionadas (`resize_terminal ... session is not running` seguido de
  `sessao_aberta`). Isso é esperado; não é bug novo.
- [x] **Projeto do Yorcookie rodando sem nada na Conversa** (causa confirmada). O cwd mora em
      `~/Library/Application Support/Claude/local-agent-mode-sessions/.../outputs/yorcookie` (214
      caracteres). Acima de 200, o Claude Code corta o nome da pasta em 200 e põe `-<djb2 base36>`
      (`...-out-egl4st`, conferido no disco). `claude_project_slug` não cortava: o caminho guardado no
      índice nunca existia, e o `timeline` pula trecho com caminho ausente ("ainda não gravado") →
      Conversa vazia com o agente trabalhando. Mesmo erro no reinício: `transcript_claude_existe`
      dava falso e o engine redigitava `claude --session-id <id>` (a CLI recusa id em uso) em vez de
      `--resume` → sessão revivida sem a conversa (visto em `terminal-...-6`, ainda com `--session-id`
      no `sessions.json`). Correção: `crates/omni-core/src/pasta_do_claude.rs` (slug igual ao da CLI,
      busca por prefixo se a CLI no Bun usar outro hash, e conserto do caminho velho guardado no
      índice), ligado em `claude_project_slug`/`claude_transcript_path`/`resolve_transcript`/`timeline`
      /`fontes` (`conversations.rs`, comentários "Fork:"). Testes: `pasta_do_claude::tests` (pasta real
      deste caminho) e `reiniciar_conversa_claude_de_caminho_longo_retoma_a_mesma` (engine).
- [x] **Codex reiniciado/revivido começava conversa nova** (pendência anterior, causa no código:
      `comando_ao_reiniciar` devolvia o comando original para tudo que não fosse Claude; `codex fork`
      bifurcava de novo). `crates/omni-engine/src/retomar_codex.rs`: com o id nativo já descoberto no
      índice, reabre com `codex resume <id> <opções>`; sem rollout ainda, fica como antes. Ligado em
      `restart_session` (`comando_ao_reiniciar_com_codex`). Testes em `retomar_codex::tests`.
- Sem causa confirmada: `sessao_terminou codigo=1` 1–5 s depois de reviver (13:03, 14:48) seguido
  de `snapshot ... session not found` = alguém fechou a aba (`close_session`); só ações do usuário
  chamam `closeTerminal`. "Entrada do CLI não reconhecida" em Claude revivido: explicado no caso do
  caminho longo (a CLI saía com erro); em projeto de caminho curto falta ver a tela.
- Pendente: abas em segundo plano seguem revivendo só ao serem selecionadas; `/clear` dentro do
  Claude cria outro `.jsonl` que o índice não acompanha (a Conversa continuaria no arquivo antigo).
  Testar no app: reabrir a aba do Yorcookie → Conversa mostra o histórico; "reiniciar" nela →
  `claude --resume`; Codex com uma mensagem → reiniciar → mesma conversa.

## Energia (2026-10-19)

Medição (antes), "Medza Agents Local" aberto, 5 shells, aba Conversa com este agente trabalhando
(a própria sessão gera saída; não dá para medir sem ela), `top -l 21 -s 2 -stats pid,command,cpu,idlew,power`,
média de 20 amostras, duas rodadas: WebContent 15,6–28,8% CPU, GPU (WebKit.GPU) 7,8–8,5%, engine
7,9–9,4%, app 3,2–5,8% (~16 wakeups/s). `sample` de 8 s: no WebContent, ~10% do tempo do main thread em
`Page::updateRendering`, quase tudo **layout** (flex) disparado a cada quadro; JS e keyframes pesam pouco.
No engine: `mobile::timeline → resumo → capabilities → provider_running` (2× `ps -A` por chamada),
`list_sessions → discover_codex_transcripts` (abre o começo de cada um dos ~120 rollouts do Codex, por
segmento sem caminho, a cada 1 s) e `has_children` (`ps` em todo snapshot de shell calado com filho).

- [x] **Achado principal: o .app de 13:13 empacotou o engine de debug** (31 MB com símbolos e
      `ub_checks`; o release é 4,3 MB). Um `npm run build:engine` (debug) rodou durante o
      `build-mac-local` e trocou `src-tauri/binaries/` antes do bundle. Guarda nova no fim de
      `scripts/build-mac-local.mjs`: falha se o engine do .app tiver mais que o dobro do release.
- [x] Pausa de energia: `src/lib/energia.ts` + `src/hooks/useEconomiaDeEnergia.ts` (ligado no `App.tsx`).
      Janela oculta, sem foco (`blur`) ou 60 s sem mouse/tecla/rolagem → `html[data-energia="pausa"]`.
      `src/styles/energia.css` (importado no `main.tsx`) põe `animation-play-state: paused` só nas
      animações **infinitas** decorativas (neon da nav, neon-bar, badge-pulse, bell-ring, shimmer,
      avatar vivo, painel vazio, lançador, fluxos, `cz-luz`/reflexo do composer, brilho do
      "trabalhando", parar-pulsa, cursor do xterm). Animação de entrada fica de fora (mensagem nova
      pausada ficaria invisível); voz e arrastar também. Congela no quadro atual, retoma de onde parou.
- [x] Fundos WebGL (GradientWaves — **o padrão de quem instala** —, FloatingLines, FaultyTerminal)
      pediam quadro a 60 fps sempre (mesmo com movimento desligado/`still`). Agora pedem por
      `proximoQuadro`/`cancelarQuadro` (`energia.ts`): pausado, o pedido fica guardado e volta ao
      retomar. Troca de 2–3 linhas em cada arquivo vendorizado (anotado em `backgrounds/registry.tsx`).
- [x] `[data-omni-context="terminal"] { contain: strict }` (energia.css): o host do xterm é
      `absolute inset-0`, então vira fronteira de layout — saída da PTY não relayouta o app inteiro.
- [x] `TerminalPane`: PTY calada há 10 leituras (~1 s) → poll 250 ms (em foco) / 500 ms (sem foco), em
      vez de 100 ms; saída nova volta a 100 ms, tecla antecipa a leitura na hora. Chat: o spinner de
      120 ms do "trabalhando" para com a pausa (`src/mobile/Chat.tsx`).
- [x] `useTerminalSessions`: `ensureEngine` só na primeira leitura e depois de falha (teste novo em
      `src/test/useTerminalSessions.test.tsx`).
- [x] Engine: `crates/omni-engine/src/economia.rs` (novo): `ps` com cache de 1 s (usado por
      `has_children` e pelo novo `provider_running_recente`, só em `capabilities`; quem vai digitar ou
      esperar o CLI sair segue lendo na hora) e busca de rollouts do Codex em `list_sessions` no máximo
      a cada 15 s por pasta de dados (o mesmo ritmo que `mobile::summaries` já tinha).
- Verificado: `npm run typecheck`, `npm test` (398), `npm run build:engine` + `cargo test` (engine 89,
  core 81), `src-tauri cargo check`. **O "depois" só vale após `node scripts/build-mac-local.mjs` e
  reabrir o app.** Medir: `ps -axo pid,ppid,command | grep -E "Medza Agents Local|WebKit"` (o
  WebContent nasce junto com o app) e `top -l 21 -s 2 -pid <app> -pid <engine> -pid <WebContent>
  -pid <GPU> -stats pid,command,cpu,idlew,power` em três estados: (a) janela em foco sem agente
  trabalhando, (b) outra app na frente, (c) agente trabalhando na Conversa. Esperado: engine perto de
  0–1% em (a)/(b) com o release; WebContent/GPU caindo em (b) e após 60 s parado em (a).
- Propostas (não feitas): o `JSON.stringify` do `useTerminalSessions` inclui `last_activity_at_ms`/
  `output_seq`, então com um agente trabalhando o App inteiro re-renderiza a cada 1 s (separar a parte
  volátil); `FaixaDoTerminal` (aba de agente) relê `/timeline` inteira a cada 3 s só para os chips —
  bastaria um resumo; `working-verb` (brilho correndo em `background-clip: text`, 60 fps) poderia ser
  `steps()`; fundos WebGL com `still` poderiam desenhar um quadro e parar; git status em dobro
  (painel + rodapé) continua; `neon`/`neon-bar` usam `filter: drop-shadow` animado (repinta a cada
  quadro) — trocar por opacidade de uma camada pré-desenhada se ainda pesar.

## Crash ao sair e Enter na voz (2026-10-19)

- [x] "Medza Agents Local encerrou inesperadamente" a cada reabertura: com o modelo whisper carregado,
      o `exit` roda os destrutores do ggml-metal, que abortam em `GGML_ASSERT([rsets->data count] == 0)`
      (ggml-metal-device.m:608). 5 relatórios em ~/Library/Logs/DiagnosticReports/omni-agents-*.ips,
      todos no reinício/saída. Agora `voz::liberar()` solta o modelo no `RunEvent::Exit` (`lib.rs` usa
      `.build().run(...)`) e antes do `app.restart()` (`atualizacao.rs`).
- [x] Gravando: Enter para e transcreve, Esc descarta (listener na captura da janela em
      `BotaoDeVoz.tsx`, para o Enter não enviar a mensagem).
- Pendente: o macOS pede a permissão do microfone a cada build porque a assinatura é ad-hoc (a
      identidade do TCC é o hash do binário). Resolve com um certificado de assinatura local estável.
- **2ª rodada (build das 13:37, impacto energético 57 no Monitor de Atividade, janela em foco).** Engine
  já é o release: 1,0–1,7%. Medido agora (top 20 s, Conversa transmitindo): app 3–10%, GPU 7–36%,
  WebContent 20–23%. `sample`: no app, thread `CVDisplayLink` viva = WebKit pedindo quadro contínuo;
  as threads `caulk` estão paradas em `timed_wait` (resto do CoreAudio, sem thread de IO: o stream do
  cpal é dropado em `encerrar_gravacao`, usado por `voz_cancelar` e `voz_transcrever` — nada a fazer).
  No WebContent: `updateRendering` → layout desde o `RenderView` por toda a cadeia de flex.
  - [x] Pausa por inatividade de 60 s → **8 s** (`INATIVIDADE_MS`): lendo a Conversa sem mexer, tudo
        que é decorativo congela.
  - [x] O que fica sempre na tela parou de animar sozinho (`energia.css`): neon da nav e barra
        acesa em repouso (pulsam só no hover/foco do botão); badge e sino pulsam 3 vezes e param; avatar
        vivo, botão parar e brilho do "trabalhando" em `steps()`; reflexo do enviar anima `transform` em
        vez de `left` (este forçava layout a cada degrau).
  - [x] Fronteiras de layout: `.aba-conversa-tela` e a lista `.chat` com `contain: strict`; lateral
        (`aside`) com `contain: size layout style`.
  - [x] `useTerminalSessions`: a assinatura ignora `output_seq` e arredonda `last_activity_at_ms` ao
        minuto (só a lateral usa, como "há N min") → o App não re-renderiza mais a cada 1 s com agente
        transmitindo.
  - [x] Fundos WebGL limitados a ~30 fps em `proximoQuadro` (setTimeout + rAF, o display link nem é
        pedido antes da hora); o id guardado durante a pausa continua cancelando depois de retomar
        (antes um efeito desmontado podia deixar o laço vivo). Testes: `src/test/energia.test.ts`.
  - Verificado: typecheck, `npm test` (401), `src-tauri cargo check`. Rust não mudou nesta rodada.

## Arquivos gerados na Conversa e modelos do Claude com versão (2026-10-19)

- [x] **Imagem gerada pelo Codex aparece.** O Codex (app e TUI) grava a imagem em
      `$CODEX_HOME/generated_images/<thread>/…png` e no rollout só um `event_msg` `item_completed` →
      `Extension` `kind: image_gen.generation` com `savedPath` — nunca cita o caminho no texto, então o
      chat dizia "gerei" e nada aparecia. `feed.rs` lê esse item como fala do agente com a imagem
      (`Item::midia`); a imagem vem pela rota que já existia (`/conversas/{id}/imagens/{n}/{k}`).
      Conferido num rollout real: 19 imagens que não apareciam.
- [x] **Documentos viram cartões** (`crates/omni-core/src/entregaveis.rs`, novo): PDF, Word, Excel,
      CSV, PowerPoint, ODF, RTF, EPUB, HTML, SVG, vídeo, áudio, ZIP. Origens: caminho citado na
      resposta (absoluto, `~/`, ou relativo à pasta da conversa entre crases/link markdown — só se
      existir), `Write` do Claude e `*** Add File:` do Codex. Código e texto puro ficam de fora. A pasta
      vem do próprio transcript (`cwd` de cada linha no Claude; `session_meta`/`turn_context` no Codex).
      Imagem citada por caminho relativo também passou a aparecer.
  - Engine: `GET /conversas/{id}/arquivos/{n}/{k}` → `{nome, tipo, dados}` (base64, até 25 MB) para o
    celular baixar. Só caminhos do transcript.
  - App do PC: comandos `abrir_entregavel` e `mostrar_entregavel_na_pasta`
    (`src-tauri/src/abrir_entregavel.rs`): só arquivo comum com extensão de entregável — um `.app` ou
    `.command` citado pelo agente nunca é aberto.
  - Front: `src/mobile/ArquivosDoAgente.tsx` (cartão: nome, pasta; PC abre no app padrão + Finder,
    celular baixa). Na fala (`MessageBubble`) e abaixo do bloco de ações (fora do recolhido).
- [x] **Modelos do Claude com versão** (`crates/omni-core/src/modelos_claude.rs`, novo): os 4 apelidos
      (mais recente de cada linha) + Fable 5.1, Opus 5.5, Sonnet 5.5, Haiku 5.5, Fable 5, Opus 5,
      Opus 4.8/4.7/4.6/4.5, Sonnet 5, Sonnet 4.6, Haiku 4.5, cada um com os esforços que aceita
      (4.6 sem `xhigh`, Opus 4.5 até `high`, Haiku 4.5 sem esforço). IDs e esforços conferidos hoje em
      platform.claude.com (models/overview e build-with-claude/effort). Usado na Conversa e nos agentes.
      `modeloDaLista` prefere o id mais específico (`claude-opus-4-5-20251101` → Opus 4.5, não `opus`).
- Verificado: typecheck, `npm test` (404), `build:engine`, `cargo test` (core 86, engine 88), `src-tauri
  cargo check` + teste do `abrir_entregavel`. Testes novos: `entregaveis.rs`, `modelos_claude.rs`,
  `src/test/entregaveis.test.tsx`.
- Pendente: testar no app com CLI real (gerar imagem no Codex, pedir um PDF ao Claude). Prévia
  embutida de PDF/HTML no chat (hoje abre fora) fica para depois, se fizer falta.

## Imagem quebrada no markdown e reordenar a lateral (2026-10-19)

- [x] Quadro com "?" acima da imagem do Codex: a resposta traz `![…](/Users/…/generated_images/…png)`;
      o markdown virava `<img>` de caminho local, que o webview não carrega. `ImagemDoMarkdown`
      (`src/mobile/Chat.tsx`) só desenha imagem `http(s)`; a local já aparece embaixo do balão.
- [x] Arrastar projetos e conversas na lateral (`src/features/projects/arrastarLinha.ts` +
      `ordemDaLateral.ts`, novos; ligado em `PastasDeProjetos.tsx`). Eventos de ponteiro (como
      `tabPointerDrag.ts`), arraste só depois de 5 px (clique continua clique; o clique do soltar é
      engolido), Esc cancela, linha de destino em destaque (`fork.css`). Conversa só se move dentro do
      próprio projeto. Teclado: ⌥↑/⌥↓ na linha em foco. Ordem em `localStorage` `omni-ordem-lateral`
      (`{projetos, conversas: {projeto: [conversationKey]}}`); sem ordem salva, a natural; item novo
      no fim. Testes: `src/test/ordemDaLateral.test.tsx`.
- Verificado: typecheck, `npm test` (410), build local.

## Arraste com prévia e conversa da lateral para o layout (2026-10-19)

- [x] A linha arrastada acompanha o mouse: `arrastarLinha.ts` clona a linha (`.lateral-fantasma`,
      `aria-hidden`, presa ao ponto onde foi segurada) e a lista se reorganiza ao vivo — o lugar
      previsto vira um espaço tracejado (`.is-arrastando`). A posição é calculada entre as **outras**
      linhas (`colocar` em `ordemDaLateral.ts`), o que deixa a prévia estável enquanto as linhas mudam
      de lugar. Esc/blur cancelam e a ordem volta.
- [x] Conversa da lateral → painel do layout: fora da lateral, o arraste usa `targetAt` (agora
      exportado de `tabPointerDrag.ts`) e avisa os painéis pelo mesmo `TAB_DRAG_EVENT` das abas (origem
      `"lateral"`), então a prévia de zona do `PaneView` aparece sem código novo ali (só o rótulo
      "Abrir neste painel" no centro). Soltar → ação `PLACE_SESSION` (`types/workspace.ts`), feita em
      `features/workspace/colocarConversa.ts` com `CREATE_TAB`/`MOVE_TAB`/`SPLIT_WITH_TAB`: centro abre
      como aba, borda divide o painel (esquerda/direita lado a lado, cima/baixo empilhado), cada nova
      conversa divide de novo; conversa já aberta é movida, nunca duplicada. Só vale para conversas do
      projeto aberto (os painéis na tela são dele). App: `onSoltarConversaNoLayout` (Sidebar → App).
- Testes: `src/test/colocarConversa.test.ts` (5) e `src/test/ordemDaLateral.test.tsx` (9: prévia,
  cópia seguindo o mouse, Esc, soltar no painel, outro projeto). Verificado: typecheck, `npm test`
  (419), build local. Falta conferir na mão o arraste sobre um painel com terminal (xterm) aberto.
- [x] Prévia do layout final no arraste (aba ou conversa da lateral): `features/workspace/PreviaDoLayout.tsx`
      roda o mesmo reducer do soltar (`PLACE_SESSION`/`MOVE_TAB`/`SPLIT_WITH_TAB`) sobre uma cópia do projeto
      e desenha todos os painéis já no lugar e tamanho finais, o novo destacado ("Vai abrir aqui"). Fica por
      cima com `pointer-events: none`: a zona continua medida nos painéis reais, que não se mexem.
- [x] Conversa de **qualquer projeto** da lateral vai para o layout na tela (antes só do projeto aberto).
      Se estava aberta no layout de outro projeto, sai de lá (`soltarDosOutrosProjetos` em
      `colocarConversa.ts`). A aba e a prévia mostram a etiqueta do projeto com cor própria, e o painel
      ganha a faixa "Conversa de outro projeto" (`features/workspace/projetosDasAbas.tsx`, contexto no App).
      `FOCUS_SESSION` vai ao projeto onde a conversa está aberta.
- [x] Bug: duas abas na mesma sessão (digitar numa aparecia na outra; fechar uma apagava a conversa).
      Causa: `ATTACH_TERMINAL` (clique na lateral, notificação) só procurava a sessão no painel ativo e
      criava outra aba se ela estava no outro painel. Agora procura em todos e foca a existente. Além
      disso, fechar aba/painel só encerra a sessão se nenhuma outra aba (em qualquer projeto) usa ela.
      Testes: `src/test/conversaEntreProjetos.test.tsx` (7); `npm test` 426. Falta conferir na mão.
- [x] Bug (2026-10-19): fechar aba/painel de agente apagava a conversa (o `PaneView` chamava
      `closeTerminal`, e o `close_session` do engine tira a sessão da lista). Agora fechar só tira a aba
      da tela: a sessão segue viva e fica na lateral. Apagar é só pelo "×" da lateral, que pede segundo
      clique ("Apagar?"). Terminal puro (shell) continua sendo encerrado ao fechar. Teste
      `src/test/fecharAbaNaoApaga.test.tsx`. As conversas fechadas antes disso continuam no índice
      (`conversations.json`) e nos transcripts: reabrir pelo Histórico.
- [x] Bug (2026-10-19): chat travava em "Entrada do CLI não reconhecida" / "Sem sessão ativa". Agora o
      motivo é detalhado (`motivo_geral` em `interaction.rs`, e evento `conversa_sem_entrada` no log, só
      metadados) e mandar mensagem destrava sozinho: rota `POST /conversas/{id}/destravar` (`mobile.rs`)
      reabre o agente se a CLI saiu, senão manda Ctrl+C (limpa rascunho/menu perdido) e, se nem assim,
      reabre com `--resume`; devolve a versão da tela pronta e o chat envia em seguida. Recusa com o
      agente trabalhando ou com pergunta/aprovação na tela. Teste em `src/test/MobileApp.test.tsx`.
- Próxima missão (sessão nova): orquestrador visual de agentes, prompt em `~/omni/PROMPT-ORQUESTRADOR.md`.
- [x] Conversa (2026-10-19): texto das mensagens (do agente e do usuário) agora seleciona e copia por
      trecho (`user-select: text` em `fork.css`; o app inteiro é `none`). Caminho entre crases vira link:
      `src/mobile/CaminhoClicavel.tsx` + comando `abrir_caminho_da_conversa` (`src-tauri/src/abrir_caminho.rs`):
      pasta abre no Finder, documento/imagem/texto no app padrão, o resto (script, `.app`) só aparece
      selecionado no Finder. Relativo resolve pela pasta da sessão. Testes `caminhoClicavel.test.ts` e
      `abrir_caminho::tests`.
- [x] Atualizar sem cortar agentes (2026-10-19): `ensure_engine` (primeira conferência da janela) não
      troca o engine se algum agente está trabalhando ou esperando aprovação (`agentes_ocupados`); marca
      `ENGINE_PENDENTE` e a janela nova roda com o engine antigo. O botão (`AvisoDeAtualizacao.tsx`) diz
      "Atualizar (agentes seguem)" com agente ocupado e "Concluir atualização" quando a troca do engine
      ficou pendente e todos pararam (aí as conversas reabrem com `--resume`, sem trabalho perdido).
      Com o engine antigo, o chat cai no envio direto se a rota `/destravar` não existir (404).

## Orquestrador visual — proposta (2026-10-19, AGUARDA APROVAÇÃO do Enzo, nada codado)

Conferido nas CLIs instaladas + docs oficiais hoje:
- Claude 2.1.303: `--mcp-config <json|arquivo>` vale só para a sessão (code.claude.com/docs/en/mcp);
  `type: "http"` + `headers` com `${VAR}` expandido do ambiente.
- Codex 0.163.0: `-c <chave.pontuada>=<toml>` sobrescreve qualquer chave do config →
  `-c mcp_servers.medza.url="…" -c mcp_servers.medza.bearer_token_env_var="MEDZA_ORQ_TOKEN"`
  (chaves em learn.chatgpt.com/docs/extend/mcp).
- Antigravity (`agy` 1.4.2): só `agy mcp add` (config global), sem flag por sessão. Entra como
  **subagente**; como orquestrador fica para depois.

Decisão: servidor MCP **Streamable HTTP dentro do engine**, loopback, porta própria derivada da
identidade; JSON-RPC simples (initialize, tools/list, tools/call; resposta JSON, sem SSE). Token por
orquestrador, só em memória, no ambiente da PTY (`MEDZA_ORQ_TOKEN`), nunca na linha de comando, nunca o
`engine.token`; novo a cada abertura/reinício da sessão; só enxerga os subagentes do próprio
orquestrador. As ferramentas chamam em processo as rotas que o chat já usa (`/sessoes`,
`/conversas/{id}/prompt|timeline|aprovar`), como `mobile::local`. Descartadas: stdio (binário extra +
canal novo até o engine), `claude mcp add` global (vaza para toda conversa/conta), orquestrador digitar
direto em outras PTYs (sem estado nem limite).

Ferramentas: `listar_contas`, `criar_agente(plataforma, conta?, modelo?, tarefa, titulo?)` (pasta = a do
orquestrador), `mandar_prompt`, `ler_conversa(agente, ultimas?)`, `esperar(agente, segundos≤120)`,
`listar_agentes`, `parar_agente`. Limite de subagentes vivos por orquestrador (padrão 3). Subagentes não
recebem as ferramentas (sem recursão na v1).

Registro: `orquestracao.json` na pasta de dados do engine (só ids, plataforma, conta, tarefa, datas).

Fatias: (a) registro + MCP + flags no spawn de Claude/Codex; (b) `src/features/orquestracao/FluxoAoVivo`
(React Flow tardio, reaproveita `ArestaDoFluxo`/visual do `NoDoFluxo`), aba `PaneKind "orquestracao"`,
botão na faixa da conversa e na lateral; (c) composer no nó, aprovar/parar pelo nó (rotas existentes).
- [x] Permissão do Claude no chat (2026-10-19): `approval` (`interaction.rs`) reescrito. Exigia "3. No"
      exato (o Claude atual escreve "3. No, and tell Claude…"), só "Do you want to proceed?" e um único `❯`
      na tela inteira (o prompt anterior também tem `❯`). Agora: qualquer "Do you want to …?", opções
      numeradas logo abaixo, "1. Yes" e a opção "No…" (tecla = número dela), `❯` contado só no diálogo,
      texto do cartão = só o diálogo. Teste `aprovacao_do_claude_atual`.
- [x] Link abre em aba de navegador (2026-10-19): `src-tauri/src/navegador.rs` (plugin `navegacao` barra a
      janela principal de sair do app e emite `abrir-link`; webview filho por aba, `tauri` com `unstable`),
      `src/features/navegador/` (`NavegadorPane`: barra voltar/avançar/recarregar/endereço/abrir fora;
      `useLinksEmAba`: intercepta `<a href>` e fecha webview de aba fechada), ação `ABRIR_LINK`
      (`workspace/abrirLink.ts`: à direita do painel do clique, reaproveita painel com navegador). Webview
      some durante arraste e quando a aba sai da tela. Página externa sem IPC (capabilities sem `remote`).
- [x] Branch por agente (2026-10-19): `GitBranchStatus` (trocar/criar) na faixa de cada aba de agente,
      com a pasta da sessão; lançador com campo "Branch": outra que não a atual roda numa worktree
      (`src-tauri/src/worktree.rs`, `<repo>/.omni-agents/worktrees/<branch>`, `.omni-agents/` no
      `info/exclude`); `AgentLaunch.cwd`, mantido na troca de conta. Testes `worktree::tests`,
      `AgentLauncher.branch.test.tsx`, `abrirLink.test.ts`.
- [x] "+" no painel abre agente no projeto do painel (2026-10-19): aba ganhou `projectId` opcional
      (`WorkspaceTab`, `CREATE_TAB`). `PaneView` usa o projeto da aba (ou o da conversa de outro projeto que o
      painel mostra) no "+", no painel vazio e no `TerminalPane` (pasta e projeto do agente novo). Antes, com
      conversas de outro projeto no layout, todo agente novo nascia no projeto dono do layout.
      Testes `maisNoPainelDeOutroProjeto.test.tsx` e `conversaEntreProjetos.test.tsx`.
- [x] Chat sem histórico, "nenhum transcript foi associado" (2026-10-19): a conta `enzo-medza` tinha
      pasta em `<dados>/OMNI-AGENTS/profiles/claude/` mas não estava no `profiles.json` da variante Local;
      `resolve_transcript` não achava a conta e desistia (e o chip mostrava "Padrão"). Agora
      `omni_core::perfis_descobertos` junta as pastas de conta fora do registro (raiz definida no início
      do engine e do app) e o `list_profiles` do app as grava de volta. Teste `perfis_orfaos`.
- [x] Seletor de conta/modelo/esforço cortado no chat (2026-10-19): a lista do `Select` (`components/ui`)
      era `absolute` dentro do painel e sumia no `overflow` dele, com a largura do botão. Agora vai para o
      `body` (portal, `position: fixed` medida do botão), abre para cima quando falta espaço, largura pelo
      texto (até 360 px) e realinha ao rolar/redimensionar. Teste `selectSobrePainel.test.tsx`.
