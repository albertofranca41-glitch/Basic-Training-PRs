#Programação / #Web / [[Internet]] / [[Cliente e Servidor]] / [[API]] / [[REST]]

## 1. HTTP - Hypertext Transfer Protocol

### Definição

**HTTP (Hypertext Transfer Protocol)** é um protocolo da camada de aplicação utilizado para realizar a transferência de *recursos* entre **clientes e servidores** em uma rede, definindo a estrutura das mensagens, os métodos de interação, os códigos de status e diversas regras que governam essa comunicação.

## 2. HTTP Request

Uma **HTTP request** é uma mensagem enviada pelo cliente ao servidor para solicitar um recurso ou realizar determinada operação.

Uma requisição HTTP possui vários componentes.

Um exemplo:

```http
GET /users/42 HTTP/1.1
Host: api.exemplo.com
Accept: application/json
Authorization: Bearer ...
```

Podemos dividi-la em:

```http
GET /users/42 HTTP/1.1
│      │
│      └── recurso solicitado
└──────── método

Headers
   ↓
Host: api.exemplo.com
Accept: application/json
Authorization: ...
```

Dependendo do método e da operação, a requisição também pode possuir um **body**.

---

## 3. Método HTTP

O **método HTTP** indica a intenção da requisição.

Alguns dos principais são:

| Método   | Semântica geral                                                   |
| -------- | ----------------------------------------------------------------- |
| `GET`    | Obter uma representação de um recurso                             |
| `POST`   | Submeter dados ou solicitar a criação/processamento de um recurso |
| `PUT`    | Substituir uma representação de recurso                           |
| `PATCH`  | Modificar parcialmente um recurso                                 |
| `DELETE` | Remover um recurso                                                |

Por exemplo:

```http
GET /users/42
```

pode significar:

> "Quero obter o usuário 42."

Enquanto:

```http
DELETE /users/42
```

pode significar:

> "Quero remover o usuário 42."

O método, portanto, carrega **semântica**.

---

## 4. URL

Na requisição:
`
``` http
GET /users/42
```

temos um identificador do recurso solicitado.

Quando utilizamos uma URL completa:

>https://api.exemplo.com/users/42

podemos dividi-la em componentes:

```
https://api.exemplo.com/users/42
│       │              │
│       │              └── caminho
│       └────────────────── autoridade/host
└────────────────────────── esquema
```

O navegador utiliza essa informação para determinar **qual recurso deseja acessar e para qual servidor deve enviar a requisição**.

---

## 5. Headers

Os **headers HTTP** são campos de metadados associados à requisição ou à resposta.

Por exemplo:

```http
Host: api.exemplo.com
Accept: application/json
Content-Type: application/json
```

Eles fornecem informações adicionais sobre a comunicação.

 `Content-Type`

Indica o formato do conteúdo da mensagem.

Por exemplo:

```
Content-Type: application/json
```

significa que o corpo da mensagem contém JSON.

`Accept`

Indica quais formatos o cliente aceita receber.

```
Accept: application/json
```

significa, aproximadamente:

> "Prefiro receber a resposta em JSON."

`Authorization`

Pode transportar informações utilizadas para autenticação/autorização:`
```
Authorization: Bearer<>
```

## 6. Body

Uma requisição HTTP também pode possuir um **body**, utilizado para transportar dados.

Por exemplo, ao criar um usuário:

```http
POST /users HTTP/1.1
Host: api.exemplo.com
Content-Type: application/json

{
    "name": "Alberto",
    "email": "alberto@example.com"
}
```

Aqui temos:

```
Request
│
├── Method: POST
├── Path: /users
├── Headers
│
└── Body
      │
      └── JSON
```

O body não é obrigatório em todas as requisições.

Um `GET`, por exemplo, normalmente não precisa de body para indicar o recurso que deseja obter.

---

# 7. HTTP Response

Depois de receber uma requisição, o servidor produz uma **HTTP response**.

Exemplo:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 42,
    "name": "Alberto"
}
```

Uma resposta possui principalmente:

```
Status
Headers
Body
```

Por exemplo:

```http
HTTP/1.1 200 OK
│        │
│        └── motivo textual
└─────────── código de status
```

---

# 8. Status Codes

O **status code** informa o resultado do processamento da requisição.

Eles são divididos em categorias:

```
1xx → informações

2xx → sucesso

3xx → redirecionamento

4xx → erro relacionado à requisição do cliente

