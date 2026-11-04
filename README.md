# Biblioteca Comunitária: Requisitos e Arquitetura de um Sistema

**Disciplina:** Projetos de Sistemas · CIESA (Centro Universitário de Ensino Superior do Amazonas)
**Trabalho:** individual
**Autora:** Luchie Bonates Simões
**Tema:** sistema de empréstimos para uma biblioteca comunitária de Manaus

<!-- Os comentários como este não aparecem no GitHub. Servem de lembrete e podem ser apagados no final. -->

---

## Sumário

- [Parte 1: Requisitos](#parte-1-requisitos)
  - [1.1 Descrição do sistema](#11-descrição-do-sistema)
  - [1.2 Tabela de requisitos](#12-tabela-de-requisitos)
- [Parte 2: Documento de arquitetura](#parte-2-documento-de-arquitetura)
  - [2.1 Diagramas C4](#21-diagramas-c4-níveis-1-a-3)
  - [2.2 Registros de decisão (ADRs)](#22-registros-de-decisão-adrs)
  - [2.3 Comunicação cliente-servidor](#23-comunicação-cliente-servidor)
  - [2.4 Rastreabilidade](#24-rastreabilidade)

---

## Parte 1: Requisitos

### 1.1 Descrição do sistema

<!-- Um parágrafo sobre o negócio e os tipos de usuário: leitor, bibliotecário e administrador. -->

A Biblioteca Comunitária de Manaus é uma biblioteca de bairro mantida por doações e trabalho voluntário, que empresta livros gratuitamente a moradores associados. Hoje o controle é feito em fichas de papel, o que causa perda de prazos, livros emprestados em duplicidade e dificuldade de saber o que existe no acervo. O sistema proposto organiza o acervo, os empréstimos, as devoluções e as multas. Há três tipos de usuário: o **leitor**, que se cadastra, pesquisa livros, reserva, renova e acompanha seus empréstimos; o **bibliotecário**, que cadastra livros e registra empréstimos, devoluções e pagamentos de multa no balcão; e o **administrador**, que gerencia as contas dos bibliotecários. O sistema conversa com dois sistemas externos: um **serviço de e-mail**, usado nos avisos de devolução, e uma **base externa de consulta por ISBN**, usada para preencher os dados dos livros cadastrados.

### 1.2 Tabela de requisitos

<!-- Mínimos: 10 RF, 6 RNF (mensuráveis, em pelo menos 4 categorias) e 5 RN. -->
<!-- RF: "O sistema deve permitir que [usuário] [ação]". RNF: sempre com medida. RN: vale mesmo sem o sistema. -->

#### Requisitos funcionais (RF)

| Código | Tipo | Descrição | Prioridade |
|---|---|---|---|
| RF01 | Funcional | O sistema deve permitir que o leitor se cadastre informando nome, documento, e-mail e telefone. | Alta |
| RF02 | Funcional | O sistema deve permitir que o leitor, o bibliotecário e o administrador entrem no sistema com e-mail e senha. | Alta |
| RF03 | Funcional | O sistema deve permitir que o leitor pesquise livros por título, autor, ISBN ou categoria e veja a disponibilidade dos exemplares. | Alta |
| RF04 | Funcional | O sistema deve permitir que o bibliotecário cadastre um livro a partir do ISBN, com os dados preenchidos por consulta a uma base externa. | Alta |
| RF05 | Funcional | O sistema deve permitir que o bibliotecário registre o empréstimo de um exemplar a um leitor. | Alta |
| RF06 | Funcional | O sistema deve permitir que o bibliotecário registre a devolução de um exemplar. | Alta |
| RF07 | Funcional | O sistema deve permitir que o leitor renove um empréstimo ativo. | Média |
| RF08 | Funcional | O sistema deve permitir que o leitor reserve um livro que está emprestado. | Média |
| RF09 | Funcional | O sistema deve permitir que o leitor consulte seus empréstimos, prazos de devolução e multas. | Média |
| RF10 | Funcional | O sistema deve permitir que o leitor receba por e-mail um aviso 2 dias antes do vencimento do empréstimo. | Média |
| RF11 | Funcional | O sistema deve permitir que o bibliotecário registre o pagamento de uma multa. | Média |
| RF12 | Funcional | O sistema deve permitir que o administrador gerencie as contas dos bibliotecários. | Média |

#### Requisitos não funcionais (RNF)

| Código | Tipo | Descrição | Prioridade |
|---|---|---|---|
| RNF01 | Desempenho | A pesquisa no acervo deve retornar resultados em até 2 segundos em 95% das requisições, com até 50 usuários simultâneos. | Alta |
| RNF02 | Segurança | Toda comunicação entre o cliente e o servidor deve usar HTTPS com TLS 1.2 ou superior em 100% das requisições. | Alta |
| RNF03 | Segurança | A conta de um usuário deve ser bloqueada por 15 minutos após 5 tentativas de login consecutivas com senha errada. | Média |
| RNF04 | Disponibilidade | O sistema deve ter disponibilidade mínima de 99% por mês no horário de funcionamento da biblioteca (8h às 20h). | Alta |
| RNF05 | Usabilidade | Um leitor novo deve conseguir concluir o cadastro em até 3 minutos, sem ajuda de outra pessoa. | Média |
| RNF06 | Compatibilidade | O sistema deve funcionar nas duas últimas versões do Chrome, Firefox, Edge e Safari, em telas a partir de 360 pixels de largura. | Alta |

#### Regras de negócio (RN)

| Código | Tipo | Descrição | Prioridade |
|---|---|---|---|
| RN01 | Regra de negócio | O prazo de empréstimo é de 14 dias corridos. | Alta |
| RN02 | Regra de negócio | O atraso na devolução gera multa de R$ 0,50 por dia, por item. | Alta |
| RN03 | Regra de negócio | Cada leitor pode ter no máximo 3 itens emprestados ao mesmo tempo. | Alta |
| RN04 | Regra de negócio | Leitor com multa pendente ou com item em atraso não pode retirar novos itens. | Média |
| RN05 | Regra de negócio | Cada empréstimo pode ser renovado uma única vez, por mais 7 dias, e somente se não houver reserva para o livro. | Média |
| RN06 | Regra de negócio | Menores de 18 anos só podem se associar com autorização de um responsável. | Média |

---

## Parte 2: Documento de arquitetura

### 2.1 Diagramas C4 (níveis 1 a 3)

<!-- Todo diagrama tem título e legenda. Os níveis não se misturam: o nível 1 não mostra banco de dados. -->

#### Nível 1: Contexto

```mermaid
---
title: C4 Nível 1 - Contexto do sistema da biblioteca
---
flowchart LR
    Leitor["Leitor"] -->|"pesquisa, reserva e renova"| Sis["Sistema da Biblioteca"]
    Bibliotecario["Bibliotecário"] -->|"registra empréstimos e devoluções"| Sis
    Admin["Administrador"] -->|"gerencia contas e relatórios"| Sis
    Sis -->|"envia aviso de devolução"| Email["Serviço de e-mail"]
    Sis -->|"consulta dados por ISBN"| Isbn["Base externa de ISBN"]
```

**Legenda:** caixas são pessoas ou sistemas; cada seta indica o que trafega entre eles.

#### Nível 2: Contêineres

```mermaid
---
title: C4 Nível 2 - Contêineres do sistema da biblioteca
---
flowchart LR
    Usuario["Usuários"] -->|"HTTPS"| Cliente["Aplicação cliente<br/>[tecnologia]"]
    Cliente -->|"HTTPS/JSON"| Api["Servidor / API<br/>[tecnologia]"]
    Api -->|"SQL"| Bd[("Banco de dados<br/>[tecnologia]")]
```

**Legenda:** cada contêiner mostra a tecnologia escolhida; cada seta mostra o protocolo.

#### Nível 3: Componentes do servidor

```mermaid
---
title: C4 Nível 3 - Componentes do servidor
---
flowchart TB
    subgraph Apresentacao["Camada de apresentação"]
        Controllers["Controladores da API"]
    end
    subgraph Negocio["Camada de negócio"]
        Auth["Autenticação"]
        Acervo["Acervo"]
        Emprestimos["Empréstimos"]
    end
    subgraph Dados["Camada de dados"]
        Repos["Acesso a dados"]
    end
    Controllers --> Auth
    Controllers --> Acervo
    Controllers --> Emprestimos
    Acervo --> Repos
    Emprestimos --> Repos
```

**Legenda:** os módulos estão agrupados nas três camadas do servidor.

---

### 2.2 Registros de decisão (ADRs)

<!-- 3 ADRs. Cada um com 2 alternativas reais, consequência negativa e um requisito apontado. -->

#### ADR-001: [Título em forma de decisão]

**Status:** proposta
**Data:** AAAA-MM-DD

**Contexto**
Que requisito (RF, RNF ou RN) exige esta decisão agora?

**Alternativas consideradas**
- **Alternativa A: [nome].** A favor: ... Contra: ...
- **Alternativa B: [nome].** A favor: ... Contra: ...

**Decisão**
Escolhi a alternativa [A], porque [razão ligada ao requisito].

**Consequências**
- Positivas: ...
- Negativas: ...

#### ADR-002: [Título em forma de decisão]

**Status:** proposta
**Data:** AAAA-MM-DD

**Contexto**

**Alternativas consideradas**
- **Alternativa A: [nome].** A favor: ... Contra: ...
- **Alternativa B: [nome].** A favor: ... Contra: ...

**Decisão**

**Consequências**
- Positivas: ...
- Negativas: ...

#### ADR-003: [Título em forma de decisão]

**Status:** proposta
**Data:** AAAA-MM-DD

**Contexto**

**Alternativas consideradas**
- **Alternativa A: [nome].** A favor: ... Contra: ...
- **Alternativa B: [nome].** A favor: ... Contra: ...

**Decisão**

**Consequências**
- Positivas: ...
- Negativas: ...

---

### 2.3 Comunicação cliente-servidor

#### Fluxo de requisição e resposta

<!-- Diagrama de sequência de um caso de uso completo: cliente → servidor → banco → resposta. -->

```mermaid
---
title: Sequência - [nome do caso de uso]
---
sequenceDiagram
    actor U as Usuário
    participant C as Cliente
    participant S as Servidor / API
    participant B as Banco de dados
    U->>C: [ação]
    C->>S: [requisição HTTPS]
    S->>B: [consulta SQL]
    B-->>S: [resultado]
    S-->>C: [resposta JSON]
    C-->>U: [resultado na tela]
```

**Legenda:** setas contínuas são pedidos; setas tracejadas são respostas.

#### Requisições HTTP

<!-- 4 requisições, com método, URL, status e JSON. Pelo menos uma com erro 4xx. Cada uma liga a um RF. -->

**Requisição 1: [nome]** (RF__)

- **Método e URL:** `GET /api/...`
- **Status da resposta:** `200 OK`

```json
{}
```

**Requisição 2: [nome]** (RF__)

- **Método e URL:** `POST /api/...`
- **Status da resposta:** `201 Created`

```json
{}
```

**Requisição 3: [nome]** (RF__)

- **Método e URL:** `PUT /api/...`
- **Status da resposta:** `200 OK`

```json
{}
```

**Requisição 4: [nome, caso de erro]** (RF__)

- **Método e URL:** `POST /api/...`
- **Status da resposta:** `4xx`

```json
{}
```

#### HTTPS

<!-- Que dados do sistema precisam de proteção e o que o TLS garante. -->

[Escrever aqui.]

---

### 2.4 Rastreabilidade

<!-- Nenhum RNF pode ficar sem decisão, e nenhuma requisição pode ficar sem RF. -->

#### RNF → decisão

| RNF | Decisão (ADR ou elemento do C4) |
|---|---|
| RNF01 | |
| RNF02 | |
| RNF03 | |
| RNF04 | |
| RNF05 | |
| RNF06 | |

#### Requisição HTTP → RF

| Requisição | RF |
|---|---|
| Requisição 1 | |
| Requisição 2 | |
| Requisição 3 | |
| Requisição 4 | |
