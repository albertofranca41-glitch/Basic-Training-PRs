# Clay UI

## 1. O que é Clay UI?

**Clay UI** é uma biblioteca de componentes de interface utilizada no ecossistema Liferay. Ela fornece componentes reutilizáveis, como botões, menus, campos de formulário, caixas de seleção, tabelas e elementos de navegação.

O objetivo é facilitar a construção de interfaces consistentes, reduzindo a necessidade de implementar cada elemento visual do zero.

Por exemplo, em vez de criar manualmente um botão e definir todo o seu estilo, podemos utilizar um componente do Clay:

```jsx
import ClayButton from '@clayui/button';

function App() {
    return <ClayButton>Clique aqui</ClayButton>;
}
```

Nesse exemplo, `ClayButton` é um componente fornecido pela biblioteca. Podemos utilizá-lo dentro da interface React como utilizamos outros componentes.

> **Importante:** Clay não é uma linguagem de programação nem um framework que substitui o React. É uma biblioteca de componentes e estilos que pode ser utilizada para construir interfaces.

Documentação oficial: <https://www.clayui.com/docs/introduction/how-to-use-clay>

## 2. Clay, React e Lexicon

Para entender o papel do Clay, é útil distinguir três conceitos.

- **React:** biblioteca JavaScript utilizada para construir interfaces por meio de componentes.
- **Clay UI:** biblioteca que disponibiliza componentes de interface prontos para serem utilizados em aplicações, especialmente no ecossistema Liferay.
- **Lexicon:** sistema de design do Liferay, que define padrões visuais e de interação para manter a experiência consistente entre as interfaces.

Uma maneira simples de visualizar a relação é:

```text
React
  └── organiza a interface em componentes
        └── utiliza componentes do Clay UI
              └── seguem padrões de design do Lexicon
```

Essa relação é conceitual: o Clay fornece componentes que implementam padrões do sistema de design, enquanto o React permite compor esses componentes na aplicação.

## 3. Componentes disponíveis

O Clay possui pacotes para diferentes necessidades da interface. Alguns exemplos:

| Componente | Finalidade |
|---|---|
| `@clayui/button` | Botões e ações |
| `@clayui/drop-down` | Menus suspensos |
| `@clayui/form` | Elementos de formulário, como checkbox e radio |
| `@clayui/card` | Apresentação de conteúdo em cartões |
| `@clayui/alert` | Mensagens de alerta |
| `@clayui/modal` | Janelas modais |
| `@clayui/table` | Tabelas |
| `@clayui/pagination` | Paginação |
| `@clayui/tooltip` | Dicas exibidas ao interagir com elementos |

A biblioteca organiza muitos desses componentes em pacotes independentes. Assim, o projeto pode instalar os pacotes de que precisa, em vez de tratar todos os componentes como um único pacote JavaScript.

Consulte a documentação de cada componente antes de utilizá-lo, pois suas propriedades, eventos e formas de composição variam.

## 4. Instalação

A documentação oficial apresenta duas formas comuns de instalar os pacotes: NPM e Yarn.

### 4.1. Instalar com NPM

Para instalar o CSS do Clay e o pacote de botões:

```bash
npm install @clayui/css @clayui/button
```

### 4.2. Instalar com Yarn

```bash
yarn add @clayui/css @clayui/button
```

O nome `@clayui/button` representa o pacote de botões. Para utilizar outro componente, instale o pacote correspondente, como `@clayui/drop-down`.

**Atenção:** `@clayui/*` é uma forma ilustrativa de representar a família de pacotes. O asterisco não é o nome de um pacote que possa ser instalado diretamente. Substitua-o pelo nome real do pacote necessário.

### 4.3. Importar o CSS

Além do pacote JavaScript do componente, a aplicação precisa carregar os estilos do Clay. Sem o CSS, os componentes podem não apresentar a aparência esperada.

Em uma aplicação React, importe o CSS no ponto de entrada apropriado:

```jsx
import '@clayui/css/lib/css/atlas.css';
```

A localização exata desse import depende da estrutura e da configuração do projeto. Siga as instruções da versão instalada do Clay.

### 4.4. Utilizar o CSS por CDN

A documentação também demonstra uma alternativa para carregar os estilos sem instalar o pacote CSS por NPM ou Yarn:

```html
<link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/@clayui/css/lib/css/atlas.css"
/>
```

Essa opção inclui o CSS na página. Ela **não instala os componentes React**: para utilizá-los como componentes JavaScript, a aplicação ainda precisa ter os pacotes e o ambiente necessários.

