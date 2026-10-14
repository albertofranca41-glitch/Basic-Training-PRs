#Programação / #Web / [[HTTP]]

## Definição

**Cliente-servidor** é um modelo arquitetural de comunicação no qual um **cliente** inicia requisições por determinados recursos ou serviços, enquanto um **servidor** recebe essas requisições, processa-as e fornece as respostas correspondentes.

A comunicação ocorre por meio de uma rede e normalmente utiliza protocolos específicos para definir como as mensagens devem ser estruturadas e transmitidas.


```
Cliente ────── requisição ──────> Servidor

Cliente <────── resposta ──────── Servidor
```

### Conceitos-chave

#### Cliente

O **cliente** é o componente que inicia uma interação com um servidor para obter algum recurso ou executar alguma operação.

Um navegador, por exemplo, atua como cliente quando solicita uma página web:
```
	Navegador
	    │
	    │ requisição
	    ▼
	Servidor
```

Mas um navegador não é o único tipo de cliente. Um programa Java, uma aplicação mobile ou outra API também podem atuar como clientes.

#### Servidor

O **servidor** é o componente responsável por receber e processar as requisições dos clientes e disponibilizar recursos ou executar serviços.

Um servidor não precisa necessariamente ser uma máquina diferente do cliente. **Cliente e servidor são papéis desempenhados por componentes em uma comunicação.**

Uma mesma máquina pode inclusive atuar nos dois papéis.

## Comunicação

Uma comunicação cliente-servidor normalmente segue este ciclo:

```
1. Cliente inicia uma requisição
          ↓
2. Requisição é transmitida pela rede
          ↓
3. Servidor recebe a requisição
          ↓
4. Servidor processa a requisição
          ↓
5. Servidor produz uma resposta
          ↓
6. Resposta é transmitida ao cliente
```

No contexto web, o protocolo **HTTP** define a estrutura dessa comunicação.

Por exemplo:

```http
GET /produtos HTTP/1.1
Host: exemplo.com
```

O servidor pode responder:

```http
HTTP/1.1 200 OK

Content-Type: application/json
```

seguido dos dados solicitados.

---

## Exemplo prático

Quando você acessa:


```
https://github.com
```

o navegador atua como **cliente**.

O servidor responsável pelo site atua como **servidor**.

A comunicação pode ser representada de forma simplificada como:


```
Navegador
    │
    │ HTTP Request
    ▼
Servidor Web
    │
    │ processamento
    ▼
Aplicação
    │
    │ dados
    ▼
Banco de dados
    │
    ▼
Aplicação
    │
    ▼
Servidor Web
    │
    │ HTTP Response
    ▼
Navegador
```

Aqui já começamos a perceber que **"servidor" não significa necessariamente apenas uma máquina**. O servidor pode envolver uma infraestrutura composta por servidor web, aplicação, banco de dados, cache, serviços auxiliares etc.