# REST — Representational State Transfer

## Definição

**REST** (*Representational State Transfer*) é um estilo arquitetural para sistemas distribuídos, apresentado por Roy Fielding em sua tese de doutorado, em 2000.

Ele define um conjunto de restrições arquiteturais que orientam como os componentes de um sistema devem interagir. Essas restrições favorecem características como escalabilidade, separação de responsabilidades e independência entre cliente e servidor.

REST é frequentemente utilizado no desenvolvimento de APIs web. Nesses casos, o HTTP costuma ser o protocolo de comunicação, mas **REST e HTTP não são a mesma coisa**: HTTP é um protocolo, enquanto REST é um estilo arquitetural.

## Conceitos-chave

### Recurso

Em REST, o conceito central é o **recurso** (*resource*).

Um recurso representa algo que pode ser identificado e acessado por meio de uma interface. Por exemplo:

- Um produto;
- Um usuário;
- Uma publicação de blog;
- Uma coleção de artigos.

Um recurso não precisa corresponder diretamente a uma tabela ou registro do banco de dados. Ele é uma abstração apresentada pela interface do sistema.

### URI — Identificação de recursos

Cada recurso pode ser identificado por uma URI (*Uniform Resource Identifier*).

Por exemplo, em uma API de produtos:

```text
https://api.exemplo.com/produtos
https://api.exemplo.com/produtos/42
```

A primeira URI pode identificar uma coleção de produtos, enquanto a segunda pode identificar um produto específico, cujo identificador é `42`.

A URI identifica o recurso, mas não determina, por si só, qual operação será executada sobre ele.

### Representação

Um cliente geralmente não recebe o recurso em si como um objeto interno do servidor. Ele recebe uma **representação do recurso**.

Essa representação contém dados que descrevem o estado do recurso em um formato adequado à comunicação.

Por exemplo, uma representação JSON de um produto poderia ser:

```json
{
  "id": 42,
  "name": "Teclado",
  "price": 150.00
}
```

JSON é um formato comum para representar recursos em APIs REST, mas não é uma exigência universal do REST. Outros formatos também podem ser utilizados.

## REST e HTTP

Em APIs web que utilizam HTTP, os métodos HTTP indicam a semântica da operação solicitada pelo cliente.

Considere uma API que disponibiliza recursos de produtos:

| Método | URI | Finalidade comum |
|---|---|---|
| `GET` | `/produtos` | Consultar a coleção de produtos |
| `GET` | `/produtos/42` | Consultar um produto específico |
| `POST` | `/produtos` | Solicitar a criação de um produto |
| `PUT` | `/produtos/42` | Substituir a representação de um produto |
| `PATCH` | `/produtos/42` | Solicitar uma alteração parcial |
| `DELETE` | `/produtos/42` | Solicitar a remoção de um produto |

Esses exemplos mostram convenções comuns em APIs HTTP. O comportamento exato de cada operação depende do contrato da API.

É importante observar que **REST não é simplesmente utilizar os métodos HTTP**. Uma API não se torna RESTful apenas porque possui endpoints e responde a requisições HTTP.

## Como funciona uma requisição REST

Uma interação típica entre um cliente e uma API web pode ser representada assim:

```text
Cliente
   |
   | Requisição HTTP
   | GET /produtos/42
   v
API / Servidor
   |
   | Interpreta a requisição
   | Executa a lógica necessária
   | Consulta dados, se necessário
   v
Resposta HTTP
   |
   | Status: 200 OK
   | Content-Type: application/json
   | Corpo: representação do produto
   v
Cliente
```

O cliente não precisa conhecer os detalhes internos da implementação do servidor. Ele precisa conhecer a interface disponibilizada pela API e respeitar seu contrato.

Da mesma forma, o servidor pode alterar sua implementação interna sem necessariamente exigir mudanças no cliente, desde que preserve o comportamento contratual esperado.

## Restrições arquiteturais do REST

REST não é apenas um conjunto de convenções para nomear URLs. O estilo arquitetural descrito por Roy Fielding apresenta seis restrições.

### 1. Cliente-servidor

A arquitetura separa as responsabilidades do cliente e do servidor.

O cliente cuida das necessidades relacionadas à interação com o usuário ou ao consumo da interface. O servidor cuida da disponibilização dos recursos e das funcionalidades correspondentes.

Essa separação permite que cada lado evolua de forma relativamente independente.

### 2. Ausência de estado de sessão no servidor (*stateless*)

Cada requisição deve conter as informações necessárias para que o servidor a compreenda e processe, sem depender de um contexto de sessão armazenado no servidor a partir de requisições anteriores.

Isso não significa que o servidor não possa armazenar dados persistentes, como usuários, produtos ou publicações. A restrição diz respeito ao estado da interação entre cliente e servidor.

Por exemplo, uma API pode receber um token de autenticação em cada requisição, em vez de depender de um contexto de sessão mantido no servidor.

### 3. Capacidade de armazenamento em cache

As respostas devem indicar se podem ou não ser armazenadas em cache.

Quando aplicável, o cache pode reduzir requisições desnecessárias e melhorar o desempenho. O comportamento depende das regras de cache e dos cabeçalhos apropriados.

