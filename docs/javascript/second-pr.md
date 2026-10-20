# Segundo PR de JavaScript

## Etapa 3 – O código

### 3.1 Visão geral

O arquivo permite selecionar um arquivo no navegador e enviá-lo para o Documents and Media do Liferay usando a Headless Delivery API.

O código está dividido em três partes:

**Configuração e elementos HTML:**
- `myForm`: formulário que recebe o evento de envio.
- `fileInput`: campo usado para selecionar o arquivo.
- `output`: elemento onde a resposta da API é exibida.
- `urlPostDoc`: URL do endpoint de upload.
- `email` e `senha`: credenciais do ambiente local.

**Função de requisição:**

`uploadDoc(file)` → envia o arquivo e seus metadados por meio de uma requisição `POST`.

**Evento do formulário:**

O `submit` é interceptado, o arquivo selecionado é recuperado e a função `uploadDoc(file)` é chamada.

O fluxo geral é:

1. O usuário seleciona um arquivo.
2. O formulário é enviado.
3. O JavaScript impede o envio tradicional da página.
4. O código verifica se existe um arquivo selecionado.
5. `uploadDoc(file)` prepara os dados e envia a requisição.
6. A resposta é exibida na página.

### 3.2 Código completo comentado

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

      // Exibe a resposta da API na página.
      output.innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
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

### 3.3 Explicação por partes

### Parte 1 – Elementos HTML e DOM

```javascript
const myForm = document.getElementById("myForm");
const fileInput = document.getElementById("fileInput");
const output = document.getElementById("output");
```

O método `document.getElementById()` permite acessar elementos HTML pelo seu atributo `id`.

Isso conecta o JavaScript à página: o código consegue detectar o envio do formulário, recuperar o arquivo selecionado e apresentar o resultado da requisição.

O campo de arquivo possui uma particularidade:

```javascript
const file = fileInput.files[0];
```

A propriedade `files` contém os arquivos selecionados. Como a atividade permite selecionar um arquivo por vez, `[0]` acessa o primeiro elemento dessa coleção.

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

O ponto principal dessa implementação é o uso de `FormData`.

**`FormData`** é uma interface do JavaScript usada para construir um conjunto de campos que pode ser enviado em uma requisição HTTP. Nesse caso, ela permite combinar um arquivo binário com informações textuais.

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

O navegador configura o tipo de conteúdo e acrescenta um parâmetro chamado *boundary*, que separa as diferentes partes do corpo da requisição. Definir manualmente esse cabeçalho pode impedir que o servidor interprete corretamente os dados enviados.

A autenticação continua usando o cabeçalho `Authorization` com Basic Auth, como no primeiro PR.

### Parte 4 – Tratamento da resposta

```javascript
if (!res.ok) {
  throw new Error(`${res.status}`);
}

const data = await res.json();

output.innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
```

Depois da requisição, o código verifica se a resposta HTTP indica sucesso. Se não indicar, lança um erro, que será tratado pelo `catch`.

Quando a resposta contém JSON, `res.json()` converte esse conteúdo em um objeto JavaScript. Já `JSON.stringify(data, null, 2)` transforma o objeto em texto formatado, facilitando a leitura na página.

**Atenção:** esse trecho pressupõe que o endpoint retorna um corpo JSON. Se a API responder sem conteúdo, será necessário tratar esse caso antes de chamar `res.json()`. Além disso, como a resposta é inserida com `innerHTML`, em um código de produção seria mais seguro criar um elemento `<pre>` e inserir o texto com `textContent`, evitando interpretar o conteúdo retornado como HTML.

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

O `addEventListener()` registra uma função que será executada quando ocorrer o evento `submit` do formulário.

- `event.preventDefault()` impede o envio tradicional do formulário.
- `fileInput.files[0]` recupera o arquivo selecionado.
- A condição `if (!file)` interrompe a execução se nenhum arquivo tiver sido escolhido.
- `await uploadDoc(file)` aguarda a conclusão da função de upload.

Essa separação deixa cada parte com uma responsabilidade clara: o evento trata da interação com o usuário, enquanto `uploadDoc()` cuida da comunicação com a API.

### 3.4 Como executar e testar

1. Deixe o Liferay rodando em `localhost:8080` e confirme o ID do site. Se necessário, ajuste `urlPostDoc`.
2. Salve o código em um arquivo HTML, por exemplo, `upload-document.html`.
3. Abra o arquivo no navegador.
4. Selecione um arquivo e clique em **Upload**.
5. Confira se a resposta da API aparece na página e se o documento foi criado no Documents and Media.
6. Abra as ferramentas do desenvolvedor com `F12`. Na aba **Console**, verifique os erros; na aba **Network (Rede)**, confira a requisição, o corpo enviado e o status HTTP.

Se a requisição for bloqueada por CORS, verifique a configuração de CORS do Liferay e siga o procedimento indicado no Basic Training.

**Observações importantes:**

- O usuário autenticado precisa ter permissão para adicionar documentos ao destino.
- O ID do site e o `folderId` precisam corresponder ao ambiente em que você está testando.
- Basic Auth com Base64 não é criptografia. Use credenciais de treinamento apenas em um ambiente local e protegido; em produção, não exponha credenciais no código do navegador.
- Se o servidor retornar um erro, consulte o status e a resposta na aba Network para identificar a causa.
