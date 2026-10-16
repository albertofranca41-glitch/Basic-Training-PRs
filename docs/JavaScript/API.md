# API (Application Programming Interface)

## 1. Introdução

**API (Application Programming Interface)** é uma interface que define como um software pode disponibilizar funcionalidades ou dados para serem utilizados por outro software.

Uma API estabelece um conjunto de operações disponíveis, as entradas necessárias para executá-las, os resultados esperados e as regras que os consumidores precisam respeitar para interagir com o sistema.

O principal objetivo de uma API é permitir a comunicação e a integração entre componentes de software sem exigir que o consumidor conheça os detalhes internos da implementação do sistema que oferece essas funcionalidades.

Podemos entender uma API como um **contrato de interação entre um consumidor e um fornecedor de funcionalidades**.

Por exemplo, imagine um sistema bancário que disponibiliza uma API para consultar o saldo de uma conta. O sistema consumidor precisa saber como solicitar essa informação e como interpretar a resposta, mas não precisa conhecer a implementação interna do sistema bancário, as consultas SQL utilizadas ou a estrutura de suas tabelas.

```text
┌────────────────────┐
│ Sistema consumidor │
└─────────┬──────────┘
          │
          │ Solicita uma operação
          ▼
┌────────────────────┐
│        API         │
│                    │
│ Contrato público   │
│ Operações          │
│ Entradas e saídas  │
└─────────┬──────────┘
          │
          │ Utiliza a implementação interna
          ▼
┌────────────────────┐
│ Sistema fornecedor │
│                    │
│ Regras de negócio  │
│ Serviços           │
│ Persistência       │
│ Banco de dados     │
└────────────────────┘
```

A API define a interface pela qual o consumidor interage com o sistema. A implementação interna é responsável por realizar o trabalho necessário para atender à solicitação.

Essa separação permite que a implementação interna seja modificada sem necessariamente exigir alterações nos consumidores, desde que o contrato da API seja preservado.

## 2. O que compõe uma API?

Uma API não é apenas um endereço para o qual enviamos uma requisição. Ela envolve um contrato que determina como a interação deve acontecer.

Os elementos desse contrato variam conforme o tipo de API, mas podem incluir:

- **Operações disponíveis:** quais ações o consumidor pode solicitar.
- **Parâmetros de entrada:** quais informações precisam ser fornecidas.
- **Formato dos dados:** como as informações são representadas.
- **Resultados:** quais informações ou resultados uma operação pode produzir.
- **Regras de utilização:** quais condições precisam ser atendidas para executar uma operação.
- **Tratamento de erros:** como falhas e situações excepcionais são comunicadas.
- **Autenticação e autorização:** como a identidade do consumidor é verificada e quais operações ele pode executar.

Considere uma API que permite cadastrar usuários:

```text
Operação: criar usuário

Entradas:
- nome
- e-mail

Resultado esperado:
- usuário criado
- identificador do usuário
- informações do cadastro

Possíveis erros:
- dados inválidos
- e-mail já utilizado
- consumidor sem autorização
```

Esse contrato informa ao consumidor como utilizar a funcionalidade, sem revelar necessariamente como o sistema valida os dados, armazena o usuário ou gera seu identificador.

**Importante:** nem toda API é uma API Web. Uma biblioteca de programação, por exemplo, também pode disponibilizar uma API por meio de classes, métodos e funções utilizados diretamente por outro código.

## 3. API, interface e implementação

Para compreender melhor o conceito, é importante distinguir três elementos.

### Interface

A interface define como uma funcionalidade pode ser acessada. Ela apresenta as operações disponíveis e a maneira de utilizá-las.

### Implementação

A implementação contém o código responsável por executar as operações oferecidas pela interface.

### Consumidor

O consumidor é o componente ou sistema que utiliza a interface para acessar as funcionalidades disponibilizadas.

Podemos representar essa relação assim:

```text
Consumidor
    │
    │ Utiliza
    ▼
Interface (API)
    │
    │ É atendida pela
    ▼
Implementação
    │
    ├── Regras de negócio
    ├── Cálculos
    ├── Consultas
    └── Persistência
```

