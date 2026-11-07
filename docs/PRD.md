# PRD — App de Gestão Escolar

| Campo | Valor |
|---|---|
| Documento | Product Requirements Document (PRD) |
| Produto | App de Gestão Escolar (nome comercial **a definir**) |
| Dono do produto | William Henrique Moreira Dias |
| Fonte | *App de Gestão Escolar — Especificação v3* (05/11/2026) |
| Versão deste documento | 1.0 |
| Status | Rascunho para validação |
| Documentos relacionados | [`SRS.md`](./SRS.md) · [`Mapa de fluxo.md`](./Mapa%20de%20fluxo.md) |

> Documento vivo. Tudo o que ainda não foi decidido está marcado como **a definir** e consolidado na seção 12.

---

## 1. Resumo

Aplicativo **real** de gestão escolar para escolas particulares, com três tipos de acesso — **escola (gestão)**, **professor** e **pai/responsável** — cada um com login e senha.

- A **escola** administra salas, alunos, professores, agenda, avisos, cobranças e o controle de entrada/saída.
- O **professor** alimenta as informações dos alunos das suas salas (frequência, notas, comportamento, eventos).
- O **pai** acompanha cada filho em uma única página e **paga dentro do app**.

**Princípio central:** o dinheiro das mensalidades vai **direto para a conta da escola**. O sistema apenas gera as cobranças e acompanha os pagamentos — o dono do produto nunca tem acesso a esse dinheiro.

## 2. Problema e oportunidade

| Para quem | Dor atual | O que o produto resolve |
|---|---|---|
| Escola | Informação espalhada (cadernos, planilhas, grupos de mensagem); cobrança e controle de inadimplência manuais; nenhuma visão de quem está no prédio | Centraliza cadastro, comunicação, agenda e financeiro; mostra quem pagou e quem está em atraso; painel do dia de entrada/saída |
| Professor | Chamada e notas em papel; avisos e justificativas chegam por canais informais | Lança frequência, notas, comportamento e eventos das suas salas em um só lugar; recebe justificativas e confirmações no app |
| Pai/responsável | Não sabe na hora se o filho chegou; perde avisos e eventos; pagamentos por boleto avulso | Página única por filho com frequência, notas, comportamento, agenda, avisos, horário de chegada/saída e pagamento integrado |

## 3. Modelo de negócio

- **Cliente pagante:** a escola particular. O dono do produto vende o sistema e cobra uma **mensalidade da escola** (preço **a definir**).
- **Cobrança do sistema à escola:** feita **à parte**, no sistema de gestão do dono do produto, depois que o app real estiver pronto. No futuro pode ser feita via **split** de pagamento ou cobrança mensal à escola (**a definir**).
- **Dinheiro das famílias:** cada escola possui uma **subconta própria no gateway** (decisão atual: Asaas). As cobranças são emitidas na subconta da escola e o dinheiro cai direto nela. Não há repasse manual.
- **Pendências de negócio:** confirmar enquadramento fiscal com um contador; confirmar com o Asaas taxas, limites e regras do período de avaliação de subcontas criadas via API.

## 4. Objetivos

| # | Objetivo | Como saberemos (indicador sugerido — **a validar**) |
|---|---|---|
| O1 | Pais acompanham o filho em tempo real | % de pais que acessam o app ao menos 1×/semana |
| O2 | Escola reduz inadimplência e trabalho manual de cobrança | % de cobranças pagas até o vencimento; tempo gasto com cobrança |
| O3 | Professor registra a rotina da sala no app | % de aulas com frequência lançada no dia |
| O4 | Escola sabe quem está no prédio | % de alunos com entrada registrada por tag |
| O5 | Conformidade com a LGPD desde o primeiro acesso | 100% dos pais com termo aceito antes de ver dados |
| O6 | Validar o produto com uma escola piloto | Escola piloto usando todos os módulos ao fim da fase 6 |

> A especificação original não define métricas numéricas; os indicadores acima são uma proposta para discussão.

## 5. Personas e perfis de acesso

| Perfil | Entra com | Vê e faz |
|---|---|---|
| **Escola (gestão)** | Login e senha | Tudo: salas, alunos, professores, agenda, avisos, financeiro, tags/leitores, termo LGPD, configurações |
| **Professor** | Login e senha próprios | Somente as salas dele: lança notas, frequência, comportamento e eventos; recebe justificativas e confirmações; envia avisos às suas salas |
| **Pai/responsável** | **Um login por pessoa**, válido em todas as escolas | Os filhos cadastrados, inclusive em escolas diferentes; escolhe qual acompanhar sem logar de novo; paga cobranças; confirma reunião; justifica falta |

