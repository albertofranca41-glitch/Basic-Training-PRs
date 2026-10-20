# Guia: Atividade de JavaScript com a Headless Delivery API (CRUD de Blog Postings)

Este guia ensina, do zero, a fazer a atividade em que você escreve um código **JavaScript** que conversa com o **Liferay DXP** por meio de **requisições HTTP** (criar, listar, buscar, atualizar e apagar blogs).

Ele está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Conceitos necessários: cliente/servidor, HTTP, API, REST, JSON, autenticação, Promises, `async/await` e `fetch` |
| **Etapa 2 – O código** | O código completo da atividade, comentado, e explicado parte por parte |

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 Cliente e servidor](#11-cliente-e-servidor)
  - [1.2 HTTP: requisição e resposta](#12-http-requisição-e-resposta)
  - [1.3 API e REST](#13-api-e-rest)
  - [1.4 JSON](#14-json)
  - [1.5 Autenticação (Basic Auth)](#15-autenticação-basic-auth)
  - [1.6 JavaScript assíncrono: Promises e async/await](#16-javascript-assíncrono-promises-e-asyncawait)
  - [1.7 fetch](#17-fetch)
  - [1.8 A Headless Delivery API do Liferay](#18-a-headless-delivery-api-do-liferay)
  - [1.9 Pré-requisitos do ambiente](#19-pré-requisitos-do-ambiente)
- [Recursos adicionais](#recursos-adicionais)    
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 Código completo comentado](#22-código-completo-comentado)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
  - [2.4 Como executar e testar](#24-como-executar-e-testar)
  - [2.5 Pontos de melhoria](#25-pontos-de-melhoria)
  - [2.6 Erros comuns](#26-erros-comuns)
---

# Etapa 1 – Teoria

Antes do código, vamos entender o "mundo" em que ele vive. Se você já conhece algum tópico, pode pular.

## 1.1 Cliente e servidor

Quase tudo na internet funciona no modelo **cliente-servidor**:

- **Cliente:** quem **pede** algo (seu navegador, um aplicativo, um script JavaScript).
- **Servidor:** quem **responde** (um computador que guarda dados e regras de negócio).

```
┌──────────┐     requisição  ──►     ┌──────────┐
│ CLIENTE  │                         │ SERVIDOR │
│ (seu JS) │     ◄──  resposta       │ (Liferay)│
└──────────┘                         └──────────┘
```

Na atividade:

- O **servidor** é o **Liferay DXP**, rodando na sua máquina em `http://localhost:8080`.
- O **cliente** é o **seu código JavaScript**.

> `localhost` significa "este próprio computador". `8080` é a **porta** em que o Liferay "escuta" pedidos.

## 1.2 HTTP: requisição e resposta

**HTTP** (*HyperText Transfer Protocol*) é a "língua" que cliente e servidor usam para conversar. Cada conversa é um par: **uma requisição** e **uma resposta**.

### Anatomia de uma requisição

| Parte | O que é | Exemplo na atividade |
|---|---|---|
| **URL** | Endereço do recurso | `http://localhost:8080/o/headless-delivery/v1.0/sites/20117/blog-postings` |
| **Método (verbo)** | O que você quer fazer | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| **Cabeçalhos (headers)** | Informações extras sobre o pedido | `Authorization`, `Content-Type` |
| **Corpo (body)** | Os dados enviados (opcional) | `{"headline": "...", "articleBody": "..."}` |

### Métodos HTTP e o CRUD

**CRUD** = **C**reate, **R**ead, **U**pdate, **D**elete (criar, ler, atualizar, apagar). Cada operação tem um método HTTP correspondente:

| Operação | Método | Significado | Tem corpo? |
|---|---|---|---|
| Criar | `POST` | Cria um novo recurso | Sim |
| Ler (lista ou um) | `GET` | Busca dados, sem alterar nada | Não |
| Atualizar (completo) | `PUT` | **Substitui** o recurso inteiro | Sim |
| Atualizar (parcial) | `PATCH` | Altera **só os campos enviados** | Sim |
| Apagar | `DELETE` | Remove o recurso | Não |

> **PUT × PATCH:** imagine um blog com título e corpo. Com `PUT` você deve enviar **os dois** (o recurso é substituído). Com `PATCH` pode enviar **só o título** e o resto continua igual.

### Anatomia de uma resposta

| Parte | O que é |
|---|---|
| **Código de status** | Número que diz se deu certo ou errado |
| **Cabeçalhos** | Informações sobre a resposta |
| **Corpo** | Os dados devolvidos (normalmente JSON) |

### Códigos de status mais comuns

| Faixa | Significado | Exemplos |
|---|---|---|
| **2xx** | Sucesso | `200 OK`, `201 Created`, `204 No Content` (deu certo, mas **sem corpo**) |
| **3xx** | Redirecionamento | `301`, `302` |
| **4xx** | Erro do **cliente** (você pediu errado) | `400 Bad Request`, `401 Unauthorized` (credenciais erradas), `403 Forbidden`, `404 Not Found` |
| **5xx** | Erro do **servidor** | `500 Internal Server Error` |

## 1.3 API e REST

**API** (*Application Programming Interface*) é um "cardápio" de operações que um sistema oferece para outros programas usarem. Em vez de clicar em botões numa tela, o seu código chama endereços da API.

**REST** (*Representational State Transfer*) é um **estilo** de organizar APIs sobre HTTP. Princípios principais:

1. **Tudo é um recurso**, identificado por uma URL (um blog, um usuário, um produto).
2. Você usa os **métodos HTTP** para dizer o que fazer com o recurso (`GET`, `POST`...).
3. Os dados trafegam em um formato padrão, normalmente **JSON**.
4. O servidor é ***stateless***: cada requisição traz tudo o que o servidor precisa saber (inclusive a autenticação); ele não "lembra" da requisição anterior.

Exemplo de como as URLs REST costumam ser organizadas:

| Ação | Método + URL |
|---|---|
| Listar blogs | `GET /blog-postings` |
| Buscar o blog 42 | `GET /blog-postings/42` |
| Criar um blog | `POST /blog-postings` |
| Atualizar o blog 42 | `PUT /blog-postings/42` |
| Apagar o blog 42 | `DELETE /blog-postings/42` |

Repare: a URL fala do **recurso** (substantivo); o **método** é o verbo.

## 1.4 JSON

**JSON** (*JavaScript Object Notation*) é o formato de texto usado para trocar dados:

```json
{
  "headline": "Meu primeiro blog",
  "articleBody": "Olá, mundo!"
}
```

Duas funções do JavaScript fazem a ponte entre objeto e texto:

| Função | O que faz | Quando usar |
|---|---|---|
| `JSON.stringify(objeto)` | Objeto JS → **texto** JSON | Ao **enviar** dados (corpo da requisição) |
| `JSON.parse(texto)` | Texto JSON → objeto JS | Ao ler texto (o `res.json()` do `fetch` já faz isso por você) |

> Na rede só trafega **texto**. Por isso o objeto precisa virar texto (`stringify`) antes de ser enviado e voltar a ser objeto ao ser recebido.

## 1.5 Autenticação (Basic Auth)

O servidor precisa saber **quem** está pedindo. A atividade usa **Basic Authentication**: você envia o usuário e a senha no cabeçalho `Authorization`.

O formato é:

```
Authorization: Basic <base64 de "email:senha">
```

Em JavaScript:

```js
"Basic " + btoa("test@liferay.com:learn")
```

- `btoa()` converte um texto para **Base64**.
- ⚠️ **Base64 não é criptografia!** Qualquer pessoa consegue decodificar. Por isso Basic Auth só é aceitável em ambiente **local de treinamento** ou sobre **HTTPS**. Em aplicações reais, usa-se OAuth 2.0 ou outros mecanismos.

## 1.6 JavaScript assíncrono: Promises e async/await

### O problema

Pedir algo a um servidor **demora** (milissegundos ou segundos). Se o JavaScript ficasse parado esperando, a página travaria. Então operações lentas são **assíncronas**: o programa pede, segue a vida e é avisado quando a resposta chegar.

### Promise

Uma **Promise** (promessa) é um objeto que representa **um resultado que ainda não existe, mas vai existir**. Ela tem três estados:

| Estado | Significado |
|---|---|
| **pending** | Ainda esperando |
| **fulfilled** | Deu certo; tem um valor |
| **rejected** | Deu errado; tem um erro |

Forma clássica de usar (com `.then` / `.catch`):

```js
fetch(url)
  .then((res) => res.json())
  .then((dados) => console.log(dados))
  .catch((erro) => console.error(erro));
```

### async / await

`async/await` é uma forma **mais legível** de escrever o mesmo, parecendo código "em sequência":

```js
const carregar = async () => {          // "async" marca a função como assíncrona
  try {
    const res = await fetch(url);       // "await" espera a Promise resolver
    const dados = await res.json();     // espera de novo
    console.log(dados);
  } catch (erro) {                      // qualquer erro cai aqui
    console.error(erro);
  }
};
```

Regras de ouro:

1. `await` só pode ser usado **dentro de uma função `async`** (ou no topo de módulos ES).
2. Uma função `async` **sempre devolve uma Promise**, mesmo que você faça `return 5`.
3. `try/catch` captura erros de qualquer `await` dentro do bloco.
4. `await` **pausa só aquela função**, não o programa inteiro.

## 1.7 fetch

`fetch(url, opções)` é a função nativa do JavaScript para fazer requisições HTTP. Devolve uma **Promise** que resolve para um objeto **Response**.

```js
const res = await fetch("http://localhost:8080/...", {
  method: "POST",                              // verbo HTTP
  headers: { "Content-Type": "application/json" }, // cabeçalhos
  body: JSON.stringify({ chave: "valor" }),    // corpo (texto)
});
```

O objeto `Response` (`res`) tem:

| Propriedade/método | O que é |
|---|---|
| `res.ok` | `true` se o status for 200–299 |
| `res.status` | Número do status (200, 404...) |
| `res.json()` | **Promise** que lê o corpo e converte para objeto JS |
| `res.text()` | Promise que lê o corpo como texto |

### ⚠️ A pegadinha mais importante do `fetch`

O `fetch` **só rejeita a Promise em falha de rede** (servidor desligado, sem internet). Se o servidor responder **404 ou 500, o `fetch` considera que deu certo!** Por isso o código **sempre** verifica `res.ok`:

```js
if (!res.ok) {
  throw new Error(`${res.status}`);   // transforma erro HTTP em exceção → vai para o catch
}
```

## 1.8 A Headless Delivery API do Liferay

O Liferay expõe várias APIs REST. A **Headless Delivery** dá acesso a conteúdos: blogs, documentos, artigos da web, etc.

Padrão das URLs:

```
http://localhost:8080/o/headless-delivery/v1.0/<recurso>
```

| Parte | Significado |
|---|---|
| `/o/` | Prefixo das APIs/módulos do Liferay |
| `headless-delivery` | Nome da API |
| `v1.0` | Versão da API |
| `sites/20117/blog-postings` | Recurso: blogs do site de ID `20117` |

Endpoints usados na atividade:

| Operação | Método | URL |
|---|---|---|
| Criar blog | `POST` | `/sites/{siteId}/blog-postings` |
| Listar blogs | `GET` | `/sites/{siteId}/blog-postings` |
| Buscar um blog | `GET` | `/blog-postings/{id}` |
| Atualizar (completo) | `PUT` | `/blog-postings/{id}` |
| Atualizar (parcial) | `PATCH` | `/blog-postings/{id}` |
| Apagar | `DELETE` | `/blog-postings/{id}` |

Campos principais de um blog:

| Campo | Descrição |
|---|---|
| `headline` | Título (obrigatório) |
| `articleBody` | Conteúdo (obrigatório) |
| `id` | Identificador, gerado pelo servidor |

A listagem devolve um objeto **paginado**:

```json
{
  "items": [ { "id": 1, "headline": "...", "articleBody": "..." } ],
  "page": 1,
  "pageSize": 20,
  "totalCount": 1,
  "lastPage": 1
}
```

> **Site ID:** cada site do Liferay tem um ID numérico (na atividade, `20117`; **o seu pode ser diferente**). Você o encontra nas configurações do site (o local exato varia um pouco conforme a versão). Outra opção é o **API Explorer** do Liferay, que costuma estar em `http://localhost:8080/o/api` e permite testar a API pelo navegador, mostrando as URLs e os formatos.

## 1.9 Pré-requisitos do ambiente

| Ferramenta | Para quê |
|---|---|
| **Liferay DXP rodando** em `localhost:8080` | O servidor que a API consulta |
| **Navegador moderno** | Abrir o HTML de teste e executar o JavaScript no contexto da página |
| **Git** e conta no **GitHub** | Versionar e enviar o PR |
| Um editor (ex.: VS Code) | Escrever o código |

Verifique:

```bash
git --version
```

## Recursos adicionais

Os arquivos abaixo aprofundam os conceitos apresentados nesta etapa. Leia-os como complemento quando quiser explorar cada assunto com mais detalhes:

- [Cliente e servidor](../theory/client-server.md): explica os papéis de cliente e servidor e como eles trocam requisições e respostas.
- [HTTP](../theory/http.md): detalha métodos, cabeçalhos, corpo das mensagens e códigos de status.
- [API](../theory/api.md): apresenta APIs como contratos de comunicação entre sistemas e como são usadas no backend.
- [REST](../theory/rest.md): aprofunda recursos, URLs e métodos HTTP em APIs REST.
- [JSON](../theory/json.md): Aprofunda o conceito de JSON.
- [API Explorer do seu ambiente](http://localhost:8080/o/api): confira os endpoints, schemas e operações presentes na versão instalada (Funciona somente se o container Liferay estiver rodando e se você fez o login corretamente da sua conta `test@liferay.com`).

---

# Etapa 2 – O código

## 2.1 Visão geral

O arquivo define **duas URLs**, **duas funções de cabeçalho** e **seis funções de requisição** (uma por operação):

```
Configuração:    urlBlogPostings, urlBlogPostingsById, email, senha
Cabeçalhos:      authHeader(), jsonHeaders()
Operações CRUD:  postBlog()          → POST   (Create)
                 getAllBlog()        → GET    (Read - lista)
                 getSingleBlog(id)   → GET    (Read - um)
                 updateSingleBlog(id)→ PUT    (Update completo)
                 patchBlog(id)       → PATCH  (Update parcial)
                 deleteBlog(id)      → DELETE (Delete)
```

Todas as funções seguem o **mesmo padrão**:

```
1. try {
2.   Faz a requisição com fetch (await)
3.   Verifica res.ok → se falhou, lança erro
4.   (se houver corpo) lê o JSON com res.json() (await)
5.   Usa/mostra o resultado
6. } catch (err) {
7.   Mostra o erro
8. }
```

Entender esse padrão uma vez é entender as seis funções.

## 2.2 Código completo comentado


```javascript

// Author: Alberto

// =====================================================================
// 1. CONFIGURAÇÃO
// =====================================================================

// URL da COLEÇÃO de blogs do site 20117.
// Usada para LISTAR (GET) e CRIAR (POST).
const urlBlogPostings =
  "http://localhost:8080/o/headless-delivery/v1.0/sites/20117/blog-postings";

// URL base para trabalhar com UM blog específico.
// O id é colocado no final na hora da chamada: `${urlBlogPostingsById}/${id}`
const urlBlogPostingsById =
  "http://localhost:8080/o/headless-delivery/v1.0/blog-postings";

// Credenciais do ambiente LOCAL de treinamento (nunca use credenciais reais aqui).
const email = "test@liferay.com";
const senha = "learn";

// =====================================================================
// 2. CABEÇALHOS (HEADERS)
// =====================================================================

// Devolve o cabeçalho de autenticação Basic.
// btoa() converte "email:senha" para Base64.
// É uma FUNÇÃO (e não um objeto fixo) para gerar um objeto novo a cada uso.
const authHeader = () => ({
  Authorization: "Basic " + btoa(`${email}:${senha}`),
});

// Cabeçalhos para requisições que ENVIAM JSON:
// - "...authHeader()" copia (spread) o Authorization para dentro do novo objeto
// - "Content-Type" avisa ao servidor que o corpo está em formato JSON
const jsonHeaders = () => ({
  ...authHeader(),
  "Content-Type": "application/json",
});

// =====================================================================
// 3. CREATE (POST) - cria um blog e devolve o id dele
// =====================================================================
const postBlog = async () => {
  try {
    const res = await fetch(urlBlogPostings, {
      method: "POST",
      headers: jsonHeaders(),
      // JSON.stringify converte o objeto JavaScript em texto JSON,
      // porque o corpo da requisição precisa ser texto.
      body: JSON.stringify({
        headline: "Test Blog Entry from REST Services",
        articleBody:
          "This article was posted via REST services provided by Liferay DXP.",
      }),
    });

    // fetch NÃO lança erro em 404/500; por isso checamos res.ok manualmente.
    // Se houve problema, lançamos um erro e o fluxo pula para o bloco catch.
    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    // O primeiro await (no fetch) espera a RESPOSTA do servidor.
    // O segundo await (res.json()) espera a LEITURA e conversão do corpo
    // (texto JSON) para um objeto JavaScript.
    // "data" passa a ser um objeto cujos valores acessamos pelas chaves.
    const data = await res.json();
    console.log(data);

    // O servidor gera o id do novo blog. Devolvemos para usar nas
    // próximas operações (buscar, atualizar, apagar).
    return data.id;
  } catch (err) {
    console.error(err);
    return null; // sinaliza que a criação falhou
  }
};

// =====================================================================
// 4. READ (GET) - lista todos os blogs do site
// =====================================================================
const getAllBlog = async () => {
  try {
    const res = await fetch(urlBlogPostings, {
      method: "GET",
      headers: jsonHeaders(),
      // GET não tem body: não há nada a enviar, só pedimos dados.
    });

    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    // A resposta é um objeto paginado: os blogs estão em data.items
    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
};

// =====================================================================
// 5. UPDATE (PUT) - substitui o blog por completo
// =====================================================================
const updateSingleBlog = async (id) => {
  try {
    const res = await fetch(`${urlBlogPostingsById}/${id}`, {
      method: "PUT",
      headers: jsonHeaders(),
      // PUT substitui o recurso: envie TODOS os campos obrigatórios
      // (headline e articleBody).
      body: JSON.stringify({
        headline: "Updated Blog from REST Services",
        articleBody:
          "This article was updated via REST services provided by Liferay DXP.",
      }),
    });

    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    // 204 = "No Content": deu certo, mas NÃO há corpo na resposta.
    // Chamar res.json() sem corpo causaria erro, então tratamos antes.
    if (res.status === 204) {
      console.log("Blog updated.");
      return;
    }

    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
};

// =====================================================================
// 6. READ (GET) - busca UM blog pelo id
// =====================================================================
const getSingleBlog = async (id) => {
  try {
    const res = await fetch(`${urlBlogPostingsById}/${id}`, {
      method: "GET",
      headers: jsonHeaders(),
    });

    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    // Sem corpo? Não tente ler JSON.
    if (res.status === 204) {
      console.log("No content.");
      return null;
    }

    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
};

// =====================================================================
// 7. UPDATE PARCIAL (PATCH) - altera só os campos enviados
// =====================================================================
const patchBlog = async (id) => {
  try {
    const res = await fetch(`${urlBlogPostingsById}/${id}`, {
      method: "PATCH",
      headers: jsonHeaders(),
      // Só o título é enviado; articleBody permanece como estava.
      body: JSON.stringify({
        headline: "Patched Blog from REST Services",
      }),
    });

    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    if (res.status === 204) {
      console.log("No content.");
      return null;
    }

    const data = await res.json();
    console.log(data);
  } catch (err) {
    console.error(err);
  }
};

// =====================================================================
// 8. DELETE - apaga um blog
// =====================================================================
const deleteBlog = async (id) => {
  try {
    const res = await fetch(`${urlBlogPostingsById}/${id}`, {
      method: "DELETE",
      // Aqui usamos só authHeader(): não enviamos corpo, então
      // não precisamos do Content-Type.
      headers: authHeader(),
    });

    if (!res.ok) {
      throw new Error(`${res.status}`);
    }

    // Normalmente o servidor responde 204 (sem corpo), por isso
    // NÃO chamamos res.json() aqui.
    console.log("Blog deleted");
  } catch (err) {
    console.error(err);
  }
};
```

## 2.3 Explicação por partes

### Parte 1 – Constantes de configuração

```js
const urlBlogPostings = ".../sites/20117/blog-postings";
const urlBlogPostingsById = ".../blog-postings";
```

Duas URLs porque a API tem **dois níveis**:

| Constante | Usada para | Por quê |
|---|---|---|
| `urlBlogPostings` | Listar e criar | Operações sobre a **coleção**, dentro de um **site** |
| `urlBlogPostingsById` | Buscar, atualizar, apagar | Operações sobre **um blog**; o `id` já o identifica, então o site não é necessário |

> Usamos `const` porque os valores não serão reatribuídos.

### Parte 2 – Cabeçalhos como funções

```js
const authHeader = () => ({ Authorization: "Basic " + btoa(`${email}:${senha}`) });
```

- `() => ({ ... })` é uma **arrow function** que devolve um objeto. Os **parênteses** ao redor das chaves são necessários; sem eles o JavaScript acharia que `{}` é o corpo da função, não um objeto.
- `` `${email}:${senha}` `` é uma **template string**: monta o texto `test@liferay.com:learn`.
- `btoa(...)` codifica em Base64.

```js
const jsonHeaders = () => ({ ...authHeader(), "Content-Type": "application/json" });
```

- `...authHeader()` é o **spread**: "despeja" as propriedades do objeto devolvido dentro do novo objeto.
- `Content-Type: application/json` informa ao servidor o formato do corpo.
- O nome com hífen precisa estar **entre aspas**.

Resultado de `jsonHeaders()`:

```js
{
  Authorization: "Basic dGVzdEBsaWZlcmF5LmNvbTpsZWFybg==",
  "Content-Type": "application/json"
}
```

### Parte 3 – O padrão `try / fetch / res.ok / res.json / catch`

Passo a passo de qualquer função (usando `postBlog` como exemplo):

| # | Linha | O que acontece |
|---|---|---|
| 1 | `const postBlog = async () => {` | Declara uma função **assíncrona**, o que permite usar `await` dentro dela |
| 2 | `try {` | Começa um bloco "tente fazer isto"; se algo lançar erro, o fluxo vai para o `catch` |
| 3 | `const res = await fetch(url, {...})` | Envia a requisição e **espera** o servidor responder; `res` é o objeto Response |
| 4 | `method: "POST"` | Define o verbo HTTP |
| 5 | `headers: jsonHeaders()` | Anexa autenticação e tipo de conteúdo |
| 6 | `body: JSON.stringify({...})` | Converte o objeto em texto JSON para enviar |
| 7 | `if (!res.ok) throw new Error(...)` | Se o status não for 2xx, **lança** um erro manualmente |
| 8 | `const data = await res.json()` | Lê o corpo da resposta e converte em objeto JS |
| 9 | `console.log(data)` | Mostra o resultado no console do navegador |
| 10 | `return data.id` | Devolve o id do blog criado (uma `async` function devolve isso dentro de uma Promise) |
| 11 | `} catch (err) {` | Recebe qualquer erro (rede, status ruim, JSON inválido) |
| 12 | `console.error(err); return null;` | Registra o erro e devolve `null` para quem chamou saber que falhou |

### Parte 4 – Diferenças entre as seis operações

| Função | Método | URL | Corpo? | Trata 204? | Observação |
|---|---|---|---|---|---|
| `postBlog` | POST | coleção | Sim (título + corpo) | Não | Devolve `data.id` |
| `getAllBlog` | GET | coleção | Não | Não | Lista fica em `data.items` |
| `getSingleBlog` | GET | `/{id}` | Não | Sim | Busca um blog |
| `updateSingleBlog` | PUT | `/{id}` | Sim (todos os campos) | Sim | Substitui o recurso |
| `patchBlog` | PATCH | `/{id}` | Sim (só o título) | Sim | Altera só o enviado |
| `deleteBlog` | DELETE | `/{id}` | Não | Não lê corpo | Usa só `authHeader()` |

### Parte 5 – Por que tratar o status `204`?

`204 No Content` significa **sucesso sem corpo**. Se o código tentasse `await res.json()` nessa resposta, o JavaScript lançaria um erro parecido com `Unexpected end of JSON input`, já que não há JSON para ler. Por isso:

```js
if (res.status === 204) {
  console.log("Blog updated.");
  return;                 // encerra a função antes do res.json()
}
```

### Parte 6 – Template string na URL

```js
`${urlBlogPostingsById}/${id}`
```

Se `id` for `42`, vira `http://localhost:8080/o/headless-delivery/v1.0/blog-postings/42`. É assim que a mesma URL base serve para qualquer blog.

## 2.4 Como executar e testar

O arquivo só **define** funções; nada roda até você **chamá-las**. Para executar o ciclo completo em sequência, pode acrescentar ao final do arquivo uma função principal como esta:

```javascript
// =====================================================================
// 9. EXECUÇÃO (teste do ciclo completo)
// =====================================================================
const main = async () => {
  const id = await postBlog();          // 1) cria e guarda o id
  if (!id) {
    console.log("Não foi possível criar o blog. Encerrando.");
    return;
  }

  await getAllBlog();                   // 2) lista todos
  await getSingleBlog(id);              // 3) busca o que criamos
  await updateSingleBlog(id);           // 4) substitui por completo (PUT)
  await getSingleBlog(id);              // 5) confere o resultado
  await patchBlog(id);                  // 6) altera só o título (PATCH)
  await getSingleBlog(id);              // 7) confere de novo
  await deleteBlog(id);                 // 8) apaga
};

main();
```

Cada `await` faz a função esperar a anterior terminar. Isso é importante: não dá para buscar o blog antes de ele ter sido criado.

Para validar o PR, porém, **priorize o teste pelo navegador com o arquivo HTML**: a documentação oficial do Basic Training pede que a atividade seja testada dessa forma. Use o [`test-blogs.html`](test-blogs.html) que acompanha esta pasta. O HTML chama cada função CRUD pelos botões, permitindo testar as operações e acompanhar as requisições feitas pelo navegador:

1. Deixe o **Liferay rodando** em `localhost:8080` e confirme o **ID do seu site**. Ajuste `urlBlogPostings` no seu JavaScript se o ID for diferente de `20117`.
2. Copie `test-blogs.html` para a pasta onde está o seu arquivo `PR.js` (ou o JavaScript com as funções CRUD). O HTML carrega `./PR.js`, então os dois arquivos precisam ficar lado a lado. Se o seu arquivo tiver outro nome, altere o atributo `src` da tag `<script>` no HTML.
3. Ao testar pelo HTML, deixe as funções CRUD disponíveis para ele e não execute `main()` automaticamente ao carregar o JavaScript. O HTML chama cada operação separadamente.
4. Abra a cópia de `test-blogs.html` no navegador. Use **Criar blog** e **Listar blogs** para testar; para buscar, atualizar ou excluir, informe antes um ID no campo. Lembre-se de que excluir remove o blog do Liferay.
5. Abra as ferramentas do desenvolvedor com **F12** (ou clique com o botão direito e escolha **Inspecionar**). Na aba **Console**, veja os `console.log`/erros; na aba **Network (Rede)**, acompanhe as requisições, respostas e códigos HTTP. O resultado da operação também aparece na página.

Se o navegador bloquear uma requisição por **CORS**, siga o tutorial do Basic Training para habilitar o CORS especificamente para esse endpoint.

## 2.5 Pontos de melhoria

Depois que tudo funcionar, estas melhorias deixam o código mais profissional (e costumam ser bem vistas em revisões):

1. **Retornar os dados.** `getAllBlog`, `getSingleBlog`, `updateSingleBlog` e `patchBlog` só fazem `console.log(data)`. Fazer `return data;` permite reaproveitar o resultado.
2. **Mensagens de erro mais claras:**
   ```js
   throw new Error(`Erro HTTP ${res.status} ao criar o blog`);
   ```
3. **Evitar repetição.** Todas as funções repetem `try / fetch / if (!res.ok) / res.json()`. Dá para extrair uma função auxiliar:
   ```js
   const request = async (url, options) => {
     const res = await fetch(url, options);
     if (!res.ok) throw new Error(`Erro HTTP ${res.status}`);
     return res.status === 204 ? null : res.json();
   };
   ```
4. **`Accept` em vez de `Content-Type` nos GET.** Em requisições sem corpo, `Content-Type` não é necessário; o cabeçalho `Accept: application/json` é o adequado para dizer qual formato você espera receber.
5. **Indentação consistente.** Use **espaços** (ou tabs), nunca os dois misturados.
6. **Credenciais.** Aqui as credenciais são as do ambiente local de treinamento, então é aceitável para a atividade. Em projetos reais, **nunca** versione senhas: use variáveis de ambiente.

## 2.6 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `Error: 401` | E-mail ou senha incorretos | Confira `email` e `senha` |
| `Error: 404` | URL ou `id` errados; site inexistente | Confira o site ID e o id do blog |
| `Error: 400` | Corpo inválido ou faltando campo obrigatório | Envie `headline` e `articleBody` no `POST`/`PUT` |
| `Error: 403` | Usuário sem permissão | Use um usuário com permissão no site |
| `Failed to fetch` | Liferay desligado, endereço incorreto ou bloqueio de CORS | Confira `localhost:8080`, o site ID e as configurações de CORS do tutorial |
| `Unexpected end of JSON input` | Tentou ler JSON de resposta sem corpo (204) | Trate o `204` antes do `res.json()` |
| Os botões não chamam as funções | O JavaScript não carregou ou as funções não estão disponíveis para o HTML | Confira o nome e o caminho do arquivo JavaScript no atributo `src` e deixe as funções acessíveis ao HTML |
| `undefined` no id | `postBlog` falhou e devolveu `null` | Veja o erro no console do navegador e corrija a criação |

### Para praticar

1. Altere `postBlog` para receber `headline` e `articleBody` como **parâmetros**.
2. Faça `getAllBlog` **devolver** `data.items` e imprima só os títulos com `.map()`.
3. Crie a função auxiliar `request` da seção 2.5 e reescreva uma das funções usando-a.
4. Provoque um erro 404 de propósito (id inexistente) e observe o tratamento.

---