Por exemplo, uma aplicação pode disponibilizar uma operação chamada `getUserById`, que retorna um usuário a partir de seu identificador.

O consumidor precisa conhecer o nome da operação, o parâmetro necessário e o resultado esperado. Ele não precisa saber se a implementação consulta um banco relacional, um serviço externo ou um sistema de cache.

Essa separação é uma forma de **abstração**: o consumidor trabalha com uma interface definida, enquanto os detalhes internos permanecem encapsulados na implementação.

Isso não significa que qualquer alteração interna seja automaticamente segura. Se a mudança alterar o comportamento observável da API, os consumidores poderão ser afetados.

## 4. Tipos de API conforme a forma de utilização

Existem APIs utilizadas dentro do mesmo processo e APIs utilizadas para comunicação entre processos ou sistemas.

### 4.1. APIs de bibliotecas e frameworks

Uma biblioteca pode disponibilizar classes, funções e métodos que outros programas utilizam.

Por exemplo, em Java:

```java
String name = "Liferay";

int length = name.length();
```

O método `length()` faz parte da API da classe `String`. O código consumidor utiliza essa funcionalidade por meio da interface pública da classe, sem precisar implementar novamente a lógica de contagem de caracteres.

Frameworks também disponibilizam APIs para que os desenvolvedores utilizem suas funcionalidades e integrem o próprio código à estrutura oferecida pelo framework.

### 4.2. APIs de sistemas operacionais

Um sistema operacional oferece APIs para que os programas solicitem funcionalidades como acesso a arquivos, gerenciamento de processos e comunicação com recursos do sistema.

O programa utiliza as interfaces disponibilizadas pelo sistema operacional, em vez de precisar implementar diretamente todos os mecanismos internos necessários para essas operações.

### 4.3. APIs de serviços remotos

Uma API remota permite que um programa solicite funcionalidades ou dados disponibilizados por outro processo ou sistema.

Essas APIs normalmente utilizam algum mecanismo de comunicação, como HTTP, para transportar as solicitações e respostas entre as partes.

É nesse contexto que aparecem as APIs Web, frequentemente utilizadas na comunicação entre frontends, backends e serviços externos.

## 5. APIs no desenvolvimento backend

No desenvolvimento backend, uma API frequentemente funciona como uma interface de acesso às funcionalidades de uma aplicação.

Por exemplo, uma aplicação pode disponibilizar operações para cadastrar, consultar, atualizar e remover registros.

```text
┌────────────────────┐
│      Frontend      │
│                    │
│ Interface do       │
│ usuário            │
└─────────┬──────────┘
          │
          │ Requisições
          ▼
┌────────────────────┐
│      API Web       │
│                    │
│ Endpoints          │
│ Validação inicial  │
│ Autenticação       │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│      Backend       │
│                    │
│ Serviços           │
│ Regras de negócio  │
│ Persistência       │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│     Banco de dados │
└────────────────────┘
```

Esse diagrama representa uma organização possível, não uma arquitetura obrigatória. Em algumas aplicações, as responsabilidades são distribuídas entre vários serviços; em outras, diferentes camadas podem estar no mesmo processo.

Uma API de backend pode permitir que um frontend:

- Consulte os dados de um usuário.
- Crie uma publicação.
- Atualize as informações de um registro.
- Exclua um recurso.
- Execute uma operação de negócio.
- Consulte dados disponibilizados por outros sistemas.

A API não precisa expor diretamente o banco de dados. Em uma aplicação bem estruturada, o backend pode validar a solicitação, aplicar as regras de negócio e controlar o acesso aos dados antes de produzir a resposta.

### Exemplo: cadastro de uma publicação

Imagine uma aplicação de blogs.

O frontend oferece um formulário para que o usuário crie uma publicação. Quando o formulário é enviado, o frontend envia uma solicitação à API do backend.

O backend recebe os dados, verifica se são válidos, aplica as regras de negócio e solicita à camada de persistência que armazene a publicação. Depois, o resultado é devolvido ao consumidor.

```text
1. Usuário preenche o formulário
              │
              ▼
2. Frontend envia a requisição
              │
              ▼
3. API recebe os dados
              │
              ▼
4. Backend valida a solicitação
              │
              ▼
5. Serviço executa as regras de negócio
              │
              ▼
6. Persistência armazena os dados
              │
              ▼
7. API devolve a resposta
              │
              ▼
8. Frontend apresenta o resultado
```

