## Introdução

**REST (Representational State Transfer)** é um **estilo arquitetural** que define um conjunto de princípios e restrições para organizar a comunicação entre sistemas distribuídos.

Em uma API RESTful, a comunicação é geralmente organizada em torno de **recursos**, que são identificados por URIs, e as operações sobre esses recursos são realizadas utilizando métodos HTTP padronizados, como `GET`, `POST`, `PUT` e `DELETE`.

A ideia central é proporcionar uma forma de comunicação **padronizada, previsível e uniforme** entre cliente e servidor.

> **REST organiza a comunicação em torno de recursos e utiliza uma interface uniforme para permitir que clientes interajam com esses recursos.**

---

## Recursos

Um dos conceitos fundamentais do REST é o **recurso**.

Um recurso representa alguma entidade ou informação que pode ser identificada pelo sistema.

Por exemplo:

```
/users
/products
/orders
```

Um recurso específico pode ser identificado por um identificador:

```
/users/10
/products/42
/orders/135
```

Nesse modelo, a URL não representa necessariamente uma operação. Ela identifica o **recurso** com o qual o cliente deseja interagir.

---

## Métodos HTTP

As operações realizadas sobre os recursos são normalmente expressas através dos métodos HTTP.

|Método|Operação comum|
|---|---|
|`GET`|Obter um recurso|
|`POST`|Criar um novo recurso|
|`PUT`|Substituir/atualizar um recurso|
|`PATCH`|Atualizar parcialmente um recurso|
|`DELETE`|Remover um recurso|

Por exemplo:

```http
GET /users/10
```

pode representar:

> "Obter o usuário identificado por 10."

Enquanto:

```http
DELETE /users/10
```

representa:

> "Remover o usuário identificado por 10."

Observe que a URL continua identificando o mesmo recurso. O que muda é a **operação solicitada**, representada pelo método HTTP.

---

## REST e HTTP

REST e HTTP não são a mesma coisa.

**HTTP** é um protocolo de comunicação.

**REST** é um estilo arquitetural.

Entretanto, HTTP possui características que se encaixam muito bem nas restrições do REST, por isso é extremamente comum encontrar APIs REST implementadas sobre HTTP.

Podemos visualizar a relação assim:

```
REST
 │
 │ estilo arquitetural
 ▼
API RESTful
 │
 │ frequentemente utiliza
 ▼
HTTP
 │
 ├── GET
 ├── POST
 ├── PUT
 ├── PATCH
 └── DELETE
```

---

## Interface uniforme

Um dos princípios mais importantes do REST é a ideia de uma **interface uniforme**.

Em vez de cada recurso possuir uma forma completamente diferente de interação, a API utiliza uma estrutura consistente.

Por exemplo:

```
GET    /users
POST   /users

GET    /users/10
PUT    /users/10
DELETE /users/10
```

O cliente aprende um conjunto relativamente pequeno de regras que pode utilizar para interagir com diferentes recursos.

---

## Stateless

Uma API REST deve seguir o princípio de **statelessness**.

Isso significa que cada requisição deve conter as informações necessárias para que o servidor consiga processá-la, sem depender de um estado de sessão armazenado no servidor entre requisições.

Por exemplo:

```
Request 1
Cliente ──────────────► Servidor

Request 2
Cliente ──────────────► Servidor

Request 3
Cliente ──────────────► Servidor
```

O servidor não deve precisar interpretar a terceira requisição apenas com base no que aconteceu nas duas anteriores.

Isso facilita aspectos como escalabilidade e distribuição das requisições entre diferentes servidores.

---

## Representação de recursos

O cliente normalmente não recebe o recurso interno exatamente como ele existe no servidor.

Ele recebe uma **representação** desse recurso.

Por exemplo:

```
Recurso interno
     │
     │ representação
     ▼
JSON

{
  "id": 10,
  "name": "Alberto"
}
```

O recurso pode existir internamente como um objeto Java, uma entidade de banco de dados ou outra estrutura, enquanto sua representação para o cliente pode ser um documento JSON.

---

# Estrutura de uma API RESTful

Podemos visualizar uma comunicação REST de maneira simplificada:

```
┌──────────┐
│  Cliente │
└────┬─────┘
     │
     │ HTTP Request
     │
     │ GET /users/10
     ▼
┌──────────────┐
│ API RESTful  │
└──────┬───────┘
       │
       │ processa a requisição
       ▼
┌──────────────┐
│   Sistema    │
│              │
│ Banco /      │
│ Serviços /   │
│ Regras       │
└──────┬───────┘
       │
       │ representação
       ▼
┌──────────────┐
│ HTTP Response│
│              │
│ JSON         │
└──────────────┘
```

---

# REST na prática

Considere uma API que disponibiliza usuários.

### Listar usuários

```http
GET /users
```

### Obter um usuário

```http
GET /users/10
```

### Criar um usuário

```http
POST /users
```

### Atualizar um usuário

```http
PUT /users/10
```

### Remover um usuário

```http
DELETE /users/10
```

A estrutura permanece consistente porque a API utiliza **recursos + uma interface uniforme de interação**.

---

# REST e outras abordagens

REST é apenas uma das maneiras de construir APIs.

- [[API]]
- [[RPC]]
- [[GraphQL]]

Uma maneira inicial de diferenciar as abordagens é:

```
REST
→ Interagir com recursos.

RPC
→ Executar operações remotas.

GraphQL
→ Solicitar exatamente os dados desejados através de uma
```