Exemplo: o pai tem o João e a Maria. Entra uma vez, escolhe o filho e abre a página individual dele. Pode trocar de filho a qualquer momento. Se o João estuda na Escola A e a Maria na Escola B, o pai usa o mesmo login e vê os dois filhos agrupados por escola.

**Perfis em aberto**
- **Aluno com acesso próprio:** **a definir** (por ora, só os pais).
- **Administrador da plataforma (dono do produto):** necessário para cadastrar escolas e criar a subconta no gateway. Não descrito na especificação — **a definir** se será tela no app ou processo interno.

## 6. Escopo do produto

### 6.1 Área do pai — página do aluno
Cada filho tem uma página (dashboard) com tudo em um lugar:
- **Frequência:** presenças, faltas e percentual.
- **Notas:** por disciplina e período.
- **Comportamento:** registros lançados pelos professores.
- **Entrada e saída:** horário de chegada e saída de hoje, se está dentro da escola agora, tempo de permanência e histórico (dia/semana/mês, incluindo atrasos e saídas antecipadas).
- **Agenda:** visão semanal, mensal e anual (provas, reuniões, feriados, eventos).
- **Notificações:** avisos da escola e dos professores.
- **Financeiro:** mensalidade e cobranças extras do filho, com opção de pagar.
- **Ações:** confirmar presença em reunião e justificar falta com mensagem.

### 6.2 Professor
- Vê apenas as salas dele.
- Lança **frequência**, **notas** e **comportamento** — o pai vê na hora.
- Pode usar a **chegada pela tag para pré-preencher a frequência** do dia.
- Lança **eventos** na agenda das suas salas (prova, trabalho, reunião), se a escola permitir; os pais daquela sala são notificados.
- Recebe **justificativas de falta** e **confirmações de reunião**.
- Envia **avisos** às suas salas.

### 6.3 Gestão da escola
- **Salas:** cria salas e vê a quantidade de alunos em cada uma.
- **Alunos:** cadastro completo, vínculo com sala e com pais (um pai pode ter vários filhos), mensalidade acordada e tag.
- **Professores:** cadastra, dá login e define quais salas (e disciplinas) cada um atende.
- **Notificações:** avisos gerais, por sala ou por aluno.
- **Agenda:** eventos gerais (feriados, reuniões, provas); vê todos os eventos; liga/desliga, sala por sala, a permissão do professor lançar eventos.
- **Visão geral:** acompanha frequência, notas e comportamento de qualquer aluno.
- **Configurações:** horário de entrada da sala, tolerância de atraso, horário limite para aviso de ausência.

### 6.4 Financeiro
- **Mensalidade por aluno:** valor acordado no cadastro, já com desconto (ex.: 30%, 70% ou outro), editável aluno por aluno.
- **Cobranças extras** (apostila, uniforme, passeio, evento): a escola escolhe quem entra (um aluno, uma sala, várias salas ou todos) e **define valor e forma de pagamento de cada aluno**. Não há divisão igual automática. A cobrança aparece só na área do pai daquele aluno.
- **Pagamento pelo pai, dentro do app:** Pix (QR Code), cartão de crédito, cartão de débito (**a confirmar** — a API do Asaas não aceita) e boleto. Com mais de um filho, o pai vê o que deve de cada um e pode pagar um só ou juntar os dois na mesma cobrança (filhos da mesma escola; por Pix ou cartão).
- **Baixa automática:** quando o gateway confirma, a cobrança fica paga.
- **Dashboard da escola:** quanto entrou, quem pagou, quem está em atraso, quem falta pagar.
- **Lembretes automáticos** antes e depois do vencimento, para mensalidade e extras (dias **a definir**).
- **Atraso (multa, juros, bloqueio):** **a definir**.

### 6.5 Agenda e comunicação
- Agenda semanal, mensal e anual. Eventos gerais vêm da escola; eventos de sala vêm do professor (só nas salas dele, se a escola permitir).
- **Notificação automática** aos pais dos alunos da sala ao criar um evento.
- **Confirmar presença em reunião:** o pai responde no app; escola e professor veem quem confirmou.
- **Justificar falta:** o pai envia mensagem; professor e escola recebem e a falta fica com a justificativa registrada.