A API define como solicitar o cadastro e como o resultado será comunicado. A implementação do backend determina como o cadastro será efetivamente realizado.

## 6. APIs Web e o protocolo HTTP

Uma **API Web** permite que softwares interajam por tecnologias da Web. Muitas APIs Web usam HTTP para transportar requisições e respostas entre cliente e servidor.

HTTP define as regras dessas mensagens, incluindo métodos, URLs, headers, bodies e códigos de status. API define o contrato de interação: quais operações estão disponíveis, quais entradas esperam e quais resultados podem retornar. Portanto, uma API Web pode usar HTTP, mas API e HTTP não são a mesma coisa.

Para os detalhes de requisições, respostas, métodos, headers e códigos de status, consulte [HTTP.md](./HTTP.md). Nesta seção, basta lembrar que o contrato da API determina como interpretar a comunicação HTTP.

---

## 7. Endpoints e recursos

Em uma API Web, um **endpoint** é um ponto de acesso por meio do qual o consumidor interage com uma funcionalidade ou recurso disponibilizado pela API. Na prática, costuma ser descrito pela combinação de um método HTTP e uma URL.

Considere uma API de publicações:

```text
GET    /api/blog-postings
POST   /api/blog-postings
GET    /api/blog-postings/42
PUT    /api/blog-postings/42
PATCH  /api/blog-postings/42
DELETE /api/blog-postings/42
```

Nesse exemplo:

- `/api/blog-postings` representa o caminho utilizado para acessar a coleção de publicações.
- `/api/blog-postings/42` representa o caminho utilizado para acessar uma publicação identificada por `42`.
- Os métodos HTTP definem diferentes tipos de interação com esses caminhos.

O caminho, por si só, não determina toda a operação. Por exemplo, `GET /api/blog-postings/42` e `DELETE /api/blog-postings/42` utilizam o mesmo caminho, mas possuem semânticas diferentes.

Também é importante não confundir **endpoint com recurso**. O recurso é aquilo que a API disponibiliza para interação; o endpoint é o ponto de acesso utilizado para realizar essa interação.

## 8. JSON e representação de dados

Uma API precisa definir como os dados serão representados para que o consumidor consiga interpretá-los.

Um dos formatos mais utilizados em APIs Web é o **JSON (JavaScript Object Notation)**, um formato textual para representar dados estruturados.

Exemplo:

```json
{
  "id": 42,
  "headline": "Introdução às APIs",
  "published": true,
  "tags": ["backend", "http", "programming"]
}
```

O JSON permite representar objetos, arrays, strings, números, valores booleanos e valores nulos.

Ele é independente da linguagem de programação: um cliente JavaScript, Java, Python ou outra linguagem pode interpretar um JSON, desde que utilize os mecanismos apropriados.

Em JavaScript, por exemplo:

```javascript
const data = {
  headline: "Introdução às APIs",
  articleBody: "Conteúdo da publicação"
};

// Converte o objeto JavaScript em uma string JSON.
const body = JSON.stringify(data);

// Converte uma resposta JSON em um objeto JavaScript.
const responseData = await response.json();
```

`JSON.stringify()` transforma um valor JavaScript em uma string JSON. Já `response.json()` lê o corpo de uma resposta e interpreta seu conteúdo como JSON.

Essas operações não são equivalentes: uma serializa dados para envio ou armazenamento, enquanto a outra interpreta dados recebidos.

Além disso, uma API não é obrigada a utilizar JSON. Dependendo do contrato e do contexto, ela pode trabalhar com XML, texto simples, formatos binários ou outros tipos de representação.

## 9. REST: uma abordagem arquitetural para APIs

**REST (Representational State Transfer)** é um estilo arquitetural para sistemas distribuídos. Ele define restrições que orientam como os componentes de um sistema interagem.

REST não é um protocolo, uma linguagem de programação ou simplesmente uma convenção para nomear URLs.

Entre suas restrições arquiteturais estão:

