# React: JSX, componentes e props

Este material apresenta os conceitos necessários para ler e organizar
componentes React, com exemplos próximos de um widget da Liferay que
exibe listas e formulários para consumir uma API.

O foco é entender **como o JSX representa a interface**, **como dividir
a interface em componentes** e **como passar informações e funções entre
componentes**. O estado e os efeitos dos Hooks são assuntos
relacionados, mas ficam para um material específico.

> **Compatibilidade:** o projeto de referência utiliza React `16.8.6`.
> Os exemplos deste documento evitam recursos que exigem versões mais
> recentes, como os recursos introduzidos no React 18.

## 1. O que é um componente React?

Um **componente** é uma parte reutilizável da interface. Ele pode
representar uma página inteira, uma seção, uma lista, um formulário ou
um botão com comportamento próprio.

Em React, componentes funcionais são funções JavaScript que retornam
elementos que o React pode renderizar.

``` jsx
import React from "react";

export default function Welcome() {
    return <h1>Bem-vindo!</h1>;
}
```

-   `function Welcome()` declara uma função chamada `Welcome`.
-   `return` informa o que o componente deve exibir.
-   `<h1>Bem-vindo!</h1>` é JSX.
-   `export default` permite importar esse componente em outro arquivo.

Por convenção, o nome de um componente React começa com letra maiúscula.
Isso ajuda o JSX a diferenciar componentes definidos pela aplicação de
elementos HTML nativos.

``` jsx
<h1>Olá</h1>        // Elemento HTML
<Welcome />         // Componente React
```

O React interpreta `<Welcome />` como uma referência ao componente
`Welcome`, e não como uma tag HTML chamada `welcome`.

## 2. O que é JSX?

**JSX** é uma sintaxe que permite escrever uma estrutura parecida com
HTML dentro do JavaScript. Ela é transformada durante o processo de
build; o navegador não executa JSX diretamente.

``` jsx
const title = "Gerenciador de blogs";

function Header() {
    return <h1>{title}</h1>;
}
```

O JSX permite expressar a estrutura visual de forma declarativa:
descrevemos o que a interface deve representar, e o React cuida de
atualizar a interface quando os dados relevantes mudam.

### 2.1 Expressões JavaScript dentro do JSX

Chaves `{}` permitem inserir uma **expressão JavaScript** em um trecho
de JSX.

``` jsx
const userName = "Ana";
const total = 3;

function Summary() {
    return (
        <section>
            <h2>Olá, {userName}!</h2>
            <p>Você tem {total} itens.</p>
            <p>Próximo total: {total + 1}</p>
        </section>
    );
}
```

Dentro das chaves, podemos usar variáveis, operações e chamadas de
funções que retornem um valor apropriado para exibição.

``` jsx
<p>{userName.toUpperCase()}</p>
<p>{total > 0 ? "Há itens disponíveis" : "A lista está vazia"}</p>
```

**Atenção:** chaves não significam que qualquer instrução JavaScript
pode ser escrita diretamente ali. Por exemplo, um `if` tradicional ou um
`for` não é usado como expressão dentro do JSX. Para decidir o que
exibir, normalmente usamos uma expressão condicional ou preparamos o
valor antes do `return`.

### 2.2 Diferenças entre JSX e HTML

Embora se pareça com HTML, JSX possui algumas diferenças importantes.

  -----------------------------------------------------------------------
  HTML                                JSX
  ----------------------------------- -----------------------------------
  `class="panel"`                     `className="panel"`

  `for="title"` em um `label`         `htmlFor="title"`

  Alguns elementos podem ser escritos Elementos vazios devem ser
  sem fechamento                      fechados, como `<input />`

  Atributos de eventos usam           Eventos React usam nomes como
  convenções HTML                     `onClick` e recebem funções
  -----------------------------------------------------------------------

Exemplo:

``` jsx
<label htmlFor="headline">Título</label>
<input id="headline" className="form-control" />
```

