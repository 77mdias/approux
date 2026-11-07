# ADR-002 — Stack: React + Vite (PWA), NestJS, PostgreSQL + Drizzle

| Campo | Valor |
|---|---|
| Status | **Aceita** |
| Data | 07/11/2026 |
| Relacionados | [ADR-001](./ADR-001-multiescola-banco-unico-rls.md) · [ADR-004](./ADR-004-hospedagem.md) |

## Contexto

Desenvolvedor único, produto mobile first para pais e professores, com API pública para leitores de tag e webhooks do gateway, tarefas agendadas (lembretes, aviso de ausência), notificações e atualização "na hora" para o pai. Orçamento de infraestrutura baixo.

## Decisão

**Monólito modular em TypeScript**, num monorepo.

| Camada | Escolha | Observação |
|---|---|---|
| Frontend | **React + Vite**, SPA instalável como **PWA** (`vite-plugin-pwa`) | Tudo atrás de login, sem necessidade de SSR/SEO |
| Backend | **NestJS** (API REST) | Um módulo Nest por domínio; guards para autenticação, contexto de escola e políticas |
| Banco | **PostgreSQL 16+** | RLS para isolamento por escola (ADR-001) |
| ORM / migrações | **Drizzle ORM + drizzle-kit** | SQL explícito, transações simples para `set_config` por requisição |
| Fila / agendamento | **pg-boss** (fila no próprio PostgreSQL) | Lembretes, notificações, processamento de webhooks, aviso de ausência — sem Redis |
| Tempo real | **Server-Sent Events (SSE)** | Fluxo servidor → cliente ("João chegou"); fallback para polling |
| Validação / contratos | **Zod** em pacote compartilhado | Mesmos schemas no front e no back |
| Integrações | Portas/adapters: `PaymentGateway` (Asaas), `Notifier` (push, e-mail, WhatsApp) | Troca de fornecedor sem mexer no domínio |

### Estrutura do monorepo (pnpm workspaces)

```
apps/
  web/        # React + Vite (PWA)
  api/        # NestJS (HTTP + SSE)
  worker/     # NestJS standalone consumindo pg-boss (mesmo código de domínio)
packages/
  shared/     # schemas Zod, tipos, enums (perfis, status de cobrança...)
  db/         # schema Drizzle, migrações, helpers withEscola()
```

### Módulos de domínio (backend)

`identidade` (usuários, vínculos, sessão) · `escolas` · `cadastros` (salas, alunos, professores, responsáveis) · `pedagogico` (frequência, notas, comportamento) · `agenda` · `comunicacao` (avisos, justificativas, notificações) · `financeiro` (cobranças, pagamentos, webhooks) · `acesso` (tags, leitores, registros) · `lgpd` (modelos, termos, aceites) · `auditoria`.

## Alternativas descartadas

- **Next.js:** SSR não agrega num app 100% autenticado; a API precisa existir de qualquer forma para leitores e webhooks.
- **Prisma:** funciona, mas RLS com `SET LOCAL` por requisição exige extensões/contornos; Drizzle é mais direto.
- **BullMQ + Redis:** mais um serviço para operar e pagar; pg-boss atende o volume previsto. Reavaliar se a fila virar gargalo.
- **WebSocket:** a comunicação em tempo real é unidirecional; SSE é mais simples e passa por proxies HTTP comuns.
- **Supabase/Firebase:** lógica financeira e de controle de acesso ficaria espalhada em funções e políticas; Firebase não é relacional.

## Consequências

- ✅ Uma linguagem só, tipos compartilhados entre front e back.
- ✅ Um processo de API + um worker + um PostgreSQL cabem numa VPS pequena.
- ⚠️ **Push em PWA no iPhone** só funciona com o app adicionado à tela inicial (iOS 16.4+). Avisos críticos (ausência, atraso de pagamento, código de confirmação do termo) precisam de canal reserva (e-mail e/ou WhatsApp) — decisão de canal ainda **a definir**.
- ⚠️ SSE por trás de proxy/CDN exige mensagens de keep-alive periódicas.
