# Inventário de contas

> Sem e-mails, tokens ou identificadores de conta. Valores de plano vêm da documentação oficial
> **na data indicada** — conferir de novo antes de codificar qualquer limite.
> Campos `?` aguardam o Enzo (ficha do brief, `/status` e `/usage` do perfil).

## Por perfil (MacBook Air do Enzo)

| Apelido (proposto) | Provider | Perfil OMNI | Config dir | Tipo | Plano | Janelas de limite | Uso extra | Validade/saldo | Quem usa |
|---|---|---|---|---|---|---|---|---|---|
| `claude-pessoal` | claude | Padrão | `~/.claude` | `assento`? | ? (Pro ou Max) | 5 h + semana | ? | — | Enzo |
| `claude-team` | claude | Enzo Medza | `.../profiles/claude/enzo-medza` | `assento` (provável) | Team, assento Premium (provável, ver abaixo) | 5 h + semana, por assento | ? (admin da org) | ano grátis a partir de ? | Enzo |
| `codex-pessoal` | codex | ? | `~/.codex`? | `assento` | ChatGPT ? | ? | créditos? | — | Enzo |
| `google-pro` | (Antigravity/Gemini, Fase 5) | — | — | `assento` | Google AI Pro, pessoal ou Workspace ? | ? | ? | — | Enzo |

O sócio terá a própria linha quando instalar no Mac dele (assentos são pessoais; nada de login
compartilhado).

## Claude for Startups (consultado em 2026-10-19)

Fonte oficial: [anúncio de 2026-10-06](https://claude.com/blog/were-expanding-the-claude-startups-program-to-help-founders-build).

- Benefício atual: **um ano grátis de Claude Team com até 5 assentos Premium** (para empresas novas
  no Team) **mais** um crédito único de **US$ 1.000 de API** (Claude Console, sujeito aos
  *Supplemental Credit Terms*; não vale em Bedrock/Vertex). Atualização de 2026-10-13: turma cheia,
  inscrições fechadas.
- Ou seja, provavelmente o Enzo tem **as duas coisas**: assento Team (perfil "Enzo Medza", tipo
  `assento`) e, separado, crédito de API (tipo `api`, só utilizável por chave de API do Console —
  hoje o OMNI não usa chave de API). Confirmar pelo `/status` do perfil e pelo Console.

## Claude Team (consultado em 2026-10-19)

Fonte oficial: [Team plan, central de ajuda](https://support.claude.com/en/articles/9266767).

- Assento Standard: 1,25× o uso por sessão do Pro. Premium: 6,25× o Pro. Os dois incluem Claude Code.
- Limites são **por membro**, não compartilhados entre assentos: janela de sessão (5 h) e limite
  semanal com reset em horário fixo por conta (Configurações → Uso).
- A organização pode **pré-pagar créditos de uso** para seguir trabalhando depois do limite (Claude,
  Cowork e Claude Code), com teto de gasto por organização e por usuário. Isso é o "uso extra" da
  regra 3: dinheiro da empresa. O OMNI nunca deve entrar nisso sem `permitir_uso_extra`.

## Termos de uso do Claude Code (regra 4, consultado em 2026-10-19)

Fonte: [Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance).

- Team usa os **Termos Comerciais**; Free/Pro/Max, os Termos de Consumidor.
- Login OAuth (claude.ai) é para os assinantes e para "uso comum do Claude Code e de outros apps
  nativos da Anthropic". Quem constrói produto (inclusive com o **Agent SDK**) deve usar **chave de
  API**; terceiros não podem oferecer login do claude.ai nem intermediar/guardar tokens.
- Explicitamente permitido: o usuário entrar no **binário oficial, sem modificação**, com a própria
  assinatura. É exatamente o que o OMNI faz (dirige `claude` / `claude -p`, cada conta logada pelo
  próprio fluxo da Anthropic).
- Consequências para o desenho:
  - (a) `claude -p` headless pela CLI oficial com o próprio assento: dentro da regra, desde que siga
    sendo o binário oficial com o login do próprio dono. Os limites anunciados de Pro/Max assumem
    "uso individual comum"; automação pesada em paralelo é zona cinza — manter concorrência baixa.
  - (b) Sessões simultâneas no mesmo assento: a página não proíbe; os limites são por membro.
    Tratar como restrito por cota (`max_concorrencia`, padrão 2).
  - (c) **Agent SDK ou harness próprio autenticando com o assento: não.** O Copiloto da Fase 4 deve
    rodar sobre a CLI oficial (`claude -p --output-format stream-json`), nunca sobre o SDK com OAuth.
    Se um dia usar o SDK, só com chave de API (o crédito de US$ 1.000 serviria para isso).

## Codex / ChatGPT (consultado em 2026-10-19, incompleto)

- Codex está incluído nos planos do ChatGPT; o uso sai de um saldo compartilhado com outros recursos
  do ChatGPT e varia por modelo e tarefa. `/status` na CLI mostra o restante.
  ([central de ajuda](https://help.openai.com/en/articles/11369540-codex-usage-limits))
- Falta confirmar nos termos oficiais se `codex exec` com login do ChatGPT pode ser usado em
  automação (Fase 3/4). Até lá, mesmo cuidado do Claude: binário oficial, concorrência baixa.

## Google (Fase 5)

Spike pendente em `roadmaps/copiloto/antigravity.md`.