- **Separação cliente-servidor:** os componentes possuem responsabilidades distintas.
- **Stateless:** cada requisição contém as informações necessárias para ser compreendida, sem depender de um contexto de sessão mantido pelo servidor entre requisições.
- **Cacheabilidade:** as respostas devem indicar se podem ser armazenadas em cache, quando aplicável.
- **Interface uniforme:** a interação segue uma interface consistente, com recursos identificáveis e representações padronizadas.
- **Sistema em camadas:** os componentes podem interagir por meio de intermediários sem precisar conhecer toda a estrutura interna.
- **Código sob demanda:** restrição opcional que permite ao servidor enviar código executável ao cliente.

Uma API REST utiliza essas restrições para organizar a interação entre clientes e recursos.

### Exemplo

```http
GET /api/users/10
```

O cliente solicita uma representação do recurso identificado pelo caminho `/api/users/10`.

Uma resposta possível seria:

```json
{
  "id": 10,
  "name": "Ana",
  "email": "ana@example.com"
}
```

Esse exemplo ilustra uma interação comum em APIs Web inspiradas em REST. Entretanto, **utilizar HTTP, JSON e URLs organizadas por recursos não é suficiente, por si só, para garantir que uma API cumpra todas as restrições de REST**.

Na prática, muitas APIs chamadas de REST são APIs HTTP com características RESTful em diferentes graus.

## 10. RPC: comunicação orientada a operações

**RPC (Remote Procedure Call)** é um modelo de comunicação que permite a um programa solicitar a execução de uma operação em outro processo ou sistema remoto.

A ideia é que o consumidor invoque uma operação disponibilizada pelo servidor, fornecendo os parâmetros necessários e recebendo um resultado.

```text
Cliente
   │
   │ sayHello("Ana")
   ▼
Mecanismo RPC
   │
   ▼
Servidor
   │
   └── Executa a operação sayHello
             │
             ▼
        Retorna o resultado
```

O mecanismo de comunicação cuida de aspectos como o transporte da solicitação e a entrega do resultado, conforme a implementação utilizada.

Uma implementação moderna conhecida é o **gRPC**, que utiliza Protocol Buffers por padrão para definir serviços e mensagens, além de utilizar HTTP/2 como transporte em sua implementação convencional.

RPC não é necessariamente limitado a HTTP ou gRPC. Existem outros sistemas RPC com diferentes protocolos e mecanismos de serialização.

## 11. GraphQL: consultas definidas pelo cliente

**GraphQL** é uma linguagem de consulta para APIs e uma especificação que define como schemas, consultas e operações são validados e executados.

Em uma API GraphQL, o servidor define um schema tipado que descreve os dados e as operações disponíveis. O cliente pode solicitar os campos de que precisa, dentro dos limites desse schema.

Por exemplo:

```graphql
query {
  user(id: 10) {
    name
    email
  }
}
```

O cliente solicita os campos `name` e `email` do usuário identificado por `10`.

Uma resposta possível seria:

```json
{
  "data": {
    "user": {
      "name": "Ana",
      "email": "ana@example.com"
    }
  }
}
```

O servidor valida a consulta, executa os mecanismos responsáveis por obter os dados — frequentemente chamados de *resolvers* — e devolve o resultado de acordo com a estrutura solicitada, respeitando o schema e as regras de execução.

GraphQL é frequentemente disponibilizado por meio de HTTP, mas não é sinônimo de HTTP. Também não significa que o cliente tenha acesso irrestrito a quaisquer dados: o schema, a autorização e a implementação do servidor continuam determinando o que pode ser consultado.

## 12. Comparação entre REST, RPC e GraphQL

As três abordagens permitem construir interfaces para interação entre sistemas, mas organizam a comunicação de maneiras diferentes.

| Característica | REST | RPC | GraphQL |
|---|---|---|---|
| Modelo central | Recursos e suas representações | Operações ou procedimentos | Consultas e operações sobre um schema |
| Como o cliente expressa a intenção | Interage com um recurso por meio de uma interface uniforme | Solicita a execução de uma operação | Declara os campos e dados que deseja consultar |
| Interface | Recursos e restrições arquiteturais | Métodos ou operações definidos pelo serviço | Schema tipado e linguagem de consulta |
| HTTP | Muito comum | Depende da implementação | Muito comum |
| Exemplo ilustrativo | `GET /users/10` | `getUser(10)` | `query { user(id: 10) { name } }` |

