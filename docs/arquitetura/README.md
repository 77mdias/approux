# Arquitetura — Registro de decisões (ADRs)

Cada decisão de arquitetura é registrada em um arquivo próprio: contexto, opções, decisão e consequências.

| ADR | Decisão | Status |
|---|---|---|
| [ADR-001](./ADR-001-multiescola-banco-unico-rls.md) | Multiescola: banco único com `escola_id` e Row-Level Security | Aceita |
| [ADR-002](./ADR-002-stack.md) | Stack: React + Vite (PWA), NestJS, PostgreSQL + Drizzle, pg-boss, SSE | Aceita |
| [ADR-003](./ADR-003-identidade-global-e-vinculos.md) | Identidade global de usuário e vínculos por escola | Aceita |
| [ADR-004](./ADR-004-hospedagem.md) | Hospedagem: Cloudflare Pages + VPS em São Paulo | Proposta |

## Próximas decisões

| # | Tema |
|---|---|
| 3 | Atualização em tempo real e canal de notificações (push, e-mail, WhatsApp) |
| 4 | Pagamento conjunto no Asaas e processamento idempotente de webhooks |
| 5 | Leitor de tag: hardware, protocolo, autenticação e buffer offline |
| 6 | Autenticação e sessão (cookies, tokens, troca de contexto), segredos e LGPD operacional |

Documentos de produto: [PRD](../PRD.md) · [SRS](../SRS.md) · [Mapa de fluxo](../Mapa%20de%20fluxo.md)
