#Programação / #Backend / #API


## Introdução

**API (Application Programming Interface)** é uma interface que define como diferentes componentes ou sistemas de software podem **interagir entre si**.

Em uma aplicação, uma API estabelece quais operações podem ser realizadas, quais dados podem ser enviados ou solicitados e como essas informações devem ser estruturadas. Dessa forma, o consumidor da API não precisa conhecer os detalhes internos da implementação do sistema para utilizá-lo.

Podemos pensar em uma API como um **contrato de comunicação** entre duas partes:

```
Cliente
   │
   │ Requisição
   ▼
┌───────────────┐
│      API      │
└───────────────┘
   │
   │ Comunicação com a aplicação
   ▼
Servidor / Sistema
```

O cliente conhece a **interface disponibilizada pela API**, enquanto os detalhes internos de como o servidor processa a solicitação permanecem abstraídos.

---

## APIs no Backend

No desenvolvimento backend, APIs são frequentemente utilizadas para permitir que diferentes partes de um sistema se comuniquem.

Por exemplo:

```
Frontend
   │
   │ API
   ▼
Backend
   │
   ├── Banco de dados
   ├── Serviços
   ├── Sistemas externos
   └── Filas / Mensageria
```

Um frontend pode utilizar uma API para:

- obter dados;
- criar recursos;
- modificar recursos;
- remover recursos;
- executar operações;
- autenticar usuários;
- comunicar-se com outros serviços.

A forma como essa comunicação é estruturada depende da abordagem utilizada para construir a API.

---

# Formas de construir APIs

Não existe uma única maneira de construir uma API. Entre as abordagens mais importantes estão:

### [[REST]]

**REST (Representational State Transfer)** é um **estilo arquitetural** para sistemas distribuídos. Ele organiza a interação em torno de recursos e utiliza um conjunto de restrições arquiteturais, como interface uniforme, comunicação stateless e identificação de recursos.

```
Cliente
   │
   │ GET /users/10
   ▼
API REST
   │
   ▼
Recurso User
```

É uma das abordagens mais utilizadas para APIs Web.

---

### [[RPC]]

**RPC (Remote Procedure Call)** é um modelo de comunicação no qual um cliente pode solicitar a execução de uma operação em outro processo ou sistema remoto como se estivesse realizando uma chamada de procedimento.

```
Cliente
   │
   │ sayHello()
   ▼
Servidor
   │
   └── executa sayHello()
```

Uma implementação moderna e bastante conhecida desse modelo é o **gRPC**, que define serviços por meio de métodos remotos e utiliza Protocol Buffers, por padrão, para descrever a interface e as mensagens.

---

### [[GraphQL]]

**GraphQL** é uma linguagem de consulta para APIs e uma especificação que permite definir, por meio de um **schema tipado**, quais dados e operações uma API disponibiliza. O cliente pode declarar quais dados deseja receber, e o servidor valida e executa essa consulta de acordo com o schema.

```
Cliente
   │
   │ query { user { name email } }
   ▼
GraphQL API
   │
   ├── valida a query
   ├── executa os resolvers
   └── retorna os campos solicitados
```

---

# Comparação conceitual

Uma forma inicial de diferenciar essas abordagens é observar **como o cliente expressa sua intenção**:

```
REST
→ "Quero interagir com este recurso."

RPC
→ "Quero executar esta operação."

GraphQL
→ "Quero estes dados desta forma."
```

Essa distinção não significa que cada abordagem esteja limitada exclusivamente a esse comportamento, mas fornece um bom modelo mental inicial para estudá-las.

---

# API não é necessariamente HTTP

É importante separar os conceitos de **API** e **protocolo de comunicação**.

Uma API pode utilizar diferentes mecanismos de comunicação. REST é frequentemente implementado sobre HTTP, GraphQL também é comumente utilizado sobre HTTP, enquanto sistemas RPC podem utilizar diferentes protocolos e transportes.

Portanto:

```
API
│
├── REST
│   └── frequentemente HTTP
│
├── GraphQL
│   └── frequentemente HTTP
│
└── RPC
    ├── gRPC
    ├── XML-RPC
    └── outros sistemas RPC
```

Assim, **HTTP, REST, RPC e GraphQL não são conceitos equivalentes**. Eles pertencem a níveis conceituais diferentes e podem se relacionar na construção de um sistema.

---

# Conceitos para estudar

A partir desta introdução, o estudo de APIs pode ser dividido em:

## Fundamentos

- [[API e Interface]]
- [[Cliente e Servidor]]
- [[Request e Response]]
- [[Protocolos de Comunicação]]
- [[HTTP]]

## Formas de construção

- [[REST]]
- [[RPC]]
- [[GraphQL]]

## Dados

- [[JSON]]
- [[XML]]
- [[Serialização]]
- [[Desserialização]]

## Segurança

- [[Autenticação]]
- [[Autorização]]
- [[OAuth 2.0]]
- [[JWT]]
- [[API Keys]]

## Arquitetura e desempenho

- [[Versionamento de APIs]]
- [[Caching]]
- [[Rate Limiting]]
- [[Paginação]]
- [[Idempotência]]
- [[Load Balancing]]


