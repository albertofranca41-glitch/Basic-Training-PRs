# Guia: Atividade de REST Builder (Primeiro PR: Client JAR com Foo e Goo) *Esse markdown nao foi revisado ainda

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
  - [1.1 Revisão rápida: HTTP, REST e JSON](#11-revisão-rápida-http-rest-e-json)
  - [1.2 JSONWS × REST (RPC × REST)](#12-jsonws--rest-rpc--rest)
  - [1.3 OpenAPI](#13-openapi)
  - [1.4 O que é o REST Builder](#14-o-que-é-o-rest-builder)
  - [1.5 Os módulos do projeto liferay-r3b2](#15-os-módulos-do-projeto-liferay-r3b2)
  - [1.6 Arquivos gerados × arquivos que você implementa](#16-arquivos-gerados--arquivos-que-você-implementa)
  - [1.7 A API de exemplo: Foo e Goo](#17-a-api-de-exemplo-foo-e-goo)
  - [1.8 API Explorer: conferindo a API no navegador](#18-api-explorer-conferindo-a-api-no-navegador)
  - [1.9 Client JAR × OSGi Resource](#19-client-jar--osgi-resource)
  - [1.10 O padrão Builder](#110-o-padrão-builder)
  - [1.11 Pré-requisitos do ambiente](#111-pré-requisitos-do-ambiente)
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

Antes do código, vamos entender o "mundo" em que ele vive. Se você já conhece algum tópico, pode pular.

## 1.1 Revisão rápida: HTTP, REST e JSON

Este guia parte do princípio de que você já viu os conceitos básicos (por exemplo, no guia da atividade de JavaScript). Um resumo do que importa aqui:

- **Cliente e servidor:** o **servidor** é o Liferay DXP em `http://localhost:8080`; o **cliente** é o seu código (agora em **Java**, não mais JavaScript).
- **HTTP:** cada conversa é uma requisição + uma resposta. Os métodos `POST`, `GET`, `PUT`, `PATCH` e `DELETE` correspondem às operações do **CRUD** (criar, ler, atualizar, apagar).
- **REST:** estilo de API em que tudo é um **recurso** identificado por uma URL, e o método HTTP é o verbo.
- **JSON:** formato de texto em que os dados trafegam.

A grande diferença desta atividade: em JavaScript você montava a requisição **à mão** (`fetch`, cabeçalhos, `JSON.stringify`). Aqui, o **client JAR** faz tudo isso por você: você trabalha com **objetos Java** (`Foo`, `Goo`) e **métodos** (`postFoo`, `getFoo`...), e a biblioteca cuida do HTTP e do JSON por baixo.

Para aprofundar, veja os arquivos em [Recursos adicionais](#recursos-adicionais).

## 1.2 JSONWS × REST (RPC × REST)

O Liferay tem duas formas principais de expor serviços por HTTP:

| | **JSONWS** | **REST (Headless)** |
|---|---|---|
| Estilo | **RPC** (*Remote Procedure Call*) sobre HTTP | **REST** |
| Ideia | Você chama **métodos remotos** dos serviços do Service Builder | Você manipula **recursos** por URL + método HTTP |
| Acoplamento | Forte com o **banco de dados**: as entidades do Service Builder representam tabelas | Independente do banco: o contrato é definido por você |
| Gerada por | Service Builder | **REST Builder** |

> **RPC × REST em uma frase:** no RPC você diz **o que executar** (`addEntry(...)`); no REST você diz **sobre qual recurso** e **qual verbo** (`POST /entries`). Um não é necessariamente melhor que o outro: são ferramentas diferentes, como garfo e colher.

Como o JSONWS reflete o banco de dados, mudar uma tabela muda a API. No REST você desenha o **contrato** da API separadamente, o que dá mais liberdade e estabilidade para quem a consome.

## 1.3 OpenAPI

**OpenAPI** é uma **especificação** para **descrever** APIs REST em um arquivo (normalmente YAML). Nele você declara:

- os **caminhos** (`paths`): `/foo`, `/foo/{fooId}`, `/foo/{fooId}/goos`, `/goo/{gooId}`;
- os **métodos** de cada caminho (`get`, `post`, `put`, `patch`, `delete`);
- os **schemas**: o formato dos objetos (`Foo` e `Goo`, com seus campos e tipos).

Pense no OpenAPI como a **planta de uma casa**: ela não é a casa, mas descreve exatamente como ela deve ser. Quem lê a planta (pessoas ou ferramentas) sabe o que esperar.

## 1.4 O que é o REST Builder

O **REST Builder** é um **gerador de código** do Liferay. Ele lê a "planta" (OpenAPI) e gera o código Java para você. Seus dois arquivos de entrada principais são:

| Arquivo | O que contém |
|---|---|
| `rest-config.yaml` | Informações básicas da API que será gerada (nome, pacote, caminho, etc.) |
| `rest-openapi.yaml` | A **definição OpenAPI**: schemas `Foo` e `Goo` e as interações disponíveis |

O comando que dispara a geração é:

```bash
./gradlew headless-r3b2-impl:buildREST
```

Resultado: dezenas de arquivos Java e de propriedades são escritos nos módulos do projeto (você vê cada um deles listado no console como `Writing ...`).

Fluxo mental:

```
rest-openapi.yaml  ──► buildREST ──► DTOs, Resources, client, GraphQL, testes...
 (você descreve)       (gerador)       (código pronto)
```

## 1.5 Os módulos do projeto liferay-r3b2

O projeto é dividido em **módulos Gradle**, cada um com um papel:

| Módulo | O que tem | Vai para o Liferay? |
|---|---|---|
| `headless-r3b2-api` | Interfaces e DTOs usados **dentro** do Liferay (ex.: `FooResource` em `com.acme.headless.r3b2.resource.v1_0`) | **Sim** (deploy) |
| `headless-r3b2-impl` | As **implementações** (`FooResourceImpl`, `GooResourceImpl`), a aplicação JAX-RS, GraphQL e a configuração OpenAPI | **Sim** (deploy) |
| `headless-r3b2-client` | A **biblioteca cliente** (client JAR) para quem usa a API **de fora**, por HTTP (ex.: `FooResource` em `com.acme.headless.r3b2.client.resource.v1_0`) | **Não** |
| `headless-r3b2-test` | Esqueletos de testes de integração | **Não** |

> ⚠️ Repare que existem **dois** `FooResource`, em pacotes diferentes: um no **api** (para uso dentro da JVM do Liferay) e outro no **client** (para uso externo, por HTTP). **Nesta atividade você usa o do client.**

## 1.6 Arquivos gerados × arquivos que você implementa

| Tipo de arquivo | Tem `@generated`? | Regenerado pelo `buildREST`? | Você edita? |
|---|---|---|---|
| `FooResource.java`, `GooResource.java`, `BaseFooResourceImpl.java`... | **Sim** | **Sempre** | **Não** (suas alterações seriam apagadas) |
| `FooResourceImpl.java`, `GooResourceImpl.java` | Não | **Só se não existirem** | **Sim**: é onde você escreve a lógica |

Os `*ResourceImpl.java` do REST Builder são parecidos com os `*ServiceImpl.java` do Service Builder: não são regenerados e são feitos para o desenvolvedor implementar. A diferença é que o REST Builder só tem implementações **remotas** (os "resources" são como os "remote services").

Cada método público de `FooResourceImpl` leva `@Override`, porque ele implementa um método declarado em `BaseFooResourceImpl`, que por sua vez é gerado a partir do `rest-openapi.yaml`.

## 1.7 A API de exemplo: Foo e Goo

A API `headless-r3b2` tem **dois schemas** com um relacionamento **pai → filho**:

```
Foo  (pai)
 └── Goo  (filho: pertence a um Foo, por isso tem fooId)
```

| Schema | Campos usados na atividade |
|---|---|
| `Foo` | `id` (gerado pelo servidor), `name`, `description` |
| `Goo` | `id` (gerado pelo servidor), `name`, `description`, `fooId` |

Endpoints (relativos a `http://localhost:8080/o/headless-r3b2/v1.0`):

| Operação | Método | Caminho | Método no client JAR |
|---|---|---|---|
| Criar Foo | `POST` | `/foo` | `fooResource.postFoo(foo)` |
| Listar Foos | `GET` | `/foo` | `fooResource.getFooPage()` |
| Buscar um Foo | `GET` | `/foo/{fooId}` | `fooResource.getFoo(fooId)` |
| Atualizar Foo (completo) | `PUT` | `/foo/{fooId}` | `fooResource.putFoo(fooId, foo)` |
| Atualizar Foo (parcial) | `PATCH` | `/foo/{fooId}` | `fooResource.patchFoo(fooId, foo)` |
| Apagar Foo | `DELETE` | `/foo/{fooId}` | `fooResource.deleteFoo(fooId)` |
| Criar Goo de um Foo | `POST` | `/foo/{fooId}/goos` | `gooResource.postFooGoo(fooId, goo)` |
| Listar Goos de um Foo | `GET` | `/foo/{fooId}/goos` | `gooResource.getFooGoosPage(fooId)` |
| Buscar um Goo | `GET` | `/goo/{gooId}` | `gooResource.getGoo(gooId)` |
| Atualizar Goo (completo) | `PUT` | `/goo/{gooId}` | `gooResource.putGoo(gooId, goo)` |
| Atualizar Goo (parcial) | `PATCH` | `/goo/{gooId}` | `gooResource.patchGoo(gooId, goo)` |
| Apagar Goo | `DELETE` | `/goo/{gooId}` | `gooResource.deleteGoo(gooId)` |

Observe o padrão REST: o Goo é **criado e listado dentro do Foo** (`/foo/{fooId}/goos`), mas **buscado, alterado e apagado direto** pelo seu próprio id (`/goo/{gooId}`).

> **PUT × PATCH:** `PUT` **substitui** o recurso inteiro (por isso, no código, o `putGoo` envia também o `fooId`); `PATCH` altera **só os campos enviados** (no código, só a `description`).

## 1.8 API Explorer: conferindo a API no navegador

Depois que o `headless-r3b2-api` e o `headless-r3b2-impl` são **implantados (deploy)**, a API passa a existir no Liferay. Para conferir:

1. Abra `http://localhost:8080/o/api`.
2. Vá em **REST Applications** e procure por **`headless-r3b2/v1.0`**.

Antes do deploy ela **não aparece** (o Liferay ainda não conhece essa API). Depois do deploy, o API Explorer mostra todos os endpoints de Foo e Goo, com os formatos e um botão para testar cada um pela interface.

O tutorial pede que você exercite todos os endpoints de três maneiras: **pela interface (GUI)**, com **curl** e com **Postman**. É um ótimo aquecimento: assim você enxerga o HTTP "cru" antes de escondê-lo atrás do client JAR.

## 1.9 Client JAR × OSGi Resource

Há duas formas de consumir uma API gerada pelo REST Builder:

| | **Client JAR** (esta atividade) | **OSGi Resource** (outro PR) |
|---|---|---|
| Onde roda seu código | **Fora** do Liferay (um programa Java comum) | **Dentro** do Liferay (ex.: um portlet) |
| Como a chamada acontece | Por **HTTP** | Dentro da **mesma JVM** |
| Autenticação | Você informa usuário e senha | Você informa o usuário logado (`themeDisplay.getUser()`) |
| Pacote do `FooResource` | `...client.resource.v1_0` | `...resource.v1_0` |
| Custo | Maior (rede + serialização JSON) | Menor (chamada de método direta) |

O **client JAR** é um arquivo `.jar` com as classes do módulo `headless-r3b2-client`. Quem o usa **não precisa** ser um projeto Gradle nem rodar dentro do Liferay: basta ter o JAR no **classpath**.

> **Classpath** é a lista de lugares onde o Java procura as classes que o seu código importa. Se o JAR do client não estiver no classpath, o `import com.acme.headless.r3b2.client...` não compila.

## 1.10 O padrão Builder

No código você vai ver:

```java
FooResource fooResource = FooResource.builder()
        .authentication("test@liferay.com", "learn")
        .build();
```

Esse é o **padrão Builder**: em vez de um construtor gigante, você **configura passo a passo** (`.authentication(...)`) e finaliza com `.build()`, que devolve o objeto pronto.

Pontos importantes:

- `FooResource.builder()` devolve um **`FooResource.Builder`**.
- `.authentication(usuario, senha)` configura a **Basic Authentication** (usuário e senha enviados em cada requisição).
- `.build()` devolve o `FooResource` já pronto para uso.
- ⚠️ Como na atividade de JavaScript, Basic Auth só é aceitável em **ambiente local de treinamento** ou sobre HTTPS.

## 1.11 Pré-requisitos do ambiente

| Ferramenta / etapa | Para quê |
|---|---|
| **Liferay DXP rodando** em `localhost:8080` | O servidor que a API consulta |
| **Projeto `liferay-r3b2`** configurado como projeto Gradle/OSGi | Ele contém os módulos `api`, `impl`, `client` e `test` |
| **`buildREST` executado** e **`api` + `impl` implantados** | Sem isso, a API não existe no Liferay |
| **Client JAR gerado** (`./gradlew headless-r3b2-client:jar`) | É a biblioteca que o seu código usa |
| **JDK** (compilador `javac` e `java`) | Compilar e executar o código |
| **Git** e conta no **GitHub** | Versionar e enviar o PR |
| Um editor (ex.: VS Code) | Escrever o código |

Verifique:

```bash
java -version
javac -version
git --version
```

E confirme no navegador que `http://localhost:8080/o/api` lista **`headless-r3b2/v1.0`**.

## Recursos adicionais

Os arquivos abaixo aprofundam conceitos usados nesta etapa. Leia-os como complemento quando quiser explorar cada assunto com mais detalhes:

- [Cliente e servidor](./Cliente%20e%20Servidor.md): papéis de cliente e servidor e como trocam requisições e respostas.
- [HTTP](./HTTP.md): métodos, cabeçalhos, corpo das mensagens e códigos de status.
- [API](./API.md): APIs como contratos de comunicação entre sistemas.
- [REST](./REST.md): recursos, URLs e métodos HTTP em APIs REST.

---

# Etapa 2 – O código

## 2.1 Visão geral

O arquivo `Main.java` tem **um método `main`** (que executa o ciclo completo) e **doze métodos auxiliares**, um por operação:

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

> A lógica é exatamente a do código da atividade. Salve o arquivo como `Main.java` (o nome do arquivo deve ser igual ao nome da classe pública).

```java
import com.acme.headless.r3b2.client.dto.v1_0.Foo;
import com.acme.headless.r3b2.client.dto.v1_0.Goo;
import com.acme.headless.r3b2.client.resource.v1_0.FooResource;
import com.acme.headless.r3b2.client.resource.v1_0.GooResource;

public class Main {

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

**4. Salve o código como `Main.java` nessa pasta**

**5. Compile, informando o client JAR no classpath**

```bash
javac -classpath .:../headless-r3b2-client/build/libs/\* *.java
```

**6. Execute**

```bash
java -classpath .:../headless-r3b2-client/build/libs/* Main
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
| `class Main is public, should be declared in a file named Main.java` | O nome do arquivo não bate com o da classe | Salve como `Main.java` |
| `Could not find or load main class Main` | Esqueceu o `.` no classpath ou está em outra pasta | Rode no mesmo diretório do `Main.class`, com `-classpath .:...` |
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