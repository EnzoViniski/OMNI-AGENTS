# Spike: Google Antigravity / Gemini CLI (Fase 5)

> Pesquisa feita em 2026-10-19, só leitura de documentação. **Nada construído.** Conferir de novo
> antes de integrar: as páginas dizem que limites e termos "podem mudar".

## O que existe

- **Antigravity CLI (`agy`)**: binário nativo, instalado em `~/.local/bin` por script oficial
  (`antigravity.google/cli/install.sh`). Fonte secundária, atualizada em 2026-10-08:
  [cheat sheet](https://computingforgeeks.com/antigravity-cli-cheat-sheet/).
  - Headless: `-p/--print/--prompt`, `--output-format text|json|stream-json`, `--input-format`,
    `--json-schema`, `--print-timeout` (padrão 5 min).
  - Sessão: `-c/--continue`, `--conversation <id>`; modelo: `--model`, `--effort low|medium|high`,
    `agy models`; `/model` dentro do TUI.
  - MCP: `agy mcp add|remove|list|enable|disable`; global em `~/.gemini/config/mcp_config.json`,
    projeto em `.agents/mcp_config.json`.
  - Uso: `/usage` (`/quota`) e `/credits` no TUI.
  - Credenciais no **keyring do sistema** (Keychain no macOS). **Nenhuma variável documentada para
    trocar a pasta de config** → hoje não há como ter dois perfis Google isolados como fazemos com
    `CLAUDE_CONFIG_DIR`/`CODEX_HOME`. Cada sócio usaria a própria conta no próprio Mac (ok pela regra 1).
  - ACP nativo: só um pedido em aberto
    ([issue #31](https://github.com/google-antigravity/antigravity-cli/issues/31)); não contar com isso.
- **Gemini CLI**: o post oficial do Google Cloud (2026-02-04,
  [Choosing Antigravity or Gemini CLI](https://cloud.google.com/blog/topics/developers-practitioners/choosing-antigravity-or-gemini-cli))
  documenta modo headless e MCP. Fonte secundária diz que deixou de atender contas individuais em
  2026-06-18 e foi substituída pelo `agy` — **não confirmado em fonte oficial**.

## Plano Google AI Pro ([página oficial de planos](https://antigravity.google/docs/plans/))

- Cota "alta", renovada a cada 5 h até bater o limite semanal (mesma forma de janela do Claude:
  encaixa no modelo de conta da Fase 3 como `assento` com janelas 5 h + semana).
- Modelos: Gemini em todos os planos. **Modelos de terceiros (Claude, GPT) só no Ultra** — o Pro não
  dá acesso a Claude/GPT pelo Antigravity.
- Créditos de IA acima da cota (cobrança em preço Gemini Enterprise), com a opção
  "AI Credit Overages: Never/Always". Para nós: equivale a uso extra pago; manter em "Never".
- CLI incluída em todos os planos. Sem BYOK.

## Termos ([antigravity.google/terms](https://antigravity.google/terms))

- Conta individual: termos derivados dos Termos de Serviço do Google (consumidor). Equipes: termos
  do Google Cloud via Gemini Enterprise.
- **Zona cinza para o OMNI:** "usar software, ferramentas ou serviços de terceiros para acessar o
  Serviço" é tratado como violação, com risco de suspensão da conta; o exemplo dado é um harness
  usando o OAuth do Antigravity. O OMNI não tocaria no OAuth — só executaria o `agy` oficial num
  terminal, como um emulador de terminal faz. Mas orquestrar `agy -p` em automação por um app de
  terceiros pode ser lido como "ferramenta de terceiros acessando o serviço". **Decisão do Enzo
  antes de construir.** Alternativa sem ambiguidade: só abrir o `agy` interativo numa aba, como
  terminal comum, sem headless nem delegação automática.
- Dados: as interações são usadas para avaliar e melhorar produtos do Google e podem ser revistas
  por pessoas; dá para mudar a preferência nas configurações. → Plano pessoal fica **fora dos
  projetos confidenciais** (política de dados por projeto, regra 6) e o opt-out deve ser ligado.

## Recomendação

1. Fase 5 começa só com o `agy` **interativo** como provider novo (launcher + leitor de uso via
   `/usage`), sem headless, até o Enzo decidir sobre a zona cinza.
2. Se o Enzo aceitar o risco do headless, integrar `agy -p --output-format stream-json` no executor
   com concorrência 1 e sem uso extra; nunca em projeto confidencial.
3. Copiloto com Claude e Codex primeiro; Google entra depois, se entrar.