Se o projeto exigir uma versão específica, fixe a versão no endereço do CDN, em vez de depender de uma versão não especificada.

## 5. Utilizando um componente

Depois de instalar o pacote e importar os estilos, podemos importar e renderizar um componente.

```jsx
import ClayButton from '@clayui/button';

function App() {
    return (
        <div>
            <h1>Minha aplicação</h1>
            <ClayButton>Clique aqui</ClayButton>
        </div>
    );
}

export default App;
```

Vamos separar as partes principais:

- `import ClayButton from '@clayui/button'`: importa o componente do pacote instalado.
- `function App()`: declara um componente React.
- `<ClayButton>Clique aqui</ClayButton>`: utiliza o componente Clay dentro do JSX.
- `export default App`: exporta o componente para que ele possa ser utilizado em outro ponto da aplicação.

O exemplo demonstra a renderização de um botão. Para executar a aplicação, ainda é necessário ter um projeto React configurado.

## 6. Composição de componentes

Um conceito importante na documentação do Clay é a **composição**: construir uma interface combinando componentes menores.

Em vez de esperar que um único componente resolva todo o comportamento de uma funcionalidade, podemos reunir componentes especializados para montar a experiência desejada.

Por exemplo, um menu suspenso pode ser formado por:

- um botão que abre o menu;
- um componente que controla a área do menu;
- uma lista de opções;
- itens que representam links ou controles.

A composição permite adaptar o conteúdo à necessidade da aplicação, mantendo a reutilização dos componentes.

## 7. Componentes de baixo nível e de alto nível

A documentação sobre menus suspensos ilustra duas formas de trabalhar com componentes Clay.

### 7.1. Componente de baixo nível (*low-level*)

Um componente de baixo nível oferece uma estrutura mais flexível. O desenvolvedor decide como compor o conteúdo e controla explicitamente partes do comportamento.

Isso costuma ser útil quando a interface precisa de uma composição personalizada.

### 7.2. Componente de alto nível (*high-level*)

Um componente de alto nível oferece uma solução mais pronta para um caso de uso comum. Ele pode receber dados estruturados e cuidar de parte da composição internamente.

Isso costuma reduzir a quantidade de código necessária quando o comportamento padrão atende à necessidade.

### 7.3. Comparação

| Abordagem | Característica | Quando pode ser útil |
|---|---|---|
| Baixo nível | Mais controle sobre a composição | Quando precisamos personalizar a estrutura |
| Alto nível | Mais comportamento pronto | Quando o padrão oferecido já atende ao caso |

Não significa que uma abordagem seja sempre melhor que a outra. A escolha depende do grau de personalização e de controle necessário.

## 8. Exemplo prático: um menu suspenso controlado

O exemplo a seguir adapta o princípio apresentado na documentação oficial. Ele utiliza `ClayDropDown`, `ClayButton` e o estado do React para controlar se o menu está aberto.

```jsx
import {useState} from 'react';

import ClayButton from '@clayui/button';
import ClayDropDown from '@clayui/drop-down';

function MenuExemplo() {
    const [expand, setExpand] = useState(false);

    return (
        <ClayDropDown
            active={expand}
            onActiveChange={setExpand}
            trigger={<ClayButton>Clique aqui</ClayButton>}
        >
            <ClayDropDown.ItemList>
                <ClayDropDown.Item href="#perfil">
                    Perfil
                </ClayDropDown.Item>

                <ClayDropDown.Item href="#configuracoes">
                    Configurações
                </ClayDropDown.Item>
            </ClayDropDown.ItemList>
        </ClayDropDown>
    );
}

export default MenuExemplo;
```

### 8.1. Entendendo o estado

```jsx
const [expand, setExpand] = useState(false);
```

`useState` é um Hook do React que permite manter um valor de estado entre as renderizações do componente.

Nesse caso:

- `expand` guarda o valor atual;
- `setExpand` é a função utilizada para atualizar esse valor;
- `false` é o valor inicial, então o menu começa fechado.

### 8.2. Controlando o menu

```jsx
<ClayDropDown
    active={expand}
    onActiveChange={setExpand}
    trigger={<ClayButton>Clique aqui</ClayButton>}
>
    {/* Conteúdo do menu */}
</ClayDropDown>
```

