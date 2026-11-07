# SRS — Especificação de Requisitos de Software

| Campo | Valor |
|---|---|
| Documento | Software Requirements Specification (SRS) |
| Produto | App de Gestão Escolar (nome **a definir**) |
| Fonte | *App de Gestão Escolar — Especificação v3* (05/11/2026) |
| Versão deste documento | 1.0 |
| Status | Rascunho para validação |
| Documentos relacionados | [`PRD.md`](./PRD.md) · [`Mapa de fluxo.md`](./Mapa%20de%20fluxo.md) |

---

## 1. Introdução

### 1.1 Propósito
Especificar os requisitos funcionais, regras de negócio, dados, interfaces externas e requisitos não funcionais do App de Gestão Escolar, de forma verificável e rastreável, para orientar o desenho da arquitetura, o desenvolvimento e os testes.

### 1.2 Escopo
Sistema multiescola com três perfis (escola, professor, pai/responsável) cobrindo: cadastro escolar, acompanhamento pedagógico, agenda e comunicação, financeiro com pagamento via gateway (subconta por escola), controle de entrada e saída por tag RFID/NFC e consentimento LGPD.

### 1.3 Convenções
- **Identificadores:** `RF-<módulo>-<nn>` (funcional), `RN-<nn>` (regra de negócio), `RNF-<nn>` (não funcional), `INT-<nn>` (interface externa).
- **Prioridade (MoSCoW):** **M** = obrigatório · **S** = importante · **C** = desejável · **W** = não nesta versão.
- **Fase:** fase do roadmap do PRD (1 a 6) em que o requisito é entregue.
- **a definir:** decisão pendente; ver seção 10.
- "Deve" indica obrigatoriedade; "pode" indica opção.

