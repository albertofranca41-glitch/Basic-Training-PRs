# Camada de API e Separação de Responsabilidades

## 1. Introdução

Em aplicações JavaScript, é comum que um componente precise buscar dados de uma API, enviar informações para um servidor ou atualizar algum registro. Em uma aplicação React, essas operações podem ser realizadas diretamente dentro dos componentes.

Entretanto, conforme a aplicação cresce, misturar a lógica de interface com as requisições HTTP pode tornar o código mais difícil de compreender, testar e manter.

Uma maneira de organizar melhor a aplicação é separar as responsabilidades: os componentes ficam responsáveis pela interface e pelo estado visual, enquanto um módulo específico concentra as funções responsáveis pela comunicação com a API.

Essa organização é especialmente útil em aplicações que consomem APIs REST, como os widgets React desenvolvidos para o Liferay.

## 2. O que é separação de responsabilidades?

**Separação de responsabilidades** (*Separation of Concerns*) é um princípio de organização de software segundo o qual cada parte do sistema deve ter uma responsabilidade bem definida.

Em vez de concentrar toda a lógica em um único arquivo, dividimos o código em partes que colaboram entre si.

Por exemplo, em um widget React que gerencia publicações de blog, podemos identificar as seguintes responsabilidades:

- **Interface:** exibir publicações, formulários, botões e mensagens.
- **Estado:** armazenar os dados que influenciam a renderização dos componentes.
- **Comunicação com a API:** realizar requisições HTTP e interpretar as respostas.
- **Regras da aplicação:** coordenar operações como criar, editar ou excluir uma publicação.

Essas responsabilidades não precisam necessariamente corresponder a arquivos separados em todos os projetos. O importante é que a organização do código torne claro o papel de cada parte.

## 3. O problema de misturar interface e requisições

Considere um componente React que precisa buscar publicações de blog no Liferay.

Uma implementação inicial poderia realizar a requisição diretamente no componente:

```jsx
import React, { useEffect, useState } from 'react';

function BlogManager() {
    const [blogs, setBlogs] = useState([]);

    useEffect(() => {
        fetch('/o/headless-delivery/v1.0/sites/20117/blog-postings')
            .then((response) => {
                if (!response.ok) {
                    throw new Error('Erro ao buscar publicações');
                }

                return response.json();
            })
            .then((data) => {
                setBlogs(data.items || []);
            })
            .catch((error) => {
                console.error(error);
            });
    }, []);

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

O código é válido como exemplo introdutório, mas o componente acumula diferentes responsabilidades:

1. Define o estado das publicações.
2. Decide quando realizar a requisição.
3. Conhece o endereço do endpoint.
4. Realiza a comunicação HTTP.
5. Trata a resposta recebida.
6. Exibe os dados na interface.

Em um componente pequeno, essa combinação pode ser administrável. Porém, imagine que outros componentes precisem acessar o mesmo endpoint ou que a aplicação tenha diversas operações de criação, edição e exclusão.

Nesse cenário, a lógica de comunicação pode acabar repetida em vários lugares.

Além disso, uma mudança no endpoint ou na maneira de tratar a resposta pode exigir alterações em diferentes componentes.

**O problema não é realizar uma requisição HTTP dentro de um componente, mas permitir que a lógica de comunicação se espalhe pela aplicação sem uma organização clara.**

## 4. Criando uma camada de API

Uma alternativa é centralizar as operações de comunicação com o servidor em um módulo específico.

Esse módulo pode exportar funções que representam as operações disponíveis na API. Os componentes importam essas funções sem precisar conhecer todos os detalhes de cada requisição.

Por exemplo, a estrutura poderia ser:

```text
src/
├── api/
│   └── liferay.js
├── components/
│   └── BlogManager.jsx
├── App.js
└── index.js
```

O arquivo `liferay.js` ficaria responsável por funções relacionadas à comunicação com as APIs do Liferay.

O componente `BlogManager.jsx`, por sua vez, utilizaria essas funções para obter os dados necessários à interface.

Essa separação permite que cada arquivo tenha um objetivo mais claro.

## 5. Exemplo prático

### 5.1. Módulo de comunicação com a API

No arquivo `src/api/liferay.js`, podemos definir uma função responsável por buscar publicações:

```javascript
export async function getBlogs() {
    const response = await fetch(
        '/o/headless-delivery/v1.0/sites/20117/blog-postings'
    );

    if (!response.ok) {
        throw new Error('Erro ao buscar publicações');
    }

    const data = await response.json();

    return data.items || [];
}
```

A função `getBlogs()` concentra os detalhes da requisição:

- O endereço do endpoint.
- A execução da requisição HTTP.
- A verificação do status da resposta.
- A leitura do JSON.
- A extração da lista de publicações.

O uso de `async` e `await` permite escrever o código assíncrono de maneira mais linear.

A função lança um erro quando a resposta HTTP não indica sucesso. A responsabilidade de decidir como esse erro será apresentado ao usuário pode permanecer no componente.

**Observação:** o endereço, a autenticação e o formato da resposta devem ser ajustados conforme o ambiente e o endpoint utilizados. O exemplo pressupõe que a resposta contém uma propriedade `items`.

### 5.2. Componente React

Agora, o componente pode importar a função e utilizá-la:

```jsx
import React, { useEffect, useState } from 'react';
import { getBlogs } from '../api/liferay';