- `active={expand}` informa ao menu se ele deve estar ativo/aberto.
- `onActiveChange={setExpand}` fornece ao componente a função que atualizará esse estado quando houver uma mudança.
- `trigger` define o elemento que aciona o menu — nesse exemplo, um botão Clay.
- O conteúdo entre as tags define o que aparece dentro do menu.

Esse é um exemplo de **componente controlado**: o estado relevante é mantido pelo componente React que o utiliza e passado ao Clay por meio de propriedades (*props*).

A ideia central é que o Clay fornece o componente de interface, mas a aplicação pode manter o controle do estado e da lógica.

> Os nomes e a API dos componentes podem mudar entre versões. Confira a documentação correspondente à versão instalada antes de reutilizar o exemplo em um projeto real.

## 9. Propriedades (*props*) e eventos

Em React, as **props** são valores enviados de um componente pai para um componente filho. O Clay utiliza props para receber configurações, conteúdo e funções de retorno (*callbacks*).

No exemplo anterior:

- `active` é uma prop que informa o estado do menu;
- `onActiveChange` é uma prop que recebe uma função;
- `trigger` é uma prop que define o elemento que aciona o menu.

Uma função como `setExpand` pode ser passada como prop para permitir que o componente comunique uma mudança ao componente que o controla.

O fluxo simplificado é:

```text
O usuário interage com o menu
          ↓
O componente comunica a mudança
          ↓
setExpand atualiza o estado do React
          ↓
React renderiza novamente a interface
          ↓
active recebe o novo valor de expand
```

Esse padrão ajuda a manter o comportamento da interface previsível.

## 10. Utilização no ecossistema Liferay

O Clay é especialmente relevante ao trabalhar com interfaces do ecossistema Liferay, pois oferece componentes alinhados aos padrões de interface utilizados pela plataforma.

Entretanto, é importante considerar o contexto do projeto:

- **Aplicação React:** os pacotes Clay podem ser importados diretamente quando o projeto está configurado para utilizá-los.
- **Portlet ou módulo Liferay tradicional:** a forma de integração depende da tecnologia usada para renderizar a interface, como JSP, JavaScript ou React.
- **Projeto com versões específicas do Liferay e do Clay:** é necessário verificar a compatibilidade e a documentação correspondente.

Não se deve presumir que qualquer exemplo React da documentação possa ser copiado diretamente para um JSP ou para qualquer módulo Liferay. A tecnologia de interface e a configuração do projeto determinam como os componentes serão incorporados.

A documentação do Clay também possui material específico sobre o uso em JSPs: <https://www.clayui.com/docs/introduction/clay-in-jsps>.

## 11. Erros comuns e cuidados

### Instalar o componente, mas esquecer o CSS

Os pacotes de componentes e os estilos cumprem funções diferentes. Verifique se o CSS do Clay está sendo carregado.

### Instalar o pacote errado

Cada pacote atende a um conjunto específico de funcionalidades. Consulte a documentação e instale o pacote correspondente ao componente desejado.

### Esperar que um componente de baixo nível venha totalmente configurado

Alguns componentes fornecem apenas a estrutura básica. É necessário compô-los com outros elementos para construir o menu, formulário ou interação completa.

### Não controlar o estado quando o componente exige isso

Componentes interativos podem precisar receber valores de estado e funções de callback. Verifique as props exigidas pela API do componente.

### Copiar exemplos sem conferir a versão

A documentação online pode evoluir. Confirme os nomes dos pacotes, as propriedades e as instruções de instalação da versão utilizada pelo projeto.

## 12. Resumo

- **Clay UI** é uma biblioteca de componentes de interface utilizada no ecossistema Liferay.
- **React** é a biblioteca que permite construir interfaces com componentes; Clay fornece componentes que podem ser usados nesse contexto.
- **Lexicon** define padrões de design que ajudam a manter consistência visual e de interação.
- Os pacotes são organizados sob o escopo `@clayui`, como `@clayui/button` e `@clayui/drop-down`.
- O CSS precisa ser carregado para que os componentes tenham a aparência esperada.
- A **composição** combina componentes menores para formar interfaces mais completas.
- Componentes de **baixo nível** permitem maior controle; componentes de **alto nível** oferecem soluções mais prontas.
- Em componentes controlados, a aplicação mantém o estado e o fornece ao Clay por meio de props.

## Referências

- [How to Use Clay — documentação oficial](https://www.clayui.com/docs/introduction/how-to-use-clay)
- [Clay UI — documentação geral](https://www.clayui.com/docs)
- [Clay in JSPs](https://www.clayui.com/docs/introduction/clay-in-jsps)