Uma forma inicial de memorizar a diferença é:

```text
REST
→ Interação organizada em torno de recursos.

RPC
→ Interação organizada em torno de operações.

GraphQL
→ Consultas e operações definidas por um schema,
  permitindo selecionar os campos desejados.
```

Essa comparação é um modelo mental introdutório, não uma divisão absoluta. Uma API REST também permite executar operações, RPC pode retornar estruturas complexas de dados e GraphQL pode disponibilizar operações que modificam o estado do sistema.

A escolha depende das necessidades do sistema, do contrato desejado, dos consumidores, das ferramentas disponíveis e dos requisitos de manutenção.

## 13. Autenticação, autorização e segurança

Uma API pode disponibilizar operações públicas ou exigir que o consumidor se autentique antes de utilizá-las.

Dois conceitos fundamentais são:

**Autenticação:** processo de verificar a identidade apresentada pelo consumidor.

**Autorização:** processo de determinar quais recursos e operações essa identidade pode acessar.

Por exemplo, uma API pode permitir que qualquer visitante consulte publicações públicas, mas exigir autenticação para criar ou excluir uma publicação. Mesmo autenticado, um usuário pode não ter autorização para excluir publicações de outra pessoa.

As APIs podem utilizar mecanismos como sessões, tokens de acesso, OAuth 2.0 e outros esquemas, dependendo do contexto.

Também é importante considerar:

- Utilização de HTTPS para proteger os dados durante o transporte.
- Validação dos dados recebidos.
- Controle de acesso no servidor.
- Limites de utilização e proteção contra abuso.
- Tratamento de erros sem expor informações internas sensíveis.
- Gestão segura de credenciais e tokens.

A existência de uma API não significa que todas as suas operações devam estar publicamente acessíveis.

## 14. Versionamento e evolução de uma API

Uma API pode ser utilizada por vários consumidores independentes. Por isso, alterações no contrato precisam ser avaliadas com cuidado.

Imagine que uma API retorne:

```json
{
  "id": 10,
  "name": "Ana"
}
```

Se o servidor alterar o nome do campo `name` para `fullName` sem preservar a compatibilidade, os consumidores que esperam encontrar `name` poderão deixar de funcionar.

Por isso, APIs precisam considerar a compatibilidade entre versões, a documentação das mudanças e a comunicação com seus consumidores.

Uma estratégia comum é indicar versões na URL, como:

```text
/api/v1/users
/api/v2/users
```

Essa é uma estratégia possível, não uma exigência universal. Também existem outras formas de versionamento e evolução de contratos.

Nem toda mudança exige uma nova versão. Alterações compatíveis podem ser introduzidas sem quebrar os consumidores existentes, desde que respeitem as regras do contrato.

## 15. Documentação e ferramentas de exploração

A documentação de uma API descreve como seus consumidores podem utilizá-la.

Uma boa documentação pode incluir:

- Endpoints e métodos disponíveis.
- Parâmetros obrigatórios e opcionais.
- Estruturas de requisição e resposta.
- Códigos de status e formatos de erro.
- Requisitos de autenticação e autorização.
- Exemplos de utilização.
- Informações sobre paginação, filtros e ordenação.
- Regras de compatibilidade e versionamento.

Em APIs HTTP, a especificação **OpenAPI** é um formato amplamente utilizado para descrever endpoints, operações, parâmetros, schemas, respostas e mecanismos de segurança.

Ferramentas de exploração de APIs permitem consultar essa documentação e, em muitos casos, executar requisições de teste.

No Liferay, por exemplo, o API Explorer permite explorar APIs disponibilizadas pela instalação, consultar suas operações e examinar os parâmetros e modelos documentados. Em uma instalação local, ele costuma estar disponível em `/o/api`, conforme a configuração e a versão do ambiente.

Esse tipo de ferramenta é especialmente útil durante o aprendizado, pois permite relacionar o contrato documentado com as requisições HTTP efetivamente realizadas.

## 16. Relação entre API, backend e banco de dados