### 4. Interface uniforme

A interação com os recursos segue uma interface padronizada. Essa restrição é uma das características centrais do REST e possui quatro aspectos:

- **Identificação dos recursos:** os recursos são identificados, normalmente, por URIs.
- **Manipulação por representações:** as representações recebidas ou enviadas são utilizadas para consultar ou alterar recursos.
- **Mensagens autodescritivas:** as mensagens carregam informações suficientes para que sejam interpretadas conforme a interface e o protocolo.
- **Hipermídia como mecanismo do estado da aplicação (HATEOAS):** as representações podem incluir links e controles que orientam o cliente sobre as próximas ações disponíveis.

O último aspecto é especialmente importante: uma API que utiliza HTTP, JSON e URLs organizadas por recursos pode seguir algumas convenções REST sem implementar integralmente a restrição de interface uniforme, especialmente HATEOAS.

### 5. Sistema em camadas

A arquitetura pode ser composta por camadas intermediárias, como proxies, gateways, balanceadores de carga e servidores de cache.

O cliente não precisa conhecer toda a estrutura interna pela qual a requisição passa. Cada camada pode cumprir uma responsabilidade específica.

### 6. Código sob demanda (opcional)

O servidor pode, opcionalmente, enviar código executável ao cliente para ampliar suas funcionalidades, como scripts JavaScript.

Essa restrição é opcional no REST; as demais restrições arquiteturais devem ser respeitadas para que o sistema seja considerado RESTful segundo o modelo completo.

## RESTful: o que significa?

O termo **RESTful** costuma descrever uma API ou um sistema que segue os princípios e as restrições do estilo arquitetural REST.

Na prática, o termo é frequentemente utilizado de maneira mais ampla para APIs que organizam recursos em URIs e utilizam os métodos e os códigos de status HTTP de forma coerente.

Entretanto, há uma diferença entre seguir convenções comuns de APIs REST e satisfazer todas as restrições arquiteturais definidas por Fielding. A presença de endpoints como `/produtos/42` e de respostas JSON, por si só, não comprova conformidade integral com REST.

## REST, API e HTTP: qual é a diferença?

Esses conceitos estão relacionados, mas representam coisas diferentes:

| Conceito | O que representa |
|---|---|
| **API** | Uma interface ou contrato por meio do qual um componente disponibiliza funcionalidades ou dados a outros componentes. |
| **HTTP** | Um protocolo que define a semântica e a estrutura das mensagens de comunicação na Web. |
| **REST** | Um estilo arquitetural que define restrições para organizar a interação entre componentes de sistemas distribuídos. |
| **JSON** | Um formato textual de intercâmbio de dados que pode ser utilizado nas representações transmitidas por uma API. |

Uma API web pode utilizar HTTP sem ser RESTful. Também pode utilizar JSON sem ser RESTful. Esses elementos são comuns em APIs REST, mas não são suficientes, isoladamente, para caracterizar o estilo arquitetural.

## Relação com o Liferay Headless Delivery API

O Liferay disponibiliza APIs Headless que permitem a outros componentes consultar e manipular recursos da plataforma por meio de requisições HTTP.

Por exemplo, no Headless Delivery, uma aplicação JavaScript pode consultar publicações de blog utilizando um endpoint semelhante a:

```text
GET /o/headless-delivery/v1.0/sites/20117/blog-postings
```

Nesse cenário:

- **JavaScript** atua como cliente;
- **HTTP** transporta a requisição e a resposta;
- **Headless Delivery API** disponibiliza uma interface para acessar os recursos do Liferay;
- **JSON** pode transportar a representação dos dados;
- **Liferay** processa a requisição e executa as operações necessárias.

O identificador `20117` é um exemplo de ID de site e pode variar conforme a instalação.

Essa separação permite que uma aplicação cliente consuma os recursos do Liferay sem precisar conhecer todos os detalhes de sua implementação interna.

## Resumo

REST é um estilo arquitetural para sistemas distribuídos, frequentemente utilizado na construção de APIs web.

Para compreender esse modelo, é importante distinguir:

- **Recursos:** aquilo que a API disponibiliza para consulta ou manipulação;
- **URIs:** identificadores utilizados para referenciar recursos;
- **Representações:** dados que descrevem os recursos em formatos como JSON;
- **HTTP:** protocolo frequentemente utilizado para transportar as mensagens;
- **Restrições REST:** princípios arquiteturais que orientam a interação entre cliente e servidor.

A ideia central não é apenas criar URLs e executar operações CRUD. É organizar a comunicação por meio de uma interface uniforme, com responsabilidades separadas e mensagens que permitam aos componentes interagir sem depender dos detalhes internos uns dos outros.

## Referências

- Fielding, Roy Thomas. *Architectural Styles and the Design of Network-based Software Architectures* (2000), capítulo 5 — [Tese de doutorado](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm).
- Fielding, Roy Thomas. *REST APIs must be hypertext-driven* (2008) — [Artigo original](https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven).
- MDN Web Docs — [An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview).
- Liferay Learn — [Headless APIs](https://learn.liferay.com/).