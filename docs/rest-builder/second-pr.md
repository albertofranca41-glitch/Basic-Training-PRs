# Guia: Atividade de REST Builder (Segundo PR: Client JAR da Headless Delivery com Documentos e Knowledge Base)

Este guia ensina, do zero, a fazer a segunda atividade de REST Builder: usar o **client JAR da Headless Delivery** (uma API que **já vem pronta** no Liferay DXP) para escrever um programa **Java** que faz o ciclo completo de **CRUD** (criar, buscar, atualizar e apagar) de **pastas de documentos**, **documentos**, **pastas da Knowledge Base** e **artigos da Knowledge Base**.

Ele segue o mesmo formato do guia do primeiro PR e está dividido em **2 etapas**:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Só o que é **novo** em relação ao primeiro PR: a Headless Delivery, como obter o client JAR dela, `siteId`, upload de arquivos e logging |
| **Etapa 2 – O código** | O código completo da atividade, comentado e explicado parte por parte, como compilar, executar, enviar o PR e uma seção de **pontos de melhoria** |

> **Pré-requisito:** este guia assume que você já leu o guia do **primeiro PR de REST Builder** (Foo e Goo). Conceitos como cliente × servidor, HTTP, REST, JSON, OpenAPI, REST Builder, client JAR × OSGi Resource, `classpath` e o **padrão Builder** foram explicados lá e **não são repetidos aqui**.