### 6.6 Entrada e saída por tag (controle de acesso)
- Cada aluno recebe uma tag RFID/NFC (cartão, chaveiro ou pulseira) vinculada ao cadastro, lendo a tag no leitor pela tela do aluno.
- Leitor na entrada, conectado à internet, envia: número da tag, data/hora e identificação do leitor.
- Primeira leitura do dia = **entrada**; a próxima = **saída**; e assim por diante. Leituras repetidas em poucos segundos são ignoradas.
- O pai vê na hora e recebe aviso de chegada, saída e de **não chegada** até o horário limite.
- **Escola:** painel do dia (chegou, está dentro, saiu, não chegou), lista por sala (chamada e emergência), gestão de tags (vincular, trocar, bloquear por perda, com histórico), correção manual com motivo e autor, regras de horário.
- **Leitor offline:** guarda os registros e reenvia quando a internet voltar. Cada leitor tem chave própria revogável.
- **Hardware:** **a definir** (modelo pronto de controle de acesso ou montagem com ESP32 + módulo leitor; mais de um leitor para entradas com fluxo grande).
- **Limitação conhecida:** a tag prova que a tag passou, não que foi o aluno. Se for um problema, considerar foto na leitura ou conferência por funcionário.

### 6.7 Termo de aceite LGPD
- No **primeiro acesso**, o pai precisa ler e aceitar o termo antes de ver qualquer dado dos filhos. Sem aceite, não passa dessa tela.
- Aceite por clique; a escola pode exigir também confirmação por código no celular ou e-mail.
- A escola escreve **um modelo de termo** (revisado por advogado) com campos preenchidos automaticamente (`{{aluno}}`, `{{nascimento}}`, `{{sala}}`, `{{responsavel}}`, `{{cpf_responsavel}}`, `{{parentesco}}`, `{{escola}}`, `{{cnpj_escola}}`, `{{encarregado_dados}}`, `{{data}}`).
- Ao cadastrar o aluno e vincular o pai, o sistema gera o termo daquele aluno, pendente de aceite.
- Nova versão do modelo ⇒ todos os pais aceitam de novo no próximo login.
- **Painel da escola:** quem aceitou e quem não, lembrete automático, comprovante de aceite por pai, publicar nova versão.
- Dois responsáveis pelo mesmo aluno: cada um aceita com o próprio login.

## 7. Fora do escopo (desta versão)

- Receber ou intermediar o dinheiro das mensalidades (vai direto para a escola).
- Cobrança do sistema à escola dentro do app (fica no sistema de gestão do dono do produto).
- Divisão igual automática de cobranças extras.
- Acesso próprio do aluno (até decisão em contrário).
- Reconhecimento facial ou foto na leitura da tag (apenas citado como alternativa futura).
- Aconselhamento jurídico: o texto final do termo é responsabilidade da escola com seu advogado.

## 8. Requisitos não funcionais de alto nível

- **Mobile first:** precisa funcionar bem no celular dos pais e professores (sugestão: PWA instalável).
- **Segurança por perfil imposta no servidor**, em toda consulta — não só escondendo botões.
- **Multiescola desde o início:** um sistema atende várias escolas, com dados isolados por escola. Um mesmo pai pode ter filhos em escolas diferentes com um único login, e nenhuma escola enxerga dados da outra ([ADR-003](./arquitetura/ADR-003-identidade-global-e-vinculos.md)).
- **LGPD:** dados de menores; consentimento, acesso restrito, senhas com hash, registro de acessos, prazo de guarda e exclusão sob pedido.
- **Auditoria:** registro de quem alterou notas, frequência, cobranças e acessos.
- **Confiabilidade financeira:** sem pagamento duplicado; webhook repetido não pode dar baixa duas vezes.
- **Tempo real percebido:** o pai vê lançamentos e entrada/saída "na hora".

Detalhamento em [`SRS.md`](./SRS.md).

## 9. Roadmap e fases

| Fase | Entrega | Inclui |
|---|---|---|
| **0 — Protótipo** | Protótipo navegável com os três perfis | Validar as telas com uma escola antes da versão real (tag apenas simulada) |
| **1 — Base** | Fundação | Login, perfis, escolas, salas, alunos, professores, pais com vários filhos |
| **2 — Acompanhamento** | Rotina pedagógica | Frequência, notas, comportamento e página do aluno para o pai |
| **3 — Agenda e comunicação** | Comunicação | Eventos, avisos, notificações, confirmação de reunião, justificativa de falta |
| **4 — Financeiro** | Cobrança | Mensalidade por aluno, cobranças extras, pagamento pelo gateway, dashboard e lembretes |
| **5 — Entrada e saída por tag** | Controle de acesso | Leitor, API, painel do dia e visão do pai |
| **6 — Acabamento** | Go-live | LGPD, auditoria, testes com escola piloto e cobrança do sistema |

> Observação: a especificação coloca LGPD na fase 6, mas o termo é exigido **antes** de o pai ver qualquer dado. Recomenda-se entregar o fluxo de aceite junto com a fase 2 (quando o pai passa a ver dados reais).

## 10. Premissas e dependências