function BlogManager() {
    const [blogs, setBlogs] = useState([]);
    const [error, setError] = useState('');

    useEffect(() => {
        async function loadBlogs() {
            try {
                const data = await getBlogs();

                setBlogs(data);
            } catch (error) {
                console.error(error);
                setError('Não foi possível carregar as publicações.');
            }
        }

        loadBlogs();
    }, []);

    return (
        <div>
            <h2>Publicações</h2>

            {error && <p>{error}</p>}

            {blogs.map((blog) => (
                <p key={blog.id}>{blog.headline}</p>
            ))}
        </div>
    );
}

export default BlogManager;
```

Observe que o componente não precisa mais conhecer o endereço do endpoint nem implementar diretamente o tratamento da resposta HTTP.

Ele importa `getBlogs()`, solicita os dados e atualiza o estado com `setBlogs()`.

A função `getBlogs()` não é responsável por renderizar a interface, e o componente não precisa implementar todos os detalhes da comunicação com o servidor.

Assim, cada parte possui uma responsabilidade mais bem delimitada.

## 6. Como as partes se comunicam?

O fluxo da operação pode ser representado da seguinte maneira:

```text
BlogManager.jsx
      |
      | chama getBlogs()
      v
src/api/liferay.js
      |
      | realiza requisição HTTP
      v
Headless Delivery API
      |
      | retorna resposta HTTP
      v
src/api/liferay.js
      |
      | retorna os dados
      v
BlogManager.jsx
      |
      | atualiza o estado com setBlogs()
      v
React renderiza a interface atualizada
```

Cada etapa contribui para a operação completa:

1. O componente identifica a necessidade de buscar os dados.
2. O módulo de API executa a requisição.
3. O servidor processa a solicitação e devolve uma resposta.
4. O módulo de API interpreta a resposta e retorna os dados.
5. O componente armazena os dados no estado.
6. O React atualiza a interface conforme o novo estado.

Esse fluxo também ajuda a distinguir duas operações diferentes: **buscar dados no servidor** e **atualizar a interface com esses dados**.

## 7. Responsabilidade de cada parte

| Parte | Responsabilidade principal |
|---|---|
| `index.js` | Inicializar o widget e montar a aplicação React, conforme a estrutura do projeto. |
| `App.js` | Organizar a estrutura principal da aplicação e seus componentes. |
| `BlogManager.jsx` | Gerenciar a interface de publicações, seu estado e as interações do usuário. |
| `DocsManager.jsx` | Gerenciar a interface e as operações relacionadas a documentos. |
| `KBManager.jsx` | Gerenciar a interface e as operações relacionadas à base de conhecimento. |
| `src/api/liferay.js` | Concentrar funções de comunicação com as APIs do Liferay. |

Essa tabela representa uma divisão possível para o projeto. A responsabilidade exata de cada arquivo depende de sua implementação.

Também é importante observar que a separação não significa que o componente deixa de lidar com operações assíncronas. Ele ainda precisa decidir quando solicitar os dados e o que fazer com o resultado. A diferença é que os detalhes da requisição ficam concentrados em outro módulo.

## 8. Uma camada de API não é a própria API

É importante distinguir os dois conceitos.

**API:** conjunto de interfaces e operações disponibilizadas por um sistema para que outros programas possam interagir com ele.

**Camada de API da aplicação:** módulo interno que organiza a maneira como o código da aplicação acessa uma ou mais APIs externas.

No exemplo, a Headless Delivery API é disponibilizada pelo Liferay. O arquivo `liferay.js` não cria nem substitui essa API. Ele apenas fornece funções JavaScript para facilitar o acesso aos seus endpoints.

Essa distinção é importante porque a camada de API pertence ao código do widget, enquanto a Headless Delivery API é um serviço com o qual o widget se comunica.

## 9. Aplicando a ideia às operações CRUD

A mesma organização pode ser utilizada para diferentes operações.

CRUD é a sigla para *Create, Read, Update and Delete*:

- **Create:** criar um registro.
- **Read:** consultar registros.
- **Update:** atualizar um registro.
- **Delete:** excluir um registro.

Em um módulo de API, poderíamos ter funções como:

```javascript
export async function getBlogs() {
    // Consultar publicações.
}

