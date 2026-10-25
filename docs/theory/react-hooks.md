# React Hooks e Renderização

## 1. Introdução

O React é uma biblioteca JavaScript utilizada para construir interfaces de usuário a partir de componentes. Para que esses componentes possam responder às interações do usuário e às mudanças nos dados, o React oferece mecanismos para controlar o estado e executar operações relacionadas ao ciclo de vida dos componentes.

Os **Hooks** são funções especiais que permitem utilizar recursos do React em componentes funcionais. Entre os Hooks mais utilizados estão:

- `useState`: permite adicionar estado a um componente.
- `useEffect`: permite executar efeitos colaterais relacionados à renderização.
- `useRef`: permite manter uma referência entre renderizações sem provocar uma nova renderização quando seu valor muda.

Esses recursos são importantes para compreender como uma aplicação React busca dados de uma API, atualiza suas informações e apresenta os resultados na interface.

Este material utiliza conceitos compatíveis com o React 16.8.6, versão que introduziu os Hooks.

## 2. O que é renderização?

**Renderização** (*rendering*) é o processo pelo qual um componente React calcula a interface que deve apresentar com base em suas propriedades, seu estado e sua lógica.

Considere o seguinte componente:

```jsx
function Greeting() {
    return <h1>Olá, mundo!</h1>;
}
```

Quando o React renderiza esse componente, ele determina que a interface deve apresentar o texto `Olá, mundo!`.

Entretanto, uma aplicação não costuma exibir informações estáticas. É comum que a interface dependa de dados recebidos de uma API, de informações inseridas pelo usuário ou de alterações no estado do componente.

Por isso, precisamos compreender como o React reage às mudanças.

### 2.1. Renderização inicial

A renderização inicial acontece quando o componente é apresentado pela primeira vez.

```jsx
function Counter() {
    return <p>Contador: 0</p>;
}
```

Nesse exemplo, o componente apresenta o valor `0`.

Como o valor está escrito diretamente no JSX, não existe um mecanismo para alterá-lo por meio do estado do React.

### 2.2. Nova renderização

Quando o estado de um componente muda, o React normalmente agenda uma nova renderização para calcular a interface atualizada.

Isso permite que a interface acompanhe as mudanças dos dados sem que seja necessário manipular manualmente cada elemento do DOM.

É importante distinguir duas coisas:

- **Renderizar novamente:** executar novamente a lógica de renderização do componente.
- **Atualizar o DOM:** aplicar as mudanças necessárias à interface apresentada no navegador.

Uma nova renderização não significa necessariamente que todos os elementos do DOM serão recriados. O React compara o resultado e determina quais alterações são necessárias.

## 3. `useState`: estado do componente

O Hook `useState` permite que um componente funcional mantenha informações que podem mudar durante sua utilização.

Sua sintaxe básica é:

```jsx
const [state, setState] = useState(initialValue);
```

Os elementos dessa declaração são:

- `state`: variável que representa o valor atual do estado naquela renderização.
- `setState`: função utilizada para solicitar uma atualização do estado.
- `initialValue`: valor inicial do estado.

O nome `state` pode ser substituído por um nome mais específico, assim como o nome `setState`.

### 3.1. Exemplo de contador

```jsx
import React, { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Contador: {count}</p>

            <button onClick={() => setCount(count + 1)}>
                Incrementar
            </button>
        </div>
    );
}

export default Counter;
```

Nesse exemplo, `count` começa com o valor `0`.

Quando o usuário clica no botão, a função `setCount` solicita que o estado seja atualizado. O React então renderiza novamente o componente, e a interface passa a apresentar o novo valor.

O fluxo é:

1. O componente é renderizado com `count = 0`.
2. O usuário clica no botão.
3. `setCount` solicita a atualização do estado.
4. O React renderiza novamente o componente.
5. O texto apresenta o valor atualizado.

### 3.2. Por que não alterar a variável diretamente?

Considere uma variável JavaScript comum:

```javascript
let count = 0;

count = count + 1;
```

Essa atribuição altera o valor da variável, mas não informa ao React que a interface precisa ser atualizada.

Com `useState`, a atualização é realizada por meio da função disponibilizada pelo Hook:

```javascript
setCount(count + 1);
```

A função de atualização permite que o React acompanhe a mudança e agende uma nova renderização quando necessário.

**Regra prática:** quando um valor precisa mudar e essa mudança deve refletir na interface, o estado do React costuma ser a ferramenta apropriada.

### 3.3. Atualização baseada no estado anterior

Quando o novo valor depende do valor anterior, podemos fornecer uma função à função de atualização:

```jsx
setCount((previousCount) => previousCount + 1);
```

A função recebe o estado anterior e retorna o próximo valor.

Essa forma é especialmente útil quando várias atualizações dependem umas das outras.

Por exemplo:

```jsx
function Counter() {
    const [count, setCount] = useState(0);

    function incrementTwice() {
        setCount((previousCount) => previousCount + 1);
        setCount((previousCount) => previousCount + 1);
    }

    return (
        <div>
            <p>Contador: {count}</p>

            <button onClick={incrementTwice}>
                Incrementar duas vezes
            </button>
        </div>
    );
}
```

As duas atualizações são expressas como funções que recebem o valor anterior. Assim, cada atualização pode considerar o resultado da anterior.

