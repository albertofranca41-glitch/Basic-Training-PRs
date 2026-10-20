# Guia: Atividade de REST Builder (Terceiro PR: OSGi Resource dentro de um Portlet)

Este guia ensina, do zero, a fazer a atividade em que você usa o **OSGi Resource** gerado pelo **REST Builder** para escrever um **portlet** (`R3B2Portlet`) que, **de dentro do próprio Liferay DXP**, executa o CRUD completo de **Foos** e **Goos** da API `headless-r3b2`.

Ele está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | O que é um portlet e qual é a função dele aqui, OSGi (`@Component` e `@Reference`), Factory e Builder do lado do servidor, `ThemeDisplay`, autenticação por usuário, e **tudo o que muda em relação ao `Test.java`** do primeiro PR |
| **Etapa 2 – O código** | O código completo do `R3B2Portlet`, comentado, explicado parte por parte, e como criar o módulo, implantar e conferir o resultado |


---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 Revisão rápida: o que você já sabe do PR anterior](#11-revisão-rápida-o-que-você-já-sabe-do-pr-anterior)
  - [1.2 O que é um portlet?](#12-o-que-é-um-portlet)
  - [1.3 Qual é a função do portlet nesta atividade?](#13-qual-é-a-função-do-portlet-nesta-atividade)
  - [1.4 OSGi, `@Component` e `@Reference`](#14-osgi-component-e-reference)
  - [1.5 Os dois `FooResource` (de novo) e os pacotes que mudam](#15-os-dois-fooresource-de-novo-e-os-pacotes-que-mudam)
  - [1.6 Factory e Builder: como o portlet obtém um `FooResource`](#16-factory-e-builder-como-o-portlet-obtém-um-fooresource)
  - [1.7 `ThemeDisplay` e `WebKeys`: de onde vem o usuário](#17-themedisplay-e-webkeys-de-onde-vem-o-usuário)
  - [1.8 Autenticação e permissões: sem senha no código](#18-autenticação-e-permissões-sem-senha-no-código)
  - [1.9 Client JAR × OSGi Resource: a resposta da pergunta do tutorial](#19-client-jar--osgi-resource-a-resposta-da-pergunta-do-tutorial)
  - [1.10 Tudo o que muda: `Test.java` × `R3B2Portlet`](#110-tudo-o-que-muda-testjava--r3b2portlet)
  - [1.11 Log em vez de `System.out`](#111-log-em-vez-de-systemout)
  - [1.12 O ciclo de vida: o `doView` roda a cada renderização](#112-o-ciclo-de-vida-o-doview-roda-a-cada-renderização)
  - [1.13 Módulos e implantação](#113-módulos-e-implantação)
  - [1.14 Pré-requisitos do ambiente](#114-pré-requisitos-do-ambiente)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 Código completo comentado](#22-código-completo-comentado)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
  - [2.4 Como criar o módulo, implantar e executar](#24-como-criar-o-módulo-implantar-e-executar)
  - [2.5 Pontos de melhoria](#25-pontos-de-melhoria)
  - [2.6 Erros comuns](#26-erros-comuns)

---

# Etapa 1 – Teoria

No primeiro PR, o código Java era um programa **externo** ao Liferay, que conversava com a API por **HTTP**. Agora o código passa a rodar **dentro** do Liferay. Isso muda **onde** o código roda, **como** ele é iniciado, **como** ele se autentica e **como** ele obtém o `FooResource`. Esta etapa explica cada uma dessas mudanças.

## 1.1 Revisão rápida: o que você já sabe do PR anterior

Estes conceitos continuam valendo e **não** serão repetidos aqui:

- A API `headless-r3b2` e seus recursos `Foo` e `Goo` (`Goo` está associado a `Foo` pelo campo `fooId`).
- As operações `POST`, `GET`, `PUT`, `PATCH` e `DELETE`, e a diferença entre `PUT` (substituição) e `PATCH` (atualização parcial).
- O REST Builder como gerador de código a partir de `rest-config.yaml` e `rest-openapi.yaml`.
- Os módulos `headless-r3b2-api`, `headless-r3b2-impl`, `headless-r3b2-client` e `headless-r3b2-test`.
- O padrão **Builder**: `X.builder()...build()`.

O que **muda** é a forma de consumir a API. Veja o resumo:

```text
PR 1 (Test.java)                          PR 3 (R3B2Portlet)

Programa Java externo                     Portlet dentro do Liferay
        |                                         |
  client JAR (HTTP + JSON)                  OSGi Resource (chamada de método)
        |                                         |
        v                                         v
   Liferay DXP  ------------------------>  mesma JVM do Liferay
```

## 1.2 O que é um portlet?

Um **portlet** é um componente Java que roda **dentro de um portal** (no nosso caso, o Liferay) e gera um **pedaço** de uma página web. Cada portlet aparece na página como um **widget**: um bloco independente, que pode ser adicionado, movido e removido pelo painel de widgets.

O padrão de portlets Java é definido pelas especificações **JSR 168**, **JSR 286** e **JSR 362**, e a interface se chama `javax.portlet.Portlet`. Em vez de implementar tudo à mão, é comum estender uma classe pronta:

- **`GenericPortlet`** (usada nesta atividade): a classe-base padrão da especificação. Ela já "despacha" cada requisição para um método diferente conforme o **modo** do portlet.
- `MVCPortlet`: uma classe-base do Liferay que estende `GenericPortlet` e acrescenta recursos para quem usa JSPs e comandos MVC. Não é usada aqui.

### O ciclo de vida de um portlet

Diferente de uma página web simples, um portlet tem **fases**. As duas principais são:

| Fase | Para que serve | Método típico |
|---|---|---|
| **Render** | Gerar o conteúdo que será exibido na página | `doView`, `render` |
| **Action** | Processar uma ação do usuário (por exemplo, o envio de um formulário) | `processAction` |

O `GenericPortlet` chama `doView` na fase de **render** quando o portlet está no modo **VIEW** (o modo normal de exibição). Por isso, esta atividade sobrescreve `doView`: é o ponto de entrada do nosso código.

### Como o portlet chega até a página

O portlet **não** é um programa que você executa. Ele é **registrado** no Liferay (por meio do `@Component`, na seção 1.4), e o Liferay o **chama** quando uma página que contém o widget é renderizada.

## 1.3 Qual é a função do portlet nesta atividade?

Esta é a pergunta central deste PR. Repare que o `R3B2Portlet` **não mostra nada na tela**: o `doView` não escreve nenhum HTML. Então para que serve o portlet?

**O portlet é o ponto de entrada que permite executar código dentro do Liferay.**

No primeiro PR havia um `main`: o próprio sistema operacional iniciava o programa. Código que roda **dentro** do Liferay não tem `main`; algo dentro do Liferay precisa **chamá-lo**. O portlet cumpre esse papel:

1. O portlet é registrado no Liferay como widget.
2. Você adiciona o widget a uma página.
3. Ao abrir a página, o Liferay renderiza o widget e chama `doView`.
4. O `doView` executa o CRUD de Foo e Goo, como o `main` do `Test.java` fazia.

Pense assim: **o `doView` é o novo `main`**, e **o portlet é o "palco" que o Liferay usa para chamá-lo**.

> Essa comparação vale para este exercício: `doView` não é um ponto de entrada único como `main`; é chamado pelo Liferay quando o portlet precisa ser renderizado.

Isso também explica por que o portlet nos dá algo que o programa externo não tinha: o **contexto do portal**. Ao ser chamado pelo Liferay durante a renderização de uma página, o portlet recebe o `RenderRequest`, de onde se obtém o **usuário logado** (seção 1.7). É esse usuário que será usado para executar as operações, sem precisar de usuário e senha no código.

> Neste exercício o portlet é um **veículo** para exercitar o OSGi Resource. Em uma aplicação real, um portlet existiria para exibir ou coletar informações do usuário.

## 1.4 OSGi, `@Component` e `@Reference`

O Liferay é construído sobre **OSGi**, uma plataforma que organiza o código em **módulos** (chamados *bundles*) e permite que um módulo **ofereça** serviços e outros módulos os **consumam**, sem se conhecerem diretamente.

Os **Declarative Services (DS)** são a forma de declarar isso usando anotações:

- **`@Component`**: declara que a classe é um componente OSGi. O Liferay descobre o componente e o registra.
- **`@Reference`**: pede ao OSGi que **injete** (forneça) um serviço registrado por outro módulo.

### `@Component` no portlet

```java
@Component(property = {
        "com.liferay.portlet.display-category=category.sample",
        "javax.portlet.display-name=R3B2"
}, service = Portlet.class)
public class R3B2Portlet extends GenericPortlet {
```

| Parte | O que significa |
|---|---|
| `service = Portlet.class` | Registra este componente como um serviço do tipo `Portlet`. É assim que o Liferay o reconhece como portlet |
| `com.liferay.portlet.display-category=category.sample` | A **categoria** onde o widget aparece no painel de widgets (aqui, "Sample") |
| `javax.portlet.display-name=R3B2` | O **nome** exibido do widget |

### `@Reference` na Factory

```java
@Reference
private FooResource.Factory _fooResourceFactory;
```

Esta linha pede ao OSGi: "forneça a implementação registrada de `FooResource.Factory`". Quem registra essa implementação é o módulo `headless-r3b2-impl` (a classe `FooResourceFactoryImpl`, gerada pelo REST Builder).

> **Consequência prática:** o portlet **depende** de `headless-r3b2-api` e `headless-r3b2-impl` estarem implantados e ativos. Se a Factory não estiver disponível, a referência não é satisfeita e o componente do portlet **não ativa**, ou seja, o widget não aparece (seção 2.6).

## 1.5 Os dois `FooResource` (de novo) e os pacotes que mudam

O guia do primeiro PR explicou que existem dois `FooResource`, em módulos diferentes. Agora você usa **o outro**. O tutorial destaca justamente isso: *"Notice that the Java packages are different"*.

| Elemento | `Test.java` (client JAR) | `R3B2Portlet` (OSGi Resource) |
|---|---|---|
| Módulo de origem | `headless-r3b2-client` | `headless-r3b2-api` |
| `FooResource` | `com.acme.headless.r3b2.client.resource.v1_0.FooResource` | `com.acme.headless.r3b2.resource.v1_0.FooResource` |
| `GooResource` | `com.acme.headless.r3b2.client.resource.v1_0.GooResource` | `com.acme.headless.r3b2.resource.v1_0.GooResource` |
| DTO `Foo` | `com.acme.headless.r3b2.client.dto.v1_0.Foo` | `com.acme.headless.r3b2.dto.v1_0.Foo` |
| DTO `Goo` | `com.acme.headless.r3b2.client.dto.v1_0.Goo` | `com.acme.headless.r3b2.dto.v1_0.Goo` |
| `Page` | Classe do próprio client JAR (`...client.pagination.Page`) | `com.liferay.portal.vulcan.pagination.Page` |

Ou seja, **mudam três coisas**: o `FooResource`/`GooResource`, os **DTOs** e a classe `Page`. Se você copiar um import do `Test.java` para o portlet (ou vice-versa), o código não compila ou usa as classes erradas. Quando algo "não encaixa", confira primeiro se o `.client.` aparece no meio do pacote.

## 1.6 Factory e Builder: como o portlet obtém um `FooResource`

No `Test.java`, o `FooResource` era criado diretamente:

```java
FooResource fooResource = FooResource.builder()
        .authentication("test@liferay.com", "learn")
        .build();
```

No portlet, o caminho tem **uma etapa a mais**:

```java
FooResource.Builder fooResourceBuilder = _fooResourceFactory.create();

FooResource fooResource = fooResourceBuilder.user(
        themeDisplay.getUser())
        .build();
```

Ou seja:

```text
Test.java:    FooResource.builder()  ->  .authentication(login, senha)  ->  .build()

R3B2Portlet:  _fooResourceFactory.create()  ->  .user(usuário)  ->  .build()
              (Factory injetada pelo OSGi)
```

### O que é uma Factory?

**Factory** (fábrica) é um objeto cuja função é **criar** outros objetos. Aqui, a `FooResource.Factory` cria os `FooResource.Builder`.

### Por que existe uma Factory?

Um `FooResource` do lado do servidor precisa saber **para qual usuário** está executando (para checar permissões, por exemplo). Esse usuário só é conhecido **no momento da requisição**. Então, em vez de o OSGi injetar um `FooResource` pronto e fixo, ele injeta a **Factory**, e você cria um `FooResource` sob medida **a cada requisição**, informando o usuário com `.user(...)`.

### Comparando os dois Builders

| Aspecto | `FooResource.builder()` (client) | `_fooResourceFactory.create()` (OSGi) |
|---|---|---|
| Como se obtém o Builder | Método estático `builder()` | Chamando `create()` na Factory injetada |
| Informa quem é o usuário | `.authentication(login, senha)` | `.user(User)` |
| O que é enviado ao servidor | Credenciais em um cabeçalho HTTP | Nada: o objeto `User` é usado diretamente na JVM |
| Termina com | `.build()` | `.build()` |

O `.build()` é igual nos dois casos: o padrão **Builder** continua o mesmo, e só mudam a origem do Builder e a configuração do usuário. O Builder gerado pode oferecer outras opções além de `.user(...)`; consulte a classe gerada em `headless-r3b2-api` para conhecê-las.

## 1.7 `ThemeDisplay` e `WebKeys`: de onde vem o usuário

```java
ThemeDisplay themeDisplay = (ThemeDisplay) renderRequest.getAttribute(
        WebKeys.THEME_DISPLAY);
```

O `ThemeDisplay` é um objeto do Liferay que guarda o **contexto da requisição atual**: o **usuário logado**, o **site** atual, a **página**, o **idioma**, as permissões, entre outros dados.

O Liferay o coloca como um **atributo** do `RenderRequest`. Para recuperá-lo, usa-se `renderRequest.getAttribute(...)` com a chave `WebKeys.THEME_DISPLAY` (uma constante que evita erros de digitação na chave). Como `getAttribute` devolve `Object`, é preciso fazer o **cast** para `ThemeDisplay`.

Nesta atividade usamos **apenas** `themeDisplay.getUser()`, que devolve o usuário que está vendo a página.

## 1.8 Autenticação e permissões: sem senha no código

Esta é uma das mudanças mais importantes em relação ao `Test.java`.

| Aspecto | `Test.java` | `R3B2Portlet` |
|---|---|---|
| Quem faz as operações | O usuário cujas credenciais estão no código | O **usuário logado** que abriu a página |
| Credenciais no código | Sim (`"test@liferay.com"`, `"learn"`) | **Não** |
| Como o Liferay identifica o usuário | Basic Authentication (cabeçalho HTTP) | O objeto `User` repassado em `.user(...)` |

Duas consequências:

1. **Não há senha no código.** Isso elimina o risco de enviar credenciais em um PR ou de deixá-las expostas no repositório.
2. **As permissões são as do usuário logado.** Se o usuário que abriu a página não tiver permissão para criar, alterar ou apagar Foos e Goos, as chamadas falham (normalmente com uma exceção de permissão, registrada no log). Por isso, ao testar, abra a página com um usuário que tenha as permissões necessárias (por exemplo, o administrador do ambiente de treinamento).

> Usuário visitante (não logado) também é um `User` para o Liferay, mas normalmente sem permissão para essas operações. Teste logado.

## 1.9 Client JAR × OSGi Resource: a resposta da pergunta do tutorial

O tutorial pergunta: *"Client resources are invoked over HTTP. OSGi resources are invoked within the same JVM. What is a benefit of invoking within the JVM versus invoking over HTTP?"*

Quando você usa o client JAR, **cada chamada** paga:

1. **Conversão** do objeto Java em JSON (serialização).
2. **Envio** pela rede (mesmo em `localhost`, passa pela pilha de rede e pelo servidor HTTP).
3. **Autenticação** da requisição.
4. **Conversão** da resposta JSON de volta em objeto Java (desserialização).

Com o OSGi Resource, você chama o método **diretamente na mesma JVM**: não há rede, não há conversão para JSON e a autenticação é substituída pelo usuário já conhecido.

**Benefício principal:** menos custo (rede e serialização) por chamada, e um código mais simples de autenticar.

Dois cuidados para não exagerar na conclusão:

- A chamada local **não é "de graça"**: a implementação em `FooResourceImpl` continua sendo executada, assim como as camadas abaixo dela (serviços, banco de dados). O ganho é nos custos de **comunicação**, não no trabalho da própria operação.
- O OSGi Resource **só funciona para código que roda dentro do Liferay**. Um programa externo continua precisando do client JAR.

| Aspecto | Client JAR | OSGi Resource |
|---|---|---|
| Onde o código roda | Programa externo | Dentro do Liferay (mesma JVM) |
| Comunicação | HTTP + JSON | Chamada de método |
| Autenticação | Credenciais no código | Usuário do contexto (`.user(...)`) |
| Ponto de entrada | `main` | Componente OSGi (aqui, um portlet) |
| Obtenção do resource | `FooResource.builder()` | `@Reference` na Factory + `create()` |

## 1.10 Tudo o que muda: `Test.java` × `R3B2Portlet`

Esta tabela reúne **todas** as diferenças entre o código do primeiro PR e o do terceiro. As seções seguintes aprofundam cada uma.

| # | Aspecto | `Test.java` (PR 1) | `R3B2Portlet` (PR 3) |
|---|---|---|---|
| 1 | Tipo da classe | Classe comum com `main` | `GenericPortlet` registrado com `@Component` |
| 2 | Ponto de entrada | `public static void main(...)` | `doView(RenderRequest, RenderResponse)` |
| 3 | Quem inicia a execução | Você, no terminal (`java Test`) | O Liferay, ao renderizar a página com o widget |
| 4 | Onde roda | Processo Java externo | Dentro da JVM do Liferay |
| 5 | Imports de `Foo`/`Goo` | `...client.dto.v1_0` | `...dto.v1_0` |
| 6 | Imports de `FooResource`/`GooResource` | `...client.resource.v1_0` | `...resource.v1_0` |
| 7 | Como cria o resource | `FooResource.builder()` | `_fooResourceFactory.create()` (Factory via `@Reference`) |
| 8 | Autenticação | `.authentication("test@liferay.com", "learn")` | `.user(themeDisplay.getUser())` |
| 9 | Credenciais no código | Sim | Não |
| 10 | Como chega ao usuário | Fixo no código | `ThemeDisplay`, via `renderRequest.getAttribute(WebKeys.THEME_DISPLAY)` |
| 11 | Como a chamada é feita | HTTP + JSON | Chamada direta de método na JVM |
| 12 | Saída | `System.out` / `System.err` | `Log` do Liferay (`_log.info`, `_log.debug`, `_log.error`) |
| 13 | Métodos auxiliares | `private static` (por causa do `main` estático) | `private` de instância (o `doView` não é estático) |
| 14 | Nome dos wrappers de Goo | `postGoo`, `getGooPage` | `postFooGoo`, `getFooGoosPage` (iguais aos métodos do resource) |
| 15 | Retorno de `put` e `patch` | Descartado | Guardado em variável e registrado no log |
| 16 | Tratamento de exceções | `try/finally` sem `catch`; a exceção sobe | `try/catch/finally`; o `catch` registra o erro e **não** propaga |
| 17 | Métodos `delete*` | Tratam a exceção e devolvem `boolean` | Declaram `throws Exception`; quem trata é o `finally` do `doView` |
| 18 | Ordem da limpeza | O Foo só é apagado se o Goo foi apagado | Goo e Foo têm tentativas de exclusão **independentes** |
| 19 | `putGoo` | Recebe `fooId` e envia no DTO | Não recebe `fooId` (só `gooId`) |
| 20 | Page | Classe `Page` do client JAR | `com.liferay.portal.vulcan.pagination.Page` |
| 21 | Dependências de build | Client JAR no classpath | Módulo novo (`headless-r3b2-web`) com dependências do Liferay e do módulo `api` |
| 22 | Como executar | `javac` + `java` | Implantar (`deploy`) e adicionar o widget a uma página |
| 23 | Quantas vezes roda | Uma por execução | **Uma por renderização** da página (seção 1.12) |

Os itens 16 a 19 (tratamento de erros e limpeza) são as diferenças de **lógica**; os demais são de **infraestrutura**. As operações REST em si (`postFoo`, `getFoo`, `putFoo`, ...) são praticamente as mesmas.

## 1.11 Log em vez de `System.out`

No `Test.java`, os resultados eram impressos com `System.out.println`. Dentro do Liferay, o padrão é usar o **sistema de log** do portal:

```java
private static final Log _log = LogFactoryUtil.getLog(R3B2Portlet.class);
```

| Elemento | Papel |
|---|---|
| `Log` | Interface de log do Liferay (`com.liferay.portal.kernel.log.Log`) |
| `LogFactoryUtil.getLog(R3B2Portlet.class)` | Cria o logger associado à classe |
| `static final` | O logger é um só para a classe e não muda |

### Níveis de log

| Método | Nível | Uso neste código |
|---|---|---|
| `_log.info(...)` | Informação | Marca cada operação (`=== POST Foo ===`) e as exclusões |
| `_log.debug(...)` | Depuração | Mostra os objetos retornados (`GET Foo: ...`) |
| `_log.error(msg, e)` | Erro | Registra uma exceção junto com o *stack trace* |

O nível `debug` normalmente **não aparece** por padrão. Para vê-lo, é necessário habilitar o nível `DEBUG` para a categoria `com.acme.headless.r3b2.web.internal.R3B2Portlet` nas configurações de log do Liferay (na administração do servidor, em "Log Levels"; o caminho exato do menu pode variar conforme a versão).

Os logs aparecem no **console** do Liferay. Se o Liferay roda em Docker, uma forma comum de acompanhá-los é:

```bash
docker logs -f <seu-container>
```

## 1.12 O ciclo de vida: o `doView` roda a cada renderização

Um ponto que surpreende: o `doView` é chamado **toda vez** que o portlet precisa ser renderizado, inclusive em cada atualização (*refresh*) da página.

Como colocamos o CRUD inteiro dentro do `doView`, **cada vez que você abrir ou atualizar a página**:

1. Um Foo e um Goo são criados.
2. Eles são consultados e atualizados.
3. Eles são apagados.

Isso é **proposital neste exercício** (é a forma mais simples de exercitar todos os endpoints), mas **não é um bom padrão para uma aplicação real**: uma simples visualização de página não deveria criar e apagar dados. Veja as melhorias na seção 2.5.

O bloco `finally` garante que, mesmo que algo falhe no meio, o código tente apagar o que foi criado, para não acumular registros de teste a cada atualização.

## 1.13 Módulos e implantação

Este PR acrescenta **um módulo novo** ao projeto:

| Módulo | Responsabilidade | Precisa ser implantado? |
|---|---|---|
| `headless-r3b2-api` | Contratos, interfaces (inclusive `FooResource` e `FooResource.Factory`) e DTOs | Sim |
| `headless-r3b2-impl` | Implementação da API e a implementação das Factories | Sim |
| `headless-r3b2-web` (**novo**) | Contém o `R3B2Portlet` | **Sim** |
| `headless-r3b2-client` | Biblioteca do client JAR | Não |
| `headless-r3b2-test` | Estruturas de teste do projeto | Não |

O `headless-r3b2-web` **depende** de `api` e `impl`. Por isso, a ordem lógica é: `api` e `impl` primeiro (o OSGi precisa da Factory disponível) e o `web` depois.

Para compilar, o módulo `web` precisa enxergar, no mínimo:

| Dependência | De onde vem | Usada em |
|---|---|---|
| Módulo `headless-r3b2-api` | Do próprio projeto | `FooResource`, `GooResource`, `Foo`, `Goo` |
| API de portlets (`javax.portlet`) | Do Liferay/portlet API | `GenericPortlet`, `Portlet`, `RenderRequest`, `RenderResponse` |
| Portal Kernel | Do Liferay | `ThemeDisplay`, `WebKeys`, `Log`, `LogFactoryUtil` |
| Portal Vulcan API | Do Liferay | `Page` |
| Anotações OSGi (DS) | OSGi | `@Component`, `@Reference` |

A forma exata de declarar essas dependências (sintaxe do `build.gradle`, versões) deve seguir o padrão dos outros módulos do seu projeto; use o `build.gradle` de `headless-r3b2-impl` como referência e confira se o módulo novo está incluído no `settings.gradle`.

> **Não** implante `headless-r3b2-client` nem `headless-r3b2-test`.

## 1.14 Pré-requisitos do ambiente

| Ferramenta ou etapa | Finalidade |
|---|---|
| Liferay DXP em execução | Executar o portlet e a API |
| Projeto `liferay-r3b2` configurado | Fornecer os módulos e a definição da API |
| REST Builder executado (`buildREST`) | Gerar `FooResource`, `GooResource`, as Factories e os DTOs |
| Módulos `api` e `impl` implantados | Disponibilizar a API e a implementação das Factories |
| Módulo `headless-r3b2-web` criado | Abrigar o `R3B2Portlet` |
| Usuário com permissões | Executar as operações pelo portlet |
| JDK compatível | Compilar o módulo |
| Acesso ao log do Liferay | Ver o resultado das operações |
| Git e conta no GitHub | Versionar as alterações |

Antes de começar, confirme que a API `headless-r3b2/v1.0` aparece no API Explorer (`http://localhost:8080/o/api`). Se ela não aparece, o portlet também não vai funcionar.

## Recursos adicionais

- [Cliente e servidor](../theory/client-server.md): papéis, requisições e respostas.
- [HTTP](../theory/http.md): métodos, URLs, cabeçalhos, corpos e códigos de status.
- [REST](../theory/rest.md): recursos, representações e métodos HTTP.
- [API](../theory/api.md): conceito de interface e contrato entre sistemas.
- [JSON](../theory/json.md): formato usado para representar e transportar dados.
- [DTOs e Schemas](../theory/dtos-and-schemas.md): objetos de transferência e modelos de dados descritos no contrato.
- [Portlet](../theory/portlet.md): Conceito de Portlet dentro do contexto do Liferay.
- [API Explorer do seu ambiente](http://localhost:8080/o/api): confira os endpoints, schemas e operações presentes na versão instalada (Funciona somente se o container Liferay estiver rodando e se você fez o login corretamente da sua conta `test@liferay.com`).
- [JVM (Wikipedia)](https://en.wikipedia.org/wiki/Java_virtual_machine): a máquina virtual onde o portlet e o Liferay rodam juntos.
- [Chamada local × chamada remota (Stack Overflow)](https://stackoverflow.com/questions/7538951/what-is-the-performance-difference-between-a-jvm-method-call-and-a-remote-call): indicado pelo tutorial para responder à pergunta da seção 1.9.

---

# Etapa 2 – O código

## 2.1 Visão geral

O arquivo `R3B2Portlet.java` tem **um método `doView`** (que executa o ciclo completo) e **doze métodos auxiliares**, um por operação:

```
doView():        orquestra tudo (cria resources, cria, consulta, atualiza, limpa)

Foo:             postFoo()        → POST   (Create)
                 getFooPage()     → GET    (Read - coleção)
                 getFoo()         → GET    (Read - um)
                 putFoo()         → PUT    (Update completo)
                 patchFoo()       → PATCH  (Update parcial)
                 deleteFoo()      → DELETE (Delete)

Goo:             postFooGoo()     → POST   (Create, dentro de um Foo)
                 getFooGoosPage() → GET    (Read - coleção de um Foo)
                 getGoo()         → GET    (Read - um)
                 putGoo()         → PUT    (Update completo)
                 patchGoo()       → PATCH  (Update parcial)
                 deleteGoo()      → DELETE (Delete)
```

O fluxo do `doView` é:

```
1. Recupera o ThemeDisplay (contexto do portal) e dele o usuário logado
2. Cria os dois "resources" (Foo e Goo) a partir das Factories, já com o usuário
3. try {
4.   Cria Foo → lista → busca → PUT → busca → PATCH → busca
5.   Cria Goo (ligado ao Foo) → lista → busca → PUT → busca → PATCH → busca
6. } catch (Exception e) {
7.   Registra o erro no log (sem propagar)
8. } finally {
9.   Tenta apagar o Goo e, em seguida, o Foo (mesmo que algo tenha falhado)
10. }
```

> O `try/catch/finally` com limpeza é uma proteção deste exercício: tenta apagar os dados de teste mesmo se uma etapa falhar. Ele **não é necessário para usar o OSGi Resource**.

## 2.2 Código completo comentado

> A lógica é exatamente a do código da atividade. O arquivo fica em `headless-r3b2-web`, no pacote `com.acme.headless.r3b2.web.internal` (o nome do arquivo deve ser igual ao nome da classe pública: `R3B2Portlet.java`).

```java
package com.acme.headless.r3b2.web.internal;

import com.acme.headless.r3b2.dto.v1_0.Foo;
import com.acme.headless.r3b2.dto.v1_0.Goo;
import com.acme.headless.r3b2.resource.v1_0.FooResource;
import com.acme.headless.r3b2.resource.v1_0.GooResource;

import com.liferay.portal.kernel.theme.ThemeDisplay;
import com.liferay.portal.kernel.util.WebKeys;
import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;


import java.io.IOException;

import javax.portlet.GenericPortlet;
import javax.portlet.Portlet;
import javax.portlet.RenderRequest;
import javax.portlet.RenderResponse;

import com.liferay.portal.vulcan.pagination.Page;

import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

/**
 * @author Alberto
 */

@Component(property = {
		"com.liferay.portlet.display-category=category.sample",
		// categoria onde o portlet aparece no painel "Add Widget"
		"javax.portlet.display-name=R3B2"
		// nome exibido do portlet
}, service = Portlet.class

)
public class R3B2Portlet extends GenericPortlet {

	private static final Log _log = LogFactoryUtil.getLog(R3B2Portlet.class);

	@Override
	public void doView(
			RenderRequest renderRequest,
			RenderResponse renderResponse)
			throws IOException {
		// chamado toda vez que o portlet precisa renderizar (inclusive em cada refresh)

		ThemeDisplay themeDisplay = (ThemeDisplay) renderRequest.getAttribute(
				WebKeys.THEME_DISPLAY);
		// recupera o contexto do portal: usuário logado, site atual, permissões etc.
		// o usuário será informado aos Resources para executar operações neste contexto

		FooResource.Builder fooResourceBuilder = _fooResourceFactory.create();

		FooResource fooResource = fooResourceBuilder.user(
				themeDisplay.getUser())
				.build();

		GooResource.Builder gooResourceBuilder = _gooResourceFactory.create();

		GooResource gooResource = gooResourceBuilder.user(
				themeDisplay.getUser())
				.build();

		Long fooId = null;
		Long gooId = null;

		try {

			/* --- Foo --- */

			fooId = postFoo(fooResource);
			getFooPage(fooResource);
			getFoo(fooResource, fooId);
			putFoo(fooResource, fooId);
			getFoo(fooResource, fooId);
			patchFoo(fooResource, fooId);
			getFoo(fooResource, fooId);

			/* --- Goo --- */

			gooId = postFooGoo(gooResource, fooId);
			getFooGoosPage(gooResource, fooId);
			getGoo(gooResource, gooId);
			putGoo(gooResource, gooId);
			getGoo(gooResource, gooId);
			patchGoo(gooResource, gooId);
			getGoo(gooResource, gooId);

		} catch (Exception e) {
			_log.error("Error: ", e); // "e" é o nome da variável que armazena a exceção que ocorreu.

		} finally {

			if (gooId != null) {
				try {
					deleteGoo(gooResource, gooId);
				} catch (Exception e) {
					_log.error("Error deleting Goo: ", e);
				}
			}

			if (fooId != null) {
				try {
					deleteFoo(fooResource, fooId);
				} catch (Exception e) {
					_log.error("Error deleting Foo: ", e);
				}
			}

		}
	}

	// =========================================================
	// METODOS DO FOO RESOURCE
	// =========================================================

	private Long postFoo(FooResource fooResource) throws Exception {

		_log.info("=== POST Foo ===");

		Foo newFoo = new Foo();

		newFoo.setName("POST Foo");
		newFoo.setDescription("Posted Foo from R3B2Portlet");

		Foo createdFoo = fooResource.postFoo(newFoo);

		return createdFoo.getId();
	}

	private void getFooPage(FooResource fooResource) throws Exception {

		_log.info("=== GET Foo Page ===");

		// O método getFooPage() retorna uma serie de Foo juntamente com
		// informações adicionais, como a quantidade total de resultados.
		// O Page organiza esse retorno em paginas, evitando que todos os
		// recursos sejam retornados de uma só vez.

		Page<Foo> fooPage = fooResource.getFooPage();

		_log.debug("GET Foo Page: " + fooPage);
	}


	private void getFoo(FooResource fooResource, Long fooId) throws Exception {

		_log.info("=== GET Foo ===");

		Foo foo = fooResource.getFoo(fooId);
		_log.debug("GET Foo: " + foo);
	}

	private void putFoo(FooResource fooResource, Long fooId) throws Exception {

		_log.info("=== PUT Foo ===");

		Foo updatedFoo = new Foo();

		updatedFoo.setName("PUT Foo");
		updatedFoo.setDescription("Updated Foo from R3B2Portlet");
		updatedFoo.setId(fooId);

		Foo foo = fooResource.putFoo(fooId, updatedFoo);
		_log.debug("PUT Foo: " + foo);
	}

	private void patchFoo(FooResource fooResource, Long fooId) throws Exception {

		_log.info("=== PATCH Foo ===");

		Foo patchedFoo = new Foo();

		patchedFoo.setDescription("Patched Foo from R3B2Portlet");

		Foo foo = fooResource.patchFoo(fooId, patchedFoo);
		_log.debug("PATCH Foo: " + foo);
	}

	private void deleteFoo(FooResource fooResource, Long fooId) throws Exception {

		_log.info("=== DELETE Foo ===");

		fooResource.deleteFoo(fooId);
		_log.info("Deleted Foo with ID: " + fooId);
	}

	// =========================================================
	// METODOS DO GOO RESOURCE
	// =========================================================

	private Long postFooGoo(GooResource gooResource, Long fooId) throws Exception {

		_log.info("=== POST Goo ===");

		Goo newGoo = new Goo();

		newGoo.setName("POST Goo");
		newGoo.setDescription("Posted Goo from R3B2Portlet");

		Goo createdGoo = gooResource.postFooGoo(fooId, newGoo);

		return createdGoo.getId();
	}

	private void getFooGoosPage(GooResource gooResource, Long fooId) throws Exception {

		_log.info("=== GET Foo Goos Page ===");

		Page<Goo> gooPage = gooResource.getFooGoosPage(fooId);

		_log.debug("GET Foo Goos Page: " + gooPage);
	}

	private void getGoo(GooResource gooResource, Long gooId) throws Exception {

		_log.info("=== GET Goo ===");

		Goo goo = gooResource.getGoo(gooId);
		_log.debug("GET Goo: " + goo);
	}

	private void putGoo(GooResource gooResource, Long gooId) throws Exception {

		_log.info("=== PUT Goo ===");

		Goo updatedGoo = new Goo();

		updatedGoo.setName("PUT Goo");
		updatedGoo.setDescription("Updated Goo from R3B2Portlet");
		updatedGoo.setId(gooId);

		Goo goo = gooResource.putGoo(gooId, updatedGoo);
		_log.debug("PUT Goo: " + goo);
	}

	private void patchGoo(GooResource gooResource, Long gooId) throws Exception {

		_log.info("=== PATCH Goo ===");

		Goo patchedGoo = new Goo();

		patchedGoo.setDescription("Patched Goo from R3B2Portlet");

		Goo goo = gooResource.patchGoo(gooId, patchedGoo);
		_log.debug("PATCH Goo: " + goo);
	}

	private void deleteGoo(GooResource gooResource, Long gooId) throws Exception {

		_log.info("=== DELETE Goo ===");

		gooResource.deleteGoo(gooId);
		_log.info("Deleted Goo with ID: " + gooId);
	}

	@Reference
	private FooResource.Factory _fooResourceFactory;
	// injeção de dependência via OSGi: o container fornece a implementação
	// registrada de FooResource.Factory, usada para construir um FooResource
	// autenticado (via .create().user(...).build())

	@Reference
	private GooResource.Factory _gooResourceFactory;
}
```

## 2.3 Explicação por partes

### Parte 1 – Os `import`

| Import | Papel |
|---|---|
| `...dto.v1_0.Foo` / `Goo` | **DTOs do lado do servidor** (módulo `api`): objetos simples que carregam os dados. Repare: **sem** `.client.` |
| `...resource.v1_0.FooResource` / `GooResource` | **Resources do OSGi** (módulo `api`): cada método é uma chamada direta, sem HTTP. Também **sem** `.client.` |
| `ThemeDisplay`, `WebKeys` | Recuperar o contexto do portal (usuário logado) |
| `Log`, `LogFactoryUtil` | Sistema de log do Liferay |
| `GenericPortlet`, `Portlet`, `RenderRequest`, `RenderResponse` | Tipos da API de portlets |
| `Page` | Tipo de retorno das listagens (`com.liferay.portal.vulcan.pagination.Page`) |
| `@Component`, `@Reference` | Anotações OSGi (Declarative Services) |
| `IOException` | Exceção declarada pelo `doView` |

### Parte 2 – A anotação `@Component`

```java
@Component(property = {
        "com.liferay.portlet.display-category=category.sample",
        "javax.portlet.display-name=R3B2"
}, service = Portlet.class)
```

Registra a classe como **portlet** no Liferay (`service = Portlet.class`), define a **categoria** em que o widget aparece no painel ("Sample") e o **nome** exibido ("R3B2"). Sem essa anotação, o Liferay não sabe que a classe é um portlet.

### Parte 3 – O logger

```java
private static final Log _log = LogFactoryUtil.getLog(R3B2Portlet.class);
```

Cria o logger usado por todos os métodos. Substitui o `System.out.println` do `Test.java`.

### Parte 4 – O método `doView`

```java
@Override
public void doView(RenderRequest renderRequest, RenderResponse renderResponse)
        throws IOException {
```

- `@Override`: o método sobrescreve `doView` de `GenericPortlet`.
- É chamado pelo Liferay **toda vez** que o portlet é renderizado (seção 1.12).
- Faz o papel do `main` do `Test.java`.
- `renderRequest` carrega o contexto da requisição; `renderResponse` serviria para gerar conteúdo na tela, mas **não é usado** neste exercício.

### Parte 5 – Recuperando o `ThemeDisplay`

```java
ThemeDisplay themeDisplay = (ThemeDisplay) renderRequest.getAttribute(
        WebKeys.THEME_DISPLAY);
```

Obtém o contexto do portal, de onde sai o **usuário logado** (`themeDisplay.getUser()`).

### Parte 6 – Criando os resources com a Factory

```java
FooResource.Builder fooResourceBuilder = _fooResourceFactory.create();

FooResource fooResource = fooResourceBuilder.user(
        themeDisplay.getUser())
        .build();
```

| # | Linha | O que acontece |
|---|---|---|
| 1 | `_fooResourceFactory.create()` | A Factory (injetada pelo OSGi) cria um **Builder** |
| 2 | `.user(themeDisplay.getUser())` | Informa o usuário em nome de quem as operações serão executadas |
| 3 | `.build()` | Constrói o `FooResource` |

O mesmo é feito para `GooResource`. Substitui o `.authentication("test@liferay.com", "learn")` do `Test.java`: **não há senha no código**.

### Parte 7 – `Long fooId = null;` e `Long gooId = null;`

Igual ao `Test.java`: os IDs são declarados **antes** do `try` e começam como `null`, para que o `finally` saiba o que chegou a ser criado. Usa-se `Long` (objeto) e não `long` (primitivo) porque só o objeto aceita `null`.

### Parte 8 – O bloco `try { ... } catch { ... } finally { ... }`

```java
try {
    // criar, consultar, atualizar...
} catch (Exception e) {
    _log.error("Error: ", e);
} finally {
    // limpeza
}
```

Aqui há uma **diferença** em relação ao `Test.java`, que não tinha `catch`:

- No `Test.java`, uma falha subia como exceção e encerrava o programa, e o terminal mostrava o erro.
- No portlet, o `catch` **captura** a exceção e a registra no log com `_log.error("Error: ", e)`. Assim a exceção **não sobe** para o Liferay.

Por que isso? Uma exceção não tratada durante a renderização faria o widget exibir uma mensagem de erro na página. Registrando no log, o erro fica documentado e o restante da página continua normal.

O `finally` continua **sempre executando**, tenha ou não havido erro.

### Parte 9 – A ordem das operações no `try`

```
Foo:  POST → GET (lista) → GET → PUT → GET → PATCH → GET
Goo:  POST → GET (lista) → GET → PUT → GET → PATCH → GET
```

É a mesma sequência do `Test.java`. O `GET` depois de cada alteração serve para conferir o resultado; o Goo só é criado **depois** do Foo porque precisa do `fooId`.

### Parte 10 – O `finally` e a ordem de exclusão

```java
if (gooId != null) {
    try {
        deleteGoo(gooResource, gooId);
    } catch (Exception e) {
        _log.error("Error deleting Goo: ", e);
    }
}

if (fooId != null) {
    try {
        deleteFoo(fooResource, fooId);
    } catch (Exception e) {
        _log.error("Error deleting Foo: ", e);
    }
}
```

Raciocínio:

1. O Goo **pertence** ao Foo, então apagamos o **Goo primeiro**.
2. Cada exclusão só é tentada se o ID correspondente não for `null` (isto é, se o objeto chegou a ser criado).
3. Cada exclusão tem seu **próprio `try/catch`**: se a exclusão do Goo falhar, o erro é registrado e o código **continua** para tentar apagar o Foo.

| Situação | `gooId` | `fooId` | O que acontece |
|---|---|---|---|
| Tudo certo | preenchido | preenchido | Apaga Goo e depois Foo |
| Falhou antes de criar o Goo | `null` | preenchido | Apaga só o Foo |
| Falhou antes de criar o Foo | `null` | `null` | Não apaga nada |
| Goo criado, mas a exclusão falhou | preenchido | preenchido | Registra o erro do Goo e **ainda tenta** apagar o Foo |

> **Comparação com o `Test.java`:** lá, o Foo só era apagado se o Goo tivesse sido removido (`gooDeleted`). Aqui as duas tentativas são independentes. Se o Goo não for apagado, a exclusão do Foo provavelmente também falhará (ou deixará o Goo órfão, dependendo da implementação) e isso aparecerá no log. Veja a seção 2.5.

### Parte 11 – Os métodos de criação (`postFoo` e `postFooGoo`)

```java
Foo newFoo = new Foo();
newFoo.setName("POST Foo");
newFoo.setDescription("Posted Foo from R3B2Portlet");

Foo createdFoo = fooResource.postFoo(newFoo);
return createdFoo.getId();
```

| # | Linha | O que acontece |
|---|---|---|
| 1 | `new Foo()` | Cria um DTO vazio (o `Foo` do módulo `api`) |
| 2 | `setName` / `setDescription` | Preenche os campos que queremos enviar |
| 3 | `fooResource.postFoo(newFoo)` | **Chama o método diretamente na JVM**: não há conversão para JSON nem requisição HTTP |
| 4 | `createdFoo.getId()` | O `id` foi gerado pelo servidor; devolvemos para as próximas operações |

O `postFooGoo` é igual, mas usa `gooResource.postFooGoo(fooId, newGoo)`: o Goo nasce **dentro** do Foo informado.

**Nota de nomes:** no `Test.java`, os métodos auxiliares se chamavam `postGoo` e `getGooPage`. Aqui eles se chamam `postFooGoo` e `getFooGoosPage`, os **mesmos nomes** dos métodos do resource. Os nomes dos wrappers são livres; é só uma escolha de estilo.

### Parte 12 – Os métodos de leitura

- `getFooPage()` e `getFooGoosPage(fooId)` devolvem um objeto **`Page`** (paginado), com os itens, a página atual e o total. Aqui o tipo é `com.liferay.portal.vulcan.pagination.Page`.
- `getFoo(id)` e `getGoo(id)` devolvem **um único** objeto.
- Todos só **registram** o resultado com `_log.debug`. Ao concatenar um objeto em uma `String`, o Java chama o `toString()` do DTO.

### Parte 13 – PUT × PATCH no código

| Método | O que o objeto enviado contém | Efeito |
|---|---|---|
| `putFoo` | `name`, `description` **e `id`** | Substitui os dados do Foo |
| `patchFoo` | só `description` | Altera só a descrição; o `name` fica como estava |
| `putGoo` | `name`, `description` **e `id`** | Substitui os dados do Goo |
| `patchGoo` | só `description` | Altera só a descrição |

Duas diferenças em relação ao `Test.java`:

1. **O retorno é usado.** `put` e `patch` devolvem o objeto atualizado, que é guardado em uma variável e registrado com `_log.debug`. No `Test.java`, o retorno era descartado.
2. **O `putGoo` não envia `fooId`.** No `Test.java`, o `putGoo` recebia o `fooId` e o enviava no DTO "para manter a ligação com o Foo". Aqui o DTO leva `id` mas não `fooId`. O código funciona se a implementação do servidor não exigir esse campo; confira nos logs e com um `GET` se o Goo continua associado ao Foo depois do `PUT`.

### Parte 14 – Os métodos de exclusão

```java
private void deleteFoo(FooResource fooResource, Long fooId) throws Exception {
    _log.info("=== DELETE Foo ===");

    fooResource.deleteFoo(fooId);
    _log.info("Deleted Foo with ID: " + fooId);
}
```

Diferente do `Test.java` (em que os `delete*` tratavam a exceção e devolviam `boolean`), aqui os métodos simplesmente **declaram `throws Exception`**. Quem trata a exceção é o `try/catch` dentro do `finally` do `doView` (Parte 10). O efeito final é parecido: um erro na exclusão não interrompe a limpeza.

### Parte 15 – Os métodos são de instância (não `static`)

No `Test.java`, os métodos auxiliares eram `private static`, porque eram chamados a partir do `main`, que é estático. No portlet, o `doView` **não** é estático, então os auxiliares são métodos **de instância** (`private`, sem `static`).

### Parte 16 – As `@Reference` no final da classe

```java
@Reference
private FooResource.Factory _fooResourceFactory;

@Reference
private GooResource.Factory _gooResourceFactory;
```

Pedem ao OSGi as duas Factories (a de Foo e a de Goo). O OSGi preenche esses campos quando o componente é ativado. Se alguma delas não estiver disponível, o portlet não ativa.

### Parte 17 – `_log.info` × `_log.debug`

| Nível | Onde é usado | Aparece por padrão? |
|---|---|---|
| `info` | Marcadores `=== POST Foo ===` e mensagens `Deleted ...` | Normalmente sim |
| `debug` | Resultado de cada `GET`, `PUT` e `PATCH` | Normalmente **não**; precisa habilitar o nível `DEBUG` |
| `error` | Exceções | Sim |

## 2.4 Como criar o módulo, implantar e executar

O código **é um módulo OSGi** (`headless-r3b2-web`) dentro do projeto `liferay-r3b2`, e não um arquivo solto como o `Test.java`. O passo a passo:

**1. Confirme que `api` e `impl` estão implantados**

```bash
./gradlew headless-r3b2-api:deploy -Ddeploy.docker.container.id=<seu-container>
./gradlew headless-r3b2-impl:deploy -Ddeploy.docker.container.id=<seu-container>
```

> Troque `<seu-container>` pelo nome do **seu** container do Liferay (no tutorial é `ephesians-liferay`). Confira em `http://localhost:8080/o/api` se `headless-r3b2/v1.0` aparece.

**2. Crie o módulo `headless-r3b2-web`**

Crie a pasta `headless-r3b2-web` na raiz do projeto, com um `build.gradle` e as dependências da seção 1.13. Use o módulo `headless-r3b2-impl` como referência e inclua o novo módulo no `settings.gradle`, se o seu projeto exigir.

**3. Crie a classe `R3B2Portlet`**

Crie o arquivo no caminho correspondente ao pacote:

```text
headless-r3b2-web/src/main/java/com/acme/headless/r3b2/web/internal/R3B2Portlet.java
```

e cole o código da seção 2.2.

**4. Compile**

```bash
./gradlew headless-r3b2-web:classes
```

Se houver erros de `package ... does not exist`, faltam dependências no `build.gradle` (seção 2.6).

**5. Implante o módulo `web`**

```bash
./gradlew headless-r3b2-web:deploy -Ddeploy.docker.container.id=<seu-container>
```

> **Não** implante `headless-r3b2-client` nem `headless-r3b2-test`.

**6. Adicione o widget a uma página**

Logado com um usuário com permissões (por exemplo, o administrador do ambiente de treinamento), abra uma página, entre no modo de edição e, no painel de widgets, procure a categoria **Sample** e adicione o widget **R3B2**.

**7. Acompanhe o log e, se quiser, habilite o `DEBUG`**

```bash
docker logs -f <seu-container>
```

Para ver as mensagens `GET Foo: ...`, habilite o nível `DEBUG` para `com.acme.headless.r3b2.web.internal.R3B2Portlet` (seção 1.11) e atualize a página.

**O que você deve ver:**

- **Na página:** o widget aparece **vazio**. Isso é esperado, pois o `doView` não gera conteúdo.
- **No log**, em ordem: `=== POST Foo ===`, `=== GET Foo Page ===`, `=== GET Foo ===` (três vezes), `=== PUT Foo ===`, `=== PATCH Foo ===`, o mesmo para o Goo (`POST Goo`, `GET Foo Goos Page`, `GET Goo`, ...) e, por fim, `Deleted Goo with ID: ...` e `Deleted Foo with ID: ...`.
- **A cada atualização da página**, essa sequência se repete (seção 1.12).

**Dica:** para conferir o efeito de cada passo, use o API Explorer (`http://localhost:8080/o/api`) em paralelo, ou comente temporariamente o `finally` de exclusão e confira os registros criados (lembrando que, sem a limpeza, **cada atualização cria novos registros**).

## 2.5 Pontos de melhoria

Depois que tudo funcionar, estas melhorias deixam o código mais profissional:

1. **Não rodar o CRUD no `doView`.** Como o `doView` roda a cada renderização, uma simples visualização da página cria e apaga dados. Em uma aplicação real, as operações de escrita ficariam na fase de **action** (por exemplo, `processAction`, disparada por um botão ou formulário), e o `doView` só exibiria dados.
2. **Mostrar o resultado na tela.** Hoje o widget aparece vazio. O `renderResponse.getWriter()` permite escrever HTML na saída do portlet (o `doView` já declara `throws IOException`, que é a exceção desses métodos).
3. **Chamar `getUser()` uma vez só.** O `themeDisplay.getUser()` aparece duas vezes; pode ser guardado em uma variável `User user`.
4. **Revisar a limpeza.** Considere repetir a lógica do `Test.java`: só tentar apagar o Foo se o Goo foi apagado, ou aceitar que o Foo seja apagado de qualquer forma e comentar a escolha.
5. **Conferir o `putGoo`.** Verifique se ele deve enviar `fooId`, como no `Test.java`.
6. **Validar, não só registrar.** Compare os campos recebidos com os esperados (ex.: após o `PATCH`, o `name` deve continuar `"PUT Foo"`).
7. **Usar `isDebugEnabled`.** É comum, no Liferay, proteger logs com concatenação de `String` usando `if (_log.isDebugEnabled()) { ... }`, para não montar a mensagem quando o `DEBUG` está desligado.
9. **Evitar valores repetidos.** Os nomes e descrições (`"POST Foo"`, `"Posted Foo from R3B2Portlet"`, ...) podem virar constantes.
10. **Mensagens de erro mais claras.** Inclua o contexto (qual operação, qual ID) na mensagem do `catch` principal, em vez de apenas `"Error: "`.

## 2.6 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `package com.acme.headless.r3b2... does not exist` na compilação | O módulo `web` não enxerga o módulo `api` | Declare a dependência de `headless-r3b2-api` no `build.gradle` do módulo `web` |
| `package com.liferay.portal.vulcan.pagination does not exist` (ou `ThemeDisplay`, `javax.portlet`...) | Faltam dependências do Liferay / da API de portlets | Adicione as dependências da seção 1.13, seguindo o `build.gradle` de outro módulo |
| O código compila, mas o `FooResource.Factory` "não existe" | Imports do pacote errado (`.client.`) ou `buildREST` não executado | Use `com.acme.headless.r3b2.resource.v1_0` e rode `./gradlew headless-r3b2-impl:buildREST` |
| O widget **R3B2** não aparece em "Sample" | O módulo `web` não foi implantado, ou o componente não ativou (Factory indisponível) | Confirme o deploy de `api`, `impl` e `web`; confira o log e o estado dos *bundles* no Gogo Shell (`lb`) |
| O widget aparece, mas está **vazio** | Esperado: o `doView` não escreve conteúdo | Veja o resultado no log |
| Nenhum `GET Foo: ...` no log | O nível `DEBUG` não está habilitado | Habilite o `DEBUG` para `com.acme.headless.r3b2.web.internal.R3B2Portlet` |
| Nada aparece no log | O widget não está na página, ou o log consultado é de outro container | Confirme que o widget está na página e use `docker logs -f <seu-container>` |
| Erro de permissão (por exemplo, `PrincipalException`) no log | O usuário logado não tem permissão para a operação | Teste com um usuário com permissão (administrador) |
| `NullPointerException` com `fooId` | O `postFoo` falhou e o ID ficou `null` | Veja o erro registrado em `Error:` e corrija a criação |
| Foos ou Goos sobrando no Liferay | O `finally` falhou ao apagar | Veja `Error deleting ...` no log e apague pelo API Explorer |
| Muitos registros criados e apagados no log | O `doView` roda a cada renderização | Esperado neste exercício (seção 1.12) |
| `headless-r3b2/v1.0` não aparece no API Explorer | `api` e `impl` ainda não foram implantados | Faça o deploy (seção 2.4, passo 1) |

### Para praticar

1. Troque o `Log` por mensagens que mostrem o **`id`** retornado por cada operação, e confira com o API Explorer.
2. Faça o `doView` **mostrar na tela** o `name` e a `description` do Foo após o `PATCH`, usando `renderResponse.getWriter()`.
3. Faça o `getFooPage` **devolver** a página e registre só os nomes dos Foos.
4. Provoque um erro de propósito (ex.: um `fooId` inexistente em `postFooGoo`) e observe como o `catch` e o `finally` se comportam no log.
5. Compare, lado a lado, o `Test.java` e o `R3B2Portlet` usando a tabela da seção 1.10 e tente explicar cada diferença com suas próprias palavras.

---