- Cada escola terá uma subconta no gateway de pagamento criada via API a partir da conta principal do dono do produto.
- Os leitores de tag terão internet (Wi-Fi ou cabo) na maior parte do tempo.
- A escola é a **controladora** dos dados; o sistema é o **operador**.
- A escola fornecerá o modelo de termo revisado por advogado e os dados do encarregado (DPO).
- Os dados obrigatórios do responsável (ex.: CPF) serão preenchidos no cadastro — sem eles o termo não é gerado.

## 11. Riscos

| Risco | Impacto | Mitigação |
|---|---|---|
| Período de avaliação do Asaas: até R$ 2.000 em cobranças por subconta, até 10 subcontas, até 60 dias; conta principal precisa ser PJ | Escola não consegue cobrar a mensalidade de todos os alunos no início | Confirmar liberação com o gerente de contas antes da escola piloto; alternativa: escola conecta a própria conta Asaas ([ADR-005](./arquitetura/ADR-005-pagamentos-asaas-e-webhooks.md)) |
| Enquadramento fiscal do dono do produto | Problemas tributários | Avaliar com contador |
| Tag usada por outro aluno | Registro de presença falso | Foto na leitura ou conferência por funcionário (se necessário) |
| Internet do leitor cai | Pais sem aviso de chegada | Buffer local no leitor e reenvio; avisos de ausência considerar atraso de sincronização |
| Vazamento de dados de menores | Jurídico e reputacional | Permissões no servidor, isolamento por escola, auditoria, testes de autorização |
| Pai recusa ou revoga o termo | Escola sem canal no app | Caminho alternativo de comunicação definido no termo |
| Hardware de leitor indefinido | Atraso da fase 5 | Decidir modelo cedo e testar com protótipo |

## 12. Pontos em aberto (consolidado)

| # | Tema | Pergunta |
|---|---|---|
| Q1 | Negócio | Nome do sistema e preço da mensalidade cobrada das escolas |
| Q2 | Negócio | Cobrança do sistema à escola: split ou cobrança mensal |
| Q3 | Financeiro | Gateway final (Asaas é a decisão atual) — taxas, limites e regras |
| Q4 | Financeiro | Multa, juros e bloqueio em caso de atraso |
| Q5 | Financeiro | Dias de antecedência e de cobrança após vencimento dos lembretes |
| Q6 | Financeiro | Dia de vencimento da mensalidade e geração mensal (automática? até quando?) |
| Q7 | Pedagógico | Regras de nota: média, bimestres, recuperação |
| Q8 | Pedagógico | Frequência mínima |
| Q9 | Comunicação | Canal das notificações e lembretes: push, e-mail, WhatsApp |
| Q10 | Perfis | O aluno terá acesso próprio? |
| Q11 | Perfis | Como escolas são cadastradas (painel de administração da plataforma?) |
| Q12 | Tag | Tipo de tag e modelo de leitor |
| Q13 | LGPD | Pai com vários filhos: um termo por filho ou termo único listando os filhos |
| Q14 | LGPD | Prazo de guarda do histórico e dos comprovantes |
| Q15 | LGPD | O que depende de consentimento vs. execução do contrato de matrícula (advogado) |
| Q17 | Financeiro | Responsável financeiro de cada aluno (pagador no gateway) |
| Q18 | Financeiro | Cartão de débito: o Asaas não aceita pela API — manter no escopo? |
| Q19 | Financeiro | Subconta criada pela plataforma ou conta própria da escola conectada |
| Q20 | Financeiro | Pagamento em duplicidade: estornar ou manter como crédito |
| ~~Q16~~ | Escala | ~~Uma ou várias escolas desde o início~~ → **Decidido:** multiescola desde o início, banco único com isolamento por escola ([ADR-001](./arquitetura/ADR-001-multiescola-banco-unico-rls.md)) |

## 13. Glossário

| Termo | Significado |
|---|---|
| Escola | Cliente do produto; administra a própria operação no app |
| Responsável / Pai | Pai, mãe ou responsável legal vinculado a um ou mais alunos |
| Sala | Turma de um ano letivo e turno |
| Cobrança | Mensalidade ou cobrança extra de um aluno, emitida no gateway |
| Subconta | Conta da escola dentro da conta principal do gateway |
| Webhook | Aviso automático enviado pelo gateway quando um pagamento é confirmado |
| Tag | Cartão, chaveiro ou pulseira RFID/NFC do aluno |
| Leitor | Aparelho na entrada da escola que lê a tag e envia ao sistema |
| Registro de acesso | Uma leitura de entrada ou saída de um aluno |
| Termo | Termo de aceite LGPD gerado por aluno a partir do modelo da escola |
| Controlador / Operador | Papéis da LGPD: escola decide sobre os dados; o sistema processa a pedido dela |
| DPO | Encarregado de dados da escola |