### 1.4 Definições
Ver glossário em [`PRD.md` §13](./PRD.md#13-glossário).

---

## 2. Descrição geral

### 2.1 Perspectiva do produto
Produto novo, SaaS multiescola. Componentes lógicos:

- **Aplicação cliente** para os três perfis (sugestão: PWA instalável, mobile first).
- **API do servidor** (sugestão: REST) que aplica todas as regras de permissão.
- **Banco de dados relacional** compartilhado com separação por escola.
- **Fila de tarefas** para lembretes, notificações e integrações.
- **Gateway de pagamento** externo (Asaas, uma subconta por escola).
- **Leitores de tag** na entrada da escola, que chamam a API.
- **Canais de notificação** (push, e-mail e/ou WhatsApp — **a definir**).

> As sugestões técnicas acima vêm da especificação e podem ser trocadas; a arquitetura será discutida em documento próprio.

### 2.2 Classes de usuários

| Classe | Descrição | Escopo de dados |
|---|---|---|
| Escola (gestão) | Equipe administrativa da escola | Todos os dados da própria escola |
| Professor | Docente com login próprio | Apenas salas (e disciplinas) atribuídas |
| Pai/responsável | Um login por pessoa, válido em todas as escolas (ADR-003) | Apenas os filhos vinculados, inclusive em escolas diferentes, após aceite do termo de cada escola |
| Leitor (sistema) | Dispositivo autenticado por chave de API | Apenas envio de leituras da própria escola |
| Gateway (sistema) | Envia webhooks de pagamento | Apenas cobranças da subconta correspondente |
| Admin da plataforma | Dono do produto; cadastra escolas | **a definir** (não descrito na especificação) |

### 2.3 Ambiente operacional
- Navegadores modernos de celular (Android/iOS) e desktop.
- Leitores RFID/NFC com Wi-Fi ou cabo, capazes de armazenar leituras offline.

### 2.4 Restrições
- **C-01** O dinheiro das famílias nunca passa pela conta do dono do produto.
- **C-02** Permissões devem ser impostas no servidor, em toda consulta.
- **C-03** Conformidade com a LGPD (dados de crianças e adolescentes).
- **C-04** Chaves do gateway e dos leitores guardadas fora do código, em local protegido.
- **C-05** O controle de acesso por tag exige servidor e API reais (no protótipo, apenas simulado).

### 2.5 Premissas e dependências
- **D-01** Disponibilidade da API de subcontas do Asaas e aprovação das subcontas pelo período de avaliação.
- **D-02** Modelo de termo fornecido pela escola e revisado por advogado.
- **D-03** Hardware de leitor escolhido (**a definir**).

---

## 3. Requisitos funcionais

### 3.1 Autenticação e contas (AUT) — Fase 1

| ID | Requisito | Prior. |
|---|---|---|
| RF-AUT-01 | O sistema deve permitir login com e-mail (ou usuário) e senha para os perfis escola, professor e pai. | M |
| RF-AUT-02 | Senhas devem ser armazenadas com hash. | M |
| RF-AUT-03 | O sistema deve bloquear a conta temporariamente após um número de tentativas erradas (limite **a definir**). | M |
| RF-AUT-04 | O usuário deve poder trocar a própria senha. | M |
| RF-AUT-05 | O sistema deve permitir recuperação de senha (canal **a definir**). | S |
| RF-AUT-06 | A sessão deve ser segura e expirar por inatividade (tempo **a definir**). | M |
| RF-AUT-07 | Usuários inativos não podem entrar. | M |
| RF-AUT-08 | Após o login, o usuário é direcionado à área do seu perfil. | M |
| RF-AUT-09 | Usuário com mais de um contexto (filhos em escolas diferentes, professor em várias escolas, professor que também é pai) escolhe o contexto ativo e pode trocá-lo sem novo login; a troca valida o vínculo no servidor. | M |

### 3.2 Gestão da escola — cadastros (ESC) — Fase 1

| ID | Requisito | Prior. |
|---|---|---|
| RF-ESC-01 | A escola deve criar, editar e desativar salas (nome, ano letivo, turno). | M |
| RF-ESC-02 | A lista de salas deve exibir a quantidade de alunos de cada uma (calculada). | M |
| RF-ESC-03 | A escola deve cadastrar alunos com cadastro completo (nome, nascimento, sala, mensalidade acordada, desconto, tag, ativo). | M |
| RF-ESC-04 | A escola deve vincular um aluno a um ou mais responsáveis, informando parentesco. | M |
| RF-ESC-05 | Um responsável pode ser vinculado a vários alunos e usa um único login. | M |
| RF-ESC-06 | Ao vincular um responsável, o sistema deve enviar um convite ao e-mail informado: se a pessoa já tiver conta, ela confirma o novo vínculo com o próprio login; se não, cria a senha. A escola vê apenas "convite enviado/aceito" e nunca descobre se a conta já existia (ADR-003). | M |
| RF-ESC-07 | A escola deve cadastrar professores, criar o login deles e definir quais salas e disciplinas cada um atende. | M |
| RF-ESC-08 | A escola deve ver a ficha do aluno com mensalidade, tag e pais. | M |
| RF-ESC-09 | A escola deve acompanhar frequência, notas e comportamento de qualquer aluno. | M |
| RF-ESC-10 | A escola deve configurar, por sala: horário de entrada, tolerância de atraso e horário limite para aviso de ausência. | M (F5) |
| RF-ESC-11 | A escola deve ligar/desligar, sala por sala, a permissão de o professor lançar eventos. | M (F3) |
| RF-ESC-12 | Admin da plataforma deve cadastrar uma nova escola (nome, CNPJ, plano) e disparar a criação da subconta no gateway. | M (forma **a definir**) |

### 3.3 Área do pai (PAI) — Fases 1–2

| ID | Requisito | Prior. |
|---|---|---|
| RF-PAI-01 | Após o login (e o aceite do termo), o pai deve ver a lista dos filhos vinculados — agrupados por escola quando houver mais de uma — e escolher qual acompanhar. | M |
| RF-PAI-02 | O pai deve poder trocar de filho a qualquer momento sem novo login. | M |
| RF-PAI-03 | Se houver só um filho, o sistema pode abrir direto a página dele. | C |
| RF-PAI-04 | A página do aluno deve mostrar: frequência (presenças, faltas, percentual), notas por disciplina e período, comportamento, entrada/saída, agenda, notificações e financeiro. | M |
| RF-PAI-05 | Lançamentos do professor devem aparecer para o pai assim que salvos. | M |
| RF-PAI-06 | O pai nunca pode acessar dados de alunos não vinculados a ele (ver RNF-SEG). | M |

### 3.4 Acompanhamento pedagógico (PED) — Fase 2

| ID | Requisito | Prior. |
|---|---|---|
| RF-PED-01 | O professor deve ver apenas as salas atribuídas a ele. | M |
| RF-PED-02 | O professor deve lançar a frequência diária da sala (presente, falta, justificada). | M |
| RF-PED-03 | A chamada deve poder vir pré-preenchida com os alunos que entraram pela tag no dia. | S (F5) |
| RF-PED-04 | O professor deve lançar notas por aluno, disciplina, período e avaliação. | M |
| RF-PED-05 | O cálculo de média, períodos e recuperação deve seguir regras configuráveis (**a definir**). | S |
| RF-PED-06 | O sistema deve calcular o percentual de frequência e sinalizar quando abaixo do mínimo (mínimo **a definir**). | S |
| RF-PED-07 | O professor deve registrar comportamento (data, tipo, descrição), visível ao pai. | M |
| RF-PED-08 | A escola pode lançar e corrigir nota, frequência e comportamento de qualquer sala. | M |
| RF-PED-09 | Toda alteração de nota e frequência deve ser registrada em auditoria (quem, quando, antes/depois). | M |

### 3.5 Agenda e eventos (AGE) — Fase 3

| ID | Requisito | Prior. |
|---|---|---|
| RF-AGE-01 | A agenda deve ter visões semanal, mensal e anual. | M |
| RF-AGE-02 | A escola deve criar eventos gerais (feriado, reunião, prova, outro) para todas ou algumas salas. | M |
| RF-AGE-03 | O professor deve criar eventos (prova, trabalho, reunião) apenas nas suas salas e apenas se a escola permitir naquela sala. | M |
| RF-AGE-04 | A escola deve ver todos os eventos. | M |
| RF-AGE-05 | Ao criar um evento, os pais dos alunos das salas atingidas devem ser notificados automaticamente. | M |
| RF-AGE-06 | Para eventos do tipo reunião, o pai deve confirmar presença no app. | M |
| RF-AGE-07 | Escola e professor devem ver quem confirmou presença em cada reunião. | M |

### 3.6 Comunicação (COM) — Fase 3

| ID | Requisito | Prior. |
|---|---|---|
| RF-COM-01 | A escola deve enviar avisos gerais, por sala ou por aluno. | M |
| RF-COM-02 | O professor deve enviar avisos apenas às suas salas. | M |
| RF-COM-03 | Cada aviso deve gerar notificação para os responsáveis do destino. | M |
| RF-COM-04 | O pai deve ver as notificações do filho selecionado e marcá-las como lidas. | M |
| RF-COM-05 | O pai deve justificar uma falta enviando uma mensagem (aluno, data da falta, texto). | M |
| RF-COM-06 | A justificativa deve chegar ao professor da sala e à escola. | M |
| RF-COM-07 | A falta deve ficar registrada com a justificativa associada (situação "justificada" — critério de aceite pela escola/professor **a definir**). | M |
| RF-COM-08 | O canal de entrega (push, e-mail, WhatsApp) deve ser configurável (**a definir**). | S |

### 3.7 Financeiro (FIN) — Fase 4

| ID | Requisito | Prior. |
|---|---|---|
| RF-FIN-01 | A escola deve definir a mensalidade de cada aluno no cadastro, já com o desconto acordado, e editá-la a qualquer momento. | M |
| RF-FIN-02 | O sistema deve gerar as cobranças de mensalidade de cada aluno ativo na subconta da escola (dia de vencimento e periodicidade **a definir**). | M |
| RF-FIN-03 | A escola deve criar cobrança extra escolhendo destinatários: um aluno, uma sala, várias salas ou todos. | M |
| RF-FIN-04 | Na cobrança extra, a escola deve informar valor e forma de pagamento **de cada aluno**; não há divisão automática. | M |
| RF-FIN-05 | Cada cobrança deve aparecer somente na área do pai do aluno correspondente. | M |
| RF-FIN-06 | O pai deve ver o que deve, por filho, e pagar dentro do app via Pix (QR Code), cartão de crédito ou boleto. Cartão de débito: a API do Asaas não aceita criar cobrança de débito — **a confirmar** com o Asaas (ADR-005). | M |
| RF-FIN-07 | O pai com mais de um filho deve poder pagar a cobrança de um só ou juntar cobranças de vários filhos **da mesma escola** em um único pagamento, por Pix ou cartão de crédito (sem boleto — **a confirmar**). Filhos em escolas diferentes são pagos separadamente, um grupo por escola. | M |
| RF-FIN-08 | A cobrança deve ser marcada como paga automaticamente ao receber a confirmação do gateway (webhook). | M |
| RF-FIN-09 | A escola deve ter um dashboard: total recebido, quem pagou, quem está em atraso, quem falta pagar. | M |
| RF-FIN-10 | A escola deve poder cancelar uma cobrança aberta. | S |
| RF-FIN-11 | O sistema deve enviar lembretes automáticos antes e depois do vencimento, para mensalidade e extras (dias **a definir**). | M |
| RF-FIN-12 | O sistema deve aplicar regras de atraso (multa, juros, bloqueio) conforme definido (**a definir**). | S |
| RF-FIN-13 | Toda alteração em cobranças deve ser auditada. | M |
| RF-FIN-14 | A escola deve indicar o **responsável financeiro** de cada aluno (pagador no gateway; CPF obrigatório). Padrão: primeiro responsável vinculado. | M |
| RF-FIN-15 | O sistema deve detectar pagamento em duplicidade de uma cobrança e alertar a escola, que escolhe estornar ou manter como crédito (**a confirmar**). | M |
| RF-FIN-16 | O sistema deve reconciliar diariamente os pagamentos de cada escola com o gateway e alertar quando a fila de webhooks parecer parada. | M |
| RF-FIN-17 | O pagamento conjunto não pago expira no fim do dia e as cobranças voltam a ficar disponíveis. | M |

### 3.8 Entrada e saída por tag (TAG) — Fase 5

| ID | Requisito | Prior. |
|---|---|---|
| RF-TAG-01 | A escola deve vincular uma tag ao aluno lendo a tag no leitor a partir da tela do aluno. | M |
| RF-TAG-02 | A escola deve trocar e bloquear tags (ex.: perdida), mantendo histórico. | M |
| RF-TAG-03 | A escola deve cadastrar leitores (nome, local) e gerar/revogar a chave de API de cada um. | M |
| RF-TAG-04 | A API deve receber leituras com chave do leitor, código da tag e data/hora, e responder confirmando o registro. | M |
| RF-TAG-05 | O sistema deve identificar o aluno pela tag e gravar o registro de acesso. | M |
| RF-TAG-06 | A primeira leitura do dia é **entrada**; a seguinte, **saída**; e assim alternadamente. | M |
| RF-TAG-07 | Leituras repetidas da mesma tag em poucos segundos devem ser ignoradas (janela **a definir**). | M |
| RF-TAG-08 | Leituras de tags bloqueadas, desconhecidas ou de leitor revogado devem ser rejeitadas e registradas para diagnóstico. | S |
| RF-TAG-09 | O leitor deve armazenar leituras sem internet e reenviá-las depois; o servidor deve usar a data/hora da leitura, não a do recebimento. | M |
| RF-TAG-10 | O pai deve ver horário de chegada e saída de hoje, se o filho está dentro da escola agora e o tempo de permanência. | M |
| RF-TAG-11 | O pai deve ver histórico por dia, semana e mês, incluindo atrasos e saídas antecipadas. | M |
| RF-TAG-12 | O pai deve ser notificado na chegada, na saída e quando o filho não chegou até o horário limite. | M |
| RF-TAG-13 | A escola deve ter o painel do dia: quem chegou, está dentro, saiu e não chegou. | M |
| RF-TAG-14 | A escola deve ter lista por sala de quem está no prédio (chamada e emergência). | M |
| RF-TAG-15 | A escola deve corrigir manualmente um registro informando motivo; o sistema registra quem corrigiu. | M |
| RF-TAG-16 | O professor deve ver quem da sala dele está presente. | M |

### 3.9 Termo de aceite LGPD (LGP) — recomendado junto à Fase 2

| ID | Requisito | Prior. |
|---|---|---|
| RF-LGP-01 | A escola deve cadastrar um modelo de termo com campos substituíveis: `{{aluno}}`, `{{nascimento}}`, `{{sala}}`, `{{responsavel}}`, `{{cpf_responsavel}}`, `{{parentesco}}`, `{{escola}}`, `{{cnpj_escola}}`, `{{encarregado_dados}}`, `{{data}}`. | M |
| RF-LGP-02 | Ao cadastrar o aluno e vincular o responsável, o sistema deve gerar o termo preenchido e deixá-lo pendente de aceite. | M |
| RF-LGP-03 | Se faltar campo obrigatório (ex.: CPF do responsável), o sistema não gera o termo e avisa a escola o que falta. | M |
| RF-LGP-04 | A escola deve visualizar o termo gerado de cada aluno e baixá-lo em PDF antes do aceite. | M |
| RF-LGP-05 | No primeiro acesso, o pai deve ver o termo completo e rolável antes de qualquer outra tela; sem aceite, não avança. | M |
| RF-LGP-06 | O aceite é por clique; a escola pode exigir confirmação adicional por código enviado ao celular ou e-mail. | M / S |
| RF-LGP-07 | O sistema deve gravar a prova do aceite (ver §5.2) e congelar a cópia exata do texto aceito. | M |
| RF-LGP-08 | Pai com mais de um filho: um termo por filho ou termo único listando os filhos (**a definir** pela escola). | M |
| RF-LGP-09 | Se os dados do pai ou do aluno mudarem, o termo pendente é gerado de novo; o já aceito permanece como estava. | M |
| RF-LGP-10 | Ao publicar nova versão do modelo, todos os pais devem aceitar novamente no próximo login. | M |
| RF-LGP-11 | O pai deve poder revogar o consentimento; o sistema registra a data da revogação e respeita os efeitos definidos no termo. | M |
| RF-LGP-12 | A escola deve ver a lista de quem aceitou e quem não, enviar lembrete automático aos pendentes e baixar o comprovante de aceite de cada pai. | M |
| RF-LGP-13 | Cada responsável de um mesmo aluno aceita com o próprio login. Pai com filhos em escolas diferentes aceita o termo de cada escola separadamente; o bloqueio por termo pendente vale por escola. | M |
| RF-LGP-14 | O sistema deve suportar pedidos do titular: consultar, corrigir e pedir exclusão (fluxo operacional **a definir**). | S |

### 3.10 Auditoria (AUD) — Fase 6

| ID | Requisito | Prior. |
|---|---|---|
| RF-AUD-01 | Registrar quem alterou notas, frequência, cobranças e registros de acesso (usuário, data/hora, antes/depois, motivo quando houver). | M |
| RF-AUD-02 | Registrar acessos de usuários a dados de alunos (log de acesso). | M |
| RF-AUD-03 | A escola deve consultar a trilha de auditoria da própria escola. | S |

---

## 4. Regras de negócio

| ID | Regra |
|---|---|
| RN-01 | Um sistema atende várias escolas; nenhum dado de uma escola é visível para outra. |
| RN-02 | Professor só acessa salas atribuídas (Professor–Sala); pai só acessa os filhos vinculados (Responsável–Aluno). |
| RN-03 | O pai não vê nenhum dado dos filhos sem termo aceito na versão vigente. |
| RN-04 | Mensalidade é definida por aluno, já com desconto, e editável a qualquer momento pela escola. |
| RN-05 | Cobrança extra tem valor e forma de pagamento definidos por aluno; não existe divisão igual automática. |
| RN-06 | Cada cobrança aparece apenas para os responsáveis do aluno a que pertence. |
| RN-07 | Cobranças são emitidas na subconta da escola; o dinheiro cai direto nela, sem repasse. |
| RN-08 | A baixa de uma cobrança ocorre apenas por confirmação do gateway (webhook); o mesmo evento não pode baixar duas vezes. |
| RN-09 | Pagamento conjunto de vários filhos quita todas as cobranças incluídas, cada uma com seu valor; só é permitido entre cobranças da mesma escola (subconta). |
| RN-24 | Uma cobrança pode estar em no máximo um pagamento conjunto ativo; ao pagar o conjunto, os títulos individuais abertos são cancelados no gateway. |
| RN-23 | Um usuário é global e pode ter vínculos (gestor, professor, responsável) com várias escolas; nenhuma escola vê os vínculos ou dados de outra. |
| RN-10 | Lembretes são enviados antes e depois do vencimento, para mensalidade e extras. |
| RN-11 | Atraso: multa, juros e bloqueio **a definir**. |
| RN-12 | Professor só cria eventos nas suas salas e somente se a escola permitir naquela sala. |
| RN-13 | Ao criar evento ou aviso, os pais dos alunos atingidos são notificados. |
| RN-14 | Confirmação de reunião e justificativa de falta são feitas somente pelo pai e chegam ao professor e à escola. |
| RN-15 | Leituras de tag alternam entrada/saída por aluno por dia, começando por entrada. |
| RN-16 | Leituras repetidas em poucos segundos são ignoradas. |
| RN-17 | Sem leitura de entrada até o horário limite da sala, o pai recebe aviso de ausência. |
| RN-18 | Atraso = entrada após horário de entrada + tolerância; saída antecipada = saída antes do fim do turno (horário de fim **a definir**). |
| RN-19 | Correção manual de registro de acesso exige motivo e registra o autor; apenas a escola pode corrigir. |
| RN-20 | O texto aceito do termo é congelado com os dados preenchidos; modelo e versão ficam registrados. |
| RN-21 | Mudança de dados regenera apenas termos pendentes; nova versão do modelo exige novo aceite de todos. |
| RN-22 | Notas: média, períodos e recuperação **a definir**. Frequência mínima **a definir**. |

---

## 5. Requisitos de dados

### 5.1 Entidades (modelo lógico da especificação)

| Entidade | Campos principais | Observações |
|---|---|---|
| Escola | nome, CNPJ, conta de pagamento (gateway), configurações (horários, tolerância, horário limite de ausência), plano do sistema | Raiz do isolamento multiescola |
| Usuário | nome, e-mail (único global), telefone, senha (hash), ativo | **Global**, sem escola (ADR-003) |
| Vínculo | usuário, escola, papel (gestor, professor, responsável), ativo | Uma pessoa pode ter vários vínculos em várias escolas |
| Sala | escola, nome, ano letivo, turno, professores | Contagem de alunos calculada |
| Professor–Sala | professor, sala, disciplina | Define o que o professor vê |
| Aluno | escola, nome, nascimento, sala, mensalidade acordada, desconto, tag, ativo | Mensalidade é por aluno |
| Responsável–Aluno | usuário pai, aluno, parentesco | Um pai com vários filhos usa um login |
| Tag | código, aluno, status (ativa, bloqueada, perdida), data do vínculo | Histórico de trocas |
| Leitor | escola, nome, local, chave da API, ativo | Chave própria, revogável |
| Registro de acesso | aluno, tag, leitor, data e hora, tipo (entrada/saída), origem (tag/manual), corrigido por, motivo | Entrada e saída alternam no dia |
| Frequência | aluno, sala, data, situação (presente, falta, justificada), justificativa, lançado por | Pode vir pré-preenchida pelo acesso |
| Nota | aluno, disciplina, período, avaliação, valor, lançado por | Regras de média a definir |
| Comportamento | aluno, data, tipo, descrição, lançado por | Visível ao pai |
| Evento | escola, título, tipo (prova, reunião, feriado, outro), data, salas, criado por | Professor só cria para as salas dele |
| Confirmação de presença | evento, pai, aluno, resposta | Reuniões |
| Aviso | escola, título, texto, destino (todos, sala, aluno), autor, data | Gera notificação |
| Mensagem de justificativa | aluno, pai, data da falta, texto, status | Vai ao professor e à escola |
| Cobrança | escola, aluno, tipo (mensalidade/extra), competência (mensalidade), descrição, valor, vencimento, forma de pagamento, status (aberta, paga, atrasada, cancelada, estornada) | Valor sempre definido por aluno; o id no gateway fica no Título (ADR-005) |
| Pagamento | cobrança, valor pago, forma (Pix, crédito, débito, boleto), data, id no gateway | Vem do webhook do gateway |
| Notificação | usuário, texto, tipo, lida, data | Canal a definir |

### 5.2 Entidades complementares (derivadas dos requisitos)

Não aparecem na tabela de modelo de dados da especificação, mas são necessárias para atender os requisitos:

| Entidade | Campos principais | Origem |
|---|---|---|
| Modelo de termo | escola, versão, texto com campos, publicado em, vigente | RF-LGP-01, RF-LGP-10 |
| Termo gerado | modelo/versão, aluno, responsável, texto preenchido, status (pendente, aceito, revogado, substituído) | RF-LGP-02, RN-20 |
| Aceite de termo | responsável, alunos vinculados, versão do termo, cópia do texto aceito, data e hora, IP, aparelho, forma de confirmação (clique, código), situação, data da revogação | RF-LGP-07, PDF §7-B |
| Disciplina | escola, nome | Professor–Sala, Nota |
| Período letivo | escola, ano, períodos (bimestres etc.) | Nota, RN-22 |
| Cobrança extra (lote) | escola, descrição, criado por, itens por aluno | RF-FIN-03/04 (agrupa as cobranças de um mesmo extra) |
| Título no gateway | escola, tipo (individual/agrupado), id no Asaas, forma, valor, vencimento, status, link de pagamento; cobranças cobertas | RF-FIN-07, ADR-005 |
| Cliente no gateway | escola, usuário (responsável financeiro), id do customer no Asaas | RF-FIN-14 |
| Log de auditoria | escola, usuário, entidade, id, ação, antes, depois, motivo, data | RF-AUD-01/02 |
| Evento de webhook | provedor, id do evento, payload, processado em | Idempotência (RN-08) |
| Subconta do gateway | escola, id da subconta, referência da chave (cofre), status de aprovação | INT-01 |

### 5.3 Retenção
Prazo de guarda do histórico de acessos, dados pedagógicos e comprovantes de aceite: **a definir** pela escola/advogado. O sistema deve suportar expurgo/anonimização após o prazo e exclusão sob pedido.

---

## 6. Matriz de permissões

| Ação | Escola | Professor | Pai |
|---|---|---|---|
| Cadastrar salas, alunos, professores, tags e leitores | Sim | Não | Não |
| Definir mensalidade e criar cobranças | Sim | Não | Não |
| Ver dashboard financeiro e inadimplência | Sim | Não | Não |
| Lançar nota, frequência e comportamento | Sim | Só nas salas dele | Não |
| Criar evento | Todos | Só nas salas dele, se a escola permitir | Não |
| Enviar aviso | Sim | Só às salas dele | Não |
| Ver dados do aluno | Todos | Só salas dele | Só os filhos dele |
| Ver horários de entrada e saída | Todos | Só salas dele | Só os filhos dele |
| Corrigir registro de acesso | Sim (com motivo) | Não | Não |
| Pagar cobranças | Não | Não | Só as dos filhos dele |
| Confirmar reunião e justificar falta | Não | Não | Sim |
| Gerenciar modelo de termo e ver aceites | Sim | Não | Não |
| Aceitar/revogar termo | Não | Não | Sim |

**Obrigatório:** essas regras valem no servidor, em toda consulta, e não só escondendo botões na tela.

---

## 7. Interfaces externas

### INT-01 — Gateway de pagamento (Asaas, subconta por escola)
- O dono do produto mantém **uma conta principal**; para cada escola o sistema cria, via API, uma **subconta** em nome da escola.
- Cada subconta tem chave de API, saldo, cobranças e dados próprios. O sistema guarda a chave de cada escola em local protegido (fora do código) e usa a chave certa em cada cobrança.
- Operações: criar subconta; criar/cancelar cobrança (Pix, cartão de crédito, cartão de débito, boleto); consultar status; receber webhooks de pagamento.
- **Webhook:** o sistema recebe o aviso de pagamento e dá baixa na cobrança. Deve validar a autenticidade e ser **idempotente**.
- **Mensalidade do sistema:** separada; futuramente via split ou cobrança mensal (**a definir**).
- **Atenção:** subcontas criadas via API passam por período de avaliação (até 10 subcontas, até R$ 2.000 em cobranças por subconta, até 60 dias) e a conta principal precisa ser PJ. Confirmar com o gerente de contas antes de fechar; alternativa: a escola conecta a própria conta Asaas (ADR-005 §7).
- Detalhamento de idempotência, pagamento conjunto e mapeamento de eventos: [ADR-005](./arquitetura/ADR-005-pagamentos-asaas-e-webhooks.md).

### INT-02 — Leitor de tags
- Rota da API (exemplo: `POST /acessos`) com: chave do leitor, código da tag e data/hora da leitura.
- A resposta confirma o registro. Leituras não confirmadas permanecem no leitor para reenvio.
- O servidor deve aceitar reenvio de leituras antigas (offline) e tratar duplicatas.
- Chave por leitor, revogável pela escola.

### INT-03 — Notificações
- Push no app, e-mail e/ou WhatsApp. Canal **a definir**.
- Disparadas por: evento criado, aviso, chegada/saída/ausência, lembretes de cobrança, lembrete de termo pendente, código de confirmação do aceite.
- Processadas de forma assíncrona (fila).

---

## 8. Requisitos não funcionais

### 8.1 Segurança (SEG)
| ID | Requisito |
|---|---|
| RNF-SEG-01 | Senhas com hash forte; bloqueio após tentativas erradas; troca de senha. |
| RNF-SEG-02 | Autorização por perfil e por vínculo verificada no servidor em toda requisição. |
| RNF-SEG-03 | Isolamento entre escolas em todas as consultas. |
| RNF-SEG-04 | Chaves dos leitores e do gateway armazenadas fora do código, em local protegido. |
| RNF-SEG-05 | Comunicação somente via HTTPS. |
| RNF-SEG-06 | Webhooks do gateway autenticados antes do processamento. |

### 8.2 Privacidade / LGPD (PRV)
| ID | Requisito |
|---|---|
| RNF-PRV-01 | Escola = controladora; sistema = operador. |
| RNF-PRV-02 | Consentimento específico e em destaque de ao menos um dos pais/responsável legal antes do acesso aos dados. |
| RNF-PRV-03 | Registro de acessos a dados de menores. |
| RNF-PRV-04 | Prazo de guarda configurável; exclusão sob pedido. |
| RNF-PRV-05 | Compartilhamento apenas com professores das salas do aluno e com o gateway de pagamento; nada vendido nem usado para publicidade. |

### 8.3 Confiabilidade (CONF)
| ID | Requisito |
|---|---|
| RNF-CONF-01 | Sem pagamento duplicado; webhook repetido não gera baixa dupla. |
| RNF-CONF-02 | Leitura dupla de tag não gera registros duplicados. |
| RNF-CONF-03 | Queda de internet do leitor não perde registros. |
| RNF-CONF-04 | Falha no envio de notificação deve ser reprocessada (retentativas). |

### 8.4 Desempenho e disponibilidade (DES) — metas sugeridas, **a validar**
| ID | Requisito |
|---|---|
| RNF-DES-01 | Leitura de tag registrada e visível ao pai em poucos segundos após o recebimento. |
| RNF-DES-02 | Telas principais carregam rapidamente em rede móvel (meta numérica **a definir**). |
| RNF-DES-03 | Pico de leituras no horário de entrada suportado (vários leitores simultâneos). |
| RNF-DES-04 | Disponibilidade alvo e janela de manutenção **a definir**. |

### 8.5 Usabilidade (USA)
| ID | Requisito |
|---|---|
| RNF-USA-01 | Interface mobile first, instalável no celular (sugestão: PWA). |
| RNF-USA-02 | Organização em abas separadas: acompanhamento do aluno, agenda, financeiro, gestão. |
| RNF-USA-03 | Pai troca de filho com um toque, sem novo login. |
| RNF-USA-04 | Interface em português do Brasil. |

### 8.6 Manutenibilidade (MAN)
| ID | Requisito |
|---|---|
| RNF-MAN-01 | Regras ainda indefinidas (média, frequência mínima, atraso, lembretes) devem ser configuráveis sem alterar código, sempre que viável. |
| RNF-MAN-02 | Gateway de pagamento e canal de notificação isolados atrás de interfaces para permitir troca. |

---

## 9. Verificação — cenários de teste obrigatórios

| # | Cenário | Requisitos |
|---|---|---|
| T-01 | Pai tenta abrir o aluno de outra família (via URL/API) → negado | RN-02, RNF-SEG-02 |
| T-02 | Professor tenta acessar sala não atribuída → negado | RN-02 |
| T-03 | Usuário da escola A tenta acessar dados da escola B → negado | RN-01, RNF-SEG-03 |
| T-04 | Pagamento duplicado da mesma cobrança | RNF-CONF-01 |
| T-05 | Webhook repetido do gateway → uma única baixa | RN-08 |
| T-06 | Leitura dupla de tag em poucos segundos → um único registro | RN-16 |
| T-07 | Leitor sem internet acumula leituras e reenvia → registros com horário correto, sem duplicar | RF-TAG-09 |
| T-08 | Pai com dois filhos paga os dois juntos → ambas as cobranças pagas | RF-FIN-07, RN-09 |
| T-16 | Pai com filhos nas escolas A e B: escola A não vê o filho, o vínculo nem as cobranças da escola B; pagamento conjunto entre escolas é recusado | RN-23, RN-09 |
| T-17 | Escola cadastra responsável com e-mail já existente → nenhuma informação sobre a conta existente é revelada | RF-ESC-06 |
| T-18 | Webhooks chegando fora de ordem (RECEIVED antes de CREATED) → estado final correto | ADR-005 |
| T-19 | Timeout ao criar cobrança no Asaas e nova tentativa → um único título no gateway | ADR-005 |
| T-20 | Pai paga o conjunto e um boleto individual no mesmo dia → alerta de duplicidade para a escola | RF-FIN-15 |
| T-21 | Rotina de mensalidades executada duas vezes no mês → nenhuma cobrança duplicada | RF-FIN-02 |
| T-22 | Webhook com evento ou campo desconhecido → responde 200 e não interrompe a fila | ADR-005 |
| T-09 | Pai sem termo aceito tenta acessar dados (via API) → negado | RN-03 |
| T-10 | Nova versão do termo publicada → pai é obrigado a reaceitar no próximo login | RF-LGP-10 |
| T-11 | Cadastro sem CPF do responsável → termo não gerado e escola avisada | RF-LGP-03 |
| T-12 | Professor tenta criar evento em sala com permissão desligada → negado | RN-12 |
| T-13 | Aluno sem entrada até o horário limite → pai recebe aviso de ausência | RN-17 |
| T-14 | Correção manual sem motivo → rejeitada | RN-19 |
| T-15 | Tag bloqueada lida no leitor → rejeitada e registrada | RF-TAG-08 |

---

## 10. Itens a definir

Consolidados em [`PRD.md` §12](./PRD.md#12-pontos-em-aberto-consolidado). Itens adicionais levantados nesta especificação:

| # | Item |
|---|---|
| S1 | Limite de tentativas de login e tempo de bloqueio |
| S2 | Tempo de expiração da sessão |
| S3 | Canal de recuperação de senha e de envio de credenciais aos pais |
| S4 | Janela (em segundos) para ignorar leituras repetidas |
| S5 | Horário de fim do turno por sala (para "saída antecipada") |
| S6 | Valores de status da justificativa de falta e quem a aceita |
| S7 | Fluxo operacional dos direitos do titular (consulta, correção, exclusão) |
| S8 | Metas numéricas de desempenho e disponibilidade |
| S9 | Interface de administração da plataforma (cadastro de escolas) |

---

## 11. Rastreabilidade módulo × fase

| Módulo | Fase | Requisitos |
|---|---|---|
| Autenticação e cadastros | 1 | RF-AUT-*, RF-AUT-09, RF-ESC-01..09, RF-ESC-12, RF-PAI-01..03 |
| Acompanhamento | 2 | RF-PED-*, RF-PAI-04..06 |
| Termo LGPD | 2 (recomendado) / 6 (original) | RF-LGP-* |
| Agenda e comunicação | 3 | RF-AGE-*, RF-COM-*, RF-ESC-11 |
| Financeiro | 4 | RF-FIN-* |
| Entrada e saída por tag | 5 | RF-TAG-*, RF-ESC-10, RF-PED-03 |
| Acabamento | 6 | RF-AUD-*, RNF-PRV-*, testes com escola piloto |