Os nomes `className` e `htmlFor` são propriedades usadas pelo JSX/React;
não são nomes de classes ou identificadores especiais da Liferay.

### 2.3 Um único elemento raiz

Em versões e configurações tradicionais do React, o `return` de um
componente precisa apresentar um único elemento raiz. Se a interface
tiver vários elementos irmãos, podemos envolvê-los em um elemento como
`div`.

``` jsx
function BlogSection() {
    return (
        <div>
            <h2>Blog postings</h2>
            <p>Gerencie as publicações do site.</p>
        </div>
    );
}
```

Também é possível usar um **Fragment** para agrupar elementos sem
adicionar uma `div` extra ao HTML renderizado:

``` jsx
function BlogSection() {
    return (
        <React.Fragment>
            <h2>Blog postings</h2>
            <p>Gerencie as publicações do site.</p>
        </React.Fragment>
    );
}
```

O Fragment é útil quando o agrupamento é necessário para o JSX, mas não
queremos criar um elemento visual adicional no DOM.

## 3. Componentes e composição

Uma aplicação React pode ser organizada como uma árvore de componentes.
Um componente maior usa outros componentes menores para compor a
interface.

Por exemplo, um widget que gerencia publicações pode ter esta estrutura:

``` text
App
├── BlogManager
│   ├── formulário de publicação
│   └── lista de publicações
├── DocsManager
└── KBManager
```

Essa árvore é apenas uma representação conceitual. No código, os
componentes podem estar em arquivos diferentes e ser importados conforme
a organização do projeto.

Um componente raiz pode compor os gerenciadores assim:

``` jsx
import React from "react";
import BlogManager from "./components/BlogManager";
import DocsManager from "./components/DocsManager";
import KBManager from "./components/KBManager";

export default function App() {
    return (
        <div>
            <h1>CRUD App</h1>
            <BlogManager />
            <hr />
            <DocsManager />
            <hr />
            <KBManager />
        </div>
    );
}
```

Observe que `App` não precisa conter toda a lógica de cada gerenciador.
Ele organiza a interface e inclui os componentes responsáveis por suas
próprias seções.

### 3.1 Por que dividir em componentes?

Dividir a interface pode ajudar a:

-   **Separar responsabilidades:** cada componente fica responsável por
    uma parte mais específica da interface.
-   **Reutilizar código:** um componente genérico pode ser utilizado em
    mais de um lugar.
-   **Facilitar a leitura:** componentes menores tornam mais fácil
    localizar a lógica de uma seção.
-   **Facilitar a manutenção:** mudanças em uma seção podem ficar
    concentradas no componente correspondente.

A divisão não precisa ser exagerada. Vale criar um componente quando ele
representa uma parte compreensível da interface ou quando a reutilização
e a separação de responsabilidades trazem benefício real.

## 4. Props: passando informações para componentes

**Props** (abreviação de *properties*) são informações que um componente
recebe de seu componente pai. Elas permitem configurar um componente sem
precisar codificar todos os valores diretamente nele.

``` jsx
import React from "react";

function WelcomeMessage(props) {
    return <p>Olá, {props.name}!</p>;
}

export default function App() {
    return (
        <div>
            <WelcomeMessage name="Ana" />
            <WelcomeMessage name="Carlos" />
        </div>
    );
}
```

O componente `App` passa a propriedade `name` para cada instância de
`WelcomeMessage`. O componente filho lê o valor por meio de
`props.name`.

### 4.1 Desestruturação de props

Em vez de escrever `props.name`, podemos extrair a propriedade
diretamente nos parâmetros da função:

``` jsx
function WelcomeMessage({ name }) {
    return <p>Olá, {name}!</p>;
}
```

`{ name }` nesse contexto é **desestruturação de objeto**: a função
recebe um objeto de props e extrai a propriedade `name`.

É a mesma ideia de desestruturação usada em JavaScript:

``` javascript
const user = { name: "Ana", role: "editor" };
const { name } = user;

console.log(name); // "Ana"
```

### 4.2 Passando valores e funções

Props podem conter textos, números, booleanos, objetos, arrays e
funções.

``` jsx
function BlogSummary({ headline, commentCount }) {
    return (
        <p>
            {headline} — comentários: {commentCount}
        </p>
    );
}

function App() {
    return (
        <BlogSummary
            headline="Introdução ao React"
            commentCount={4}
        />
    );
}
```

Para passar um valor JavaScript como número, booleano ou variável,
usamos chaves:

``` jsx
<BlogSummary headline={currentHeadline} commentCount={4} />
```

Sem chaves, `commentCount="4"` passaria o texto `"4"`, enquanto
`commentCount={4}` passaria o número `4`.

Uma função também pode ser passada como prop:

``` jsx
function DeleteButton({ onDelete }) {
    return <button onClick={onDelete}>Excluir</button>;
}

function App() {
    function handleDelete() {
        console.log("Solicitação de exclusão");
    }

    return <DeleteButton onDelete={handleDelete} />;
}
```

Aqui, `App` fornece a função, e `DeleteButton` decide em qual evento ela
será executada. Essa técnica é comum quando um componente filho precisa
comunicar uma ação ao componente pai.

### 4.3 Props não devem ser alteradas pelo componente filho

Props devem ser tratadas como dados somente para leitura. Um componente
não deve tentar modificar diretamente o valor recebido.

``` jsx
function Greeting({ name }) {
    // Não faça: name = "Outro nome";
    return <p>Olá, {name}!</p>;
}
```

Se um valor precisa mudar durante o uso da interface, ele normalmente
deve ser gerenciado como estado pelo componente responsável por essa
informação. O tema de estado e Hooks será aprofundado em um material
próprio.

## 5. Renderização condicional

Muitas interfaces precisam mostrar conteúdos diferentes dependendo de
uma condição. Em JSX, isso costuma ser feito com o operador ternário ou
com `&&`.

### 5.1 Operador ternário

``` jsx
function ResultMessage({ hasItems }) {
    return (
        <p>
            {hasItems ? "Existem itens cadastrados." : "Nenhum item encontrado."}
        </p>
    );
}
```

A forma geral é:

``` javascript
condicao ? valorSeVerdadeiro : valorSeFalso
```

O ternário é útil quando existem duas alternativas claras.

### 5.2 Operador `&&`

``` jsx
function ErrorMessage({ error }) {
    return (
        <div>
            {error && <p className="alert alert-danger">{error}</p>}
        </div>
    );
}
```

Se `error` for verdadeiro, o parágrafo será renderizado. Se não houver
erro, a expressão não produzirá o parágrafo.

**Cuidado com números:** `count && <p>...</p>` pode exibir `0` quando
`count` for zero, porque `0` é um valor falsy, mas também é renderizável
pelo React como texto. Para uma condição baseada em quantidade, prefira
`count > 0 && <p>...</p>`.

## 6. Renderizando listas com `map()`

É comum receber uma lista de objetos de uma API e transformá-la em
elementos da interface. O método JavaScript `map()` percorre um array e
devolve um novo array com o resultado de uma função aplicada a cada
item.

Imagine que a API devolva:

``` javascript
const blogs = [
    { id: 101, headline: "Introdução ao React" },
    { id: 102, headline: "Componentes na prática" }
];
```

Podemos renderizar a lista com JSX:

``` jsx
function BlogList({ blogs }) {
    return (
        <ul>
            {blogs.map((blog) => (
                <li key={blog.id}>{blog.headline}</li>
            ))}
        </ul>
    );
}
```

O que acontece:

1.  `blogs.map(...)` percorre os itens do array.
2.  Para cada `blog`, a função retorna um elemento `<li>`.
3.  O React renderiza os elementos resultantes dentro do `<ul>`.
4.  `key={blog.id}` fornece uma identificação estável para cada item da
    lista.