Um dos pontos mais importantes para compreender o desenvolvimento backend é perceber que a API não precisa corresponder diretamente à estrutura do banco de dados.

Considere uma aplicação que permite criar uma publicação.

A API pode receber um JSON contendo o título e o conteúdo. O backend pode validar esses campos, verificar as permissões do usuário, aplicar regras de negócio e, somente então, solicitar à camada de persistência que armazene os dados.

```text
Requisição HTTP
      │
      ▼
API / Controller
      │
      │ Interpreta a requisição
      ▼
Service
      │
      │ Executa as regras de negócio
      ▼
Repository / Persistence
      │
      │ Consulta ou modifica os dados
      ▼
Banco de dados
```

Essa organização é uma possibilidade comum de arquitetura. Os nomes e a divisão exata das camadas variam conforme o framework e as decisões do projeto.

O ponto central é que cada camada pode possuir uma responsabilidade distinta:

- **API ou controller:** recebe a interação externa e a encaminha para o processamento adequado.
- **Service:** coordena operações e aplica regras de negócio, conforme a arquitetura.
- **Repository ou persistence:** abstrai operações de acesso e armazenamento de dados.
- **Banco de dados:** mantém os dados de acordo com o modelo de persistência.

Essa separação permite que a interface pública permaneça relativamente estável mesmo quando a implementação interna evolui.

## 17. Relação com o Service Builder do Liferay

No Liferay Service Builder, é importante distinguir os serviços locais, os serviços remotos e as APIs HTTP que podem ser construídas sobre eles.

Um **Local Service** disponibiliza operações para utilização dentro do ambiente da aplicação Liferay. Ele pode implementar regras de negócio e interagir com a persistência gerada pelo Service Builder. A chamada a um Local Service não é, por si só, uma requisição HTTP.

Um **Remote Service** pode disponibilizar operações para acesso remoto, conforme a configuração e os mecanismos de exposição utilizados pelo Liferay. A existência de um Remote Service também não significa que toda operação será automaticamente disponibilizada como um endpoint HTTP.

Já uma API HTTP, como uma API construída com REST Builder, define uma interface de comunicação Web para que consumidores externos ou internos possam enviar requisições HTTP e receber respostas.

Uma representação simplificada desse fluxo é:

```text
Cliente HTTP
     │
     │ Requisição HTTP
     ▼
API REST / Camada de exposição
     │
     ▼
Serviço da aplicação
     │
     ▼
Local Service / Regras de negócio
     │
     ▼
Persistência do Service Builder
     │
     ▼
Banco de dados
```

O diagrama representa uma organização possível, não uma sequência obrigatória para toda aplicação Liferay. A implementação pode utilizar diferentes serviços, adaptadores e camadas conforme o projeto.

O conceito essencial é que **a API define como o consumidor interage com o sistema, enquanto os serviços e a persistência implementam o comportamento necessário para atender às solicitações**.

## 18. Conclusão

Uma API é uma interface que estabelece como um software pode utilizar as funcionalidades ou os dados disponibilizados por outro componente ou sistema.

Para compreender uma API, é necessário ir além de seus endpoints. Também é importante entender seu contrato, as operações disponíveis, os formatos de dados, o mecanismo de comunicação, as regras de segurança e o comportamento esperado em situações de sucesso ou erro.

No desenvolvimento backend, esses conceitos se relacionam da seguinte maneira:

- A **API** define a interface de interação.
- O **protocolo**, como HTTP, estabelece as regras de comunicação utilizadas na troca de mensagens.
- O **formato de dados**, como JSON, define como as informações são representadas.
- A **implementação do backend** processa as solicitações e executa o comportamento correspondente.
- As **regras de negócio** determinam quais operações são válidas e como o sistema deve se comportar.
- A **persistência** é responsável por armazenar e recuperar dados quando necessário.

REST, RPC e GraphQL são abordagens diferentes para organizar essa interação. Cada uma possui características próprias e pode ser adequada a diferentes necessidades.

Compreender essas distinções é fundamental para estudar APIs Web, desenvolver aplicações backend e entender como tecnologias como o Liferay conectam a comunicação HTTP aos serviços, às regras de negócio e à persistência de dados.