# Guia: Atividade de JavaScript com a Headless Delivery API (Upload de documentos)

Neste segundo PR, você cria um formulário HTML para selecionar um arquivo, enviá-lo ao **Documents and Media** do Liferay e exibir a resposta da API na própria página.

O guia está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Como o navegador trata o envio do formulário, prepara o arquivo com `FormData` e atualiza o DOM. |
| **Etapa 2 – O código** | Como as partes do exemplo funcionam juntas e como testar o upload no seu ambiente. |

> Para revisar HTTP, JSON, Basic Auth, `fetch` e `async/await`, consulte o [primeiro PR de JavaScript](first-pr.md).

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 O formulário HTML e o DOM](#11-o-formulário-html-e-o-dom)
  - [1.2 O evento `submit` e o método `preventDefault()`](#12-o-evento-submit-e-o-método-preventdefault)
  - [1.3 Seleção de arquivos com `FileList` e `File`](#13-seleção-de-arquivos-com-filelist-e-file)
  - [1.4 Upload com `FormData` e `multipart/form-data`](#14-upload-com-formdata-e-multipartform-data)
  - [1.5 Exibição da resposta com `<pre>`](#15-exibição-da-resposta-com-pre)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 Código completo comentado](#22-código-completo-comentado)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
    - [Parte 1 – Elementos HTML e DOM](#parte-1--elementos-html-e-dom)
    - [Parte 2 – A função `uploadDoc(file)`](#parte-2--a-função-uploaddocfile)
    - [Parte 3 – Enviando o arquivo com `fetch`](#parte-3--enviando-o-arquivo-com-fetch)
    - [Parte 4 – Tratamento da resposta](#parte-4--tratamento-da-resposta)
    - [Parte 5 – Evento `submit`](#parte-5--evento-submit)
  - [2.4 Como executar e testar](#24-como-executar-e-testar)
  - [2.5 Pontos de melhoria](#25-pontos-de-melhoria)
  - [2.6 Erros comuns](#26-erros-comuns)

---

# Etapa 1 – Teoria

Vamos acompanhar o que acontece **no navegador**, desde o momento em que alguém escolhe um arquivo até a exibição do documento devolvido pelo Liferay.

## 1.1 O formulário HTML e o DOM

Ao ler o HTML, o navegador constrói o **DOM** (*Document Object Model*), uma árvore de objetos que representa a página. No exemplo, o formulário contém o campo de arquivo e o botão. A `<div id="output">` fica fora do formulário e serve para mostrar o resultado.

`document.getElementById("myForm")` procura o elemento cujo atributo é `id="myForm"`. A constante `myForm` guarda uma referência ao formulário que já está na página. Por isso podemos registrar nele uma função que reage ao envio.

> A posição do `<script>` também importa. Quando o navegador chega a ele, já leu os três elementos. Se o ID estiver escrito de outra forma, a busca retorna `null` e a chamada seguinte a `myForm.addEventListener(...)` falha.

## 1.2 O evento `submit` e o método `preventDefault()`

```javascript
myForm.addEventListener("submit", async (event) => {
  event.preventDefault();
  // A leitura do arquivo e o upload vêm em seguida.
});
```

Ao carregar a página, o JavaScript executa `addEventListener` para **registrar** uma função. O primeiro argumento, `"submit"`, indica qual evento interessa. O segundo é a própria função, chamada de *callback*. O navegador guarda essa função e a chama a cada envio do formulário. Assim, abrir a página apenas deixa o formulário pronto para uso.

### Por que escutar o formulário?

O clique acontece no botão, mas o botão tem `type="submit"` e aciona o envio do formulário. Escutar `submit` permite tratar o envio inclusive quando o botão é ativado pelo teclado. Quando o evento ocorre, o navegador passa à função um objeto com informações sobre aquele envio. O nome `event` é o parâmetro que usamos para acessar esse objeto.

### O efeito de `event.preventDefault()`

Sem essa chamada, o navegador também tentaria fazer o envio nativo. Como o formulário do exemplo omite `action` e `method`, o destino padrão é a URL da própria página e o método padrão é `GET`. Essa navegação poderia interromper o upload ou fazer desaparecer a resposta exibida. O endereço em `urlPostDoc` só é usado quando nosso código chama `fetch`.

`event.preventDefault()` cancela a ação padrão **desse envio**. A função continua na linha seguinte e pode validar o arquivo, executar o `POST` e atualizar a página. A chamada vem antes de qualquer `await` porque o navegador pode iniciar a ação padrão assim que termina de despachar o evento, enquanto uma função assíncrona ainda aguarda sua operação. Já o `return` usado na validação tem outra finalidade: encerrar a execução da função naquele ponto.

## 1.3 Seleção de arquivos com `FileList` e `File`

O campo `<input type="file">` abre o seletor de arquivos. A escolha fica disponível em `fileInput.files`, uma coleção do tipo `FileList`. Escolher o arquivo apenas altera a seleção. O envio acontece depois, quando a função registrada lê `fileInput.files[0]`.

Como o campo aceita um arquivo por vez, a posição zero contém o objeto `File` escolhido. Esse objeto dá acesso ao conteúdo do arquivo e a informações como `file.name`. Se a seleção estiver vazia, o acesso por índice resulta em `undefined`. Nesse caso, `if (!file)` entra no bloco, o alerta pede uma seleção e o `return` evita chamar `uploadDoc`.

## 1.4 Upload com `FormData` e `multipart/form-data`

`new FormData()` cria um conjunto vazio de campos. Cada chamada a `append(nome, valor)` acrescenta uma entrada. Na atividade, `append("file", file)` inclui o próprio arquivo, enquanto `append("title", file.name)` envia apenas seu nome como título. Por exemplo, ao selecionar `relatorio.pdf`, o servidor recebe tanto os bytes do PDF quanto o texto `relatorio.pdf`. A descrição e o identificador da pasta completam o corpo. O valor numérico passado a `append("folderId", 0)` é convertido em texto.

Quando `fetch` recebe esse `FormData` como `body`, o navegador monta um corpo `multipart/form-data`. Cada campo ocupa uma parte identificada pelo nome usado em `append`. O navegador escolhe um delimitador (*boundary*), insere esse delimitador entre as partes e informa o mesmo valor no cabeçalho `Content-Type`. Por isso o exemplo deixa esse cabeçalho a cargo do navegador. Escrevê-lo manualmente como apenas `multipart/form-data` omite o delimitador de que o servidor precisa para separar os campos.

## 1.5 Exibição da resposta com `<pre>`

Depois de obter o objeto `data` com `await res.json()`, o código executa três operações distintas:

```javascript
const pre = document.createElement("pre");
pre.textContent = JSON.stringify(data, null, 2);
output.replaceChildren(pre);
```

1. **Criar o elemento.** `document.createElement("pre")` cria um elemento HTML `<pre>` em memória. A constante `pre` guarda uma referência a ele. Nesse momento ele ainda está fora da árvore da página. A tag foi escolhida porque preserva espaços e quebras de linha, o que torna a indentação visível.
2. **Preencher o elemento.** `JSON.stringify(data, null, 2)` produz uma string a partir de `data`. O segundo argumento, `null`, indica que não aplicamos um filtro ou função de substituição. O terceiro, `2`, pede dois espaços de indentação por nível. A atribuição a `textContent` coloca essa string dentro do elemento como texto. Assim, um título como `<b>Relatório</b>` aparece com os sinais de menor e maior, sem virar uma tag em negrito.
3. **Mostrar o resultado.** `output.replaceChildren(pre)` coloca o elemento pronto dentro de `<div id="output">`, substituindo seus filhos atuais. No primeiro upload bem-sucedido, a área estava vazia. No segundo, o novo `<pre>` ocupa o lugar do anterior. A própria `div` continua disponível para o próximo resultado.

## Recursos adicionais

- [Evento `submit` do formulário (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event) — veja em quais situações o navegador dispara o evento e como registrar uma função para recebê-lo.
- [`event.preventDefault()` (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) — aprofunde a diferença entre cancelar a ação padrão e interromper a propagação do evento.
- [Upload de arquivos com `FormData` (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects) — acompanhe exemplos de envio de arquivos e veja por que o navegador precisa definir o `Content-Type`.
- [`replaceChildren()` (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Element/replaceChildren) — veja como substituir os filhos de um elemento, como fazemos ao exibir o novo `<pre>` na área de resultado.
- [API Explorer do Liferay local](http://localhost:8080/o/api) — consulte a operação `POST /sites/{siteId}/documents` da Headless Delivery para conferir o corpo da requisição e a resposta esperada (Funciona somente se o container Liferay estiver rodando e se você fez o login corretamente da sua conta `test@liferay.com`).

---

# Etapa 2 – O código

## 2.1 Visão geral

A página permite selecionar um arquivo no navegador e enviá-lo para o Documents and Media do Liferay usando a Headless Delivery API.

| Parte do código | Responsabilidade |
|---|---|
| Configuração | Localiza `myForm`, `fileInput` e `output` e define o endereço da API e as credenciais de treinamento. |
| `uploadDoc(file)` | Monta o corpo da requisição, envia o documento e exibe a resposta do Liferay. |
| Evento `submit` | Lê a seleção atual, verifica se há um arquivo e chama `uploadDoc(file)`. |

O fluxo geral é:

1. O usuário seleciona um arquivo.
2. O formulário é enviado.
3. O JavaScript impede o envio tradicional da página.
4. O código verifica se existe um arquivo selecionado.
5. `uploadDoc(file)` prepara os dados e envia a requisição.
6. A resposta é exibida na página.

## 2.2 Código completo comentado

A lógica é a mesma da atividade; apenas organizei a indentação e acrescentei comentários para facilitar o estudo.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Uploading a document</title>
  </head>
  <body>
    <h1>Post documents to Documents and Media</h1>

    <form id="myForm">
      <input type="file" id="fileInput" />
      <button type="submit">Upload</button>
    </form>

    <div id="output"></div>

<script>
  // Author: Alberto
  
  // ================================================================
  // 1. CONFIGURAÇÃO
  // ================================================================

  // Elementos HTML que serão utilizados pelo JavaScript.
  const myForm = document.getElementById("myForm");
  const fileInput = document.getElementById("fileInput");
  const output = document.getElementById("output");

  // Endpoint responsável por receber os documentos do site 20117.
  const urlPostDoc =
    "http://localhost:8080/o/headless-delivery/v1.0/sites/20117/documents";

  // Credenciais do ambiente local de treinamento.
  const email = "test@liferay.com";
  const senha = "learn";

  // ================================================================
  // 2. UPLOAD DO ARQUIVO
  // ================================================================

  const uploadDoc = async (file) => {
    // FormData permite enviar o arquivo e os metadados no mesmo corpo.
    const formData = new FormData();

    formData.append("file", file);
    formData.append("title", file.name);
    formData.append("description", "File Uploaded");
    formData.append("folderId", 0);

    try {
      const res = await fetch(urlPostDoc, {
        method: "POST",
        headers: {
          // Envia as credenciais usando autenticação Basic.
          Authorization: "Basic " + btoa(`${email}:${senha}`),
        },

        // O navegador prepara o corpo multipart e seu boundary.
        // Não definir Content-Type manualmente neste caso.
        body: formData,
      });

      // fetch não lança erro automaticamente para respostas HTTP
      // como 400 ou 500.
      if (!res.ok) {
        throw new Error(`${res.status}`);
      }

      // Converte a resposta JSON em um objeto JavaScript.
      const data = await res.json();

      // Exibe a resposta da API como texto, sem interpretá-la como HTML.
      const pre = document.createElement("pre");
      pre.textContent = JSON.stringify(data, null, 2);
      output.replaceChildren(pre);
    } catch (err) {
      console.error(err);
    }
  };

  // ================================================================
  // 3. EVENTO DO FORMULÁRIO
  // ================================================================

  myForm.addEventListener("submit", async (event) => {
    // Impede o comportamento padrão do formulário,
    // que enviaria os dados e poderia recarregar a página.
    event.preventDefault();

    // Recupera o primeiro arquivo selecionado.
    const file = fileInput.files[0];

    // Não tenta enviar se nenhum arquivo tiver sido selecionado.
    if (!file) {
      alert("Please, select a file.");
      return;
    }

    // Envia o arquivo para a API.
    await uploadDoc(file);
  });
</script>
  </body>
</html>
```

## 2.3 Explicação por partes

### Parte 1 – Elementos HTML e DOM

```javascript
const myForm = document.getElementById("myForm");
const fileInput = document.getElementById("fileInput");
const output = document.getElementById("output");
```

As três referências acompanham o fluxo da atividade: `myForm` recebe o evento, `fileInput` fornece a seleção atual e `output` recebe o resultado.

### Parte 2 – A função `uploadDoc(file)`

```javascript
const uploadDoc = async (file) => {
  const formData = new FormData();

  formData.append("file", file);
  formData.append("title", file.name);
  formData.append("description", "File Uploaded");
  formData.append("folderId", 0);
};
```

A função recebe o arquivo selecionado como argumento. Em seguida, prepara os dados que serão enviados ao Liferay.

Confira quais dados a função associa a cada campo:

| Campo | Finalidade |
|---|---|
| `file` | Contém o arquivo selecionado. |
| `title` | Define o título do documento com o nome do arquivo. |
| `description` | Define a descrição do documento. |
| `folderId` | Identifica a pasta de destino; `0` é o valor usado nesta atividade. |

A API precisa receber esses campos com os nomes esperados pelo endpoint.

### Parte 3 – Enviando o arquivo com `fetch`

```javascript
const res = await fetch(urlPostDoc, {
  method: "POST",
  headers: {
    Authorization: "Basic " + btoa(`${email}:${senha}`),
  },
  body: formData,
});
```

A requisição usa `POST` porque estamos criando um documento no Liferay.

Observe duas diferenças em relação ao primeiro PR:

- **Não usamos `JSON.stringify()`** para o corpo da requisição. O objeto `FormData` é enviado diretamente.
- **Não definimos `Content-Type: application/json`**. O corpo não é JSON: é enviado como `multipart/form-data`.

Na aba **Network**, confira o `boundary` em **Request Headers** e os nomes dos campos em **Payload**.

### Parte 4 – Tratamento da resposta

```javascript
if (!res.ok) {
  throw new Error(`${res.status}`);
}

const data = await res.json();

const pre = document.createElement("pre");
pre.textContent = JSON.stringify(data, null, 2);
output.replaceChildren(pre);
```

A atualização de `output` ocorre somente depois de `res.ok` e da leitura de um corpo JSON válido. Se uma dessas etapas falhar, o fluxo vai para o `catch`, que registra o erro no console. Se um resultado anterior já estiver na página, ele continua visível.

### Parte 5 – Evento `submit`

```javascript
myForm.addEventListener("submit", async (event) => {
  event.preventDefault();

  const file = fileInput.files[0];

  if (!file) {
    alert("Please, select a file.");
    return;
  }

  await uploadDoc(file);
});
```

Para acompanhar o fluxo, tente enviar o formulário com o campo vazio. O alerta aparece e a aba **Network** não registra um `POST` de upload. Selecione um arquivo e tente novamente. Agora a execução alcança `await uploadDoc(file)`, e você pode inspecionar a requisição e comparar a resposta com o texto exibido em `output`.

## 2.4 Como executar e testar

1. Deixe o Liferay rodando em `localhost:8080` e confirme o ID do site. Se necessário, ajuste `urlPostDoc`.
2. Salve o código em um arquivo HTML, por exemplo, `upload-document.html`.
3. Abra o arquivo no navegador.
4. Selecione um arquivo e clique em **Upload**.
5. Confira se a resposta da API aparece na página e se o documento foi criado no Documents and Media.
6. Abra as ferramentas do desenvolvedor com `F12`. Na aba **Console**, verifique os erros; na aba **Network (Rede)**, confira a requisição, o corpo enviado e o status HTTP.

## 2.5 Pontos de melhoria

- **Configuração do ambiente.** O exemplo fixa a URL e o ID do site no script. Centralizar esses valores facilita testar o formulário em outro site.
- **Retorno ao usuário.** O `catch` registra a falha apenas no console. A área `output` também pode informar quando o upload falhar, principalmente se ela ainda estiver mostrando o último envio bem-sucedido.

## 2.6 Erros comuns

| Sintoma | O que conferir |
|---|---|
| A aba Network mostra `403` | Confira se o usuário autenticado tem permissão para adicionar documentos ao destino. |
| O documento aparece em outro local | Verifique o ID do site na URL e o `folderId` enviado no corpo. |
| O navegador informa um bloqueio de CORS | Confira a origem da página e a configuração de CORS indicada no Basic Training. |

> Use as credenciais do exemplo apenas no ambiente local de treinamento. Elas ficam visíveis no JavaScript entregue ao navegador.

### Para praticar

Envie dois arquivos com nomes diferentes, um de cada vez. Compare as duas requisições na aba **Network** e inspecione a `div` de resultado na aba **Elements**. Por que a página mostra apenas o documento do último envio bem-sucedido?
