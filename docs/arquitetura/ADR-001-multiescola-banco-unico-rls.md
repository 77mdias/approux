# ADR-001 — Multiescola: banco único com `escola_id` e Row-Level Security

| Campo | Valor |
|---|---|
| Status | **Aceita** |
| Data | 07/11/2026 |
| Relacionados | [ADR-003](./ADR-003-identidade-global-e-vinculos.md) · `RN-01`, `RNF-SEG-03`, `T-03` do [SRS](../SRS.md) |

## Contexto

Um único sistema atende várias escolas, e os dados de uma escola nunca podem aparecer para outra (`RN-01`). São dados de menores (LGPD). O volume por escola é pequeno (centenas de alunos), o time é de uma pessoa e o orçamento inicial de infraestrutura é de até R$ 200/mês.

## Opções consideradas

| Opção | Prós | Contras |
|---|---|---|
| **A. Banco único, `escola_id` em cada tabela** | Uma migração para todos; custo e operação mínimos; relatórios da plataforma simples | Um filtro esquecido vaza dados |
| B. Um schema por escola | Isolamento mais forte | Migrações N vezes; pool de conexões e ORM mais complexos |
| C. Um banco por escola | Isolamento máximo | Custo e operação multiplicados; inviável no orçamento |

## Decisão

**Opção A**, com isolamento garantido em camadas:

1. **Contexto da escola vem da sessão.** O token/sessão carrega o contexto ativo (`usuario_id`, `escola_id`, `papel`), validado contra os vínculos do usuário (ver ADR-003). Nenhuma rota confia em `escola_id` vindo do corpo/URL sem validar contra esse contexto. Leitor de tag → escola da chave do leitor. Webhook → escola da subconta do gateway. Jobs → escola explícita no payload do job.
2. **Row-Level Security (RLS) no PostgreSQL** em toda tabela com `escola_id`, como rede de segurança:
   ```sql
   ALTER TABLE aluno ENABLE ROW LEVEL SECURITY;
   ALTER TABLE aluno FORCE ROW LEVEL SECURITY;
   CREATE POLICY isolamento_escola ON aluno
     USING (escola_id = current_setting('app.escola_id', true)::uuid)
     WITH CHECK (escola_id = current_setting('app.escola_id', true)::uuid);
   ```
   A API executa cada requisição dentro de uma transação que começa com `SELECT set_config('app.escola_id', $1, true)` (equivalente a `SET LOCAL`, seguro com pool de conexões).
3. **Papéis de banco separados:**
   - `app_migrator` — dono das tabelas, usado só nas migrações.
   - `app_api` — usado pela API; **não é dono das tabelas nem superusuário** (donos e superusuários ignoram RLS).
   - `app_platform` — com `BYPASSRLS`, usado apenas por rotinas da plataforma (admin, relatórios, jobs que varrem todas as escolas), em código isolado e auditado.
4. **Chaves estrangeiras compostas** incluindo `escola_id` (ex.: `aluno (escola_id, sala_id) → sala (escola_id, id)`), para que o banco recuse ligar registros de escolas diferentes.
5. **UUIDs** (v7, ordenáveis) como chave primária — sem IDs sequenciais enumeráveis.
6. **Testes automatizados de isolamento** (`T-01`..`T-03`) rodando na CI contra um PostgreSQL real com RLS ativo.

A autorização **dentro** da escola (professor ↔ salas, pai ↔ filhos, termo aceito) fica na camada de políticas da aplicação, não em RLS — as regras são mais ricas e mais fáceis de testar em código.

## Consequências

- ✅ Infra simples e barata; uma migração atende todas as escolas.
- ✅ Defesa em profundidade: bug na aplicação não basta para vazar dados entre escolas.
- ⚠️ Toda tabela nova precisa de `escola_id` + política RLS — incluir checagem na CI (consulta em `pg_tables`/`pg_policies` que falha se faltar política).
- ⚠️ Toda consulta precisa rodar dentro da transação com contexto; centralizar num helper (`withEscola(escolaId, tx => ...)`) e proibir acesso direto ao pool.
- 🔁 Se uma rede grande exigir isolamento físico no futuro, é possível migrar aquela escola para um banco próprio com o mesmo schema.
