
# Guia: Atividade de React com Clay UI e Headless Delivery API (CRUD de Blogs, Documents e Knowledge Base como widget)

Este guia ensina, do zero, a fazer a atividade em que você cria um **widget em React** para o **Liferay DXP**. O widget usa os componentes visuais do **Clay UI** e conversa com a **Headless Delivery API** para fazer o CRUD de três recursos: **Blogs**, **Documents and Media** e **Knowledge Base**.

Ele está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Conceitos necessários: React como widget na Liferay, hooks, Clay UI, os endpoints da API usados no app, `fetch` (autenticação, upload e status 204) e como o widget é construído e implantado |
| **Etapa 2 – O código** | Os arquivos da atividade, um a um, com código comentado e explicado parte por parte |

> Este guia **não ensina a enviar o PR**: isso você já tem no seu próprio markdown. Aqui o foco é entender o assunto da atividade.

> A configuração inicial da pasta React utilizada no terceiro PR pode ser consultada neste repositório de referência: liferay-react-example. Use-o como referência para preparar a estrutura inicial antes de aplicar as alterações descritas neste guia.

> Os conceitos de HTTP, REST, JSON, Promises e `async/await` são os mesmos do guia da atividade de JavaScript. Aqui eles são só retomados quando necessário.

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 React como widget na Liferay](#11-react-como-widget-na-liferay)
  - [1.2 Hooks usados no projeto](#12-hooks-usados-no-projeto)
  - [1.3 Clay UI](#13-clay-ui)
  - [1.4 A Headless Delivery API neste app](#14-a-headless-delivery-api-neste-app)
  - [1.5 fetch na prática](#15-fetch-na-prática)
  - [1.6 Build e deploy do widget](#16-build-e-deploy-do-widget)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 O package.json](#22-o-packagejson)
  - [2.3 A camada de API (liferay.js)](#23-a-camada-de-api-liferayjs)
  - [2.4 O ponto de entrada (index.js)](#24-o-ponto-de-entrada-indexjs)
  - [2.5 O componente raiz (App.js)](#25-o-componente-raiz-appjs)
  - [2.6 BlogManager.jsx](#26-blogmanagerjsx)
  - [2.7 DocsManager.jsx](#27-docsmanagerjsx)
  - [2.8 KBManager.jsx](#28-kbmanagerjsx)
  - [2.9 Pontos de melhoria](#29-pontos-de-melhoria)

---

# Etapa 1 – Teoria

Antes do código, vamos entender o "mundo" em que ele vive. Se você já conhece algum tópico, pode pular.

## 1.1 React como widget na Liferay

### O que é um JS portlet

**React** é uma biblioteca JavaScript para construir interfaces a partir de **componentes** (pedaços de tela reutilizáveis, escritos como funções). Na Liferay, uma aplicação React pode ser empacotada como um **widget** (também chamado de *portlet*): um bloco que o administrador arrasta para dentro de uma página.

Para isso, o projeto precisa ser transformado em um **módulo OSGi** (o formato de pacote que a Liferay entende). Quem faz essa transformação é o **`liferay-npm-bundler`**: ele pega o código já compilado e as dependências do `node_modules` e gera um arquivo **`.jar`** que a Liferay consegue instalar.

### Como a Liferay "liga" o seu React na página

A Liferay não conhece o seu `App`. O que ela conhece é o **ponto de entrada** do projeto, que é o arquivo indicado no campo `main` do `package.json` (aqui, `index.js`). Esse arquivo precisa **exportar uma função** chamada `main`. Quando o widget aparece na página, a Liferay chama essa função e passa um objeto com informações do widget. A que nos interessa é o **`portletElementId`**: o `id` de um `<div>` vazio que a Liferay reservou na página para o seu widget.

```
Página da Liferay
└── <div id="...portletElementId..."> ← a Liferay cria esse espaço vazio
└── <App /> ← o seu código desenha aqui dentro
```

### O `package.json` como "cola" do projeto

No widget, o `package.json` não serve só para listar dependências. Ele também descreve o widget para a Liferay:

| Bloco/campo | Para que serve |
|---|---|
| `main` | Aponta o arquivo de entrada (`index.js`) que exporta a função `main` |
| `portlet` | Metadados do widget: nome (`javax.portlet.name`), categoria em que ele aparece, se pode ser colocado mais de uma vez na página (`instanceable`), papéis com acesso etc. |
| `scripts` | Comandos que você roda com `npm run <nome>`: `build`, `deploy`, `start`... |
| `dependencies` | Bibliotecas usadas **em tempo de execução** (React, Clay) |
| `devDependencies` | Ferramentas usadas **só para construir** o projeto (Babel, bundler) |

## 1.2 Hooks usados no projeto

Os componentes do projeto são **funções**. Para que uma função "lembre" de coisas entre uma renderização e outra, o React oferece os **hooks**. Eles foram introduzidos no React **16.8**, e a versão fixada no projeto (`16.8.6`) já os suporta.

| Hook | O que faz | Onde aparece no projeto |
|---|---|---|
| `useState` | Guarda um valor e **redesenha o componente** quando ele muda | Listas (`blogs`, `documents`, `articles`), campos do formulário (`headline`, `body`...) e o `editId` |
| `useEffect` | Executa um código **depois** que o componente é desenhado | Carregar a lista da API logo que o componente aparece na tela |
| `useRef` | Guarda uma **referência** a um elemento HTML, sem causar redesenho | O `<input type="file">` escondido do `DocsManager` |

### `useState`

```js
const [headline, setHeadline] = useState("");
// ↑ valor atual ↑ função para trocar o valor ↑ valor inicial
```
Nunca altere a variável diretamente (`headline = "x"` não funciona). Sempre use a função `setHeadline("x")`: é isso que avisa o React para redesenhar a tela.

### `useEffect` e o array de dependências

```js
useEffect(() => {
	load();
}, []); // ← array vazio: executa UMA vez, quando o componente aparece
```
O segundo argumento controla **quando** o efeito roda. Com `[]`, ele roda só na primeira vez. Sem o array, rodaria a cada redesenho, o que causaria um laço de requisições.

### Componentes controlados

Um **input controlado** é aquele cujo valor vem do estado do React, e não do próprio HTML:

```jsx
<ClayInput value={headline} onChange={(e) => setHeadline(e.target.value)} />;
```
O ciclo é: o usuário digita → `onChange` dispara → `setHeadline` atualiza o estado → o React redesenha o input com o novo valor. Assim o estado é sempre a "fonte da verdade" do que está no formulário, e fica fácil limpar o campo (`setHeadline("")`) ou preenchê-lo para edição.

## 1.3 Clay UI

**Clay** é o sistema de design da Liferay, disponível como componentes React. Usar o Clay faz o seu widget ficar **com o mesmo visual** do resto da plataforma, sem você escrever CSS.

Componentes usados neste projeto:

| Componente | Pacote | Para que serve |
|---|---|---|
| `ClayButton` | `@clayui/button` | Botões (`displayType` define a cor: `primary`, `secondary`...) |
| `ClayForm` e `ClayInput` | `@clayui/form` | Formulário, grupos de campos e inputs (inclusive `textarea`) |
| `ClayTable` | `@clayui/table` | Tabelas com cabeçalho, linhas e células |

### Por que as versões ficam fixas

Repare que no `package.json` as versões do Clay não têm `^` (`"3.1.0"`, e não `"^3.1.0"`). O Clay é dividido em **vários pacotes que dependem uns dos outros**. Fixar as versões garante que todos os pacotes sejam compatíveis entre si e com a versão da Liferay usada no treinamento. Uma atualização automática poderia quebrar o visual ou o build.

### O bloco `overrides`

Cada pacote Clay também declara suas próprias dependências (por exemplo, `@clayui/icon`, `@clayui/shared`). Sem controle, o npm pode acabar instalando **duas versões diferentes do mesmo pacote** (uma para cada pacote que o pede), o que costuma gerar conflitos no build ou no visual.

O campo **`overrides`** do `package.json` resolve isso: ele **obriga** o npm a usar a versão indicada, em qualquer lugar da árvore de dependências.

> O campo `overrides` é reconhecido pelo npm a partir da versão **8.3**. Em versões mais antigas, ele é simplesmente ignorado.

## 1.4 A Headless Delivery API neste app

A API é a mesma do guia de JavaScript (`/o/headless-delivery/v1.0`). A diferença é que, aqui, o código roda **dentro de uma página da própria Liferay**, então podemos usar uma URL **relativa** (sem `http://localhost:8080`) e pegar o ID do site dinamicamente.

**Por que não usamos URLs absolutas nem IDs fixos?** A atividade exige evitar hard-coded URLs e IDs de site. Por isso, o código define apenas o caminho-base relativo da API em `BASE_URL` e obtém o ID do site em tempo de execução com `Liferay.ThemeDisplay.getScopeGroupId()`. Assim, o widget usa o servidor da página em que está instalado e o site em que foi colocado, em vez de depender de um endereço como `http://localhost:8080\` ou de um ID específico, como `20117`. Os caminhos dos recursos (por exemplo, `blog-postings`) continuam definidos no código porque identificam os endpoints que a aplicação precisa consumir; o que evitamos é fixar o host completo e o ID do site.

### Recurso × método no app

| Recurso | Listar (GET) | Criar (POST) | Atualizar (PUT) | Excluir (DELETE) |
|---|---|---|---|---|
| **Blogs** (`blog-postings`) | Sim | Sim | Sim | Sim |
| **Knowledge Base** (`knowledge-base-articles`) | Sim | Sim | Sim | Sim |
| **Documents** (`documents`) | Sim | Sim (upload) | Não implementado | Sim |

> Neste app, Documents só lista, envia e exclui. A edição de documentos não foi implementada.

### Endpoints por site × endpoints por id

Assim como no guia de JavaScript, a API tem **dois níveis** de URL:

| Nível | Formato | Usado para |
|---|---|---|
| **Coleção, dentro de um site** | `/sites/{siteId}/blog-postings` | **Listar** e **criar** |
| **Um item específico** | `/blog-postings/{id}` | **Atualizar** e **excluir** |

O mesmo padrão vale para `documents` e `knowledge-base-articles`.

### Campos usados em cada recurso

| Recurso | Campos enviados | Campos lidos na tela |
|---|---|---|
| Blog | `headline`, `articleBody` | `id`, `headline`, `articleBody` |
| Knowledge Base | `title`, `articleBody` | `id`, `title`, `articleBody` |
| Documents | arquivo (`file`), `title`, `description` | `id`, `title`, `fileName` |

## 1.5 fetch na prática

O `fetch`, o JSON e o `async/await` funcionam como no guia de JavaScript. Aqui há três detalhes que merecem atenção.

### Autenticação Basic

O app envia `email:senha` em Base64 no cabeçalho `Authorization`, como na atividade de JavaScript:

```js
"Basic " + btoa("test@liferay.com:learn");
```
> ⚠️ **Base64 não é criptografia.** Isso só é aceitável em ambiente de treinamento. Veja os [pontos de melhoria](#29-pontos-de-melhoria) para a alternativa correta dentro de um widget.

### Upload com `FormData`

Para enviar um **arquivo**, o corpo da requisição não é JSON: é um formulário do tipo **multipart**, montado com `FormData`:

```js
const formData = new FormData();
formData.append("file", file); // o arquivo escolhido pelo usuário
```
E aqui há uma regra importante: **não defina o `Content-Type` manualmente.** Um formulário multipart precisa de um *boundary* (um separador gerado automaticamente) dentro do cabeçalho. Se você fixar `"Content-Type": "multipart/form-data"` à mão, o separador some e o servidor não consegue ler o arquivo. Deixando o navegador montar o cabeçalho, ele já coloca o valor certo.

Por isso o upload usa só `authHeader()` (sem `Content-Type`), e não `jsonHeaders()`.

### O status `204` no DELETE

Como no guia de JavaScript, `204 No Content` significa sucesso **sem corpo**. Tentar ler `res.json()` numa resposta assim lança um erro. O `handleResponse` do projeto trata isso devolvendo `null` quando o status é 204, e os `DELETE` passam por ele.

## 1.6 Build e deploy do widget

O código que você escreve (JSX, imports de módulos) **não roda direto** na Liferay. Ele precisa ser transformado e empacotado.

### O script `build`

```
babel --source-maps -d build src && npm run copy-assets && liferay-npm-bundler
```

| Passo | O que acontece |
|---|---|
| `babel ... -d build src` | **Compila** tudo o que está em `src/` (converte JSX e JavaScript moderno em JavaScript comum) e grava o resultado na pasta `build/`. `--source-maps` gera arquivos que ajudam a depurar |
| `npm run copy-assets` | Copia os arquivos estáticos do projeto (CSS, idiomas...) para dentro do pacote |
| `liferay-npm-bundler` | **Empacota** o resultado e as dependências do `node_modules` em um **`.jar`** dentro da pasta `dist/` |

O `&&` faz cada comando só começar se o anterior terminar sem erro.

### Por que o deploy precisa ser feito com um único comando?

Nesta atividade, o projeto deve poder ser compilado, empacotado e enviado para o ambiente Liferay usando **um único comando**. Por isso existe o script `deploy` no `package.json`: ele encadeia o build e a cópia do pacote para a pasta de deploy do container. Depois da configuração inicial do ambiente, basta executar `npm run deploy` no terminal, dentro da pasta do projeto. Não é necessário executar manualmente cada etapa separadamente.

### O script `deploy`

```
npm run build && docker cp dist/test-react-1.0.0.jar liferay:/opt/liferay/deploy
```

Ele roda o `build` e depois **copia o `.jar`** para a pasta de deploy da Liferay **dentro do container Docker**. A Liferay fica "olhando" essa pasta: assim que o arquivo aparece, ela instala o módulo automaticamente, sem precisar reiniciar o servidor. Portanto, o comando usado no fluxo normal da atividade é `npm run deploy`, que executa as etapas necessárias em sequência.

## Recursos adicionais

Os links abaixo complementam os conceitos apresentados nesta etapa e ajudam a consultar detalhes que dependem da versão do ambiente:

### Configuração inicial

- liferay-react-example: referência para fazer o setup inicial da pasta React usado no terceiro PR.

### React

- [React: JSX, componentes e props](../theory/react-jsx-components.md): material do repositório sobre a estrutura dos componentes e a passagem de dados.

- [React Hooks e renderização](../theory/react-hooks.md): material do repositório sobre estado, efeitos e renderização.

- [Aprenda React](https://react.dev/learn): introdução a componentes, JSX, props e estado.

- [Referência de `useState`](https://react.dev/reference/react/useState): declaração e atualização de estado.

- [Referência de `useEffect`](https://react.dev/reference/react/useEffect): sincronização com sistemas externos, como a API usada pelo widget.

> A documentação atual do React pode mostrar recursos que não existem na versão `16.8.6` usada neste projeto. Confira a versão instalada no `package.json` antes de aplicar exemplos.

### Clay UI

- [Guia de Clay UI do repositório](../theory/clay.md): material interno sobre Clay e seu uso na Liferay.

- [Documentação do Clay](https://www.clayui.com/docs): componentes React do sistema de design da Liferay.

- [Componentes React do Clay](https://www.clayui.com/docs/components): referência de propriedades e exemplos dos componentes, como botões, formulários, tabelas e modais.

> As versões de Clay deste projeto estão fixadas no `package.json`; consulte a documentação compatível com essas versões caso alguma propriedade dos exemplos atuais seja diferente.

### Liferay e Headless Delivery API

- [Camada de API e separação de responsabilidades](../theory/api-layer-and-separation-of-concerns.md): organização das chamadas HTTP fora dos componentes de interface.

- [Headless APIs no Liferay Learn](https://learn.liferay.com/w/dxp/integration/headless-apis): visão geral das APIs REST disponíveis na plataforma.

- [Consumo de serviços REST](https://learn.liferay.com/en/w/dxp/integration/headless-apis/using-liferay-as-a-headless-platform/consuming-apis/consuming-rest-services): autenticação, chamadas e consulta dos endpoints.

- **API Explorer do ambiente:** abra `/o/api` no seu Liferay para conferir os endpoints e os esquemas da versão em execução. Isso é especialmente útil para validar upload, permissões e campos dos recursos.

### APIs do navegador

- [Fetch API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API): envio de requisições HTTP e leitura das respostas.

- [FormData (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/FormData): envio de arquivos e campos no formato `multipart/form-data`.

---

# Etapa 2 – O código

## 2.1 Visão geral

O projeto tem **uma camada de API**, **três componentes de tela** (um por recurso) e dois arquivos que montam tudo:

```
src/
├── index.js ← ponto de entrada (a Liferay chama a função main)
├── App.js ← junta os três componentes na mesma tela
├── api/
│ └── liferay.js ← TODAS as chamadas HTTP (fetch) ficam aqui
└── components/
├── BlogManager.jsx ← CRUD de Blogs
├── DocsManager.jsx ← listar, enviar e excluir Documents
└── KBManager.jsx ← CRUD de Knowledge Base
```

O caminho dos dados em qualquer operação é sempre o mesmo:

```
Componente (botão clicado)
│ chama
▼
Função de liferay.js (ex.: createBlog)
│ fetch
▼
Headless Delivery API (Liferay)
│ resposta JSON
▼
Componente guarda no estado (useState) → a tela é redesenhada
```

A separação é proposital: os componentes só cuidam da **tela**, e o `liferay.js` só cuida da **comunicação**. Se a URL da API mudar, você corrige em um único lugar.

O que foi feito em cada arquivo:

| Arquivo | Situação | Papel |
|---|---|---|
| `package.json` | Linhas implementadas | Dependências do Clay, `overrides` e script `deploy` |
| `src/api/liferay.js` | Criado e implementado | Funções de requisição para os três recursos |
| `src/index.js` | Implementado | Entrega o `<App />` ao elemento da Liferay |
| `src/App.js` | Criado e implementado | Monta a tela com os três gerenciadores |
| `src/components/BlogManager.jsx` | Criado e implementado | Tela do CRUD de Blogs |
| `src/components/DocsManager.jsx` | Criado e implementado | Tela de upload, listagem e exclusão de Documents |
| `src/components/KBManager.jsx` | Criado e implementado | Tela do CRUD de Knowledge Base |

Vamos seguir **da base para a interface**: primeiro o que sustenta o projeto (`package.json` e `liferay.js`), depois o que desenha a tela.

## 2.2 O package.json

O `package.json` do projeto já vem com uma estrutura pronta. Na atividade, você **implementa três trechos**.

### Trecho 1: as dependências do Clay

```jsonc
// ATENÇÃO: JSON de verdade NÃO aceita comentários. Eles estão aqui só
// para explicar; no arquivo real, não escreva os comentários.
"dependencies": {
// Alertas (mensagens de aviso/erro). Instalado, mas ainda não usado no código.
"@clayui/alert": "3.1.0",
// Botões. USADO em todos os componentes (ClayButton).
"@clayui/button": "3.3.0",
// Formulários e inputs. USADO em todos os componentes (ClayForm, ClayInput).
"@clayui/form": "3.1.0",
// Janelas modais (pop-ups). Instalado, mas ainda não usado no código.
"@clayui/modal": "3.1.0",
// Navegação (menus, abas). Instalado, mas ainda não usado no código.
"@clayui/nav": "3.1.0",
// Tabelas. USADO em todos os componentes (ClayTable).
"@clayui/table": "3.1.0",
// React e ReactDOM já vinham no projeto. A versão 16.8.6 é a primeira
// linha que suporta hooks (useState, useEffect, useRef).
"react": "16.8.6",
"react-dom": "16.8.6"
},
```

Os versões estão **sem `^`** de propósito (veja a [seção 1.3](#13-clay-ui)). Hoje o projeto usa `button`, `form` e `table`; os outros três pacotes ficam disponíveis para melhorias futuras (veja a [seção 2.9](#29-pontos-de-melhoria)).

### Trecho 2: o bloco `overrides`

```jsonc
// (comentários só para explicação; não existem no arquivo real)
"overrides": {
// Pacotes "internos" do Clay. Os pacotes principais (button, form, table)
// dependem deles. Sem fixar, o npm poderia instalar versões diferentes
// do MESMO pacote para cada dependente, e isso gera conflito.
"@clayui/icon": "3.0.4", // ícones usados pelos botões e campos
"@clayui/shared": "3.1.0", // utilitários compartilhados entre os pacotes
// Aqui o "button" aparece de novo: ele também é dependência de outros
// pacotes do Clay. Fixar no override garante UMA ÚNICA versão na árvore.
"@clayui/button": "3.3.0",
"@clayui/link": "3.1.0" // links estilizados, usados internamente
},
```

### Trecho 3: o script `deploy`

```jsonc
"scripts": {
// (build, copy-assets, translate e start já vinham no projeto)
// 1) "npm run build" → compila e gera o .jar dentro de dist/
// 2) "&&" → só continua se o build terminou sem erro
// 3) "docker cp" → copia um arquivo do seu computador para dentro de um container
// dist/test-react-1.0.0.jar → ORIGEM: o .jar gerado pelo build
// liferay:/opt/liferay/deploy → DESTINO: container "liferay", pasta de deploy
"deploy": "npm run build && docker cp dist/test-react-1.0.0.jar liferay:/opt/liferay/deploy"
}
```

> ⚠️ **Atenção ao nome do `.jar`: ele está fixo no script.** O arquivo gerado em `dist/` segue o `name` e a `version` do `package.json` (`<name>-<version>.jar`), e o `name` costuma coincidir com o nome da pasta do projeto. Se você renomear a pasta, o `name` ou a `version`, o `docker cp` **não vai achar o arquivo** e o deploy falha. Ajuste o nome no script sempre que isso mudar.

>

> O mesmo vale para o nome do **container** (`liferay`): ele precisa ser igual ao nome do container no seu ambiente.

### O que já vinha pronto

Estas partes só precisam de uma menção, porque já vieram prontas:

- **`devDependencies`:** as ferramentas de build (`liferay-npm-bundler`, Babel, Webpack e afins).

- **Bloco `portlet`:** nome, categoria e papéis do widget.

- **`main`:** o arquivo de entrada (`index.js`).

- **Scripts `build`, `copy-assets`, `translate` e `start`.**

## 2.3 A camada de API (liferay.js)

Este arquivo concentra **todas** as chamadas à API. Tem quatro partes: configuração, funções auxiliares, e as funções de cada recurso.

```javascript
// =====================================================================
// 1. CONFIGURAÇÃO
// =====================================================================
// Caminho base da Headless Delivery API.
// É RELATIVO (começa em "/o/..."), sem "http://localhost:8080", porque este
// código roda DENTRO de uma página da Liferay: o navegador já sabe qual é
// o servidor. Isso também evita problemas de CORS.
const BASE_URL = "/o/headless-delivery/v1.0";
// Devolve o ID do site em que o widget está colocado.
// "Liferay" é um objeto global que a própria Liferay injeta em todas as páginas.
// Diferente do guia de JavaScript (onde o ID 20117 era fixo no código), aqui o ID
// é descoberto na hora, então o widget funciona em qualquer site.
// É uma FUNÇÃO (e não uma constante) para o valor ser lido só quando for usado.
const getSiteId = () => Liferay.ThemeDisplay.getScopeGroupId();
// Credenciais do ambiente LOCAL de treinamento (nunca use credenciais reais aqui).
const EMAIL = "test@liferay.com";
const PASSWORD = "learn";
// =====================================================================
// 2. FUNÇÕES AUXILIARES
// =====================================================================
// Cabeçalho de autenticação Basic. btoa() converte "email:senha" para Base64.
// Os parênteses em volta das chaves "({ ... })" fazem a arrow function
// devolver um OBJETO (sem eles, o JavaScript acharia que "{}" é o corpo da função).
const authHeader = () => ({
	Authorization: "Basic " + btoa(`${EMAIL}:${PASSWORD}`),
});
// Cabeçalhos para requisições que ENVIAM JSON:
// - "...authHeader()" copia (spread) o Authorization para dentro do novo objeto
// - "Content-Type" avisa ao servidor que o corpo está em formato JSON
const jsonHeaders = () => ({
	...authHeader(),
	"Content-Type": "application/json",
});
// Trata a resposta de QUALQUER requisição. Recebe o objeto Response do fetch.
// É "async" porque lê o corpo da resposta, o que também leva tempo.
const handleResponse = async (response) => {
	// O fetch NÃO considera 404/500 como erro (só falha de rede). Por isso
	// checamos response.ok (true para status 200–299) e lançamos o erro nós mesmos.
	if (!response.ok) {
		// Lê o corpo como TEXTO para incluir a mensagem do servidor no erro.
		const text = await response.text();
		// Lançar um erro faz a Promise ser REJEITADA, e quem chamou a função
		// pode capturar isso com try/catch.
		throw new Error(`HTTP ${response.status}: ${text}`);
	}
	// 204 = sucesso SEM corpo (típico do DELETE). Não existe JSON para ler:
	// chamar response.json() aqui lançaria "Unexpected end of JSON input".
	if (response.status === 204) {
		return null;
	}
	// Caso normal: converte o corpo (texto JSON) em objeto JavaScript.
	return response.json();
};
// =====================================================================
// 3. BLOGS
// =====================================================================
// Padrão de todas as funções abaixo: arrow function SEM chaves, ou seja,
// o resultado de "fetch(...).then(handleResponse)" é devolvido direto
// (return implícito). Cada função devolve uma PROMISE, e os componentes
// usam "await" para esperar o resultado.
// GET: lista os blogs do site (endpoint da COLEÇÃO, com o ID do site).
export const getBlogs = () =>
	fetch(`${BASE_URL}/sites/${getSiteId()}/blog-postings`, {
		headers: jsonHeaders(),
	}).then(handleResponse);
// POST: cria um blog. As chaves curtas "{ headline, articleBody }" são o
// atalho para "{ headline: headline, articleBody: articleBody }".
export const createBlog = (headline, articleBody) =>
	fetch(`${BASE_URL}/sites/${getSiteId()}/blog-postings`, {
		method: "POST",
		headers: jsonHeaders(),
		// JSON.stringify converte o objeto em TEXTO, porque o corpo da
		// requisição precisa ser texto.
		body: JSON.stringify({
			headline,
			articleBody,
		}),
	}).then(handleResponse);
// PUT: substitui um blog existente. Note a URL: SEM "sites/{id}", porque
// o id do blog já o identifica (endpoint de UM item).
export const updateBlog = (id, headline, articleBody) =>
	fetch(`${BASE_URL}/blog-postings/${id}`, {
		method: "PUT",
		headers: jsonHeaders(),
		body: JSON.stringify({
			headline,
			articleBody,
		}),
	}).then(handleResponse);
// DELETE: apaga um blog. Sem corpo, então usamos só authHeader()
// (não precisa de Content-Type). A resposta é 204, tratada no handleResponse.
export const deleteBlog = (id) =>
	fetch(`${BASE_URL}/blog-postings/${id}`, {
		method: "DELETE",
		headers: authHeader(),
	}).then(handleResponse);
// =====================================================================
// 4. DOCUMENTS
// =====================================================================
// GET: lista os documentos do site.
export const getDocuments = () =>
	fetch(`${BASE_URL}/sites/${getSiteId()}/documents`, {
		headers: jsonHeaders(),
	}).then(handleResponse);
// POST: envia um ARQUIVO. Aqui a função tem chaves { }, então precisa de
// "return" explícito (há várias instruções antes do fetch).
export const uploadDoc = (file) => {
	// FormData monta um formulário "multipart", o formato para enviar arquivos.
	const formData = new FormData();
	// "file" é o arquivo escolhido pelo usuário (um objeto File do navegador).
	formData.append("file", file);
	// Campos de texto que acompanham o arquivo: título (o próprio nome do
	// arquivo) e uma descrição fixa.
	formData.append("title", file.name);
	formData.append("description", "Uploaded via React App");
	return fetch(`${BASE_URL}/sites/${getSiteId()}/documents`, {
		method: "POST",
		// SÓ authHeader(), SEM Content-Type. O navegador monta sozinho o
		// "multipart/form-data; boundary=..." correto. Se você fixar o
		// Content-Type à mão, o separador some e o upload falha.
		headers: authHeader(),
		// Aqui o corpo é o próprio FormData, e NÃO um JSON.stringify.
		body: formData,
	}).then(handleResponse);
};
// DELETE: apaga um documento pelo id.
export const deleteDoc = (id) =>
	fetch(`${BASE_URL}/documents/${id}`, {
		method: "DELETE",
		headers: authHeader(),
	}).then(handleResponse);
// =====================================================================
// 5. KNOWLEDGE BASE
// =====================================================================
// Mesma lógica dos Blogs. A única diferença é o nome do recurso na URL
// ("knowledge-base-articles") e o campo do título ("title", e não "headline").
// GET: lista os artigos do site.
export const getKBArticles = () =>
	fetch(`${BASE_URL}/sites/${getSiteId()}/knowledge-base-articles`, {
		headers: jsonHeaders(),
	}).then(handleResponse);
// POST: cria um artigo.
export const createKBArticle = (title, articleBody) =>
	fetch(`${BASE_URL}/sites/${getSiteId()}/knowledge-base-articles`, {
		method: "POST",
		headers: jsonHeaders(),
		body: JSON.stringify({
			title,
			articleBody,
		}),
	}).then(handleResponse);
// PUT: substitui um artigo existente.
export const updateKBArticle = (id, title, articleBody) =>
	fetch(`${BASE_URL}/knowledge-base-articles/${id}`, {
		method: "PUT",
		headers: jsonHeaders(),
		body: JSON.stringify({
			title,
			articleBody,
		}),
	}).then(handleResponse);
// DELETE: apaga um artigo.
export const deleteKBArticle = (id) =>
	fetch(`${BASE_URL}/knowledge-base-articles/${id}`, {
		method: "DELETE",
		headers: authHeader(),
	}).then(handleResponse);
```
### Resumo das funções

| Função | Método | URL | Corpo | Observação |
|---|---|---|---|---|
| `getBlogs` | GET | `/sites/{siteId}/blog-postings` | Não | Lista paginada em `data.items` |
| `createBlog` | POST | `/sites/{siteId}/blog-postings` | JSON | `headline` + `articleBody` |
| `updateBlog` | PUT | `/blog-postings/{id}` | JSON | Substitui o recurso |
| `deleteBlog` | DELETE | `/blog-postings/{id}` | Não | Resposta 204 |
| `getDocuments` | GET | `/sites/{siteId}/documents` | Não | |
| `uploadDoc` | POST | `/sites/{siteId}/documents` | **FormData** | Sem `Content-Type` manual |
| `deleteDoc` | DELETE | `/documents/{id}` | Não | Resposta 204 |
| `getKBArticles` | GET | `/sites/{siteId}/knowledge-base-articles` | Não | |
| `createKBArticle` | POST | `/sites/{siteId}/knowledge-base-articles` | JSON | `title` + `articleBody` |
| `updateKBArticle` | PUT | `/knowledge-base-articles/{id}` | JSON | |
| `deleteKBArticle` | DELETE | `/knowledge-base-articles/{id}` | Não | |

## 2.4 O ponto de entrada (index.js)

```javascript
// "React" precisa estar no escopo para o JSX funcionar: o Babel transforma
// "<App />" em uma chamada React.createElement(App).
import React from "react";
// ReactDOM é quem "desenha" os componentes React dentro de uma página HTML.
import ReactDOM from "react-dom";
// O componente raiz da aplicação (o próximo arquivo).
import App from "./App";
// Função que a LIFERAY chama quando o widget aparece na página.
// "export default" é o que torna esta função o ponto de entrada.
//
// O parâmetro "{portletElementId}" é uma DESESTRUTURAÇÃO: a Liferay passa um
// objeto com várias informações do widget, e aqui pegamos só a propriedade
// "portletElementId" (o id do <div> reservado para este widget).
export default function main({ portletElementId }) {
	// ReactDOM.render(O_QUE_DESENHAR, ONDE_DESENHAR)
	ReactDOM.render(
		// 1º argumento: o componente raiz.
		<App />,
		// 2º argumento: o elemento HTML da página onde o React vai desenhar.
		// getElementById procura o <div> pelo id que a Liferay informou.
		document.getElementById(portletElementId),
	);
}
```
> Em React 16, usa-se `ReactDOM.render`. Nas versões mais novas (18+), isso mudou para `createRoot`. O projeto fixa a versão 16.8.6, então `render` é o correto aqui.

## 2.5 O componente raiz (App.js)

```javascript
import React from "react";
// Cada gerenciador é um componente independente, em seu próprio arquivo.
// O nome do arquivo precisa ser IGUAL ao do import (DocsManager, com "s").
// Em sistemas que diferenciam maiúsculas de minúsculas (Linux/Docker),
// "docsmanager" e "DocsManager" são arquivos diferentes.
import BlogManager from "./components/BlogManager";
import DocsManager from "./components/DocsManager";
import KBManager from "./components/KBManager";
// Um componente React é uma FUNÇÃO que devolve JSX (HTML dentro do JavaScript).
export default function App() {
	return (
		// O JSX exige UM único elemento "pai". Este <div> só serve de
		// embrulho para os demais.
		<div>
			<h1>CRUD App</h1>
			{/* <hr /> é uma linha horizontal, só para separar as seções. */}
			<hr />
			{/* Cada tag abaixo "instancia" o componente: o React executa a
função correspondente e desenha o resultado aqui. Cada um tem
o seu PRÓPRIO estado e faz as suas PRÓPRIAS chamadas à API. */}
			<BlogManager />
			<hr />
			<DocsManager />
			<hr />
			<KBManager />
		</div>
	);
}
```
> Os três gerenciadores **não se conhecem**. Cada um carrega os seus dados e atualiza a sua própria lista. É por isso que o `App` não precisa de estado nem de props.

## 2.6 BlogManager.jsx

Este é o componente mais completo e serve de modelo para o `KBManager`. Por isso ele é explicado **por inteiro**.

```jsx
// Hooks do React: useState (guardar valores) e useEffect (rodar código
// depois que o componente aparece na tela).
import React, { useState, useEffect } from "react";
// Componentes visuais do Clay. "ClayForm" é o export padrão do pacote;
// "ClayInput" é um export nomeado, por isso vem entre chaves.
import ClayButton from "@clayui/button";
import ClayForm, { ClayInput } from "@clayui/form";
import ClayTable from "@clayui/table";
// As funções de requisição da camada de API. Os componentes NUNCA chamam
// fetch diretamente: eles só usam estas funções.
import { getBlogs, createBlog, updateBlog, deleteBlog } from "../api/liferay";
export default function BlogManager() {
	// -----------------------------------------------------------------
	// ESTADOS
	// -----------------------------------------------------------------
	// A lista de blogs vinda da API. Começa vazia ([]) até a resposta chegar.
	const [blogs, setBlogs] = useState([]);
	// Os dois campos do formulário. Começam como texto vazio.
	const [headline, setHeadline] = useState("");
	const [body, setBody] = useState("");
	// Guarda o id do blog que está sendo EDITADO. É o que diferencia
	// "criar" de "atualizar":
	// null → o formulário está criando um blog novo
	// número → o formulário está editando o blog com esse id
	const [editId, setEditId] = useState(null);
	// -----------------------------------------------------------------
	// CARREGAR A LISTA
	// -----------------------------------------------------------------
	// Busca os blogs na API e guarda no estado.
	const load = async () => {
		// Espera a resposta. O resultado é o objeto paginado da API:
		// { items: [...], page: 1, pageSize: 20, ... }
		const data = await getBlogs();
		// Os blogs estão em "data.items". O "|| []" é uma segurança: se
		// "items" não existir, usamos uma lista vazia em vez de undefined
		// (que quebraria o ".map" mais abaixo).
		setBlogs(data.items || []);
	};
	// Roda "load" UMA vez, logo que o componente aparece na tela.
	// O array vazio [] no final é o que garante "só uma vez".
	useEffect(() => {
		load();
	}, []);
	// -----------------------------------------------------------------
	// SALVAR (CRIAR OU ATUALIZAR)
	// -----------------------------------------------------------------
	// Chamada ao clicar no botão do formulário.
	const handleSave = async () => {
		// Se existe um editId, estamos EDITANDO: usa PUT (updateBlog).
		if (editId) {
			await updateBlog(editId, headline, body);
		}
		// Senão, estamos CRIANDO: usa POST (createBlog).
		else {
			await createBlog(headline, body);
		}
		// Depois de salvar, limpa o formulário e sai do "modo edição".
		setHeadline("");
		setBody("");
		setEditId(null);
		// Recarrega a lista para a tela mostrar o que mudou no servidor.
		load();
	};
	// -----------------------------------------------------------------
	// EDITAR
	// -----------------------------------------------------------------
	// Chamada ao clicar em "Editar" numa linha da tabela.
	// Recebe o OBJETO do blog daquela linha.
	const handleEdit = (blog) => {
		// Entra no "modo edição": o botão do formulário passa a dizer
		// "Atualizar" e o handleSave passa a usar o PUT.
		setEditId(blog.id);
		// Preenche o formulário com os dados atuais do blog.
		setHeadline(blog.headline);
		// "|| ''" evita passar undefined para um input controlado
		// (o React reclama quando um input alterna entre "sem valor" e "com valor").
		setBody(blog.articleBody || "");
	};
	// -----------------------------------------------------------------
	// EXCLUIR
	// -----------------------------------------------------------------
	// Chamada ao clicar em "Excluir". Recebe só o id.
	const handleDelete = async (id) => {
		await deleteBlog(id);
		// Recarrega a lista para o item apagado sumir da tela.
		load();
	};
	// -----------------------------------------------------------------
	// A TELA (JSX)
	// -----------------------------------------------------------------
	return (
		<div>
			<h2>Blogs</h2>
			{/* ---------- FORMULÁRIO ---------- */}
			<ClayForm>
				{/* ClayForm.Group agrupa um rótulo e o seu campo, com o
espaçamento padrão do Clay. */}
				<ClayForm.Group>
					{/* Em JSX, o atributo "for" do HTML se chama "htmlFor".
Ele liga o rótulo ao input de mesmo "id": clicar no
texto foca o campo, e leitores de tela entendem a relação. */}
					<label htmlFor="blog-headline">Título</label>
					<ClayInput
						id="blog-headline"
						placeholder="Título"
						// value + onChange = INPUT CONTROLADO: o texto exibido
						// vem sempre do estado.
						value={headline}
						// "e" é o evento; "e.target.value" é o texto digitado.
						// Cada tecla atualiza o estado, que redesenha o input.
						onChange={(e) => setHeadline(e.target.value)}
					/>
				</ClayForm.Group>
				<ClayForm.Group>
					<label htmlFor="blog-body">Conteúdo</label>
					<ClayInput
						// "component" troca o elemento HTML que o Clay desenha:
						// aqui vira um <textarea> (campo de várias linhas).
						component="textarea"
						id="blog-body"
						placeholder="Conteúdo"
						value={body}
						onChange={(e) => setBody(e.target.value)}
					/>
				</ClayForm.Group>
				<ClayButton
					// displayType define a cor/estilo do botão (primary = destaque).
					displayType="primary"
					onClick={handleSave}
					// type="button" evita que o botão tente "enviar" o formulário
					// e recarregar a página.
					type="button"
				>
					{/* O texto muda conforme o modo: com editId (editando) mostra
"Atualizar"; sem editId (criando) mostra "Criar". */}
					{editId ? "Atualizar" : "Criar"}
				</ClayButton>
			</ClayForm>
			<hr />
			{/* ---------- TABELA ---------- */}
			{/* As propriedades são estilos do Clay: bordas entre colunas, destaque
ao passar o mouse, rolagem em telas pequenas e linhas zebradas. */}
			<ClayTable borderedColumns hover responsive striped>
				{/* Cabeçalho da tabela */}
				<ClayTable.Head>
					<ClayTable.Row>
						<ClayTable.Cell headingCell>Título</ClayTable.Cell>
						<ClayTable.Cell headingCell>Ações</ClayTable.Cell>
					</ClayTable.Row>
				</ClayTable.Head>
				{/* Corpo: uma linha para cada blog */}
				<ClayTable.Body>
					{/* .map percorre a lista e devolve UMA linha de tabela para
cada blog. O "key" é obrigatório em listas: ajuda o React
a saber qual linha é qual quando a lista muda. Use um
valor único e estável (o id). */}
					{blogs.map((blog) => (
						<ClayTable.Row key={blog.id}>
							<ClayTable.Cell>{blog.headline}</ClayTable.Cell>
							<ClayTable.Cell>
								{/* Grupo de botões, com espaço entre eles. */}
								<ClayButton.Group spaced>
									<ClayButton
										displayType="secondary"
										// Atenção: "() => handleEdit(blog)" e NÃO
										// "handleEdit(blog)". Sem a arrow function,
										// a função seria executada na hora de
										// desenhar a tela, e não no clique.
										onClick={() => handleEdit(blog)}
										small
										type="button"
									>
										Editar
									</ClayButton>
									<ClayButton
										displayType="secondary"
										onClick={() => handleDelete(blog.id)}
										small
										type="button"
									>
										Excluir
									</ClayButton>
								</ClayButton.Group>
							</ClayTable.Cell>
						</ClayTable.Row>
					))}
				</ClayTable.Body>
			</ClayTable>
		</div>
	);
}
```
### O ciclo de vida do componente

| Momento | O que acontece |
|---|---|
| Componente aparece | `useEffect` chama `load()` → `getBlogs()` → `setBlogs(...)` → a tabela é desenhada |
| Usuário digita | `onChange` → `setHeadline` / `setBody` → o input é redesenhado com o novo texto |
| Clica em **Criar** (`editId` é `null`) | `handleSave` → `createBlog` (POST) → limpa o formulário → `load()` |
| Clica em **Editar** numa linha | `handleEdit` → `setEditId(id)` e preenche o formulário → o botão passa a dizer **Atualizar** |
| Clica em **Atualizar** (`editId` tem valor) | `handleSave` → `updateBlog` (PUT) → limpa o formulário e volta a `editId = null` → `load()` |
| Clica em **Excluir** | `handleDelete` → `deleteBlog` (DELETE) → `load()` |

> **Um detalhe importante:** `handleSave` usa **o mesmo botão e o mesmo formulário** para criar e para atualizar. O `editId` é o "interruptor" entre os dois modos.

## 2.7 DocsManager.jsx

O `DocsManager` segue a mesma ideia do `BlogManager` (estado, `load`, `useEffect`, tabela), mas com **três diferenças**:

1. Em vez de título e conteúdo, o formulário tem um **seletor de arquivo**.

2. **Não há edição** (não existe `editId`, nem `handleEdit`, nem botão **Editar**).

3. Usa **`useRef`** para controlar um `<input type="file">` escondido.

```jsx
// Aqui entra também o useRef.
import React, { useState, useEffect, useRef } from "react";
import ClayButton from "@clayui/button";
import ClayForm, { ClayInput } from "@clayui/form";
import ClayTable from "@clayui/table";
// Só três funções: não há "update" para documentos neste app.
import { getDocuments, uploadDoc, deleteDoc } from "../api/liferay";
export default function DocsManager() {
	// Lista de documentos vinda da API.
	const [documents, setDocuments] = useState([]);
	// O arquivo ESCOLHIDO pelo usuário (um objeto File), ou null se nenhum.
	const [file, setFile] = useState(null);
	// Referência ao <input type="file"> real. Diferente do useState, mudar
	// uma ref NÃO redesenha o componente. Usamos para dar comandos ao elemento
	// (abrir o seletor, limpar o valor).
	const fileInputRef = useRef(null);
	// Igual ao BlogManager: busca a lista e guarda no estado.
	const load = async () => {
		const data = await getDocuments();
		setDocuments(data.items || []);
	};
	useEffect(() => {
		load();
	}, []);
	// Chamada ao clicar em "Upload".
	const handleUpload = async () => {
		// Proteção: se não há arquivo, não faz nada. (O botão já fica
		// desabilitado nesse caso, mas é bom garantir também no código.)
		if (!file) {
			return;
		}
		// Envia o arquivo (POST multipart, em liferay.js).
		await uploadDoc(file);
		// Limpa o estado do arquivo escolhido...
		setFile(null);
		// ...e também o <input> real. O input de arquivo é "teimoso": o
		// navegador não deixa o React controlá-lo com "value", então
		// zeramos o valor direto pela ref. Sem isso, escolher o MESMO
		// arquivo de novo não dispararia o onChange.
		// O "if" evita erro caso a ref ainda não aponte para nada.
		if (fileInputRef.current) {
			fileInputRef.current.value = "";
		}
		load();
	};
	const handleDelete = async (id) => {
		await deleteDoc(id);
		load();
	};
	return (
		<div>
			<h2>Documents and Media</h2>
			<ClayForm>
				<ClayForm.Group>
					<label htmlFor="document-file">Arquivo</label>
					{/* O <input type="file"> padrão do navegador é difícil de
estilizar. A solução: ESCONDÊ-LO (display: none) e
abrir o seletor por um botão do Clay, via ref. */}
					<input
						id="document-file"
						// Liga este elemento à variável fileInputRef.
						ref={fileInputRef}
						style={{ display: "none" }}
						type="file"
						// e.target.files é a lista de arquivos escolhidos;
						// pegamos o primeiro ([0]) e guardamos no estado.
						onChange={(e) => setFile(e.target.files[0])}
					/>
					{/* ClayInput.Group coloca itens lado a lado (campo + botão). */}
					<ClayInput.Group>
						{/* Este item ocupa o espaço restante da linha. */}
						<ClayInput.GroupItem>
							{/* Campo SOMENTE LEITURA, só para mostrar o nome. */}
							<ClayInput
								readOnly
								// Operador ternário: se há arquivo, mostra o nome
								// dele; senão, mostra o aviso.
								value={file ? file.name : "Nenhum arquivo selecionado"}
							/>
						</ClayInput.GroupItem>
						{/* "shrink" faz este item ocupar só o tamanho do botão. */}
						<ClayInput.GroupItem shrink>
							<ClayButton
								displayType="secondary"
								// Simula um clique no <input> escondido, o que
								// abre a janela de escolha de arquivo.
								onClick={() => fileInputRef.current.click()}
								type="button"
							>
								Selecionar arquivo
							</ClayButton>
						</ClayInput.GroupItem>
					</ClayInput.Group>
				</ClayForm.Group>
				<ClayButton
					// Desabilitado enquanto não houver arquivo escolhido.
					disabled={!file}
					displayType="primary"
					onClick={handleUpload}
					type="button"
				>
					Upload
				</ClayButton>
			</ClayForm>
			<hr />
			<ClayTable borderedColumns hover responsive striped>
				<ClayTable.Head>
					<ClayTable.Row>
						<ClayTable.Cell headingCell>Nome</ClayTable.Cell>
						<ClayTable.Cell headingCell>Ações</ClayTable.Cell>
					</ClayTable.Row>
				</ClayTable.Head>
				<ClayTable.Body>
					{documents.map((doc) => (
						<ClayTable.Row key={doc.id}>
							<ClayTable.Cell>
								{/* Mostra o título; se não existir, usa o nome
do arquivo. O "||" devolve o primeiro valor
que não for vazio/undefined. */}
								{doc.title || doc.fileName}
							</ClayTable.Cell>
							{/* Só um botão (Excluir): não há edição, então não
precisa do ClayButton.Group. */}
							<ClayTable.Cell>
								<ClayButton
									displayType="secondary"
									onClick={() => handleDelete(doc.id)}
									small
									type="button"
								>
									Excluir
								</ClayButton>
							</ClayTable.Cell>
						</ClayTable.Row>
					))}
				</ClayTable.Body>
			</ClayTable>
		</div>
	);
}
```
### O que muda em relação ao BlogManager

| Ponto | BlogManager | DocsManager |
|---|---|---|
| Estados do formulário | `headline`, `body`, `editId` | `file` |
| Campo de entrada | `ClayInput` de texto e `textarea` | `<input type="file">` escondido + botão que o aciona |
| Salvar | `handleSave` (cria **ou** atualiza) | `handleUpload` (só envia) |
| Editar | Sim (`handleEdit` + `editId`) | **Não** |
| Hook extra | Nenhum | `useRef` |
| Corpo da requisição | JSON | `FormData` (multipart) |
| Texto da linha | `blog.headline` | `doc.title \|\| doc.fileName` |

## 2.8 KBManager.jsx

O `KBManager` é **praticamente uma cópia** do `BlogManager`: a estrutura, o ciclo de vida e a lógica de criar/atualizar/excluir são idênticos. O que muda é só o **nome dos campos e das funções**, porque a API da Knowledge Base usa `title` em vez de `headline`.

### O que muda em relação ao BlogManager

| Ponto | BlogManager | KBManager |
|---|---|---|
| Nome do componente | `BlogManager` | `KBManager` |
| Lista no estado | `blogs` / `setBlogs` | `articles` / `setArticles` |
| Campo do título | `headline` / `setHeadline` | `title` / `setTitle` |
| Campo do conteúdo | `body` / `setBody` | `content` / `setContent` |
| Funções da API | `getBlogs`, `createBlog`, `updateBlog`, `deleteBlog` | `getKBArticles`, `createKBArticle`, `updateKBArticle`, `deleteKBArticle` |
| Campo lido na API | `blog.headline` | `article.title` |
| `id`s dos campos | `blog-headline`, `blog-body` | `kb-title`, `kb-content` |
| Título da seção (`<h2>`) | `Blogs` | `Knowledge Base` |

### Os trechos que mudam, comentados

```jsx
// IMPORTS: as funções da API agora são as da Knowledge Base.
import {
	getKBArticles,
	createKBArticle,
	updateKBArticle,
	deleteKBArticle,
} from "../api/liferay";
export default function KBManager() {
	// Lista de artigos (no BlogManager era "blogs").
	const [articles, setArticles] = useState([]);
	// "title" e "content" no lugar de "headline" e "body".
	const [title, setTitle] = useState("");
	const [content, setContent] = useState("");
	// Mesmo papel do BlogManager: null = criando, número = editando.
	const [editId, setEditId] = useState(null);
	const load = async () => {
		const data = await getKBArticles();
		setArticles(data.items || []);
	};
	// handleSave: a MESMA lógica, trocando só as funções chamadas.
	const handleSave = async () => {
		if (editId) {
			await updateKBArticle(editId, title, content);
		} else {
			await createKBArticle(title, content);
		}
		setTitle("");
		setContent("");
		setEditId(null);
		load();
	};
	// handleEdit: o título do artigo vem de "article.title"
	// (no blog era "blog.headline"). O corpo continua sendo "articleBody"
	// nos dois recursos.
	const handleEdit = (article) => {
		setEditId(article.id);
		setTitle(article.title);
		setContent(article.articleBody || "");
	};
	// ... handleDelete, o JSX do formulário e a tabela seguem o mesmo
	// formato do BlogManager, trocando os nomes conforme a tabela acima.
}
```
> Repare que `useEffect`, `handleDelete` e todo o JSX seguem o mesmo molde. Se você entendeu o `BlogManager`, entendeu o `KBManager`.

## 2.9 Pontos de melhoria

Depois que tudo funcionar, estas melhorias deixam o código mais profissional (e costumam ser bem vistas em revisões):

### 1. Credenciais fixas no front-end

`test@liferay.com:learn` está escrito no código, e **tudo o que vai para o navegador é visível** para quem abrir as ferramentas do desenvolvedor. Dentro de um widget, o usuário **já está logado**, então dá para aproveitar a sessão dele, com o token anti-CSRF da própria Liferay:

```javascript
// Em vez de enviar email:senha (Basic Auth), usa a SESSÃO do usuário logado.
// "Liferay.authToken" é um token que a Liferay gera para proteger contra
// ataques CSRF. A API aceita esse token no cabeçalho "x-csrf-token".
const authHeader = () => ({
	"x-csrf-token": Liferay.authToken,
});
// O cookie de sessão vai junto automaticamente, porque a URL é relativa
// (mesma origem). Assim, as credenciais SAEM do código.
```
### 2. Sem tratamento de erro nos componentes

O `handleResponse` **lança** o erro, mas nenhum componente faz `try/catch`. Se a API falhar, o erro some no console e a tela não mostra nada (e, pior, o formulário é limpo como se tivesse dado certo). Exemplo para o `BlogManager`:

```jsx
// Novo estado para guardar a mensagem de erro (null = sem erro).
const [error, setError] = useState(null);
const handleSave = async () => {
	// try: tenta fazer o que está dentro...
	try {
		if (editId) {
			await updateBlog(editId, headline, body);
		} else {
			await createBlog(headline, body);
		}
		// Só limpa o formulário se TUDO deu certo.
		setHeadline("");
		setBody("");
		setEditId(null);
		setError(null);
		load();
	} catch (err) {
		// ...e se qualquer await acima lançar erro, o fluxo cai aqui.
		setError(err.message);
	}
};
// E no JSX, mostra o erro (se houver) com o ClayAlert, que já está instalado:
// {error && <ClayAlert displayType="danger" title="Erro">{error}</ClayAlert>}
```
### 3. Dependências instaladas e não usadas

`@clayui/alert`, `@clayui/modal` e `@clayui/nav` estão no `package.json`, mas nenhum componente os importa. Duas oportunidades diretas:

- **`ClayAlert`:** exibir os erros do item anterior.

- **`ClayModal`:** pedir confirmação antes de **excluir** (hoje o clique em "Excluir" apaga na hora).

Se não forem usadas, o ideal é removê-las do `package.json`.

### 4. Sem paginação

O `GET` devolve só a **primeira página** (em geral, 20 itens). Se existirem mais, eles não aparecem. A resposta traz `page`, `pageSize`, `totalCount` e `lastPage`, e a API aceita parâmetros como `?page=2&pageSize=20`, o que permite criar botões de "próxima página".

### 5. Documents sem edição e sem link para o arquivo

O `DocsManager` só lista, envia e exclui. Dá para acrescentar a edição e um link para **abrir/baixar** o arquivo (a resposta da API traz a URL de conteúdo do documento). Vale também conferir no **API Explorer** (`/o/api`) da sua versão como a API espera os metadados do upload (título, descrição), já que o código atual os envia como campos soltos do formulário.

### 6. Código repetido entre `BlogManager` e `KBManager`

Os dois componentes são quase idênticos. Uma solução é extrair um componente genérico (por exemplo, `ContentManager`) que receba por props os nomes dos campos e as funções da API. Outra é criar um **hook próprio** (`useCrud`) com o `load`, o `handleSave` e o `handleDelete`.

### 7. Deploy com valores fixos

O script `deploy` tem o nome do `.jar` e do container escritos à mão. Uma alternativa é ler o nome e a versão do próprio `package.json` (o npm expõe essas informações como variáveis de ambiente durante o script):

```jsonc
// (comentário só para explicação)
// $npm_package_name e $npm_package_version são preenchidos pelo npm com os
// valores do próprio package.json. Em Linux/macOS (shell sh/bash) isso funciona;
// no Windows (cmd), a sintaxe de variável é outra (%npm_package_name%).
"deploy": "npm run build && docker cp dist/$npm_package_name-$npm_package_version.jar liferay:/opt/liferay/deploy"
```

Assim, renomear o projeto ou mudar a versão não quebra mais o deploy.

### Para praticar

1. Adicione o `try/catch` com `ClayAlert` ao `BlogManager` (item 2).

2. Use o `ClayModal` para pedir confirmação antes de excluir um blog.

3. Troque o Basic Auth pelo `x-csrf-token` no `liferay.js` (item 1) e confirme que o CRUD continua funcionando.

4. Extraia um componente genérico que substitua `BlogManager` e `KBManager`.

