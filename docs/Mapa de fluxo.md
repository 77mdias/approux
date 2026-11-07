# Mapa de fluxo — App de Gestão Escolar

| Campo | Valor |
|---|---|
| Documento | Mapa de fluxo (navegação, fluxos de usuário e fluxos de sistema) |
| Fonte | *App de Gestão Escolar — Especificação v3* (05/11/2026) |
| Versão deste documento | 1.0 |
| Documentos relacionados | [`PRD.md`](./PRD.md) · [`SRS.md`](./SRS.md) |

> Diagramas em [Mermaid](https://mermaid.js.org/) — renderizam direto no GitHub. Os códigos entre colchetes (ex.: `RF-FIN-07`) apontam para os requisitos do [`SRS.md`](./SRS.md).

## Índice

1. [Mapa geral de navegação](#1-mapa-geral-de-navegação)
2. [Login e roteamento por perfil](#2-login-e-roteamento-por-perfil)
3. [Primeiro acesso do pai e termo LGPD](#3-primeiro-acesso-do-pai-e-termo-lgpd)
4. [Área do pai: escolher filho e página do aluno](#4-área-do-pai-escolher-filho-e-página-do-aluno)
5. [Escola: onboarding da escola](#5-escola-onboarding-da-escola)
6. [Escola: cadastro de aluno, pais, tag e termo](#6-escola-cadastro-de-aluno-pais-tag-e-termo)
7. [Escola: publicar nova versão do termo](#7-escola-publicar-nova-versão-do-termo)
8. [Professor: chamada, notas e comportamento](#8-professor-chamada-notas-e-comportamento)
9. [Agenda: criar evento e confirmar reunião](#9-agenda-criar-evento-e-confirmar-reunião)
10. [Avisos e notificações](#10-avisos-e-notificações)
11. [Justificativa de falta](#11-justificativa-de-falta)
12. [Financeiro: mensalidade e cobrança extra](#12-financeiro-mensalidade-e-cobrança-extra)
13. [Financeiro: pagamento pelo pai e baixa por webhook](#13-financeiro-pagamento-pelo-pai-e-baixa-por-webhook)
14. [Financeiro: lembretes automáticos](#14-financeiro-lembretes-automáticos)
15. [Tag: leitura de entrada e saída](#15-tag-leitura-de-entrada-e-saída)
16. [Tag: leitor offline](#16-tag-leitor-offline)
17. [Tag: aviso de ausência](#17-tag-aviso-de-ausência)
18. [Tag: correção manual e gestão de tags](#18-tag-correção-manual-e-gestão-de-tags)
19. [Ciclos de vida (estados)](#19-ciclos-de-vida-estados)

---

## 1. Mapa geral de navegação

Telas por perfil, conforme a seção 12 da especificação.

```mermaid
flowchart TD
    LOGIN["Login"] --> R{"Perfil?"}

    R -->|Escola| E0["Painel do dia"]
    R -->|Professor| P0["Minhas salas"]
    R -->|Pai| T0{"Termo aceito na versão vigente?"}

    subgraph ESCOLA["Escola (gestão)"]
        E0
        E1["Salas"]
        E2["Alunos: ficha, mensalidade, tag, pais"]
        E3["Professores"]
        E4["Agenda"]
        E5["Avisos"]
        E6["Acesso e tags: leitores, tags, correções"]
        E7["Financeiro: dashboard, cobranças, atrasados, lembretes"]
        E8["Termo LGPD: modelo, versões, aceites"]
        E9["Configurações: horários, tolerância, permissões"]
    end

    subgraph PROFESSOR["Professor"]
        P0
        P1["Chamada e frequência"]
        P2["Notas"]
        P3["Comportamento"]
        P4["Agenda da sala"]
        P5["Justificativas recebidas"]
        P6["Avisos"]
        P7["Presentes agora"]
    end

    subgraph PAI["Pai / responsável"]
        T1["Termo LGPD"]
        F0["Escolha do filho"]
        F1["Página do aluno"]
        F2["Frequência"]
        F3["Notas"]
        F4["Comportamento"]
        F5["Entrada e saída"]
        F6["Agenda semanal, mensal, anual"]
        F7["Notificações"]
        F8["Financeiro: por filho, pagar um ou vários"]
        F9["Confirmar reunião"]
        F10["Justificar falta"]
    end

    E0 --- E1 & E2 & E3 & E4 & E5 & E6 & E7 & E8 & E9
    P0 --- P1 & P2 & P3 & P4 & P5 & P6 & P7

    T0 -->|Não| T1
    T1 -->|Aceitou| F0
    T0 -->|Sim| F0
    F0 --> F1
    F1 --- F2 & F3 & F4 & F5 & F6 & F7 & F8
    F6 --> F9
    F2 --> F10
    F1 -->|Trocar filho| F0
```

---

## 2. Login e roteamento por perfil

`RF-AUT-01..08`

```mermaid
flowchart TD
    A(["Usuário abre o app"]) --> B["Informa e-mail/usuário e senha"]
    B --> C{"Conta bloqueada?"}
    C -->|Sim| C1["Mensagem: aguarde ou recupere a senha"]
    C -->|Não| D{"Credenciais válidas?"}
    D -->|Não| D1["Incrementa tentativas erradas"]
    D1 --> D2{"Atingiu o limite?"}
    D2 -->|Sim| D3["Bloqueia a conta temporariamente"]
    D2 -->|Não| B
    D -->|Sim| E{"Usuário ativo?"}
    E -->|Não| E1["Acesso negado"]
    E -->|Sim| F["Cria sessão segura e zera tentativas"]
    F --> G{"Perfil"}
    G -->|Escola| H["Área da escola"]
    G -->|Professor| I["Área do professor"]
    G -->|Pai| J["Verificação do termo LGPD"]
```

---

## 3. Primeiro acesso do pai e termo LGPD

`RF-LGP-05..07`, `RF-LGP-13`, `RN-03`. O termo aparece **antes de qualquer outra tela**; sem aceite o pai não avança.

```mermaid
flowchart TD
    A(["Pai autenticado"]) --> B{"Existe termo pendente para algum filho?"}
    B -->|Não| Z["Escolha do filho"]
    B -->|Sim| C["Exibe termo completo e rolável, já com nome do filho"]
    C --> D{"Pai marca o aceite e confirma?"}
    D -->|Não| C
    D -->|Sim| E{"Escola exige código de confirmação?"}
    E -->|Não| G["Grava prova do aceite"]
    E -->|Sim| F["Envia código por SMS ou e-mail"]
    F --> F1{"Código correto?"}
    F1 -->|Não| F
    F1 -->|Sim| G
    G --> H["Congela cópia do texto aceito; registra versão, data e hora, IP, aparelho e forma de confirmação"]
    H --> I{"Há outro termo pendente?"}
    I -->|Sim| C
    I -->|Não| Z
```

**O que é gravado em cada aceite:** pai e alunos vinculados · versão do termo e cópia do texto · data e hora · IP e aparelho · forma de confirmação · situação (aceito/revogado) e data da revogação.

> Pai com mais de um filho: um termo por filho **ou** termo único listando os filhos — **a definir** pela escola (`RF-LGP-08`).

### 3.1 Revogação do consentimento

`RF-LGP-11`

```mermaid
flowchart LR
    A["Pai solicita revogação"] --> B["Sistema registra situação revogado e data"]
    B --> C["Escola é avisada"]
    C --> D{"Termo prevê bloqueio do acesso?"}
    D -->|Sim| E["Acesso aos dados do filho bloqueado; escola usa canal alternativo fora do app"]
    D -->|Não| F["Aplica efeito definido no termo pelo advogado"]
```

---

## 4. Área do pai: escolher filho e página do aluno

`RF-PAI-01..06`

```mermaid
flowchart TD
    A["Escolha do filho"] --> B{"Quantos filhos vinculados?"}
    B -->|Um| C["Abre direto a página do filho"]
    B -->|Vários| D["Lista: João, Maria, ..."]
    D --> C
    C --> E["Página do aluno"]
    E --> E1["Hoje: chegou às 07:12, está na escola"]
    E --> E2["Frequência: presenças, faltas, %"]
    E --> E3["Notas por disciplina e período"]
    E --> E4["Comportamento"]
    E --> E5["Agenda"]
    E --> E6["Notificações"]
    E --> E7["Financeiro"]
    E -->|Trocar filho, sem novo login| A
```

---

## 5. Escola: onboarding da escola

`RF-ESC-12`, `INT-01`. Responsável pelo cadastro da escola (admin da plataforma) — **a definir**.

```mermaid
sequenceDiagram
    autonumber
    actor Admin as Admin da plataforma
    participant S as Sistema
    participant G as Asaas (conta principal)
    participant V as Cofre de segredos

    Admin->>S: Cadastra escola (nome, CNPJ, plano)
    S->>G: Cria subconta em nome da escola
    G-->>S: Id da subconta e chave de API
    S->>V: Guarda a chave da subconta fora do código
    S-->>Admin: Escola criada (subconta em avaliação)
    Note over G: Subconta nova passa por período de avaliação com limites
    Admin->>S: Cria usuário gestor da escola
    S-->>Admin: Credenciais do gestor enviadas
```

---

## 6. Escola: cadastro de aluno, pais, tag e termo

`RF-ESC-03..06`, `RF-TAG-01`, `RF-LGP-02..04`

```mermaid
flowchart TD
    A["Escola abre Alunos > Novo"] --> B["Preenche cadastro: nome, nascimento, sala"]
    B --> C["Informa mensalidade acordada já com desconto"]
    C --> D["Vincula responsável e parentesco"]
    D --> E{"Responsável já tem login?"}
    E -->|Sim, é pai de outro aluno| F["Vincula ao login existente"]
    E -->|Não| G["Cria usuário pai e envia convite"]
    F --> H
    G --> H{"Campos obrigatórios do termo completos? ex: CPF"}
    H -->|Não| H1["Termo não é gerado; escola é avisada do que falta"]
    H1 --> D
    H -->|Sim| I["Gera termo preenchido a partir do modelo vigente; status pendente"]
    I --> J["Escola pode pré-visualizar e baixar o PDF"]
    J --> K{"Vincular tag agora?"}
    K -->|Sim| L["Escola aciona leitura e encosta a tag no leitor"]
    L --> L1{"Tag livre?"}
    L1 -->|Não| L2["Avisa: tag em uso ou bloqueada"]
    L1 -->|Sim| M["Tag vinculada ao aluno; status ativa"]
    K -->|Depois| N(["Aluno cadastrado"])
    M --> N
```

> Mudança posterior nos dados do pai ou do aluno: o termo **pendente** é regenerado; o já **aceito** permanece como estava (`RF-LGP-09`).

---

## 7. Escola: publicar nova versão do termo

`RF-LGP-10`, `RF-LGP-12`

```mermaid
flowchart TD
    A["Escola edita o modelo de termo"] --> B["Publica nova versão"]
    B --> C["Termos aceitos da versão anterior ficam guardados como estavam"]
    B --> D["Sistema gera novos termos pendentes para todos os alunos"]
    D --> E["No próximo login cada pai vê o termo novo antes de qualquer tela"]
    D --> F["Painel de aceites: quem aceitou e quem não"]
    F --> G["Lembrete automático para pendentes"]
    F --> H["Baixar comprovante de aceite por pai"]
```

---

## 8. Professor: chamada, notas e comportamento

`RF-PED-01..09`, `RF-TAG-16`

```mermaid
flowchart TD
    A["Professor: Minhas salas"] --> B["Escolhe a sala"]
    B --> C{"O que lançar?"}

    C -->|Frequência| D["Abre chamada do dia"]
    D --> D1{"Usar chegada pela tag?"}
    D1 -->|Sim| D2["Pré-preenche presentes com quem entrou hoje"]
    D1 -->|Não| D3["Lista em branco"]
    D2 --> D4["Ajusta presente, falta"]
    D3 --> D4
    D4 --> D5["Salva"]

    C -->|Notas| E["Escolhe disciplina, período e avaliação"]
    E --> E1["Lança valor por aluno"]
    E1 --> E2["Salva"]

    C -->|Comportamento| F["Escolhe aluno"]
    F --> F1["Tipo, data e descrição"]
    F1 --> F2["Salva"]

    D5 & E2 & F2 --> G["Registro de auditoria: quem, quando, antes e depois"]
    G --> H["Pai vê na hora na página do filho"]
```

---

## 9. Agenda: criar evento e confirmar reunião

`RF-AGE-01..07`, `RN-12`, `RN-13`

```mermaid
flowchart TD
    A{"Quem cria?"} -->|Escola| B["Evento geral: feriado, reunião, prova, outro"]
    B --> B1["Escolhe salas atingidas ou todas"]
    A -->|Professor| C{"Sala é dele e a escola permite eventos nela?"}
    C -->|Não| C1["Bloqueado no servidor"]
    C -->|Sim| D["Evento da sala: prova, trabalho, reunião"]
    B1 --> E["Evento salvo na agenda"]
    D --> E
    E --> F["Notificação automática aos pais dos alunos das salas atingidas"]
    F --> G{"Tipo reunião?"}
    G -->|Não| H(["Pai vê na agenda"])
    G -->|Sim| I["Pai responde no app: vou ou não vou"]
    I --> J["Escola e professor veem quem confirmou"]
```

---

## 10. Avisos e notificações

`RF-COM-01..04`, `INT-03`

```mermaid
flowchart LR
    A["Escola"] -->|Todos, sala ou aluno| C["Aviso"]
    B["Professor"] -->|Só suas salas| C
    C --> D["Resolve destinatários: responsáveis dos alunos atingidos"]
    D --> E["Fila de notificações"]
    E --> F["Notificação no app"]
    E --> G["Push, e-mail ou WhatsApp - canal a definir"]
    F --> H["Pai lê e marca como lida"]
```

---

## 11. Justificativa de falta

`RF-COM-05..07`

```mermaid
sequenceDiagram
    autonumber
    actor Pai
    participant S as Sistema
    actor Prof as Professor
    actor Esc as Escola

    Pai->>S: Justificar falta (filho, data, mensagem)
    S->>S: Valida que o aluno é filho dele
    S->>S: Grava mensagem com status enviada
    S-->>Prof: Notifica justificativa recebida
    S-->>Esc: Notifica justificativa recebida
    Prof->>S: Visualiza justificativa
    S->>S: Falta fica registrada com a justificativa
    Note over S: Situação da frequência passa a justificada (critério de aceite a definir)
```

---

## 12. Financeiro: mensalidade e cobrança extra

`RF-FIN-01..05`, `RN-04..07`

### 12.1 Mensalidade

```mermaid
flowchart LR
    A["Cadastro do aluno: mensalidade acordada com desconto"] --> B["Rotina de geração mensal - vencimento a definir"]
    B --> C["Cria cobrança na subconta da escola"]
    C --> D["Cobrança aparece só na área do pai daquele aluno"]
    A -.->|Escola edita valor a qualquer momento| B
```

### 12.2 Cobrança extra

```mermaid
flowchart TD
    A["Escola: Nova cobrança extra"] --> B["Descrição: apostila, uniforme, passeio, evento"]
    B --> C{"Quem entra?"}
    C -->|Um aluno| D["Seleciona aluno"]
    C -->|Uma sala| E["Seleciona sala"]
    C -->|Várias salas| F["Seleciona salas"]
    C -->|Todos| G["Todos os alunos ativos"]
    D & E & F & G --> H["Lista de alunos selecionados"]
    H --> I["Define valor e forma de pagamento de CADA aluno - sem divisão automática"]
    I --> J["Confirma"]
    J --> K["Cria uma cobrança por aluno na subconta da escola"]
    K --> L["Cada pai vê só a cobrança do próprio filho"]
```

---

## 13. Financeiro: pagamento pelo pai e baixa por webhook

`RF-FIN-06..09`, `RN-08`, `RN-09`, `INT-01`

```mermaid
sequenceDiagram
    autonumber
    actor Pai
    participant App as App
    participant S as Sistema
    participant G as Asaas (subconta da escola)

    Pai->>App: Abre Financeiro
    App->>S: Lista cobranças em aberto
    S-->>App: Cobranças por filho (João, Maria)
    Pai->>App: Seleciona cobranças - uma ou de vários filhos
    Pai->>App: Escolhe forma: Pix, crédito, débito ou boleto
    App->>S: Solicita pagamento
    S->>S: Valida que as cobranças são dos filhos dele e estão abertas
    S->>G: Cria cobrança no gateway com a chave da subconta da escola
    G-->>S: Id, QR Code Pix, link do boleto ou checkout do cartão
    S-->>App: Dados de pagamento
    Pai->>G: Paga
    G-)S: Webhook pagamento confirmado
    S->>S: Valida autenticidade do webhook
    alt Evento já processado
        S-->>G: 200 OK sem nova baixa
    else Evento novo
        S->>S: Registra Pagamento e marca cobranças como pagas
        S-->>G: 200 OK
        S-)Pai: Notificação de pagamento confirmado
    end
    Note over S: Dashboard da escola atualiza: quanto entrou, quem pagou, quem está em atraso
```

> Detalhe técnico para a fase de arquitetura: como representar "juntar os dois filhos na mesma cobrança" no gateway (cobrança agregada vs. várias cobranças pagas em um checkout).

---

## 14. Financeiro: lembretes automáticos

`RF-FIN-11`, `RF-FIN-12`

```mermaid
flowchart TD
    A["Rotina diária agendada"] --> B["Busca cobranças abertas da escola"]
    B --> C{"Situação"}
    C -->|Faltam N dias para o vencimento| D["Lembrete antes do vencimento"]
    C -->|Venceu e não foi paga| E["Marca como atrasada"]
    E --> F["Lembrete de atraso a cada M dias"]
    E --> G["Regras de multa, juros, bloqueio - a definir"]
    C -->|Paga ou cancelada| H(["Nada a fazer"])
    D & F --> I["Fila de notificações para os responsáveis"]
```

> N e M: **a definir**.

---

## 15. Tag: leitura de entrada e saída

`RF-TAG-04..07`, `RF-TAG-10`, `RF-TAG-12`, `RN-15`, `RN-16`

```mermaid
sequenceDiagram
    autonumber
    participant T as Tag do aluno
    participant L as Leitor na entrada
    participant API as API POST /acessos
    participant S as Sistema
    actor Pai

    T->>L: Aluno encosta a tag
    L->>API: Chave do leitor, código da tag, data e hora
    API->>S: Autentica leitor
    alt Leitor revogado ou inativo
        S-->>L: Rejeitado
    else Leitor válido
        S->>S: Busca aluno pela tag
        alt Tag desconhecida ou bloqueada
            S-->>L: Rejeitado e registrado para diagnóstico
        else Leitura repetida em poucos segundos
            S-->>L: Confirmado, ignorada
        else Leitura válida
            S->>S: Conta leituras do aluno no dia
            Note over S: Ímpar = entrada, par = saída
            S->>S: Grava registro de acesso (origem tag)
            S-->>L: Confirmado
            S-)Pai: João chegou às 07:12
        end
    end
```

### 15.1 Regra de alternância

```mermaid
flowchart LR
    A["Leitura válida"] --> B{"Já existe registro do aluno hoje?"}
    B -->|Não| C["ENTRADA"]
    B -->|Sim| D{"Último registro de hoje"}
    D -->|Entrada| E["SAÍDA"]
    D -->|Saída| C
    C --> F{"Após horário de entrada + tolerância?"}
    F -->|Sim| G["Marca atraso"]
    E --> H{"Antes do fim do turno?"}
    H -->|Sim| I["Marca saída antecipada"]
```

---

## 16. Tag: leitor offline

`RF-TAG-09`, `RNF-CONF-03`

```mermaid
sequenceDiagram
    autonumber
    participant L as Leitor
    participant M as Memória local do leitor
    participant API as API

    L->>API: Envia leitura
    API--xL: Falha, sem internet
    L->>M: Guarda leitura com data e hora original
    Note over L,M: Continua lendo e acumulando
    loop Quando a internet voltar
        L->>API: Reenvia leituras pendentes em ordem
        API-->>L: Confirma cada uma
        L->>M: Remove as confirmadas
    end
    Note over API: Servidor usa o horário da leitura, não o do recebimento, e descarta duplicatas
```

---

## 17. Tag: aviso de ausência

`RN-17`, `RF-TAG-12`, `RF-TAG-13`

```mermaid
flowchart TD
    A["Horário limite de ausência da sala"] --> B["Verifica alunos ativos da sala"]
    B --> C{"Tem entrada hoje?"}
    C -->|Sim| D(["OK"])
    C -->|Não| E{"Dia letivo? sem feriado na agenda"}
    E -->|Não| D
    E -->|Sim| F["Avisa o pai: João não chegou até 07:30"]
    F --> G["Aparece em Não chegou no painel do dia"]
```

> Ponto de atenção: se um leitor estiver offline, o aviso pode ser falso. Considerar verificar o último contato dos leitores antes de disparar.

---

## 18. Tag: correção manual e gestão de tags

`RF-TAG-02`, `RF-TAG-15`, `RN-19`

```mermaid
flowchart TD
    subgraph CORRECAO["Correção manual de registro"]
        A["Escola abre registro do aluno"] --> B["Inclui, altera ou remove entrada/saída"]
        B --> C{"Informou motivo?"}
        C -->|Não| C1["Bloqueia: motivo obrigatório"]
        C -->|Sim| D["Grava com origem manual, corrigido por e motivo"]
        D --> E["Auditoria"]
    end

    subgraph GESTAO["Gestão de tags"]
        F["Tag perdida"] --> G["Escola bloqueia a tag"]
        G --> H["Vincula nova tag ao aluno"]
        H --> I["Histórico de trocas preservado"]
    end
```

---

## 19. Ciclos de vida (estados)

### 19.1 Cobrança

```mermaid
stateDiagram-v2
    [*] --> Aberta: Cobrança criada no gateway
    Aberta --> Paga: Webhook de pagamento
    Aberta --> Atrasada: Venceu sem pagamento
    Atrasada --> Paga: Webhook de pagamento
    Aberta --> Cancelada: Escola cancela
    Atrasada --> Cancelada: Escola cancela
    Paga --> [*]
    Cancelada --> [*]
```

### 19.2 Tag

```mermaid
stateDiagram-v2
    [*] --> Ativa: Vinculada ao aluno
    Ativa --> Bloqueada: Escola bloqueia
    Ativa --> Perdida: Pai ou escola informa perda
    Bloqueada --> Ativa: Escola desbloqueia
    Perdida --> [*]: Substituída por nova tag
    Bloqueada --> [*]: Substituída por nova tag
```

### 19.3 Termo LGPD

```mermaid
stateDiagram-v2
    [*] --> Pendente: Gerado no cadastro ou nova versão
    Pendente --> Pendente: Dados mudaram, regenerado
    Pendente --> Aceito: Pai aceita
    Pendente --> Substituido: Nova versão publicada
    Aceito --> Substituido: Nova versão publicada
    Aceito --> Revogado: Pai revoga
    Substituido --> [*]
    Revogado --> [*]
```

### 19.4 Justificativa de falta

```mermaid
stateDiagram-v2
    [*] --> Enviada: Pai envia
    Enviada --> Lida: Professor ou escola visualiza
    Lida --> Registrada: Falta passa a justificada
    Registrada --> [*]
    note right of Lida: Valores exatos de status a definir
```

### 19.5 Leitor

```mermaid
stateDiagram-v2
    [*] --> Ativo: Cadastrado com chave de API
    Ativo --> Revogado: Escola revoga a chave
    Revogado --> Ativo: Nova chave gerada
    Ativo --> Inativo: Escola desativa
    Inativo --> Ativo: Escola reativa
```