5xx → erro no servidor
```

Alguns códigos importantes:

 `200 OK`

A requisição foi processada com sucesso.

```http
HTTP/1.1 200 OK
```

`201 Created`

Um recurso foi criado com sucesso.

Comum em operações de criação:

```http
POST /users
```

Resposta:

```http
HTTP/1.1 201 Created
```

 `204 No Content`

A requisição foi processada com sucesso, mas não há conteúdo para retornar.

`400 Bad Request`

A requisição possui algum problema que impede seu processamento.

`401 Unauthorized`

A requisição exige autenticação ou as credenciais fornecidas não são suficientes.

`403 Forbidden`

O servidor entendeu a requisição, mas não permite aquela operação.

`404 Not Found`

O recurso solicitado não foi encontrado.

`500 Internal Server Error`

Ocorreu um erro no processamento do lado do servidor.

---

## 9. Um exemplo completo

Imagine uma aplicação que possui:

```http
GET https://api.exemplo.com/products/10
```

O processo pode ser entendido assim.

### 1. DNS

O domínio:

```
api.exemplo.com
```

é resolvido para um endereço IP.

```
api.exemplo.com
       ↓
      DNS
       ↓
203.0.113.50
```

### 2. Conexão

O cliente estabelece a comunicação necessária com o servidor.

## 3. HTTP Request

O cliente envia:

```http
GET /products/10 HTTP/1.1
Host: api.exemplo.com
Accept: application/json
```

### 4. Servidor processa

O servidor pode executar:

```
Request
   ↓
Backend
   ↓
Banco de dados
   ↓
Produto 10
```

### 5. HTTP Response

O servidor retorna:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": 10,
    "name": "Notebook",
    "price": 3500
}
```

### 6. Cliente utiliza a resposta

Uma aplicação frontend pode utilizar esse JSON para construir uma interface:

```
Notebook
R$ 3.500
```

---

## 10. HTTP e APIs

Aqui começa a conexão direta com **REST**, que você já estudou.

Uma API pode utilizar HTTP como mecanismo de comunicação:

```
Cliente
   │
   │ HTTP
   ▼
  API
   │
   ▼
Backend
```

Por exemplo:
```http
GET /users/10
```

```http
POST /users
```

```http
PUT /users/10
```

```http
DELETE /users/10
```

O HTTP fornece os **métodos, mensagens, headers, status codes etc.**

Uma arquitetura como REST utiliza essas capacidades do HTTP para estruturar a comunicação com recursos.

Portanto, **REST e HTTP não são sinônimos**.

Podemos pensar:

```
HTTP
│
├── métodos
├── headers
├── status codes
├── request
├── response
└── outras regras do protocolo
       │
       ▼
	  REST
	   │
	   └── utiliza HTTP seguindo determinados princípios
	       para estruturar APIs orientadas a recursos
```

---

## 11. HTTP e HTTPS

Você provavelmente já percebeu que normalmente acessamos:

>https://google.com

e não:

>http://google.com

O `S` representa **Secure**.

O **HTTPS é HTTP transportado através de uma conexão protegida por TLS**.

O TLS fornece propriedades como:

- confidencialidade;
- integridade;
- autenticação do servidor por certificados.

Assim, quando você envia:

```http
POST /login

{
    "username": "...",
    "password": "..."
}
```

a comunicação pode ser protegida contra interceptação e alteração por terceiros na rede.

---

## 12. HTTP é stateless

Uma característica importante do HTTP é que ele é, em sua essência, **stateless**.

Isso significa que cada requisição HTTP é tratada como uma mensagem independente e o protocolo não exige que o servidor mantenha um estado de sessão entre requisições.

Por exemplo:

```
Request 1:
GET /products/10

Request 2:
GET /products/20

Request 3:
GET /products/30
```

O protocolo HTTP não determina que o servidor precise "lembrar" automaticamente que as três requisições vieram do mesmo usuário.

Quando uma aplicação precisa manter estado, ela utiliza mecanismos adicionais, como:

- Cookies
- Sessões
- Tokens
- Banco de dados
- Cache

Esse conceito será importante quando você estudar **autenticação e APIs**.

---

# 15. HTTP não define a lógica da aplicação

Outro ponto fundamental:

HTTP define **como as mensagens são comunicadas**, mas não define o que sua aplicação deve fazer internamente.

Por exemplo:

```http
POST /orders
```

O HTTP não sabe o que significa "criar pedido".

Quem decide isso é a aplicação.

O backend pode fazer:

```
POST /orders
      ↓
Validar usuário
      ↓
Validar produtos
      ↓
Calcular preço
      ↓
Verificar estoque
      ↓
Salvar pedido
      ↓
Retornar resposta
```

Portanto:

> **HTTP fornece o protocolo de comunicação; a aplicação fornece a lógica de negócio.**

---

# 16. O que você deve guardar sobre HTTP

Se fosse necessário condensar todo esse conceito em alguns pontos:

> **HTTP é um protocolo da camada de aplicação utilizado para comunicação entre clientes e servidores. Ele define uma estrutura de requisição e resposta, incluindo métodos, URLs, headers, corpos de mensagem e códigos de status.**