> Este guia cobre **somente o segundo PR** de REST Builder: o uso do client JAR da **Headless Delivery** (seção "Client Resource", segunda parte do tutorial). O uso do **OSGi Resource** (`R3B2Portlet`) é outro PR e não está aqui.

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 O que muda em relação ao primeiro PR](#11-o-que-muda-em-relação-ao-primeiro-pr)
  - [1.2 Headless Delivery: uma API que já vem pronta](#12-headless-delivery-uma-api-que-já-vem-pronta)
  - [1.3 Como obter o client JAR da Headless Delivery](#13-como-obter-o-client-jar-da-headless-delivery)
  - [1.4 Os quatro recursos e como eles se relacionam](#14-os-quatro-recursos-e-como-eles-se-relacionam)
  - [1.5 O que é o `siteId`?](#15-o-que-é-o-siteid)
  - [1.6 Upload de arquivos: `multipart/form-data` e `Map<String, File>`](#16-upload-de-arquivos-multipartform-data-e-mapstring-file)
  - [1.7 Nem todo recurso usa os mesmos campos](#17-nem-todo-recurso-usa-os-mesmos-campos)
  - [1.8 Logging com `Log` em vez de `System.out`](#18-logging-com-log-em-vez-de-systemout)
  - [1.9 Como verificar o resultado (banco de dados e widgets)](#19-como-verificar-o-resultado-banco-de-dados-e-widgets)
  - [1.10 Pré-requisitos do ambiente](#110-pré-requisitos-do-ambiente)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral](#21-visão-geral)
  - [2.2 Código completo comentado](#22-código-completo-comentado)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
  - [2.4 Como compilar e executar](#24-como-compilar-e-executar)
  - [2.5 Pontos de melhoria](#25-pontos-de-melhoria)
  - [2.6 Erros comuns](#26-erros-comuns)

---

# Etapa 1 – Teoria

No primeiro PR você usou um client JAR **gerado por você** (a partir da API `headless-r3b2`, que você mesmo construiu com o REST Builder). Agora a ideia é a mesma, mas aplicada a uma API **do próprio Liferay**. Esta etapa explica o que muda.

## 1.1 O que muda em relação ao primeiro PR

| Aspecto | 1º PR (`headless-r3b2`) | 2º PR (`headless-delivery`) |
|---|---|---|
| Quem criou a API | Você, com o REST Builder | O Liferay (já vem instalada no DXP) |
| Precisa rodar `buildREST` e fazer `deploy`? | Sim | **Não** |
| De onde vem o client JAR | Gerado por você (`./gradlew headless-r3b2-client:jar`) | **Baixado** do repositório de artefatos do Liferay |
| Pacote dos imports | `com.acme.headless.r3b2.client...` | `com.liferay.headless.delivery.client...` |
| URL-base | `http://localhost:8080/o/headless-r3b2/v1.0` | `http://localhost:8080/o/headless-delivery/v1.0` |
| Recursos usados | `Foo` e `Goo` | Pastas de documentos, documentos, pastas da Knowledge Base e artigos |
| Precisa de `siteId`? | Não | **Sim** (para criar as pastas) |
| Envia arquivo? | Não | **Sim** (o PDF do documento) |
| Como verificar | `GET` e API Explorer | `GET`, banco de dados (MySQL) e os widgets do Liferay |

O **padrão Builder**, a **autenticação básica** e a lógica de `Resource` + DTO continuam exatamente iguais. O que você aprende de novo é quais recursos existem, como eles se encaixam e como enviar um arquivo.

## 1.2 Headless Delivery: uma API que já vem pronta

A **Headless Delivery** é uma das APIs REST que o Liferay já traz instaladas. Ela também foi construída com o REST Builder (é o mesmo tipo de projeto que você montou com o `liferay-r3b2`), mas é mantida pelo Liferay e cobre conteúdos como documentos, artigos da Knowledge Base, posts de blog, conteúdo estruturado e muito mais.

Como ela já está no servidor, **você não escreve nem implanta nada do lado do servidor**: o seu trabalho é só o lado do **cliente**, que é exatamente o que o client JAR oferece.

Você pode vê-la no API Explorer, em `http://localhost:8080/o/api`, procurando `headless-delivery/v1.0`.

## 1.3 Como obter o client JAR da Headless Delivery

No primeiro PR o client JAR saiu da sua própria compilação. Aqui ele precisa ser **baixado**, e a versão importa: ela deve corresponder à versão do **seu** Liferay.

Passo a passo (é o que o tutorial pede):

1. No API Explorer (`http://localhost:8080/o/api`), abra `headless-delivery/v1.0`.
2. Procure o texto que informa o `group ID`, o `artifact ID` e a `version` do client JAR. Algo como: group ID `com.liferay`, artifact ID `com.liferay.headless.delivery.client` e uma versão (por exemplo, `3.0.10`).
3. Acesse o repositório de artefatos do Liferay:
   `https://repository.liferay.com/nexus/content/groups/public/com/liferay/com.liferay.headless.delivery.client`
4. Clique na **sua** versão e baixe o arquivo `com.liferay.headless.delivery.client-<versão>.jar`.
5. Baixe também o `...-sources.jar`. Ele contém o **código-fonte** do cliente. Você pode abri-lo com um gerenciador de arquivos compactados (o tutorial usa `xarchiver`) para ver quais métodos cada `Resource` oferece.

> **Dica:** o `-sources.jar` é a melhor "documentação" do client JAR. Os nomes de métodos como `postSiteDocumentFolder` ou `postDocumentFolderDocument` não são adivinhados: eles aparecem ali, e seguem uma regra de nomes baseada no caminho do endpoint.

Uma versão do JAR diferente da do servidor pode funcionar parcialmente, mas pode também apresentar campos ou métodos ausentes. Use a versão indicada pelo seu API Explorer.

## 1.4 Os quatro recursos e como eles se relacionam

A atividade mexe com dois "mundos" do Liferay, e cada um tem uma **pasta** que contém um **item**:

```text
Site (siteId)
 |
 +---- Pasta de documentos (DocumentFolder)
 |        |
 |        +---- Documento (Document)  <- com um arquivo (PDF)
 |
 +---- Pasta da Knowledge Base (KnowledgeBaseFolder)
          |
          +---- Artigo (KnowledgeBaseArticle)
```

Tanto a pasta de documentos quanto a pasta da Knowledge Base são criadas **diretamente dentro de um site**. Já o documento e o artigo são criados **dentro de uma pasta**, informando o `id` dela. É a mesma ideia do Goo dentro do Foo no primeiro PR.

| Classe (DTO) | Classe (Resource) | Representa |
|---|---|---|
| `DocumentFolder` | `DocumentFolderResource` | Pasta da Biblioteca de Documentos |
| `Document` | `DocumentResource` | Arquivo (com metadados) dentro de uma pasta |
| `KnowledgeBaseFolder` | `KnowledgeBaseFolderResource` | Pasta da Knowledge Base |
| `KnowledgeBaseArticle` | `KnowledgeBaseArticleResource` | Artigo da Knowledge Base |

### Endpoints utilizados

A URL-base é `http://localhost:8080/o/headless-delivery/v1.0`. Os caminhos abaixo são relativos a ela.

| Recurso | Operação | Método HTTP | Caminho | Método no client JAR |
|---|---|---|---|---|
| Pasta de documentos | Criar | `POST` | `/sites/{siteId}/document-folders` | `postSiteDocumentFolder(siteId, folder)` |
| | Buscar | `GET` | `/document-folders/{documentFolderId}` | `getDocumentFolder(id)` |
| | Atualizar | `PUT` | `/document-folders/{documentFolderId}` | `putDocumentFolder(id, folder)` |
| | Atualizar parcialmente | `PATCH` | `/document-folders/{documentFolderId}` | `patchDocumentFolder(id, folder)` |
| | Apagar | `DELETE` | `/document-folders/{documentFolderId}` | `deleteDocumentFolder(id)` |
| Documento | Criar | `POST` | `/document-folders/{documentFolderId}/documents` | `postDocumentFolderDocument(folderId, doc, files)` |
| | Buscar | `GET` | `/documents/{documentId}` | `getDocument(id)` |
| | Atualizar | `PUT` | `/documents/{documentId}` | `putDocument(id, doc, files)` |
| | Atualizar parcialmente | `PATCH` | `/documents/{documentId}` | `patchDocument(id, doc, files)` |
| | Apagar | `DELETE` | `/documents/{documentId}` | `deleteDocument(id)` |
| Pasta da KB | Criar | `POST` | `/sites/{siteId}/knowledge-base-folders` | `postSiteKnowledgeBaseFolder(siteId, folder)` |
| | Buscar | `GET` | `/knowledge-base-folders/{knowledgeBaseFolderId}` | `getKnowledgeBaseFolder(id)` |
| | Atualizar | `PUT` | `/knowledge-base-folders/{knowledgeBaseFolderId}` | `putKnowledgeBaseFolder(id, folder)` |
| | Atualizar parcialmente | `PATCH` | `/knowledge-base-folders/{knowledgeBaseFolderId}` | `patchKnowledgeBaseFolder(id, folder)` |
| | Apagar | `DELETE` | `/knowledge-base-folders/{knowledgeBaseFolderId}` | `deleteKnowledgeBaseFolder(id)` |
| Artigo da KB | Criar | `POST` | `/knowledge-base-folders/{knowledgeBaseFolderId}/knowledge-base-articles` | `postKnowledgeBaseFolderKnowledgeBaseArticle(folderId, article)` |
| | Buscar | `GET` | `/knowledge-base-articles/{knowledgeBaseArticleId}` | `getKnowledgeBaseArticle(id)` |
| | Atualizar | `PUT` | `/knowledge-base-articles/{knowledgeBaseArticleId}` | `putKnowledgeBaseArticle(id, article)` |
| | Atualizar parcialmente | `PATCH` | `/knowledge-base-articles/{knowledgeBaseArticleId}` | `patchKnowledgeBaseArticle(id, article)` |
| | Apagar | `DELETE` | `/knowledge-base-articles/{knowledgeBaseArticleId}` | `deleteKnowledgeBaseArticle(id)` |

Repare no padrão: o recurso **criado dentro de outro** usa um caminho aninhado (`/document-folders/{id}/documents`), mas, depois de criado, ele tem o **seu próprio** caminho (`/documents/{id}`). Isso é igual ao que você viu com Foo e Goo.

> Esta atividade usa o ciclo **CRUD** de cada recurso. Os endpoints de **listagem** (que devolvem uma `Page`) também existem na Headless Delivery, mas não são exigidos aqui.

## 1.5 O que é o `siteId`?

O Liferay organiza conteúdos em **sites**. Cada site tem um identificador numérico, o `siteId`. Como as pastas de documentos e da Knowledge Base ficam dentro de um site, o endpoint de criação precisa saber **qual**:

```text
POST /sites/{siteId}/document-folders
```

No código, isso aparece como o primeiro argumento de `postSiteDocumentFolder(SITE_ID, folder)`.

Cada instalação tem os seus próprios IDs, então o valor do código de exemplo (`20117L`) provavelmente **não é o do seu ambiente**. Você pode encontrar o ID do seu site nas configurações do site no Liferay (o caminho exato do menu varia conforme a versão) ou consultando os sites pelo API Explorer.

> Note o sufixo `L` em `20117L`: em Java, ele indica que o número literal é um `long`. IDs no Liferay são números grandes, que não cabem em um `int`.

## 1.6 Upload de arquivos: `multipart/form-data` e `Map<String, File>`

Até agora, todas as requisições enviaram apenas **JSON**. Um documento, porém, tem **dois tipos de conteúdo**: os **metadados** (título, descrição) e o **arquivo** em si (os bytes do PDF).

Para enviar os dois na mesma requisição, o HTTP usa o formato **`multipart/form-data`**: o corpo é dividido em "partes", e cada parte tem um nome. Na Headless Delivery:

- uma parte carrega os **metadados** (o DTO `Document`, convertido para JSON pelo client JAR);
- outra parte, chamada `file`, carrega o **arquivo**.

No client JAR, você não monta isso à mão. Em vez disso, o método recebe:

1. o **DTO** com os metadados (`Document`);
2. um **`Map<String, File>`**, em que a **chave** é o nome da parte (`"file"`) e o **valor** é o objeto `java.io.File` que aponta para o arquivo no seu computador.

```java
File file = new File("/caminho/do/arquivo.pdf");

Map<String, File> files = new HashMap<>();
files.put("file", file);
```

O objeto `File` é apenas uma **referência a um caminho**: ele não carrega o arquivo para a memória nem verifica se ele existe. Se o caminho estiver errado, o erro só aparece quando o client JAR tenta ler o arquivo.

### E no `PUT` e no `PATCH`?

Os métodos `putDocument` e `patchDocument` **também** exigem o `Map<String, File>`, porque o contrato aceita um novo arquivo nessas operações. Quando você quer alterar **só os metadados**, basta passar um mapa vazio (`Map.of()`), que significa "sem arquivo novo". É por isso que o código usa `Map.of()` nesses dois casos.

> `Map.of()` existe a partir do **Java 9**. Em Java 8, use `Collections.emptyMap()`.

## 1.7 Nem todo recurso usa os mesmos campos

Ao contrário de `Foo` e `Goo` (que tinham `name` e `description`), cada recurso aqui tem campos próprios. Fique atento a esta diferença, porque trocar um pelo outro causa erro de compilação:

| DTO | Campos usados na atividade |
|---|---|
| `DocumentFolder` | `name`, `description` |
| `Document` | `title`, `description` (o arquivo vai à parte, no `Map`) |
| `KnowledgeBaseFolder` | `name`, `description` |
| `KnowledgeBaseArticle` | `title`, `description`, `articleBody` |

Duas observações:

- Pastas têm **`name`**; documentos e artigos têm **`title`**.
- O `articleBody` de um artigo é **HTML**, por isso o exemplo usa `"<p>KBArticle Body.</p>"`.

O `PUT` e o `PATCH` seguem a mesma ideia do primeiro PR: o `PUT` envia os campos principais (substituição), e o `PATCH` envia **somente** o que se quer alterar (aqui, a `description`).

## 1.8 Logging com `Log` em vez de `System.out`

No primeiro PR, a saída era feita com `System.out.println` e `System.err.println`. Neste código, usa-se o **logging do Liferay**:

```java
import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;

private static final Log log = LogFactoryUtil.getLog(Test.class);
```

**Log** é uma forma de registrar mensagens com um **nível de severidade**. Os mais usados:

| Nível | Uso típico |
|---|---|
| `debug` | Detalhes para diagnóstico (ex.: o conteúdo completo de uma resposta) |
| `info` | Marcos normais da execução (ex.: "Document deleted.") |
| `warn` | Algo estranho, mas que não impediu a execução |
| `error` | Falhas |

A vantagem sobre `System.out` é que o nível pode ser **filtrado por configuração**: em produção você pode mostrar só `info` e acima, e ligar o `debug` apenas quando precisar investigar.

O código usa `log.info(...)` para **marcar o início de cada operação** (`=== POST DocumentFolder ===`) e `log.debug(...)` para mostrar **o conteúdo das respostas**, concatenando o texto e o objeto.

### Dois cuidados importantes

1. **O `Log` vem do `portal-kernel`.** Essas classes fazem parte do Liferay, e não do client JAR. Em um programa solto (fora do Liferay), o `portal-kernel.jar` e as dependências necessárias precisam estar no `classpath`.
2. **O nível de log.** O nível `debug` pode estar desligado por padrão; nesse caso, as respostas registradas com `log.debug(...)` não aparecem.

## 1.9 Como verificar o resultado (banco de dados e widgets)

O tutorial pede que você **confira** o que o seu programa fez, por duas vias:

1. **Banco de dados (MySQL):** o tutorial indica consultas como `select * from KBArticle;` e `select * from KBFolder;` para a Knowledge Base. Para a Biblioteca de Documentos, as tabelas costumam se chamar `DLFolder` e `DLFileEntry` (confira os nomes no seu banco).
2. **Widgets no Liferay:** o widget da **Knowledge Base** mostra pastas e artigos; o widget da **Biblioteca de Documentos** (*Documents and Media*) mostra pastas e arquivos.

Quando as exclusões são bem-sucedidas, os registros somem no final. Para ver os dados no banco ou no widget, você pode, **temporariamente**, comentar as chamadas de exclusão (ou colocar uma pausa antes do `finally`) e depois voltar ao código original antes de enviar o PR.

## 1.10 Pré-requisitos do ambiente

| Ferramenta ou etapa | Finalidade |
|---|---|
| Liferay DXP em execução | Disponibilizar a Headless Delivery em `localhost:8080` |
| Client JAR da Headless Delivery (versão do seu Liferay) | Fornecer as classes `...headless.delivery.client...` |
| `portal-kernel.jar` no classpath | Fornecer `Log` e `LogFactoryUtil` (usados no código) |
| JDK 9 ou superior | `Map.of()` não existe no Java 8 |
| Um arquivo PDF no seu computador | Ser enviado como o conteúdo do documento |
| `siteId` do seu site | Criar as pastas no site correto |
| Credenciais do ambiente de treinamento | Autenticar com Basic Auth |
| Git e conta no GitHub | Versionar e enviar o PR |

Para verificar Java e Git:

```bash
java -version
javac -version
git --version
```

## Recursos adicionais

Estes materiais aprofundam os conceitos da atividade e já foram citados no guia do primeiro PR:

- [JSON](../theory/json.md)
- [OpenAPI](../theory/openapi.md)
- [DTOs e Schemas](../theory/dtos-and-schemas.md)
- [Cliente e servidor](../theory/client-server.md)
- [HTTP](../theory/http.md)
- [REST](../theory/rest.md)
- [API](../theory/api.md)

---

# Etapa 2 – O código

## 2.1 Visão geral

O arquivo `Test.java` tem **um método `main`**, **quatro métodos de CRUD** (um para cada recurso) e **quatro métodos de exclusão**:

```
main():                       orquestra tudo (cria, atualiza, e limpa no finally)

CRUD (cada um faz POST → GET → PUT → GET → PATCH → GET e devolve o ID criado):
  documentFolderCrud()        → pasta de documentos
  documentCrud(docFolderId)   → documento (com upload de PDF), dentro da pasta
  knowledgeBaseFolderCrud()   → pasta da Knowledge Base
  knowledgeBaseArticleCrud(kbFolderId) → artigo, dentro da pasta da KB

Exclusão (DELETE):
  deleteKnowledgeBaseArticle()
  deleteKnowledgeBaseFolder()
  deleteDocument()
  deleteDocumentFolder()
```

O fluxo do `main` é:

```
1. try {
2.   Cria pasta de documentos  (CRUD completo)  → docFolderId
3.   Cria documento na pasta   (CRUD completo)  → docId
4.   Cria pasta da KB          (CRUD completo)  → kbFolderId
5.   Cria artigo na pasta      (CRUD completo)  → kbArticleId
6. } finally {
7.   Tenta apagar na ordem INVERSA da criação:
     artigo → pasta da KB → documento → pasta de documentos
8. }
```

> Diferença em relação ao primeiro PR: lá o `main` chamava cada `GET`/`PUT`/`PATCH` diretamente. Aqui, cada recurso tem **um método de CRUD** que executa o ciclo inteiro e devolve o `id` criado. O `main` fica curto e só orquestra.

## 2.2 Código completo comentado

> Salve o arquivo como `Test.java` (o nome do arquivo deve ser igual ao nome da classe pública). Antes de executar, troque `SITE_ID`, as credenciais e o caminho do PDF pelos do **seu** ambiente.


```java
// ===============================================================
//   DTOs são as classes que representam os recursos da API.
// ===============================================================

import com.liferay.headless.delivery.client.dto.v1_0.Document;
import com.liferay.headless.delivery.client.dto.v1_0.DocumentFolder;
import com.liferay.headless.delivery.client.dto.v1_0.KnowledgeBaseArticle;
import com.liferay.headless.delivery.client.dto.v1_0.KnowledgeBaseFolder;

// ===============================================================
//   Resource é a classe responsavel por realizar as requisicoes 
//   HTTP para a API Headless da Liferay.
// ===============================================================

import com.liferay.headless.delivery.client.resource.v1_0.DocumentResource;
import com.liferay.headless.delivery.client.resource.v1_0.DocumentFolderResource;
import com.liferay.headless.delivery.client.resource.v1_0.KnowledgeBaseArticleResource;
import com.liferay.headless.delivery.client.resource.v1_0.KnowledgeBaseFolderResource;

import java.io.File;
import java.util.HashMap;
import java.util.Map;

import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;

public class Test {

    private static final Log log = LogFactoryUtil.getLog(Test.class);

    private static final long SITE_ID = 20117L;

    private static final String USER = "test@liferay.com";

    private static final String PASSWORD = "learn";

    // -------------------------------------------------------------
    // Cria o client responsavel por consumir os endpoints
    // respectivos a cada rescurso.
    // -------------------------------------------------------------

    private static final DocumentFolderResource DOCUMENT_FOLDER_RESOURCE = DocumentFolderResource.builder()
            .authentication(USER, PASSWORD)
            .build();

    private static final DocumentResource DOCUMENT_RESOURCE = DocumentResource.builder()
            .authentication(USER, PASSWORD)
            .build();

    private static final KnowledgeBaseFolderResource KNOWLEDGE_BASE_FOLDER_RESOURCE = KnowledgeBaseFolderResource
            .builder()
            .authentication(USER, PASSWORD)
            .build();

    private static final KnowledgeBaseArticleResource KNOWLEDGE_BASE_ARTICLE_RESOURCE = KnowledgeBaseArticleResource
            .builder()
            .authentication(USER, PASSWORD)
            .build();

    public static void main(String[] args) throws Exception {

        Long docFolderId = null;
        Long docId = null;

        Long kbFolderId = null;
        Long kbArticleId = null;

        // --------------------------------------------------------------
        // O finally tenta remover os recursos na ordem inversa
        // em que foram criados, mesmo em caso de exceção.
        // --------------------------------------------------------------

        try {

            docFolderId = documentFolderCrud();

            docId = documentCrud(docFolderId);

            kbFolderId = knowledgeBaseFolderCrud();

            kbArticleId = knowledgeBaseArticleCrud(kbFolderId);

        } finally {

            boolean kbArticleDeleted = (kbArticleId == null);

            if (kbArticleId != null) {
                kbArticleDeleted = deleteKnowledgeBaseArticle(kbArticleId);
            }

            if (kbFolderId != null && kbArticleDeleted) {
                deleteKnowledgeBaseFolder(kbFolderId);
            }

            boolean documentDeleted = (docId == null);

            if (docId != null) {
                documentDeleted = deleteDocument(docId);
            }

            if (docFolderId != null && documentDeleted) {
                deleteDocumentFolder(docFolderId);
            }

        }
    }

    private static Long documentFolderCrud() throws Exception {

        // ---------------------------------------------------------
        // POST /v1.0/sites/{siteId}/document-folders
        // Cria uma nova pasta de documentos dentro do site
        // e armazena seu ID para as proximas operações.
        // ---------------------------------------------------------

        log.info("=== POST DocumentFolder ===");

        DocumentFolder newDocumentFolder = new DocumentFolder();

        newDocumentFolder.setName("DocFolder");
        newDocumentFolder.setDescription("DocFolder Description");

        DocumentFolder createdDocumentFolder = DOCUMENT_FOLDER_RESOURCE.postSiteDocumentFolder(SITE_ID,
                newDocumentFolder);
        log.debug("Created DocumentFolder: " + createdDocumentFolder);

        // Guarda o ID gerado pelo servidor.
        Long docFolderId = createdDocumentFolder.getId();

        // ---------------------------------------------------------
        // GET /v1.0/document-folders/{documentFolderId}
        // Busca a pasta de documentos criada por meio do ID.
        // ---------------------------------------------------------

        log.info("=== GET DocumentFolder ===");

        log.debug(DOCUMENT_FOLDER_RESOURCE.getDocumentFolder(docFolderId));

        // ---------------------------------------------------------
        // PUT /v1.0/document-folders/{documentFolderId}
        // Atualiza totalmente os atributos da pasta de documentos.
        // ---------------------------------------------------------

        log.info("=== PUT DocumentFolder ===");

        DocumentFolder updatedDocumentFolder = new DocumentFolder();

        updatedDocumentFolder.setName("Updated DocFolder");
        updatedDocumentFolder.setDescription("Updated DocFolder Description");

        log.debug("Response: " + DOCUMENT_FOLDER_RESOURCE.putDocumentFolder(docFolderId, updatedDocumentFolder));

        // ---------------------------------------------------------
        // GET /v1.0/document-folders/{documentFolderId}
        // Confere como ficou a pasta de documentos após o PUT.
        // ---------------------------------------------------------

        log.info("=== GET DocumentFolder ===");

        log.debug(DOCUMENT_FOLDER_RESOURCE.getDocumentFolder(docFolderId));

        // ---------------------------------------------------------
        // PATCH /v1.0/document-folders/{documentFolderId}
        // Atualiza parcialmente a pasta de documentos criada.
        // ---------------------------------------------------------

        log.info("=== PATCH DocumentFolder ===");

        DocumentFolder patchedDocumentFolder = new DocumentFolder();

        patchedDocumentFolder.setDescription("Patched DocFolder Description");

        log.debug("Response: " + DOCUMENT_FOLDER_RESOURCE.patchDocumentFolder(docFolderId, patchedDocumentFolder));

        // ---------------------------------------------------------
        // GET /v1.0/document-folders/{documentFolderId}
        // Confere o resultado após o PATCH.
        // ---------------------------------------------------------

        log.info("=== GET DocumentFolder ===");

        log.debug(DOCUMENT_FOLDER_RESOURCE.getDocumentFolder(docFolderId));

        return docFolderId;

    }

    private static Long documentCrud(Long docFolderId) throws Exception {

        // ---------------------------------------------------------
        // POST /v1.0/document-folders/{documentFolderId}/documents
        // ---------------------------------------------------------

        log.info("=== POST Document ===");

        Document newDocument = new Document();

        newDocument.setTitle("Doc");
        newDocument.setDescription("Doc Description");

        File file = new File("/home/me/Downloads/Westminster_Confession.pdf");

        Map<String, File> files = new HashMap<>();
        files.put("file", file);

        Document createdDocument = DOCUMENT_RESOURCE.postDocumentFolderDocument(
                docFolderId, // Id da pasta em que o documento será criado
                newDocument, // Metadados do documento
                files); // Arquivo a ser enviado
        log.debug("Created Document: " + createdDocument);

        Long docId = createdDocument.getId();

        // ---------------------------------------------------------
        // GET /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== GET Document ===");

        log.debug(DOCUMENT_RESOURCE.getDocument(docId));

        // ---------------------------------------------------------
        // PUT /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== PUT Document ===");

        Document changedDocument = new Document();

        changedDocument.setTitle("Updated Doc");
        changedDocument.setDescription("Updated Doc Description");

        Document updatedDocument = DOCUMENT_RESOURCE.putDocument(
            docId, 
            changedDocument, 
            Map.of());
        log.debug("Updated Document: " + updatedDocument);

        // ---------------------------------------------------------
        // GET /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== GET Document ===");

        log.debug(DOCUMENT_RESOURCE.getDocument(docId));

        // ---------------------------------------------------------
        // PATCH /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== PATCH Document ===");

        Document partialChangeDoc = new Document();

        partialChangeDoc.setDescription("Patched Doc Description");

        Document patchedDocument = DOCUMENT_RESOURCE.patchDocument(
            docId, 
            partialChangeDoc, 
            Map.of());

        log.debug("Patched Document: " + patchedDocument);

        // ---------------------------------------------------------
        // GET /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== GET Document ===");
        
        log.debug(DOCUMENT_RESOURCE.getDocument(docId));

        return docId;
    }

    private static Long knowledgeBaseFolderCrud() throws Exception {

        // ---------------------------------------------------------
        // POST /v1.0/sites/{siteId}/knowledge-base-folders
        // ---------------------------------------------------------

        log.info("=== POST KnowledgeBaseFolder ===");

        KnowledgeBaseFolder newKnowledgeBaseFolder = new KnowledgeBaseFolder();

        newKnowledgeBaseFolder.setName("KBFolder");
        newKnowledgeBaseFolder.setDescription("KBFolder Description");

        KnowledgeBaseFolder createdKnowledgeBaseFolder = KNOWLEDGE_BASE_FOLDER_RESOURCE.postSiteKnowledgeBaseFolder(
                SITE_ID,
                newKnowledgeBaseFolder);
        log.debug("Created KnowledgeBaseFolder: " + createdKnowledgeBaseFolder);

        // Guarda o ID gerado pelo servidor.
        Long kbFolderId = createdKnowledgeBaseFolder.getId();

        // ---------------------------------------------------------
        // GET /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ---------------------------------------------------------

        log.info("=== GET KnowledgeBaseFolder ===");

        log.debug(KNOWLEDGE_BASE_FOLDER_RESOURCE.getKnowledgeBaseFolder(kbFolderId));

        // ---------------------------------------------------------
        // PUT /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ---------------------------------------------------------

        log.info("=== PUT KnowledgeBaseFolder ===");

        KnowledgeBaseFolder updatedKnowledgeBaseFolder = new KnowledgeBaseFolder();

        updatedKnowledgeBaseFolder.setName("Updated KBFolder");
        updatedKnowledgeBaseFolder.setDescription("Updated KBFolder Description");

        log.debug("Response: " + KNOWLEDGE_BASE_FOLDER_RESOURCE.putKnowledgeBaseFolder(kbFolderId, updatedKnowledgeBaseFolder));

        // ---------------------------------------------------------
        // GET /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ---------------------------------------------------------

        log.info("=== GET KnowledgeBaseFolder ===");

        log.debug(KNOWLEDGE_BASE_FOLDER_RESOURCE.getKnowledgeBaseFolder(kbFolderId));

        // ---------------------------------------------------------
        // PATCH /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ---------------------------------------------------------

        log.info("=== PATCH KnowledgeBaseFolder ===");

        KnowledgeBaseFolder patchedKnowledgeBaseFolder = new KnowledgeBaseFolder();

        patchedKnowledgeBaseFolder.setDescription("Patched KBFolder Description");

        log.debug("Response: " + KNOWLEDGE_BASE_FOLDER_RESOURCE.patchKnowledgeBaseFolder(kbFolderId, patchedKnowledgeBaseFolder));

        // ---------------------------------------------------------
        // GET /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ---------------------------------------------------------

        log.info("=== GET KnowledgeBaseFolder ===");

        log.debug(KNOWLEDGE_BASE_FOLDER_RESOURCE.getKnowledgeBaseFolder(kbFolderId));

        return kbFolderId;

    }

    private static Long knowledgeBaseArticleCrud(Long kbFolderId) throws Exception {

        // ---------------------------------------------------------------------------------
        // POST /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}/knowledge-base-articles
        // ---------------------------------------------------------------------------------

        log.info("=== POST KnowledgeBaseArticle ===");

        KnowledgeBaseArticle newKnowledgeBaseArticle = new KnowledgeBaseArticle();

        newKnowledgeBaseArticle.setTitle("KBArticle");
        newKnowledgeBaseArticle.setDescription("KBArticle Description");
        newKnowledgeBaseArticle.setArticleBody("<p>KBArticle Body.</p>");

        KnowledgeBaseArticle createdKnowledgeBaseArticle = KNOWLEDGE_BASE_ARTICLE_RESOURCE
                .postKnowledgeBaseFolderKnowledgeBaseArticle(kbFolderId,
                        newKnowledgeBaseArticle);
        log.debug("Created KnowledgeBaseArticle: " + createdKnowledgeBaseArticle);

        Long kbArticleId = createdKnowledgeBaseArticle.getId();

        // ---------------------------------------------------------------------------------
        // GET /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // ---------------------------------------------------------------------------------

        log.info("=== GET KnowledgeBaseArticle ===");

        log.debug(KNOWLEDGE_BASE_ARTICLE_RESOURCE.getKnowledgeBaseArticle(kbArticleId));

        // ---------------------------------------------------------------------------------
        // PUT /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // ---------------------------------------------------------------------------------

        log.info("=== PUT KnowledgeBaseArticle ===");

        KnowledgeBaseArticle updatedKnowledgeBaseArticle = new KnowledgeBaseArticle();

        updatedKnowledgeBaseArticle.setTitle("Updated KBArticle");
        updatedKnowledgeBaseArticle.setDescription("Updated KBArticle Description");
        updatedKnowledgeBaseArticle.setArticleBody("<p>Updated KBArticle Body.</p>");

        log.debug("Response: " + KNOWLEDGE_BASE_ARTICLE_RESOURCE.putKnowledgeBaseArticle(kbArticleId, updatedKnowledgeBaseArticle));

        // ---------------------------------------------------------------------------------
        // GET /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // ---------------------------------------------------------------------------------

        log.info("=== GET KnowledgeBaseArticle ===");

        log.debug(KNOWLEDGE_BASE_ARTICLE_RESOURCE.getKnowledgeBaseArticle(kbArticleId));

        // ---------------------------------------------------------------------------------
        // PATCH /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // ---------------------------------------------------------------------------------

        log.info("=== PATCH KnowledgeBaseArticle ===");

        KnowledgeBaseArticle patchedKnowledgeBaseArticle = new KnowledgeBaseArticle();

        patchedKnowledgeBaseArticle.setDescription("Patched KBArticle Description");

        log.debug("Response: " + KNOWLEDGE_BASE_ARTICLE_RESOURCE.patchKnowledgeBaseArticle(kbArticleId, patchedKnowledgeBaseArticle));

        // ---------------------------------------------------------------------------------
        // GET /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // ---------------------------------------------------------------------------------

        log.info("=== GET KnowledgeBaseArticle ===");

        log.debug(KNOWLEDGE_BASE_ARTICLE_RESOURCE.getKnowledgeBaseArticle(kbArticleId));

        return kbArticleId;
    }

    private static boolean deleteDocumentFolder(Long docFolderId) {

        // ---------------------------------------------------------
        // DELETE /v1.0/document-folders/{documentFolderId}
        // ---------------------------------------------------------

        log.info("=== DELETE DocumentFolder ===");

        try {
            DOCUMENT_FOLDER_RESOURCE.deleteDocumentFolder(docFolderId);
            log.info("DocumentFolder deleted.");
            return true;
        }
        catch (Exception e) {
            log.error("Erro ao remover o DocumentFolder de ID " + docFolderId, e);
            return false;
        }
    }

    private static boolean deleteDocument(Long docId) {

        // ---------------------------------------------------------
        // DELETE /v1.0/documents/{documentId}
        // ---------------------------------------------------------

        log.info("=== DELETE Document ===");

        try {
            DOCUMENT_RESOURCE.deleteDocument(docId);
            log.info("Document deleted.");
            return true;
        }
        catch (Exception e) {
            log.error("Erro ao remover o Document de ID " + docId, e);
            return false;
        }
    }

    private static boolean deleteKnowledgeBaseFolder(Long kbFolderId) {

        // ------------------------------------------------------------
        // DELETE /v1.0/knowledge-base-folders/{knowledgeBaseFolderId}
        // ------------------------------------------------------------

        log.info("=== DELETE KnowledgeBaseFolder ===");

        try {
            KNOWLEDGE_BASE_FOLDER_RESOURCE.deleteKnowledgeBaseFolder(kbFolderId);
            log.info("KnowledgeBaseFolder deleted.");
            return true;
        }
        catch (Exception e) {
            log.error("Erro ao remover o KnowledgeBaseFolder de ID " + kbFolderId, e);
            return false;
        }
    }

    private static boolean deleteKnowledgeBaseArticle(Long kbArticleId) {

        // -------------------------------------------------------------
        // DELETE /v1.0/knowledge-base-articles/{knowledgeBaseArticleId}
        // -------------------------------------------------------------

        log.info("=== DELETE KnowledgeBaseArticle ===");

        try {
            KNOWLEDGE_BASE_ARTICLE_RESOURCE.deleteKnowledgeBaseArticle(kbArticleId);
            log.info("KnowledgeBaseArticle deleted.");
            return true;
        }
        catch (Exception e) {
            log.error("Erro ao remover o KnowledgeBaseArticle de ID " + kbArticleId, e);
            return false;
        }
    }

}
```

## 2.3 Explicação por partes

### Parte 1 – Os `import`

```java
import com.liferay.headless.delivery.client.dto.v1_0.Document;
import com.liferay.headless.delivery.client.dto.v1_0.DocumentFolder;
import com.liferay.headless.delivery.client.dto.v1_0.KnowledgeBaseArticle;
import com.liferay.headless.delivery.client.dto.v1_0.KnowledgeBaseFolder;

import com.liferay.headless.delivery.client.resource.v1_0.DocumentResource;
...
```

Mesma ideia do primeiro PR: **DTOs** (`dto.v1_0`) carregam os dados e **Resources** (`resource.v1_0`) fazem as chamadas HTTP. A diferença está no pacote: agora é `com.liferay.headless.delivery.client`, e não `com.acme.headless.r3b2.client`.

Os demais imports têm outra origem:

| Import | De onde vem | Para que serve |
|---|---|---|
| `java.io.File` | Java (JDK) | Referenciar o PDF que será enviado |
| `java.util.HashMap` e `java.util.Map` | Java (JDK) | Montar o mapa de arquivos do upload |
| `com.liferay.portal.kernel.log.Log` e `LogFactoryUtil` | `portal-kernel` do Liferay | Logging |

### Parte 2 – As constantes

```java
private static final Log log = LogFactoryUtil.getLog(Test.class);
private static final long SITE_ID = 20117L;
private static final String USER = "test@liferay.com";
private static final String PASSWORD = "learn";
```

- `private static final` define uma **constante da classe**: só é visível dentro de `Test` (`private`), pertence à classe e não a um objeto (`static`) e não pode ser reatribuída (`final`).
- `LogFactoryUtil.getLog(Test.class)` cria o logger associado à classe `Test`, o que identifica de onde cada mensagem veio.
- `SITE_ID`, `USER` e `PASSWORD` centralizam os valores que dependem do ambiente. No primeiro PR, as credenciais apareciam repetidas; aqui ficam em um único lugar.

### Parte 3 – Os resources como constantes estáticas

```java
private static final DocumentFolderResource DOCUMENT_FOLDER_RESOURCE =
    DocumentFolderResource.builder()
        .authentication(USER, PASSWORD)
        .build();
```

É o **padrão Builder** (já explicado no primeiro PR), com duas mudanças:

1. Em vez de criar os resources dentro do `main`, eles são criados **uma única vez**, como constantes da classe. Assim, todos os métodos auxiliares os usam sem precisar recebê-los por parâmetro (no primeiro PR, `fooResource` era passado de método em método).
2. Há **quatro** resources, um para cada recurso da atividade.

Por serem `static final`, eles são criados quando a classe `Test` é **carregada**, antes do `main` começar. Os nomes em `MAIÚSCULAS_COM_UNDERSCORE` seguem a convenção Java para constantes.

### Parte 4 – O `main` e os IDs iniciados como `null`

```java
Long docFolderId = null;
Long docId = null;
Long kbFolderId = null;
Long kbArticleId = null;
```

Mesmo raciocínio do primeiro PR: os IDs são declarados **antes** do `try` e começam como `null`, para que o `finally` saiba **o que chegou a ser criado**. Usa-se `Long` (objeto), e não `long` (primitivo), porque só o objeto aceita `null`.

### Parte 5 – O `try { ... } finally { ... }` e a ordem de exclusão

```java
try {
    docFolderId = documentFolderCrud();
    docId = documentCrud(docFolderId);
    kbFolderId = knowledgeBaseFolderCrud();
    kbArticleId = knowledgeBaseArticleCrud(kbFolderId);
} finally {
    boolean kbArticleDeleted = (kbArticleId == null);
    if (kbArticleId != null) {
        kbArticleDeleted = deleteKnowledgeBaseArticle(kbArticleId);
    }
    if (kbFolderId != null && kbArticleDeleted) {
        deleteKnowledgeBaseFolder(kbFolderId);
    }

    boolean documentDeleted = (docId == null);
    if (docId != null) {
        documentDeleted = deleteDocument(docId);
    }
    if (docFolderId != null && documentDeleted) {
        deleteDocumentFolder(docFolderId);
    }
}
```

- Cada método de CRUD **devolve o ID** do que criou, e o `main` o guarda. Os IDs de pastas são passados adiante porque os itens (documento, artigo) são criados **dentro** delas.
- Não há `catch` no bloco principal: se uma operação falhar, a exceção continua subindo, mas o `finally` **sempre executa antes**. Cada método de exclusão trata a própria falha e devolve `true` ou `false`.
- A exclusão acontece na **ordem inversa da criação**. O que foi criado por último é apagado primeiro, e as pastas só são apagadas depois dos itens que estão dentro delas:

| Ordem de criação | Ordem de exclusão |
|---|---|
| 1. Pasta de documentos | 4. `deleteDocumentFolder` |
| 2. Documento | 3. `deleteDocument` |
| 3. Pasta da KB | 2. `deleteKnowledgeBaseFolder` |
| 4. Artigo da KB | 1. `deleteKnowledgeBaseArticle` |

- Cada `if (xxx != null)` garante que só se tente apagar o que realmente foi criado. Se o programa falhar, por exemplo, ao criar o documento, `docId` e os IDs seguintes continuam `null` e nada é apagado por engano.
- Se a exclusão de um artigo ou documento falhar, o resultado `false` impede que a pasta correspondente seja apagada. A outra cadeia de exclusões ainda pode continuar.

### Parte 6 – O padrão dos métodos de CRUD

Os quatro métodos de CRUD seguem **exatamente o mesmo roteiro**, o que facilita entendê-los de uma vez:

```
log.info("=== POST X ===")    → cria  e guarda o ID gerado pelo servidor
log.info("=== GET X ===")     → busca pelo ID
log.info("=== PUT X ===")     → substitui os dados
log.info("=== GET X ===")     → confere o resultado do PUT
log.info("=== PATCH X ===")   → altera só a descrição
log.info("=== GET X ===")     → confere o resultado do PATCH
return id;
```

Depois de cada alteração há um `GET` para **conferir o resultado**: o `GET` após o `PUT` mostra os campos substituídos, e o `GET` após o `PATCH` mostra que **só a `description` mudou**.

Cada bloco é precedido por um comentário com o **verbo e o caminho** do endpoint (por exemplo, `PUT /v1.0/document-folders/{documentFolderId}`), o que serve de guia para quem lê o código.

### Parte 7 – `documentFolderCrud` e `knowledgeBaseFolderCrud`

```java
DocumentFolder newDocumentFolder = new DocumentFolder();
newDocumentFolder.setName("DocFolder");
newDocumentFolder.setDescription("DocFolder Description");

DocumentFolder createdDocumentFolder =
    DOCUMENT_FOLDER_RESOURCE.postSiteDocumentFolder(SITE_ID, newDocumentFolder);

Long docFolderId = createdDocumentFolder.getId();
```

| # | Linha | O que acontece |
|---|---|---|
| 1 | `new DocumentFolder()` | Cria um DTO vazio |
| 2 | `setName` / `setDescription` | Preenche os campos que serão enviados |
| 3 | `postSiteDocumentFolder(SITE_ID, ...)` | Faz `POST /sites/{siteId}/document-folders`: o client JAR converte o DTO em JSON, envia, e converte a resposta em um **novo** `DocumentFolder` |
| 4 | `getId()` | O servidor gerou o `id`; guardamos para as próximas operações |

O `knowledgeBaseFolderCrud` é igual, trocando o DTO e o método (`postSiteKnowledgeBaseFolder`).

Repare em uma diferença sutil: no `PUT` e no `PATCH` de pastas, a resposta do método é concatenada à mensagem de log (`log.debug("Response: " + ...putDocumentFolder(...))`). O método devolve o objeto atualizado, que é aproveitado para ser exibido.

### Parte 8 – `documentCrud`: o upload do PDF

```java
Document newDocument = new Document();
newDocument.setTitle("Doc");
newDocument.setDescription("Doc Description");

File file = new File("/home/me/Downloads/Westminster_Confession.pdf");

Map<String, File> files = new HashMap<>();
files.put("file", file);

Document createdDocument = DOCUMENT_RESOURCE.postDocumentFolderDocument(
    docFolderId,   // Id da pasta em que o documento será criado
    newDocument,   // Metadados do documento
    files);        // Arquivo a ser enviado
```

É a parte mais nova da atividade (veja [1.6](#16-upload-de-arquivos-multipartform-data-e-mapstring-file)):

- Os **metadados** vão no DTO (`title` e `description`).
- O **arquivo** vai no `Map<String, File>`, sob a chave `"file"`.
- `postDocumentFolderDocument(docFolderId, ...)` é o método para criar um documento **dentro de uma pasta** (`POST /document-folders/{id}/documents`).

> O caminho `/home/me/Downloads/Westminster_Confession.pdf` é do computador de quem escreveu o código. **Troque** pelo caminho de um PDF que exista no seu.

No `PUT` e no `PATCH`:

```java
DOCUMENT_RESOURCE.putDocument(docId, changedDocument, Map.of());
DOCUMENT_RESOURCE.patchDocument(docId, partialChangeDoc, Map.of());
```

O terceiro argumento é o mapa de arquivos. Como queremos mudar **apenas os metadados**, passamos um mapa vazio (`Map.of()`), sem arquivo novo.

### Parte 9 – `knowledgeBaseArticleCrud`

Segue o mesmo roteiro, com duas particularidades:

- O artigo é criado dentro de uma pasta da KB: `postKnowledgeBaseFolderKnowledgeBaseArticle(kbFolderId, article)`.
- Além de `title` e `description`, ele tem o `articleBody`, que é **HTML** (`"<p>KBArticle Body.</p>"`).

```java
newKnowledgeBaseArticle.setTitle("KBArticle");
newKnowledgeBaseArticle.setDescription("KBArticle Description");
newKnowledgeBaseArticle.setArticleBody("<p>KBArticle Body.</p>");
```

### Parte 10 – Os métodos de exclusão

```java
private static boolean deleteDocumentFolder(Long docFolderId) {
    log.info("=== DELETE DocumentFolder ===");

    try {
        DOCUMENT_FOLDER_RESOURCE.deleteDocumentFolder(docFolderId);
        log.info("DocumentFolder deleted.");
        return true;
    }
    catch (Exception e) {
        log.error("Erro ao remover o DocumentFolder de ID " + docFolderId, e);
        return false;
    }
}
```

Os quatro seguem o mesmo padrão: registram o início, tentam chamar o `deleteXxx` do resource e registram a conclusão. Se ocorrer uma exceção, registram o erro com `log.error(...)` e devolvem `false`; assim, a falha de uma exclusão não interrompe as demais operações independentes do `finally`.

### Parte 11 – Como o log aparece

O código usa dois tipos de chamada:

```java
log.info("=== POST DocumentFolder ===");                      // marcos da execução
log.debug("Created DocumentFolder: " + createdDocumentFolder); // conteúdo das respostas
```

O `Log` do Liferay recebe a mensagem como um objeto; por isso, o texto e o objeto são concatenados antes de chamar `debug`. O nível `debug` pode estar desligado, então essas linhas talvez não apareçam na saída.

## 2.4 Como compilar e executar

O código **não é um projeto Gradle**: é um arquivo `.java` solto que usa JARs. O passo a passo:

**1. Baixe o client JAR da Headless Delivery**

Siga o passo a passo de [1.3](#13-como-obter-o-client-jar-da-headless-delivery) e baixe o `com.liferay.headless.delivery.client-<versão>.jar` (a versão do seu Liferay).

**2. Reúna os JARs em uma pasta**

```bash
mkdir not-a-gradle-project && cd not-a-gradle-project
mkdir libs
```

Coloque em `libs/` o client JAR da Headless Delivery e o `portal-kernel.jar` (junto com as dependências que ele exigir no seu ambiente, caso o compilador ou a execução reclamem de classes ausentes).

**3. Ajuste o código ao seu ambiente**

No `Test.java`, altere:

- `SITE_ID` para o ID do seu site;
- `USER` e `PASSWORD`, se forem diferentes;
- o caminho do PDF em `documentCrud`.

**4. Compile, informando os JARs no classpath**

```bash
javac -classpath .:libs/\* Test.java
```

**5. Execute**

```bash
java -classpath .:libs/* Test
```

> No **Windows**, o separador do classpath é `;` em vez de `:` (ex.: `-classpath ".;libs\*"`).

**O que você deve ver no terminal**, em ordem: os marcos `=== POST DocumentFolder ===`, `=== GET ... ===`, `=== PUT ... ===` e `=== PATCH ... ===` de cada um dos quatro recursos, seguidos das mensagens de exclusão bem-sucedida ou de erro emitidas durante o `finally`. Se a exclusão de um documento ou artigo falhar, a pasta que o contém será preservada. As linhas de `debug` (conteúdo das respostas) só aparecem se esse nível estiver habilitado na configuração de log.

## 2.5 Pontos de melhoria

Depois que tudo funcionar, estes pontos deixam o código mais robusto e profissional. Eles estão em ordem aproximada de importância.

### 1. Exibir as respostas dos GETs

Como os `GET` servem para conferir o resultado, você pode registrá-los em `info` para que apareçam quando esse nível estiver habilitado:

```java
log.info(DOCUMENT_FOLDER_RESOURCE.getDocumentFolder(docFolderId));
```

Uma alternativa, para um programa solto fora do Liferay, é usar `System.out.println` (como no primeiro PR) ou `java.util.logging`, que dispensam o `portal-kernel.jar`.

### 2. Não deixar o caminho do PDF fixo

`/home/me/Downloads/Westminster_Confession.pdf` só existe no computador de quem escreveu o código. Uma opção é receber o caminho pela linha de comando:

```java
File file = new File(args[0]);

if (!file.isFile()) {
    throw new IllegalArgumentException("Arquivo não encontrado: " + file);
}
```

Para isso, o `main` precisaria repassar `args[0]` para `documentCrud`. Validar com `isFile()` dá um erro claro **antes** da requisição, em vez de uma exceção obscura durante o upload.

### 3. Evitar nomes fixos que colidem em reexecuções

Se uma execução falhar no meio e a limpeza não completar, pastas ou itens de teste podem permanecer. Na execução seguinte, tentar criar recursos com os mesmos nomes no mesmo local pode causar conflito. Uma forma simples de evitar isso é acrescentar um sufixo único aos nomes usados no teste:

```java
String suffix = String.valueOf(System.currentTimeMillis());

newDocumentFolder.setName("DocFolder " + suffix);
```

### 4. Validar, não só registrar

Os `GET` após o `PUT` e o `PATCH` apenas **imprimem** o resultado. Quem executa precisa olhar e concluir. Uma melhoria é devolver o objeto e **comparar** o esperado:

```java
DocumentFolder folder = DOCUMENT_FOLDER_RESOURCE.getDocumentFolder(docFolderId);

if (!"Updated DocFolder".equals(folder.getName())) {
    throw new IllegalStateException("O PATCH não deveria ter alterado o name");
}
```

Dessa forma, o programa passa a **testar** de fato o comportamento do `PUT` e do `PATCH`.

### 5. Credenciais, `SITE_ID` e configuração fora do código

Para o ambiente local de treinamento, constantes são aceitáveis. Em projetos reais, **nunca** versione senhas: leia de variáveis de ambiente (`System.getenv("LIFERAY_PASSWORD")`) ou de argumentos. O mesmo vale para o `SITE_ID`, que muda de instalação para instalação.

### 6. Criar os resources dentro do `main` (ou de um método)

Os resources são criados em inicializadores estáticos. Se a criação falhar, o Java lança um `ExceptionInInitializerError`, que é menos claro do que um erro dentro do `main`. Para um exercício, não é um problema, mas criar os resources no início do `main` (como no primeiro PR) deixa o erro mais fácil de entender.


## 2.6 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `package com.liferay.headless.delivery.client... does not exist` | O client JAR não está no classpath | Coloque o JAR em `libs/` e confira o `-classpath` |
| `package com.liferay.portal.kernel.log does not exist` | O `portal-kernel.jar` não está no classpath | Adicione-o ao `-classpath` ou troque o logging (veja [2.5](#25-pontos-de-melhoria), ponto 1) |
| `NoClassDefFoundError` ao executar | Falta uma dependência do `portal-kernel.jar` ou o classpath da execução está diferente do da compilação | Use o mesmo `-classpath` no `javac` e no `java` e adicione as dependências ausentes |
| `cannot find symbol ... Map.of` | JDK anterior ao Java 9 | Use JDK 9+ ou `Collections.emptyMap()` |
| `class Test is public, should be declared in a file named Test.java` | O nome do arquivo não bate com o da classe | Salve como `Test.java` |
| `FileNotFoundException` / `NoSuchFileException` | O caminho do PDF não existe no seu computador | Ajuste o caminho em `documentCrud` |
| `HTTP 401` (Unauthorized) | Usuário ou senha incorretos | Confira `USER` e `PASSWORD` |
| `HTTP 404` (Not Found) | `SITE_ID` inexistente, ID de recurso inválido ou versão do client incompatível | Confira o `SITE_ID` no seu Liferay e a versão do client JAR |
| `HTTP 403` (Forbidden) | O usuário não tem permissão no site | Use um usuário com permissão (administrador do ambiente de treinamento) |
| `HTTP 4xx` ao criar uma pasta | Já existe uma pasta com o mesmo nome | Apague a pasta antiga ou use nomes únicos (veja [2.5](#25-pontos-de-melhoria), ponto 3) |
| `Connection refused` | Liferay desligado ou porta errada | Suba o Liferay e confira `localhost:8080` |
| Nenhuma linha de `debug` aparece | O nível `debug` está desligado | Use `info`, ou habilite o nível na configuração de log |
| Registros de teste ficaram no Liferay | Uma exclusão falhou ou não foi possível apagar uma pasta que ainda contém um item | Apague manualmente (widget ou API Explorer) e confira os erros registrados no log |

### Para praticar

1. Use o **API Explorer** para fazer à mão o CRUD dos quatro recursos antes de rodar o programa.
2. Se necessário, ajuste o nível de log para conseguir ver as respostas das operações.
3. Receba o caminho do PDF por `args[0]` e valide com `isFile()`.
4. Provoque uma falha de exclusão para observar como o `finally` continua limpando os recursos independentes.
5. Faça o `PATCH` do documento mudar apenas a `description` e **valide** por código que o `title` continuou o do `PUT`.

---