### 6.1 Por que a propriedade `key` é importante?

A `key` ajuda o React a identificar os elementos de uma lista entre
renderizações. Assim, ele consegue reconhecer quais itens foram
mantidos, removidos ou adicionados.

Prefira um identificador estável do próprio registro, como o `id`
retornado pela API:

``` jsx
{blogs.map((blog) => (
    <li key={blog.id}>{blog.headline}</li>
))}
```

Evite usar o índice do array como `key` quando a lista pode mudar de
ordem, receber itens novos ou ter itens removidos. Nessas situações, o
índice pode passar a representar outro registro e dificultar a
reconciliação correta da interface.

A `key` é uma informação usada pelo React; ela não é passada
automaticamente ao componente como uma prop comum. Se o componente
também precisar do identificador, passe-o separadamente:

``` jsx
{blogs.map((blog) => (
    <BlogRow key={blog.id} blogId={blog.id} headline={blog.headline} />
))}
```

## 7. Eventos: responder às ações do usuário

Um componente pode executar uma função quando o usuário clica em um
botão, digita em um campo ou envia um formulário.

``` jsx
function RefreshButton() {
    function handleClick() {
        console.log("Atualizar lista");
    }

    return <button onClick={handleClick}>Atualizar</button>;
}
```

O atributo React `onClick` recebe uma **referência para a função**.

### 7.1 Passar uma função não é o mesmo que executá-la

Correto:

``` jsx
<button onClick={handleClick}>Atualizar</button>
```

Incorreto para este caso:

``` jsx
<button onClick={handleClick()}>Atualizar</button>
```

Na primeira forma, o React recebe a função e pode executá-la quando
ocorrer o clique. Na segunda, `handleClick()` é executada imediatamente
durante a renderização, e seu resultado é passado para `onClick`.

Se for necessário passar um argumento, podemos usar uma função
intermediária:

``` jsx
function DeleteButton({ blogId, onDelete }) {
    return (
        <button onClick={() => onDelete(blogId)}>
            Excluir
        </button>
    );
}
```

Nesse exemplo, a arrow function só chama `onDelete(blogId)` quando o
usuário clica.

## 8. Formulários e campos de entrada

Em formulários React, é importante distinguir o valor que o usuário
digita da função que será executada quando ele enviar os dados.

Um campo pode ser ligado ao estado do componente por meio de `value` e
`onChange`. Esse padrão é chamado de **componente controlado**. Como o
estado e o Hook `useState` merecem uma explicação própria, o exemplo
abaixo destaca apenas a estrutura e os eventos.

``` jsx
function BlogForm({ headline, onHeadlineChange, onSubmit }) {
    return (
        <form onSubmit={onSubmit}>
            <label htmlFor="headline">Título</label>
            <input
                id="headline"
                value={headline}
                onChange={(event) => onHeadlineChange(event.target.value)}
            />

            <button type="submit">Salvar</button>
        </form>
    );
}
```

-   `value={headline}` define o valor exibido no campo.
-   `onChange` executa quando o usuário altera o campo.
-   `event.target.value` contém o texto atual digitado.
-   `onSubmit` é a função que o componente pai fornece para tratar o
    envio.

Em um formulário real, o tratamento de envio normalmente deve impedir o
comportamento padrão do navegador quando a aplicação controla o envio,
validar os dados e apresentar os estados de sucesso ou erro. A validação
e o gerenciamento desses estados podem ser estudados em materiais
separados.

## 9. Como esses conceitos se relacionam com o widget da Liferay?

No widget que gerencia recursos da Headless Delivery API, os componentes
podem ser organizados conforme suas responsabilidades:

-   `App.js` compõe as seções principais, como `BlogManager`,
    `DocsManager` e `KBManager`.
-   `BlogManager.jsx` descreve a interface de gerenciamento de
    publicações.