## 4. `useEffect`: efeitos colaterais

O Hook `useEffect` permite executar operações que precisam acontecer em relação à renderização do componente.

Essas operações são chamadas de **efeitos colaterais** (*side effects*), pois envolvem interações que vão além do cálculo direto da interface.

Alguns exemplos são:

- Buscar dados em uma API.
- Registrar ou remover um evento externo.
- Criar uma assinatura em um serviço.
- Sincronizar o componente com um sistema externo.

Sua sintaxe básica é:

```jsx
useEffect(() => {
    // Operação a ser executada após a renderização.

    return () => {
        // Limpeza opcional do efeito.
    };
}, [dependencies]);
```

O segundo argumento é um array de dependências. Ele permite que o React determine quando o efeito precisa ser executado novamente.

A função de limpeza é opcional e será explicada adiante.

### 4.1. Executar um efeito apenas na montagem inicial

Quando um efeito não depende de valores reativos do componente e recebe um array de dependências vazio, ele é executado após a renderização inicial do componente.

```jsx
import React, { useEffect } from 'react';

function Example() {
    useEffect(() => {
        console.log('Componente montado');
    }, []);

    return <p>Exemplo de useEffect</p>;
}
```

O array vazio indica que o efeito não precisa ser repetido em função de mudanças nas dependências.

Esse padrão é frequentemente utilizado para iniciar operações como a busca inicial de dados.

Entretanto, a expressão “executar uma única vez” exige uma ressalva: o comportamento pode variar em ambientes de desenvolvimento e conforme a configuração do React. No React 16.8.6, não existe o comportamento adicional de montagem e limpeza repetidas introduzido posteriormente pelo Strict Mode do React 18.

### 4.2. Executar um efeito quando uma dependência muda

Considere um componente que apresenta informações relacionadas a um identificador:

```jsx
function UserDetails({ userId }) {
    useEffect(() => {
        console.log('Identificador alterado:', userId);
    }, [userId]);

    return <p>Usuário: {userId}</p>;
}
```

Nesse exemplo, o efeito é executado após a renderização inicial e novamente quando o valor de `userId` muda.

O array `[userId]` informa ao React que o efeito depende desse valor.

Isso é útil quando uma mudança na propriedade exige uma nova operação externa, como buscar os detalhes de outro usuário.

### 4.3. Efeito sem array de dependências

Se o segundo argumento for omitido, o efeito será executado após cada renderização do componente:

```jsx
useEffect(() => {
    console.log('Componente renderizado');
});
```

Esse comportamento exige cuidado. Se o efeito atualizar um estado que provoca outra renderização, ele poderá ser executado novamente e gerar um ciclo repetitivo.

Por isso, o array de dependências deve representar corretamente os valores dos quais o efeito depende.

## 5. Buscando dados de uma API com `useEffect`

Um dos usos mais comuns de `useEffect` é iniciar uma requisição HTTP quando o componente é renderizado pela primeira vez.

Considere um componente que busca publicações de blog na Headless Delivery API do Liferay.

```jsx
import React, { useEffect, useState } from 'react';

function BlogManager() {
    const [blogs, setBlogs] = useState([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState('');

    useEffect(() => {
        async function loadBlogs() {
            try {
                const response = await fetch(
                    '/o/headless-delivery/v1.0/sites/20117/blog-postings'
                );

                if (!response.ok) {
                    throw new Error('Erro ao buscar publicações');
                }

                const data = await response.json();

                setBlogs(data.items || []);
            } catch (error) {
                console.error(error);
                setError('Não foi possível carregar as publicações.');
            } finally {
                setLoading(false);
            }
        }

        loadBlogs();
    }, []);

    if (loading) {
        return <p>Carregando publicações...</p>;
    }

    if (error) {
        return <p>{error}</p>;
    }

    return (
        <div>
            <h2>Publicações</h2>

            {blogs.map((blog) => (
                <p key={blog.id}>{blog.headline}</p>
            ))}
        </div>
    );
}

export default BlogManager;
```

O exemplo reúne três estados:

- `blogs`: armazena as publicações recebidas.
- `loading`: indica se a operação de carregamento está em andamento.
- `error`: armazena uma mensagem de erro para a interface.

O fluxo funciona da seguinte maneira:

1. O componente é renderizado com a lista vazia e `loading` igual a `true`.
2. O efeito é executado após a renderização.
3. A função `loadBlogs()` realiza a requisição.
4. Se a resposta for bem-sucedida, os dados são convertidos para JSON.
5. `setBlogs()` solicita a atualização da lista.
6. Se ocorrer um erro, a mensagem é registrada em `error`.
7. O bloco `finally` altera `loading` para `false`.
8. O React renderiza a interface de acordo com os estados atualizados.

O endereço do endpoint e o formato da resposta devem ser adequados ao ambiente do Liferay. O exemplo pressupõe uma resposta JSON com a propriedade `items`.

Também é importante observar que `fetch()` não rejeita automaticamente a Promise para todos os status HTTP de erro. Por isso, a verificação de `response.ok` é necessária quando queremos tratar essas respostas como erros.

## 6. O que acontece quando o estado muda?

Considere a seguinte sequência:

```text
Renderização inicial
        |
        v
Estado: loading = true
        |
        v
React apresenta "Carregando..."
        |
        v
useEffect inicia