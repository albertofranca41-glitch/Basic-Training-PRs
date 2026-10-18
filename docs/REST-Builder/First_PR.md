# Guia: Atividade de REST Builder (Primeiro PR: Client JAR com Foo e Goo)

Este guia ensina, do zero, a fazer a atividade em que você usa o **client JAR** gerado pelo **REST Builder** para escrever um código **Java** que conversa com a API `headless-r3b2` do **Liferay DXP** (criar, listar, buscar, atualizar e apagar **Foos** e **Goos**).

Ele está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Conceitos necessários: JSONWS × REST, OpenAPI, o que é o REST Builder, módulos gerados, client JAR e o padrão Builder |
| **Etapa 2 – O código** | O código completo da atividade, comentado, explicado parte por parte, e como compilar, executar e enviar o PR |

> Este guia cobre **somente o primeiro PR** de REST Builder: o uso do **client JAR** (seção "Client Resource" do tutorial). O uso do **OSGi Resource** (`R3B2Portlet`) é outro PR e não está aqui.

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 Revisão rápida: cliente, servidor, HTTP, REST e JSON](#11-revisão-rápida-cliente-servidor-http-rest-e-json)
  - [1.2 JSONWS × REST: RPC e recursos HTTP](#12-jsonws--rest-rpc-e-recursos-http)
  - [1.3 OpenAPI: o contrato da API](#13-openapi-o-contrato-da-api)
  - [1.4 O que é o REST Builder?](#14-o-que-é-o-rest-builder)
  - [1.5 Os módulos do projeto `liferay-r3b2`](#15-os-módulos-do-projeto-liferay-r3b2)
  - [1.6 Código gerado e código implementado pelo desenvolvedor](#16-código-gerado-e-código-implementado-pelo-desenvolvedor)
  - [1.7 A API de exemplo: Foo e Goo](#17-a-api-de-exemplo-foo-e-goo)
  - [1.8 API Explorer: verificando a API no navegador](#18-api-explorer-verificando-a-api-no-navegador)
  - [1.9 Client JAR × OSGi Resource](#19-client-jar--osgi-resource)
  - [1.10 O padrão Builder](#110-o-padrão-builder)
  - [1.11 Pré-requisitos do ambiente](#111-pré-requisitos-do-ambiente)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 Código completo comentado](#22-código-completo-comentado)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
  - [2.4 Como compilar e executar](#24-como-compilar-e-executar)
  - [2.5 Como enviar o PR](#25-como-enviar-o-pr)
  - [2.6 Pontos de melhoria](#26-pontos-de-melhoria)
  - [2.7 Erros comuns](#27-erros-comuns)

---

# Etapa 1 – Teoria

Antes de escrever o código, vamos entender como o REST Builder se encaixa no ecossistema do Liferay e como um programa Java pode consumir uma API REST por meio de um *client JAR*.

Esta etapa apresenta os conceitos necessários para acompanhar a atividade. Se você já conhece algum deles, pode utilizar os links de recursos adicionais no final para consultar apenas o que precisar.

## 1.1 Revisão rápida: cliente, servidor, HTTP, REST e JSON

Esta atividade utiliza conceitos semelhantes aos da atividade de JavaScript com a Headless Delivery API. A diferença principal está na maneira como o programa cliente realiza as requisições.

### Cliente e servidor

O **cliente** é o programa que solicita uma operação. O **servidor** é o sistema que recebe a solicitação, processa a operação e devolve uma resposta.

Nesta atividade:

- **Servidor:** o Liferay DXP, executado em `http://localhost:8080`.
- **Cliente:** o programa Java que utiliza o *client JAR* para consumir a API `headless-r3b2`.

Embora ambos estejam sendo executados no mesmo computador, eles desempenham papéis diferentes. O programa Java continua sendo um cliente externo ao Liferay.

### HTTP

O **HTTP (Hypertext Transfer Protocol)** é o protocolo utilizado na comunicação entre o cliente e o servidor.

Uma requisição HTTP pode conter:

- **Método:** indica a operação solicitada, como `GET`, `POST`, `PUT`, `PATCH` ou `DELETE`.
- **URL:** identifica o destino da requisição.
- **Cabeçalhos:** transportam informações adicionais, como o tipo de conteúdo e as credenciais de autenticação.
- **Corpo:** contém dados enviados ao servidor, quando necessário.

O servidor processa a requisição e devolve uma resposta com um código de status e, dependendo da operação, um corpo com os dados retornados.

### REST

**REST (Representational State Transfer)** é um estilo arquitetural utilizado para estruturar sistemas que se comunicam por meio de recursos.

Em uma API REST, os recursos são identificados por URLs, enquanto os métodos HTTP indicam as operações solicitadas sobre eles.

Por exemplo:

- `POST /foo`: solicita a criação de um Foo.
- `GET /foo`: solicita a listagem de Foos.
- `GET /foo/123`: solicita a consulta de um Foo específico.
- `DELETE /foo/123`: solicita a exclusão de um Foo.

Esses caminhos são exemplos ilustrativos. Os caminhos efetivamente utilizados nesta atividade estão definidos no contrato da API.

### JSON

**JSON (JavaScript Object Notation)** é um formato textual utilizado para representar e transportar dados estruturados.

Uma representação de um Foo poderia ser:

```json
{
  "name": "Exemplo",
  "description": "Um recurso de demonstração"
}
```

O cliente envia ou recebe esses dados conforme o contrato da API.

É importante perceber que JSON e Java são coisas diferentes: JSON é um formato de dados, enquanto Java é uma linguagem de programação.

### O que muda nesta atividade?

Na atividade de JavaScript, você utilizava `fetch`, configurava os cabeçalhos HTTP e convertia objetos JavaScript em JSON com `JSON.stringify`.

Nesta atividade, você utiliza objetos Java, como `Foo` e `Goo`, e métodos como `postFoo` e `getFoo`. O *client JAR* cuida de boa parte da comunicação HTTP e da conversão entre os objetos Java e os dados transmitidos.

Isso não elimina o HTTP nem o JSON. Apenas permite trabalhar em um nível de abstração mais alto.

## 1.2 JSONWS × REST: RPC e recursos HTTP

O Liferay oferece diferentes mecanismos para disponibilizar funcionalidades por meio de serviços remotos. Dois conceitos importantes nesse contexto são JSONWS e REST.

### JSONWS e RPC

O **JSONWS** é o mecanismo do Liferay que permite expor métodos de serviços por meio de chamadas remotas, utilizando JSON como formato de comunicação.

Ele está associado ao modelo **RPC (Remote Procedure Call)**, no qual o cliente solicita a execução de uma operação disponibilizada remotamente.

Por exemplo, conceitualmente, o cliente pode solicitar a execução de um método como `addEntry(...)`.

Nesse modelo, a operação é apresentada principalmente como uma chamada de método.

### REST e recursos

Em uma API REST, o contrato é organizado em torno de recursos e operações HTTP.

Por exemplo:

```http
POST /entries
```

A requisição indica que o cliente deseja criar um recurso na coleção de entradas, conforme o contrato da API.

A diferença fundamental não é que um mecanismo utiliza métodos e o outro não. Ambos podem envolver métodos de programação internamente. A diferença está na maneira como a interface de comunicação é modelada e apresentada ao consumidor.

| Aspecto | JSONWS / RPC | REST |
|---|---|---|
| Modelo de interação | Chamada de operações remotas | Interação com recursos |
| Interface | Métodos e parâmetros | URLs, métodos HTTP e representações |
| Contrato | Operações disponibilizadas pelo serviço | Recursos, operações, parâmetros e respostas |
| No ecossistema Liferay | Associado à exposição remota de serviços | Pode ser implementado com REST Builder |

O JSONWS frequentemente expõe operações relacionadas aos serviços do Liferay e pode refletir conceitos internos da aplicação. Entretanto, isso não significa que ele seja uma chamada direta ao banco de dados.

Da mesma forma, uma API REST pode utilizar entidades do Service Builder internamente. REST não exige que a aplicação tenha um banco de dados específico nem garante, por si só, independência total da persistência.

**Em resumo:** RPC enfatiza a chamada de uma operação remota; REST organiza a interface em torno de recursos e das operações definidas para eles.

## 1.3 OpenAPI: o contrato da API

O **OpenAPI** é uma especificação utilizada para descrever APIs HTTP, especialmente APIs REST, de maneira estruturada.

Em vez de depender exclusivamente de documentação escrita em linguagem natural, uma API pode ter seu contrato descrito em um arquivo YAML ou JSON que ferramentas conseguem interpretar.

Entre outras informações, o contrato pode definir:

- **Paths:** os caminhos disponíveis na API, como `/foo` e `/foo/{fooId}`.
- **Operations:** os métodos HTTP disponíveis em cada caminho, como `get`, `post`, `put`, `patch` e `delete`.
- **Parameters:** os parâmetros aceitos, incluindo parâmetros de caminho, consulta e cabeçalho.
- **Request bodies:** os dados que podem ser enviados em uma requisição.
- **Responses:** as respostas e os códigos de status esperados.
- **Schemas:** os formatos dos dados utilizados pela API, incluindo campos, tipos e estruturas.

Na API desta atividade, os schemas `Foo` e `Goo` descrevem os modelos de dados que o cliente e o servidor utilizam para representar esses recursos.

Podemos pensar no OpenAPI como a planta de uma casa: ele descreve a estrutura esperada, mas não é a própria casa.

O arquivo OpenAPI também não implementa automaticamente todas as regras de negócio. Ele descreve o contrato de comunicação; o comportamento efetivo depende da implementação correspondente.

## 1.4 O que é o REST Builder?

O **REST Builder** é uma ferramenta do Liferay que gera código a partir da definição de uma API REST.

No projeto `liferay-r3b2`, os principais arquivos de entrada são:

| Arquivo | Responsabilidade |
|---|---|
| `rest-config.yaml` | Contém informações de configuração da API utilizada pelo gerador, como identificadores e informações de organização do projeto. |
| `rest-openapi.yaml` | Descreve o contrato da API, incluindo caminhos, operações, parâmetros, respostas e schemas. |

Os detalhes exatos de configuração dependem da versão e da estrutura do projeto.

No projeto desta atividade, o comando utilizado para executar a geração é:

```bash
./gradlew headless-r3b2-impl:buildREST
```

O REST Builder lê a definição e gera classes e outros arquivos necessários à estrutura da API. Entre os resultados podem estar DTOs, interfaces de recursos, classes-base, classes de cliente e outros artefatos previstos pela configuração do projeto.

O processo pode ser representado assim:

```text
rest-config.yaml
        +
rest-openapi.yaml
        |
        v
   REST Builder
        |
        v
Código gerado para a API
        |
        +----> Contratos e DTOs
        +----> Estrutura de implementação REST
        +----> Client JAR
        +----> Outros artefatos previstos
```

O conjunto exato de arquivos gerados depende da configuração e da versão utilizada.

É importante distinguir duas atividades:

1. **Gerar o código:** transformar a definição da API em estruturas de programação.
2. **Implementar o comportamento:** escrever a lógica necessária para que as operações façam o que a aplicação exige.

O REST Builder automatiza boa parte da estrutura, mas não substitui o trabalho de implementar o comportamento da API.

## 1.5 Os módulos do projeto `liferay-r3b2`

O projeto está dividido em módulos Gradle. Cada módulo possui uma responsabilidade específica.

| Módulo | Responsabilidade | Precisa ser implantado no Liferay? |
|---|---|---|
| `headless-r3b2-api` | Contém contratos, interfaces e DTOs utilizados pela implementação no servidor. | Sim |
| `headless-r3b2-impl` | Contém a implementação da API e os componentes necessários para disponibilizá-la no Liferay. | Sim |
| `headless-r3b2-client` | Contém a biblioteca Java utilizada por programas clientes para consumir a API por HTTP. | Não |
| `headless-r3b2-test` | Contém estruturas e testes relacionados ao projeto, conforme sua configuração. | Não, para o consumo normal da API |

Os módulos `api` e `impl` participam da disponibilização da API no servidor. Já o módulo `client` permite que outro programa Java consuma essa API.

### Por que existem dois `FooResource`?

Neste projeto, há classes com nomes semelhantes em pacotes diferentes.

O `FooResource` do módulo `api` pertence ao lado servidor da aplicação. Ele faz parte do contrato utilizado pela implementação REST dentro do Liferay.

Já o `FooResource` do módulo `client` pertence à biblioteca cliente. Ele oferece métodos Java que permitem ao programa cliente fazer requisições à API remota.

A semelhança dos nomes existe porque as duas classes participam do mesmo contrato, mas suas responsabilidades são diferentes.

Na atividade deste primeiro PR, o programa utiliza as classes do pacote:

```java
com.acme.headless.r3b2.client.resource.v1_0
```

Portanto, é importante conferir o pacote dos imports: o código utiliza o cliente gerado, e não a interface de recurso interna do servidor.

## 1.6 Código gerado e código implementado pelo desenvolvedor

O REST Builder gera estruturas que servem de base para a API. Entretanto, nem todo arquivo gerado deve ser tratado da mesma maneira.

No projeto, é importante distinguir:

- **Código gerado e mantido pelo gerador:** contém estruturas que podem ser substituídas quando a geração é executada novamente.
- **Classes de implementação:** são pontos de extensão destinados ao desenvolvedor escrever a lógica específica da aplicação.
- **Código do cliente:** contém as classes utilizadas por programas externos para consumir a API.

Em geral, classes como `FooResource.java`, classes-base e DTOs gerados não devem ser editados manualmente sem uma razão específica. Alterações nesses arquivos podem ser perdidas em uma nova geração.

Já classes como `FooResourceImpl.java` são pontos de implementação destinados a receber a lógica da aplicação, conforme a estrutura gerada pelo projeto.

Os métodos implementados podem utilizar `@Override` para indicar que sobrescrevem métodos declarados em uma classe-base ou implementam métodos definidos por uma interface.

O comportamento exato da geração e da preservação de arquivos deve ser confirmado na estrutura do projeto e na versão utilizada. Não é recomendável presumir que todos os arquivos terminados em `Impl.java`, em qualquer projeto, sejam tratados de maneira idêntica.

### Qual é a relação com o Service Builder?

O Service Builder também gera código e disponibiliza pontos de extensão para o desenvolvedor.

Entretanto, os dois geradores atuam em responsabilidades diferentes:

- **Service Builder:** gera estruturas relacionadas às entidades, à persistência e aos serviços da aplicação.
- **REST Builder:** gera estruturas relacionadas ao contrato e à disponibilização de uma API REST, além de artefatos de cliente.

O `FooResourceImpl` não deve ser confundido com uma implementação de serviço de negócio. Ele pertence à camada de exposição HTTP e pode delegar operações às camadas de serviço apropriadas.

## 1.7 A API de exemplo: Foo e Goo

A API `headless-r3b2` utiliza dois recursos de exemplo: `Foo` e `Goo`.

No contrato desta atividade, `Goo` está associado a `Foo`. Esse relacionamento é representado pelo campo `fooId` e pelos caminhos de API que permitem criar e listar Goos associados a um Foo.

Podemos visualizar o relacionamento assim:

```text
Foo
 |
 +---- Goo
 |
 +---- Goo
 |
 +---- Goo
```

Os campos relevantes para a atividade são:

| Schema | Campos utilizados |
|---|---|
| `Foo` | `id`, `name`, `description` |
| `Goo` | `id`, `name`, `description`, `fooId` |

O campo `id` identifica o recurso. Na atividade, os identificadores são obtidos a partir das respostas do servidor, de acordo com o comportamento implementado.

O campo `fooId` identifica o Foo ao qual um Goo está associado.

### Endpoints utilizados

A URL-base da API é:

```text
http://localhost:8080/o/headless-r3b2/v1.0
```

Os caminhos abaixo são relativos a essa URL-base.

| Operação | Método HTTP | Caminho | Método no client JAR |
|---|---|---|---|
| Criar Foo | `POST` | `/foo` | `fooResource.postFoo(foo)` |
| Listar Foos | `GET` | `/foo` | `fooResource.getFooPage()` |
| Buscar um Foo | `GET` | `/foo/{fooId}` | `fooResource.getFoo(fooId)` |
| Atualizar Foo | `PUT` | `/foo/{fooId}` | `fooResource.putFoo(fooId, foo)` |
| Atualizar parcialmente um Foo | `PATCH` | `/foo/{fooId}` | `fooResource.patchFoo(fooId, foo)` |
| Apagar Foo | `DELETE` | `/foo/{fooId}` | `fooResource.deleteFoo(fooId)` |
| Criar Goo associado a Foo | `POST` | `/foo/{fooId}/goos` | `gooResource.postFooGoo(fooId, goo)` |
| Listar Goos de um Foo | `GET` | `/foo/{fooId}/goos` | `gooResource.getFooGoosPage(fooId)` |
| Buscar um Goo | `GET` | `/goo/{gooId}` | `gooResource.getGoo(gooId)` |
| Atualizar Goo | `PUT` | `/goo/{gooId}` | `gooResource.putGoo(gooId, goo)` |
| Atualizar parcialmente um Goo | `PATCH` | `/goo/{gooId}` | `gooResource.patchGoo(gooId, goo)` |
| Apagar Goo | `DELETE` | `/goo/{gooId}` | `gooResource.deleteGoo(gooId)` |

Os nomes dos métodos Java correspondem ao cliente gerado para o contrato utilizado no projeto. Caso o OpenAPI seja alterado e o cliente seja gerado novamente, os métodos disponíveis podem mudar.

Observe que o Goo pode ser criado e listado por meio de caminhos associados ao Foo, mas também possui um identificador próprio para operações individuais.

Isso ilustra uma característica importante das APIs REST: um recurso pode participar de uma relação com outro recurso e, ainda assim, ter operações próprias.

### PUT e PATCH

Os métodos `PUT` e `PATCH` são utilizados para operações de atualização, mas possuem semânticas diferentes.

- **PUT:** normalmente representa a substituição da representação do recurso no estado desejado pelo cliente.
- **PATCH:** representa uma atualização parcial, conforme as regras definidas para a operação.

Na atividade, o exemplo de `PATCH` envia apenas o campo `description`. Já a operação `PUT` deve respeitar os campos e o comportamento definidos no contrato e na implementação.

Não se deve concluir que qualquer API REST sempre substitui todos os campos de uma mesma maneira em toda operação `PUT`: é necessário consultar o contrato específico e entender como o servidor processa os dados recebidos.

## 1.8 API Explorer: verificando a API no navegador

O **API Explorer** do Liferay permite consultar e testar APIs disponibilizadas pelo servidor.

Para verificar a API desta atividade:

1. Confirme que o Liferay DXP está em execução.
2. Implante os módulos `headless-r3b2-api` e `headless-r3b2-impl`.
3. Acesse `http://localhost:8080/o/api`.
4. Procure pela API `headless-r3b2/v1.0`.

A API precisa estar corretamente disponibilizada no servidor para que seus endpoints possam ser executados. Se ela não aparecer, é necessário investigar a implantação, a configuração e eventuais erros no Liferay.

O API Explorer permite visualizar operações e, quando configurado e autorizado, enviar requisições de teste pela interface.

O tutorial também propõe exercitar os endpoints com `curl` e Postman. Essas ferramentas são úteis para entender como a API se comporta antes de utilizar o cliente Java.

Por exemplo, com o API Explorer, `curl` ou Postman, é possível observar a URL, o método HTTP, os cabeçalhos, o corpo da requisição e a resposta do servidor.

Essa observação ajuda a compreender o que o client JAR automatiza posteriormente.

**Importante:** o API Explorer, a API implantada e o client JAR são elementos diferentes. O Explorer é uma interface para consultar e testar a API; a implantação disponibiliza a API no servidor; o client JAR fornece uma biblioteca Java para consumi-la.

## 1.9 Client JAR × OSGi Resource

Nesta atividade, o programa utiliza o **client JAR**. Em outro PR, o foco será o consumo do recurso OSGi dentro do próprio Liferay.

Embora ambos possam utilizar classes relacionadas ao mesmo contrato REST, o modo de comunicação é diferente.

| Aspecto | Client JAR | OSGi Resource |
|---|---|---|
| Onde o código consumidor é executado | Em um programa Java externo | Dentro do ambiente OSGi do Liferay |
| Como a operação é chamada | Por HTTP, através do cliente gerado | Por chamada de método dentro do ambiente Java/OSGi |
| Autenticação | Precisa configurar credenciais ou outro mecanismo apropriado para a API remota | Pode utilizar o contexto do usuário já autenticado, conforme a implementação |
| Pacote de referência | `...client.resource.v1_0` | `...resource.v1_0` |
| Comunicação | Envolve requisição, resposta e serialização | Não precisa atravessar HTTP quando utiliza uma chamada local |

No caso do client JAR, a biblioteca não executa a implementação REST dentro do processo do cliente. Ela fornece uma interface Java para enviar requisições ao servidor.

No caso de um OSGi Resource utilizado dentro do Liferay, o código pode acessar componentes disponíveis no ambiente OSGi e realizar chamadas internas. Isso não significa que toda chamada local seja automaticamente mais rápida ou que esteja livre de outras operações: o custo real depende do caminho executado.

### O que é um JAR?

**JAR (Java Archive)** é um formato de arquivo que empacota classes Java e outros recursos.

O client JAR reúne as classes geradas para facilitar o consumo da API por outros programas Java.

Para utilizá-lo, o programa precisa ter acesso ao JAR e às dependências exigidas por ele.

### O que é classpath?

O **classpath** é o conjunto de caminhos e arquivos nos quais o Java procura classes e recursos necessários à compilação ou execução de um programa.

Se o client JAR não estiver disponível no classpath — ou nas configurações equivalentes do sistema de build —, o compilador não conseguirá resolver imports como:

```java
import com.acme.headless.r3b2.client.resource.v1_0.FooResource;
```

Além disso, o JAR do cliente pode depender de outras bibliotecas. Ter apenas o arquivo principal não garante que todas as dependências necessárias estejam disponíveis.

Em projetos Gradle ou Maven, essas dependências podem ser declaradas e resolvidas pelo sistema de build. Em uma compilação manual, é necessário disponibilizar os arquivos necessários explicitamente.

## 1.10 O padrão Builder

No código da atividade, você encontrará uma construção semelhante a esta:

```java
FooResource fooResource = FooResource.builder()
    .authentication("test@liferay.com", "learn")
    .build();
```

Esse código utiliza o **padrão Builder**, um padrão de projeto que permite construir e configurar objetos em etapas.

Em vez de exigir que todos os parâmetros sejam fornecidos diretamente em um único construtor, o Builder disponibiliza métodos para configurar o objeto e, ao final, construir a instância.

No exemplo:

1. `FooResource.builder()` inicia o processo de construção e devolve um objeto Builder.
2. `.authentication(...)` configura a autenticação do cliente.
3. `.build()` constrói e devolve a instância de `FooResource`.

O objeto retornado é o cliente que será utilizado para chamar as operações disponíveis na API.

A implementação gerada pode oferecer outras opções de configuração. É necessário consultar as classes geradas para conhecer os métodos disponíveis e os valores padrão.

### Autenticação e segurança

O método `.authentication(usuario, senha)` configura a autenticação básica HTTP (Basic Authentication) no cliente gerado para esta atividade.

Em termos gerais, a autenticação básica transmite as credenciais em um cabeçalho HTTP, utilizando uma representação codificada em Base64. Essa codificação não é criptografia.

Por isso, credenciais não devem ser transmitidas em conexões HTTP desprotegidas em ambientes reais. A autenticação precisa ser utilizada com HTTPS e com as demais medidas de segurança apropriadas.

As credenciais `test@liferay.com` e `learn` são exemplos do ambiente de treinamento, e não devem ser tratadas como credenciais universais de instalações Liferay.

## 1.11 Pré-requisitos do ambiente

Antes de executar o código, confira os seguintes requisitos.

| Ferramenta ou etapa | Finalidade |
|---|---|
| Liferay DXP em execução | Disponibilizar a API em `localhost:8080` |
| Projeto `liferay-r3b2` configurado | Fornecer os módulos e a definição da API |
| REST Builder executado | Gerar as estruturas correspondentes ao contrato |
| Módulos `api` e `impl` implantados | Disponibilizar a API no servidor |
| Client JAR gerado | Fornecer as classes Java do cliente |
| JDK compatível | Compilar e executar o programa Java |
| Dependências do cliente | Permitir a compilação e execução das classes utilizadas |
| Git e conta no GitHub | Versionar as alterações e enviar o PR |
| Editor de código | Escrever e revisar o programa |

Para verificar se Java e Git estão disponíveis no terminal:

```bash
java -version
javac -version
git --version
```

O JDK deve ser compatível com a versão de Java exigida pelo projeto e pelas bibliotecas utilizadas.

Antes de executar o programa, confirme também que a API `headless-r3b2/v1.0` está disponível no API Explorer e que o usuário utilizado tem as permissões necessárias para realizar as operações.

## Recursos adicionais

Os materiais abaixo aprofundam conceitos usados nesta atividade. Os links levam a documentos existentes no repositório ou a referências oficiais.

### Fundamentos da comunicação

- [Cliente e servidor](../JavaScript/Cliente%20e%20Servidor.md): papéis, requisições e respostas.
- [HTTP](../JavaScript/HTTP.md): métodos, URLs, cabeçalhos, corpos e códigos de status.
- [REST](../JavaScript/REST.md): recursos, representações e métodos HTTP.
- [API](../JavaScript/API.md): conceito de interface e contrato entre sistemas.

### Documentação oficial

- [REST Builder — documentação do Liferay](https://learn.liferay.com/w/dxp/integration/headless-apis/rest-builder): documentação oficial sobre geração de APIs REST no Liferay.
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html): especificação oficial do formato OpenAPI.
- [HTTP — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP): referência sobre o protocolo HTTP e suas operações.


# Etapa 2 – O código

## 2.1 Visão geral

O arquivo `Test.java` tem **um método `main`** (que executa o ciclo completo) e **doze métodos auxiliares**, um por operação:

```
main():          orquestra tudo (cria, consulta, atualiza, limpa)

Foo:             postFoo()      → POST   (Create)
                 getFooPage()   → GET    (Read - coleção)
                 getFoo()       → GET    (Read - um)
                 putFoo()       → PUT    (Update completo)
                 patchFoo()     → PATCH  (Update parcial)
                 deleteFoo()    → DELETE (Delete)

Goo:             postGoo()      → POST   (Create, dentro de um Foo)
                 getGooPage()   → GET    (Read - coleção de um Foo)
                 getGoo()       → GET    (Read - um)
                 putGoo()       → PUT    (Update completo)
                 patchGoo()     → PATCH  (Update parcial)
                 deleteGoo()    → DELETE (Delete)
```

O fluxo do `main` é:

```
1. Cria os dois "resources" (Foo e Goo) já autenticados
2. try {
3.   Cria Foo → lista → busca → PUT → busca → PATCH → busca
4.   Cria Goo (ligado ao Foo) → lista → busca → PUT → busca → PATCH → busca
5. } finally {
6.   Apaga o Goo primeiro, depois o Foo (mesmo que algo tenha falhado)
7. }
```

> O `finally` é o coração da organização do código: **o que foi criado é sempre limpo no final**, mesmo se uma etapa do meio lançar exceção.

## 2.2 Código completo comentado

> A lógica é exatamente a do código da atividade. Salve o arquivo como `Test.java` (o nome do arquivo deve ser igual ao nome da classe pública).

```java
import com.acme.headless.r3b2.client.dto.v1_0.Foo;
import com.acme.headless.r3b2.client.dto.v1_0.Goo;
import com.acme.headless.r3b2.client.resource.v1_0.FooResource;
import com.acme.headless.r3b2.client.resource.v1_0.GooResource;

public class Test {

    public static void main(String[] args) throws Exception {
        // Guardam os IDs gerados pelo servidor. Começam como null para
        // que o bloco finally saiba o que chegou a ser criado.
        Long fooId = null;
        Long gooId = null;

        // Cria os "resources" do client JAR já com autenticação (Basic Auth).
        FooResource fooResource = FooResource.builder()
                .authentication("test@liferay.com", "learn")
                .build();

        GooResource gooResource = GooResource.builder()
                .authentication("test@liferay.com", "learn")
                .build();

        try {
            // Cria um Foo e guarda seu ID para as próximas operações.
            fooId = postFoo(fooResource);

            // Consulta a coleção de Foos.
            getFooPage(fooResource);

            // Consulta o Foo criado pelo seu ID.
            getFoo(fooResource, fooId);

            // Atualiza os dados do Foo usando PUT.
            putFoo(fooResource, fooId);

            // Consulta o Foo para verificar a atualização.
            getFoo(fooResource, fooId);

            // Altera parcialmente os dados do Foo usando PATCH.
            patchFoo(fooResource, fooId);

            // Consulta o Foo para verificar a alteração parcial.
            getFoo(fooResource, fooId);

            // Cria um Goo associado ao Foo.
            gooId = postGoo(gooResource, fooId);

            // Consulta a coleção de Goos associados ao Foo.
            getGooPage(gooResource, fooId);

            // Consulta o Goo criado pelo seu ID.
            getGoo(gooResource, gooId);

            // Atualiza os dados do Goo usando PUT.
            putGoo(gooResource, gooId, fooId);

            // Consulta o Goo para verificar a atualização.
            getGoo(gooResource, gooId);

            // Altera parcialmente os dados do Goo usando PATCH.
            patchGoo(gooResource, gooId);

            // Consulta o Goo para verificar a alteração parcial.
            getGoo(gooResource, gooId);

        } finally {
            /*
             * Remove primeiro o Goo, pois ele está associado ao Foo.
             * Depois, tenta remover o Foo, desde que a exclusão do Goo
             * tenha sido concluída com sucesso.
             */
            boolean gooDeleted = (gooId == null);

            if (gooId != null) {
                gooDeleted = deleteGoo(gooResource, gooId);
            }

            if (fooId != null && gooDeleted) {
                deleteFoo(fooResource, fooId);
            } else if (fooId != null) {
                System.err.println(
                        "O Foo de ID " + fooId
                                + " não foi removido porque a exclusão do Goo falhou."
                );
            }
        }
    }

    // Cria um Foo e retorna o ID gerado pelo servidor.
    private static Long postFoo(FooResource fooResource) throws Exception {
        Foo foo = new Foo();

        foo.setName("Foo 1");
        foo.setDescription("Foo POST");

        // POST /v1.0/foo
        Foo createdFoo = fooResource.postFoo(foo);

        System.out.println("Foo criado: " + createdFoo);

        return createdFoo.getId();
    }

    // Consulta a coleção de Foos.
    private static void getFooPage(FooResource fooResource) throws Exception {
        // GET /v1.0/foo
        System.out.println("Coleção de Foos: " + fooResource.getFooPage());
    }

    // Consulta um Foo específico pelo ID.
    private static void getFoo(
            FooResource fooResource,
            Long fooId
    ) throws Exception {
        // GET /v1.0/foo/{fooId}
        System.out.println("Foo consultado: " + fooResource.getFoo(fooId));
    }

    // Substitui os dados do Foo usando PUT.
    private static void putFoo(
            FooResource fooResource,
            Long fooId
    ) throws Exception {
        Foo foo = new Foo();

        foo.setName("Different name");
        foo.setDescription("Another description");

        // PUT /v1.0/foo/{fooId}
        fooResource.putFoo(fooId, foo);

        System.out.println("Foo atualizado com PUT.");
    }

    // Altera parcialmente os dados do Foo usando PATCH.
    private static void patchFoo(
            FooResource fooResource,
            Long fooId
    ) throws Exception {
        Foo foo = new Foo();

        foo.setDescription("Foo PATCH");

        // PATCH /v1.0/foo/{fooId}
        fooResource.patchFoo(fooId, foo);

        System.out.println("Foo atualizado com PATCH.");
    }

    // Exclui um Foo e informa se a operação foi concluída.
    private static boolean deleteFoo(
            FooResource fooResource,
            Long fooId
    ) {
        try {
            // DELETE /v1.0/foo/{fooId}
            fooResource.deleteFoo(fooId);

            System.out.println("Foo removido: " + fooId);

            return true;
        } catch (Exception e) {
            System.err.println("Erro ao remover o Foo de ID " + fooId);
            e.printStackTrace();

            return false;
        }
    }

    // Cria um Goo associado ao Foo e retorna o ID gerado pelo servidor.
    private static Long postGoo(
            GooResource gooResource,
            Long fooId
    ) throws Exception {
        Goo goo = new Goo();

        goo.setName("Goo 1");
        goo.setDescription("Goo POST");

        // POST /v1.0/foo/{fooId}/goos
        Goo createdGoo = gooResource.postFooGoo(fooId, goo);

        System.out.println("Goo criado: " + createdGoo);

        return createdGoo.getId();
    }

    // Consulta os Goos associados a um Foo.
    private static void getGooPage(
            GooResource gooResource,
            Long fooId
    ) throws Exception {
        // GET /v1.0/foo/{fooId}/goos
        System.out.println(
                "Coleção de Goos: " + gooResource.getFooGoosPage(fooId)
        );
    }

    // Consulta um Goo específico pelo ID.
    private static void getGoo(
            GooResource gooResource,
            Long gooId
    ) throws Exception {
        // GET /v1.0/goo/{gooId}
        System.out.println("Goo consultado: " + gooResource.getGoo(gooId));
    }

    // Substitui os dados do Goo usando PUT.
    private static void putGoo(
            GooResource gooResource,
            Long gooId,
            Long fooId
    ) throws Exception {
        Goo goo = new Goo();

        goo.setName("Different name");
        goo.setDescription("Another description");
        goo.setFooId(fooId);

        // PUT /v1.0/goo/{gooId}
        gooResource.putGoo(gooId, goo);

        System.out.println("Goo atualizado com PUT.");
    }

    // Altera parcialmente os dados do Goo usando PATCH.
    private static void patchGoo(
            GooResource gooResource,
            Long gooId
    ) throws Exception {
        Goo goo = new Goo();

        goo.setDescription("Goo PATCH");

        // PATCH /v1.0/goo/{gooId}
        gooResource.patchGoo(gooId, goo);

        System.out.println("Goo atualizado com PATCH.");
    }

    // Exclui um Goo e informa se a operação foi concluída.
    private static boolean deleteGoo(
            GooResource gooResource,
            Long gooId
    ) {
        try {
            // DELETE /v1.0/goo/{gooId}
            gooResource.deleteGoo(gooId);

            System.out.println("Goo removido: " + gooId);

            return true;
        } catch (Exception e) {
            System.err.println("Erro ao remover o Goo de ID " + gooId);
            e.printStackTrace();

            return false;
        }
    }
}
```

## 2.3 Explicação por partes

### Parte 1 – Os `import`

```java
import com.acme.headless.r3b2.client.dto.v1_0.Foo;
import com.acme.headless.r3b2.client.dto.v1_0.Goo;
import com.acme.headless.r3b2.client.resource.v1_0.FooResource;
import com.acme.headless.r3b2.client.resource.v1_0.GooResource;
```

| Classe | Papel |
|---|---|
| `Foo`, `Goo` | **DTOs** (*Data Transfer Objects*): objetos simples que carregam os dados (`name`, `description`, `id`...). Têm `get`/`set` para cada campo |
| `FooResource`, `GooResource` | **Resources do client**: cada método faz uma chamada HTTP à API |

Repare no `.client.` no meio do pacote: são as classes do **client JAR** (por HTTP), e não as do módulo `api` (dentro do Liferay).

### Parte 2 – Criando os resources com autenticação

```java
FooResource fooResource = FooResource.builder()
        .authentication("test@liferay.com", "learn")
        .build();
```

É o **padrão Builder** (seção 1.10). O resultado é um objeto que, a cada chamada, já envia o usuário e a senha. Criamos **um resource para cada entidade** (Foo e Goo).

> Use as credenciais do **seu** ambiente local de treinamento. Elas podem ser diferentes das do exemplo (no tutorial, o `Test.java` usa a senha `test`, enquanto esta atividade usa `learn`).

### Parte 3 – `Long fooId = null;` e `Long gooId = null;`

Os IDs são declarados **antes** do `try` e começam como `null`. Por quê?

- Eles são usados no bloco `finally`, que está **fora** do `try`; uma variável declarada dentro do `try` não existe no `finally`.
- `null` significa "ainda não foi criado". Assim, o `finally` sabe o que precisa (ou não) ser apagado.
- Usamos `Long` (objeto), e não `long` (primitivo), justamente porque o primitivo **não pode ser `null`**.

### Parte 4 – O bloco `try { ... } finally { ... }`

```java
try {
    // criar, consultar, atualizar...
} finally {
    // limpeza
}
```

Diferente do `try/catch`, aqui **não há `catch`**: se algo falhar, a exceção continua subindo (e o programa mostra o erro), mas o `finally` **sempre executa antes**. Resultado: **não ficam Foos e Goos "órfãos"** de teste no seu Liferay, mesmo quando algo dá errado no meio.

É por isso também que `main` declara `throws Exception`: as exceções que não tratamos aqui são repassadas.

### Parte 5 – A ordem das operações no `try`

```
Foo:  POST → GET (lista) → GET → PUT → GET → PATCH → GET
Goo:  POST → GET (lista) → GET → PUT → GET → PATCH → GET
```

Depois de cada alteração há um `GET` para **conferir o resultado**: o `GET` depois do `PUT` mostra o Foo com os campos substituídos; o `GET` depois do `PATCH` mostra que **só a `description` mudou** e o `name` continua o do `PUT`.

O Goo só é criado **depois** do Foo porque ele precisa do `fooId`.

### Parte 6 – O `finally` e a ordem de exclusão

```java
boolean gooDeleted = (gooId == null);

if (gooId != null) {
    gooDeleted = deleteGoo(gooResource, gooId);
}

if (fooId != null && gooDeleted) {
    deleteFoo(fooResource, fooId);
} else if (fooId != null) {
    System.err.println("O Foo de ID " + fooId + " não foi removido porque a exclusão do Goo falhou.");
}
```

Raciocínio:

1. O Goo **pertence** ao Foo, então apagamos o **Goo primeiro**.
2. `gooDeleted` começa como `true` se **nunca criamos** um Goo (`gooId == null`), pois não há nada a apagar.
3. Se criamos um Goo, `deleteGoo` devolve `true` (apagou) ou `false` (falhou).
4. O Foo só é apagado se existir (`fooId != null`) **e** o Goo já tiver sido removido (`gooDeleted`).
5. Se o Goo não pôde ser removido, o programa **avisa** que o Foo foi mantido, em vez de tentar apagá-lo e falhar.

| Situação | `gooId` | `gooDeleted` | O que acontece |
|---|---|---|---|
| Tudo certo | preenchido | `true` | Apaga Goo e depois Foo |
| Falhou antes de criar o Goo | `null` | `true` | Apaga só o Foo |
| Falhou antes de criar o Foo | `null` | `true` | Não apaga nada (`fooId == null`) |
| Goo criado, mas a exclusão falhou | preenchido | `false` | Avisa e **não** apaga o Foo |

### Parte 7 – Os métodos auxiliares de criação (`postFoo` e `postGoo`)

```java
Foo foo = new Foo();
foo.setName("Foo 1");
foo.setDescription("Foo POST");

Foo createdFoo = fooResource.postFoo(foo);
return createdFoo.getId();
```

| # | Linha | O que acontece |
|---|---|---|
| 1 | `new Foo()` | Cria um DTO vazio |
| 2 | `setName` / `setDescription` | Preenche os campos que queremos enviar |
| 3 | `fooResource.postFoo(foo)` | Faz o `POST /v1.0/foo`: o client JAR converte o objeto em JSON, envia, e converte a resposta em um **novo** `Foo` |
| 4 | `createdFoo.getId()` | O servidor gerou o `id`; devolvemos para as próximas operações |

O `postGoo` é igual, mas usa `gooResource.postFooGoo(fooId, goo)`, que faz `POST /v1.0/foo/{fooId}/goos`: o Goo nasce **dentro** do Foo informado.

### Parte 8 – Os métodos de leitura (`getFooPage`, `getFoo`, `getGooPage`, `getGoo`)

- `getFooPage()` e `getFooGoosPage(fooId)` devolvem um objeto **`Page`** (paginado), com os itens da coleção, a página atual e o total.
- `getFoo(id)` e `getGoo(id)` devolvem **um único** objeto.
- Todos só **imprimem** o resultado. Ao concatenar um objeto em uma `String` (`"Foo consultado: " + ...`), o Java chama o `toString()` do DTO, que o client JAR já gera em formato JSON.

### Parte 9 – PUT × PATCH no código

| Método | O que o objeto enviado contém | Efeito |
|---|---|---|
| `putFoo` | `name` **e** `description` | Substitui o Foo inteiro |
| `patchFoo` | só `description` | Altera só a descrição; o `name` fica como estava |
| `putGoo` | `name`, `description` **e `fooId`** | Substitui o Goo inteiro; o `fooId` é enviado para manter a ligação com o Foo |
| `patchGoo` | só `description` | Altera só a descrição |

> Os retornos de `put` e `patch` não são usados no código: apenas chamamos o método e imprimimos uma mensagem. A conferência é feita pelo `getFoo`/`getGoo` seguinte.

### Parte 10 – Os métodos de exclusão devolvem `boolean`

```java
private static boolean deleteFoo(FooResource fooResource, Long fooId) {
    try {
        fooResource.deleteFoo(fooId);
        System.out.println("Foo removido: " + fooId);
        return true;
    } catch (Exception e) {
        System.err.println("Erro ao remover o Foo de ID " + fooId);
        e.printStackTrace();
        return false;
    }
}
```

Diferente dos outros métodos, **estes tratam a exceção com `try/catch`** em vez de repassá-la (`throws`). Motivo: eles rodam dentro do `finally`; se lançassem exceção, ela poderia **esconder** o erro original e interromper a limpeza. Em vez disso, o método avisa o problema no `System.err` e devolve `false`, para o `finally` decidir o que fazer.

### Parte 11 – `System.out` × `System.err`

- `System.out.println` → saída **normal** (mensagens de sucesso e resultados).
- `System.err.println` → saída de **erro**.

Assim fica fácil separar, no terminal, o que deu certo do que deu errado.

## 2.4 Como compilar e executar

O código **não é um projeto Gradle**: é um arquivo `.java` solto que usa o client JAR. O passo a passo, a partir da raiz do projeto `liferay-r3b2`:

**1. Gere o client JAR**

```bash
./gradlew headless-r3b2-client:jar
ls headless-r3b2-client/build/libs
```

Você deve ver um arquivo parecido com `com.acme.headless.r3b2.client-1.0.0.jar`.

**2. Confirme que a API está implantada**

```bash
./gradlew headless-r3b2-api:deploy -Ddeploy.docker.container.id=<seu-container>
./gradlew headless-r3b2-impl:deploy -Ddeploy.docker.container.id=<seu-container>
```

> Troque `<seu-container>` pelo nome do **seu** container do Liferay (no tutorial é `ephesians-liferay`). **Não** implante `headless-r3b2-client` nem `headless-r3b2-test`. Depois, confira em `http://localhost:8080/o/api` se `headless-r3b2/v1.0` aparece.

**3. Crie uma pasta para o código e entre nela**

```bash
mkdir not-a-gradle-project && cd not-a-gradle-project
```

**4. Salve o código como `Test.java` nessa pasta**

**5. Compile, informando o client JAR no classpath**

```bash
javac -classpath .:../headless-r3b2-client/build/libs/\* *.java
```

**6. Execute**

```bash
java -classpath .:../headless-r3b2-client/build/libs/* Test
```

> No **Windows**, o separador do classpath é `;` em vez de `:` (ex.: `-classpath ".;..\headless-r3b2-client\build\libs\*"`).

**O que você deve ver no terminal**, em ordem: `Foo criado`, `Coleção de Foos`, `Foo consultado` (três vezes, mostrando os valores mudando), as mensagens de `PUT`/`PATCH`, o mesmo para o Goo e, por fim, `Goo removido` e `Foo removido`.

**Dica:** para ver o efeito de cada passo no Liferay, use o API Explorer (`http://localhost:8080/o/api`) em paralelo, ou rode temporariamente o programa sem o `finally` de exclusão e confira os registros criados.

## 2.5 Como enviar o PR

O tutorial pede: *"Modifique o arquivo Java para usar o client JAR para acessar todos os endpoints relacionados a foos e goos. Envie um pull request do seu arquivo para o **brianchandotcom**."*

Checklist antes de enviar:

1. **Todos** os endpoints de Foo e Goo estão cobertos (os 12 da tabela da seção 1.7).
2. O programa **compila e executa** sem erros, e a saída mostra cada operação.
3. As credenciais são as do ambiente **local de treinamento** (nunca credenciais reais).
4. O arquivo enviado é apenas o `.java` modificado (não envie `.class`, JARs nem a pasta `build`).
5. Siga o fluxo habitual de Git do curso: sincronize o fork, crie o branch, faça o commit e abra o PR para o Brian.

## 2.6 Pontos de melhoria

Depois que tudo funcionar, estas melhorias deixam o código mais profissional:

1. **Retornar os dados.** Os métodos `get*` só imprimem. Devolver o objeto (`return fooResource.getFoo(fooId);`) permite reaproveitá-lo, por exemplo para **verificar** que o `PATCH` alterou só a descrição.
2. **Validar, não só imprimir.** Compare os campos recebidos com os esperados (ex.: após o `PATCH`, `name` deve continuar `"Different name"`).
3. **Evitar valores repetidos.** `"test@liferay.com"` e `"learn"` aparecem duas vezes. Guarde-os em constantes (`private static final String`).
4. **Credenciais.** Aqui são as do ambiente local de treinamento, então é aceitável. Em projetos reais, **nunca** versione senhas: use variáveis de ambiente.
5. **Mensagens de erro mais claras** nos `catch`, incluindo a mensagem da exceção (`e.getMessage()`).
6. **Pergunta do tutorial:** qual é o **benefício** de invocar um serviço dentro da JVM (OSGi Resource) em vez de por HTTP (client JAR)? Pense nos custos de rede e de serialização JSON que o client JAR paga em toda chamada.

## 2.7 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `package com.acme.headless.r3b2.client... does not exist` | O client JAR não está no classpath, ou não foi gerado | Rode `./gradlew headless-r3b2-client:jar` e confira o `-classpath` |
| `class Test is public, should be declared in a file named Test.java` | O nome do arquivo não bate com o da classe | Salve como `Test.java` |
| `Could not find or load main class Test` | Esqueceu o `.` no classpath ou está em outra pasta | Rode no mesmo diretório do `Test.class`, com `-classpath .:...` |
| `HTTP 401` (Unauthorized) | Usuário ou senha incorretos | Confira em `.authentication(...)` |
| `HTTP 404` (Not Found) | API não implantada, ou o id não existe | Faça o deploy de `api` e `impl` e confira em `/o/api` |
| `Connection refused` | Liferay desligado ou porta errada | Suba o Liferay e confira `localhost:8080` |
| `headless-r3b2/v1.0` não aparece no API Explorer | `api`/`impl` ainda não foram implantados | Faça o deploy (seção 2.4, passo 2) |
| Erros de compilação ao rodar `headless-r3b2-impl:classes` | Arquivos Java antigos ainda não compiláveis do projeto base | Siga as etapas do tutorial: apague-os e rode `buildREST` |
| `NullPointerException` em `fooId` | O `postFoo` falhou e o id ficou `null` | Veja o erro impresso e corrija a criação |
| "O Foo de ID ... não foi removido porque a exclusão do Goo falhou" | O `deleteGoo` falhou | Veja o erro impresso e apague manualmente pelo API Explorer |

### Para praticar

1. Use o **API Explorer** (GUI), o **curl** e o **Postman** para acessar todos os endpoints de Foo e Goo, antes de usar o client JAR.
2. Altere `postFoo` para receber `name` e `description` como **parâmetros**.
3. Faça `getFooPage` **devolver** a página e imprima só os nomes dos Foos.
4. Provoque um erro de propósito (ex.: um `fooId` inexistente em `postGoo`) e observe como o `finally` limpa o que foi criado.

---