-   O JSX define os campos, botões, mensagens e listas que aparecem na
    página.
-   Props permitem que componentes menores recebam os dados e as funções
    de que precisam.
-   `map()` transforma arrays de registros em listas de elementos.
-   Eventos como `onClick`, `onChange` e `onSubmit` conectam as ações do
    usuário às funções JavaScript.
-   A camada de API, como `src/api/liferay.js`, fica responsável pelas
    requisições HTTP; os componentes usam suas funções em vez de
    misturar toda a comunicação de rede com a marcação da interface.

Esse arranjo é uma separação de responsabilidades, não uma regra rígida.
O importante é conseguir identificar onde a interface é descrita, de
onde vêm os dados e qual função executa cada ação.

## 10. Erros comuns para reconhecer

  -----------------------------------------------------------------------
  Erro                    Por que é um problema   Alternativa
  ----------------------- ----------------------- -----------------------
  Usar `class` no JSX     JSX espera `className`  `className="..."`

  Esquecer o fechamento   Elementos vazios devem  `<input />`
  de `<input>`            ser fechados em JSX     

  Usar `<blogManager />`  Nome em minúscula é     `<BlogManager />`
                          interpretado como tag   
                          nativa                  

  Esquecer a `key` em     O React não recebe uma  `key={item.id}`
  listas                  identificação estável   
                          para cada item          

  Escrever                Executa a função        `onClick={save}`
  `onClick={save()}`      durante a renderização  

  Tentar alterar uma prop Props são dados         Gerenciar a mudança no
                          recebidos pelo          componente responsável
                          componente              pelo estado

  Colocar vários          É necessário um         Envolver com `<div>` ou
  elementos irmãos        elemento raiz ou        `<React.Fragment>`
  diretamente no `return` Fragment                
  -----------------------------------------------------------------------

## 11. Resumo

-   **Componente:** função que descreve uma parte da interface.
-   **JSX:** sintaxe parecida com HTML que permite descrever a interface
    dentro do JavaScript.
-   **Expressão JSX:** código JavaScript inserido entre chaves `{}`.
-   **Composição:** uso de componentes dentro de outros componentes para
    formar a aplicação.
-   **Props:** dados e funções recebidos de um componente pai.
-   **Renderização condicional:** escolha do que exibir com base em uma
    condição.
-   **`map()`:** transforma os itens de um array em elementos para
    renderizar uma lista.
-   **`key`:** identificação estável dos elementos de uma lista.
-   **Eventos:** funções associadas a ações como clique, alteração de
    campo e envio de formulário.
-   **Componente controlado:** campo cujo valor é determinado pelo
    estado React.

## Recursos adicionais

### React e JSX

-   [Aprenda React --- React](https://react.dev/learn): introdução a
    componentes, JSX, props e renderização.
-   [JavaScript in JSX with Curly
    Braces](https://react.dev/learn/javascript-in-jsx-with-curly-braces):
    uso de expressões JavaScript dentro do JSX.
-   [Your First
    Component](https://react.dev/learn/your-first-component): declaração
    e composição de componentes.
-   [Passing Props to a
    Component](https://react.dev/learn/passing-props-to-a-component):
    como passar dados e funções por props.
-   [Conditional
    Rendering](https://react.dev/learn/conditional-rendering):
    renderização condicional.
-   [Rendering Lists](https://react.dev/learn/rendering-lists): listas,
    `map()` e `key`.
-   [Responding to
    Events](https://react.dev/learn/responding-to-events): eventos e
    manipuladores de eventos.

> A documentação atual do React pode mostrar recursos que não existem na
> versão `16.8.6` utilizada pelo projeto. Confira as versões de `react`
> e `react-dom` no `package.json` antes de aplicar exemplos. Os
> conceitos básicos deste material são adequados para compreender a
> estrutura do projeto, mas exemplos da documentação moderna podem
> exigir adaptações.