export async function getBlogById(id) {
    // Consultar uma publicação específica.
}

export async function createBlog(blogData) {
    // Criar uma publicação.
}

export async function updateBlog(id, blogData) {
    // Atualizar uma publicação.
}

export async function deleteBlog(id) {
    // Excluir uma publicação.
}
```

O código acima apresenta apenas a organização possível das funções. A implementação real deve definir os endpoints, os métodos HTTP, os cabeçalhos, o corpo das requisições e o tratamento das respostas conforme a documentação da API.

Por exemplo, uma operação de exclusão pode retornar o status HTTP `204 No Content`, indicando sucesso sem um corpo de resposta. Nesse caso, o código não deve tentar interpretar a resposta como JSON se não houver conteúdo.

Centralizar essas operações facilita a manutenção porque os detalhes de comunicação ficam reunidos em um único módulo.

## 10. Benefícios da separação de responsabilidades

### 10.1. Menos duplicação

Se vários componentes precisam consultar o mesmo recurso, eles podem reutilizar uma função comum em vez de repetir a requisição.

### 10.2. Manutenção mais simples

Se um endpoint mudar, a alteração pode ficar concentrada na função correspondente do módulo de API, desde que a mudança não exija alterações na maneira como os componentes utilizam os dados.

### 10.3. Código mais legível

O componente passa a destacar o fluxo da interface, enquanto o módulo de API concentra os detalhes de comunicação.

### 10.4. Reutilização

Uma mesma função pode ser utilizada por diferentes componentes, desde que eles precisem da mesma operação e os dados retornados sejam adequados às suas necessidades.

### 10.5. Testes mais organizados

É possível testar as funções de comunicação separadamente dos componentes e testar os componentes substituindo as funções importadas por versões simuladas (*mocks*).

A separação não elimina a necessidade de testes de integração. Ela apenas facilita a identificação das responsabilidades que precisam ser verificadas.

## 11. Limites dessa abordagem

Separar a camada de API não significa que toda a lógica deva ser transferida para ela.

O módulo de API deve concentrar principalmente os detalhes de comunicação com os serviços externos. Já as decisões relacionadas à interface, como exibir uma mensagem de erro, mostrar um indicador de carregamento ou atualizar a tela após uma operação, geralmente pertencem aos componentes ou a outras partes da aplicação responsáveis por essas decisões.

Da mesma forma, a camada de API não precisa se transformar em uma estrutura complexa para um widget pequeno. A organização deve acompanhar as necessidades reais do projeto.

O objetivo é reduzir o acoplamento entre responsabilidades, e não criar arquivos ou abstrações desnecessárias.

## 12. Relação com o widget React no Liferay

No widget estudado, diferentes componentes podem trabalhar com recursos disponibilizados pela Headless Delivery API, como publicações de blog, documentos e itens da base de conhecimento.

Uma organização que concentra as requisições em `src/api/liferay.js` permite que esses componentes utilizem funções específicas para acessar os recursos.

Isso ajuda a separar dois conhecimentos:

- O componente conhece a operação necessária para sua interface.
- O módulo de API conhece os detalhes necessários para realizar a comunicação com o Liferay.

Essa organização também facilita a leitura do código por quem está aprendendo o projeto. Ao encontrar uma chamada como `getBlogs()`, por exemplo, é possível compreender a intenção pelo nome da função e consultar sua implementação apenas quando for necessário entender os detalhes da requisição.

A separação, entretanto, não garante automaticamente que o código esteja correto. Ainda é necessário verificar os endpoints, os métodos HTTP, a autenticação, as permissões e o formato das respostas.

## 13. Resumo

- **Separação de responsabilidades** significa organizar o código de acordo com as funções desempenhadas por cada parte.
- Os componentes React podem cuidar da interface, das interações e do estado necessário à renderização.
- Uma **camada de API** reúne funções responsáveis pela comunicação com serviços externos.
- A camada de API não substitui a API do Liferay; ela organiza o acesso a essa API dentro do widget.
- A separação reduz duplicação, facilita a manutenção e torna as responsabilidades mais claras.
- A organização deve ser proporcional à complexidade da aplicação, evitando abstrações desnecessárias.

## 14. Referências adicionais

- [React — Your First Component](https://react.dev/learn/your-first-component)
- [React — Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- [MDN — JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [Liferay — Headless APIs](https://learn.liferay.com/w/dxp/integration/headless-apis)
