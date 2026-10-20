# Guia: Atividade de REST Builder (Quarto PR: OSGi Resources reais da Headless Delivery)

Este guia acompanha o **quarto PR** da sequência. Em vez de trabalhar com recursos fictícios como `Foo` e `Goo`, o código usa **recursos reais da API Headless Delivery do próprio Liferay** para operar com Documentos e Media e com a Base de Conhecimento.

O portlet `R3B2Portlet` executa operações de criação, consulta, atualização e exclusão sobre quatro recursos:

- `DocumentFolder`: pastas de Documentos e Media;
- `Document`: documentos e arquivos;
- `KnowledgeBaseFolder`: pastas da Base de Conhecimento;
- `KnowledgeBaseArticle`: artigos da Base de Conhecimento.

O guia mantém a parte teórica mais curta, pois conceitos como REST, CRUD, OSGi, `@Reference`, Factory, Builder e portlets já foram abordados anteriormente. O foco agora é entender **os recursos reais, os métodos que eles disponibilizam e o upload multipart de arquivos**.

| Etapa | Conteúdo |
|---|---|
| **Etapa 1 – Contexto necessário** | O que muda neste PR, os quatro recursos, suas relações e o envio de arquivos. |
| **Etapa 2 – O código** | Código completo, explicação por blocos, implantação, execução, melhorias e erros comuns. |

---

## Sumário

- [Etapa 1 – Contexto necessário](#etapa-1--contexto-necessário)
  - [1.1 O que muda neste PR](#11-o-que-muda-neste-pr)
  - [1.2 Os quatro recursos reais da Headless Delivery](#12-os-quatro-recursos-reais-da-headless-delivery)
  - [1.3 Métodos Java e endpoints correspondentes](#13-métodos-java-e-endpoints-correspondentes)
  - [1.4 Resources do servidor e contexto do usuário](#14-resources-do-servidor-e-contexto-do-usuário)
  - [1.5 Upload de arquivos com `MultipartBody`](#15-upload-de-arquivos-com-multipartbody)
  - [1.6 Pré-requisitos](#16-pré-requisitos)
- [Recursos adicionais](#recursos-adicionais)
- [Etapa 2 – O código](#etapa-2--o-código)
  - [2.1 Visão geral do fluxo](#21-visão-geral-do-fluxo)
  - [2.2 Código completo](#22-código-completo)
  - [2.3 Explicação por partes](#23-explicação-por-partes)
  - [2.4 Criar o módulo, implantar e executar](#24-criar-o-módulo-implantar-e-executar)
  - [2.5 Pontos de atenção e melhorias](#25-pontos-de-atenção-e-melhorias)
  - [2.6 Erros comuns](#26-erros-comuns)
  - [Para praticar](#para-praticar)

---

# Etapa 1 – Contexto necessário

## 1.1 O que muda neste PR

No PR anterior, o portlet consumia `FooResource` e `GooResource`, recursos da API personalizada `headless-r3b2`. Neste PR, o portlet continua usando OSGi Resources no lado do servidor, mas passa a consumir serviços que já fazem parte do Liferay:

```text
PR anterior                         Quarto PR

headless-r3b2                        Headless Delivery
FooResource / GooResource            DocumentFolderResource
                                    DocumentResource
                                    KnowledgeBaseFolderResource
                                    KnowledgeBaseArticleResource
```

**Este PR não define uma nova API Headless Delivery nem exige executar `buildREST` para esses quatro recursos.** A atividade consome serviços já fornecidos pelo Liferay. O módulo desenvolvido no exercício é o módulo web que contém o portlet.

## 1.2 Os quatro recursos reais da Headless Delivery

### Documents and Media

| Resource Java | O que representa | Operações deste exercício |
|---|---|---|
| `DocumentFolderResource` | Pastas de Documentos e Media | Criar pasta no site, listar pastas, consultar por ID, `PUT`, `PATCH` e `DELETE`. |
| `DocumentResource` | Documentos e arquivos | Enviar arquivo para uma pasta, listar os documentos da pasta, consultar por ID, substituir com `PUT`, atualizar metadados com `PATCH` e excluir. |

O documento é criado **dentro da pasta criada no início do fluxo**. Por isso, o ID retornado ao criar a pasta é passado para `postDocumentFolderDocument`.

### Knowledge Base

| Resource Java | O que representa | Operações deste exercício |
|---|---|---|
| `KnowledgeBaseFolderResource` | Pastas da Base de Conhecimento | Criar pasta no site, listar pastas, consultar por ID, `PUT`, `PATCH` e `DELETE`. |
| `KnowledgeBaseArticleResource` | Artigos da Base de Conhecimento | Criar artigo dentro de uma pasta, listar os artigos da pasta, consultar por ID, `PUT`, `PATCH` e `DELETE`. |

O artigo também é criado dentro da pasta criada anteriormente; o código passa o ID de `KnowledgeBaseFolder` para a operação de criação do artigo.

```text
Site atual do Liferay
│
├── DocumentFolder
│   └── Document
│       └── arquivo enviado (Document.txt)
│
└── KnowledgeBaseFolder
    └── KnowledgeBaseArticle
        └── conteúdo HTML do artigo
```

Este desenho representa a estrutura criada pela atividade. Cada objeto tem seu próprio ID. O portlet guarda os IDs retornados pelos `POST` para consultar, atualizar e excluir exatamente os objetos criados.

## 1.3 Métodos Java e endpoints correspondentes

Os métodos abaixo são métodos Java dos Resources do Liferay. Eles correspondem às operações disponibilizadas pela API HTTP, mas este portlet **chama os métodos diretamente**, sem montar URLs nem enviar requisições HTTP manualmente.

As rotas são apresentadas para ajudar a relacionar os métodos Java à API pública. Confirme a assinatura exata e os parâmetros na versão instalada, usando o API Explorer em `http://localhost:8080/o/api`.

| Resource | Método Java | Rota REST correspondente |
|---|---|---|
| `DocumentFolderResource` | `postSiteDocumentFolder(siteId, folder)` | `POST /sites/{siteId}/document-folders` |
| `DocumentFolderResource` | `getSiteDocumentFoldersPage(...)` | `GET /sites/{siteId}/document-folders` |
| `DocumentFolderResource` | `getDocumentFolder(id)` | `GET /document-folders/{documentFolderId}` |
| `DocumentFolderResource` | `putDocumentFolder(id, folder)` | `PUT /document-folders/{documentFolderId}` |
| `DocumentFolderResource` | `patchDocumentFolder(id, folder)` | `PATCH /document-folders/{documentFolderId}` |
| `DocumentFolderResource` | `deleteDocumentFolder(id)` | `DELETE /document-folders/{documentFolderId}` |
| `DocumentResource` | `postDocumentFolderDocument(folderId, body)` | `POST /document-folders/{documentFolderId}/documents` |
| `DocumentResource` | `getDocumentFolderDocumentsPage(...)` | `GET /document-folders/{documentFolderId}/documents` |
| `DocumentResource` | `getDocument(id)` | `GET /documents/{documentId}` |
| `DocumentResource` | `putDocument(id, body)` | `PUT /documents/{documentId}` |
| `DocumentResource` | `patchDocument(id, body)` | `PATCH /documents/{documentId}` |
| `DocumentResource` | `deleteDocument(id)` | `DELETE /documents/{documentId}` |
| `KnowledgeBaseFolderResource` | `postSiteKnowledgeBaseFolder(siteId, folder)` | `POST /sites/{siteId}/knowledge-base-folders` |
| `KnowledgeBaseFolderResource` | `getSiteKnowledgeBaseFoldersPage(...)` | `GET /sites/{siteId}/knowledge-base-folders` |
| `KnowledgeBaseFolderResource` | `getKnowledgeBaseFolder(id)` | `GET /knowledge-base-folders/{knowledgeBaseFolderId}` |
| `KnowledgeBaseFolderResource` | `putKnowledgeBaseFolder(id, folder)` | `PUT /knowledge-base-folders/{knowledgeBaseFolderId}` |
| `KnowledgeBaseFolderResource` | `patchKnowledgeBaseFolder(id, folder)` | `PATCH /knowledge-base-folders/{knowledgeBaseFolderId}` |
| `KnowledgeBaseFolderResource` | `deleteKnowledgeBaseFolder(id)` | `DELETE /knowledge-base-folders/{knowledgeBaseFolderId}` |
| `KnowledgeBaseArticleResource` | `postKnowledgeBaseFolderKnowledgeBaseArticle(folderId, article)` | `POST /knowledge-base-folders/{knowledgeBaseFolderId}/knowledge-base-articles` |
| `KnowledgeBaseArticleResource` | `getKnowledgeBaseFolderKnowledgeBaseArticlesPage(...)` | `GET /knowledge-base-folders/{knowledgeBaseFolderId}/knowledge-base-articles` |
| `KnowledgeBaseArticleResource` | `getKnowledgeBaseArticle(id)` | `GET /knowledge-base-articles/{knowledgeBaseArticleId}` |
| `KnowledgeBaseArticleResource` | `putKnowledgeBaseArticle(id, article)` | `PUT /knowledge-base-articles/{knowledgeBaseArticleId}` |
| `KnowledgeBaseArticleResource` | `patchKnowledgeBaseArticle(id, article)` | `PATCH /knowledge-base-articles/{knowledgeBaseArticleId}` |
| `KnowledgeBaseArticleResource` | `deleteKnowledgeBaseArticle(id)` | `DELETE /knowledge-base-articles/{knowledgeBaseArticleId}` |

As chamadas de listagem usam `Pagination.of(1, 20)`, solicitando a primeira página com até 20 resultados. Os argumentos `null` em algumas chamadas representam parâmetros opcionais que o exemplo não utiliza, como opções de busca, filtros ou ordenação.

Há duas operações com nomes parecidos, mas elas representam relações diferentes. `postKnowledgeBaseArticleKnowledgeBaseArticle(parentKnowledgeBaseArticleId, article)` cria um artigo filho de outro artigo; `postKnowledgeBaseFolderKnowledgeBaseArticle(knowledgeBaseFolderId, article)` cria um artigo dentro de uma pasta. Como este exercício passa `kbFolderId`, deve usar a segunda operação. A chamada do arquivo Java fornecido foi ajustada para `postKnowledgeBaseFolderKnowledgeBaseArticle(...)`; confirme a assinatura no API Explorer da versão instalada.

## 1.4 Resources do servidor e contexto do usuário

Observe os imports:

```java
import com.liferay.headless.delivery.dto.v1_0.Document;
import com.liferay.headless.delivery.resource.v1_0.DocumentResource;
```

Eles vêm da API de servidor do Liferay, e não dos pacotes do client JAR. O client JAR é apropriado quando um programa externo precisa chamar a API por HTTP. Neste exercício, o portlet roda dentro do portal e recebe as Factories por `@Reference`.

O fluxo segue o mesmo princípio estudado no terceiro PR:

1. O OSGi injeta a Factory do Resource.
2. O portlet chama `create()` para obter um Builder.
3. O Builder recebe o usuário atual por `.user(themeDisplay.getUser())`.
4. `.build()` constrói o Resource para esse contexto.
5. O portlet chama os métodos Java do Resource.

O contexto do usuário é importante: as operações continuam sujeitas às permissões do Liferay. A chamada local não ignora autorização nem concede permissões adicionais.

## 1.5 Upload de arquivos com `MultipartBody`

Para criar ou substituir um documento, o código passa `MultipartBody` em vez de enviar somente um DTO `Document`:

```java
MultipartBody.of(
    mapaDeArquivos,
    objectMapperProvider,
    camposDeTexto
)
```

| Argumento | Função no exemplo |
|---|---|
| `Map<String, BinaryFile>` | Fornece o arquivo no campo multipart chamado `file`. |
| `MultipartBody.ObjectMapperProvider` | Fornece um `ObjectMapper`, exigido pelo construtor usado de `MultipartBody`. |
| `Map<String, String>` | Fornece campos de texto que acompanham o arquivo, como `title` ou `description`. |

O `BinaryFile` recebe o tipo MIME (`text/plain`), o nome do arquivo, o `InputStream` que disponibiliza os bytes e o tamanho do conteúdo.

```java
R3B2Portlet.class.getResourceAsStream("/Document.txt")
```

Essa chamada procura o arquivo no classpath. Portanto, `Document.txt` e `UpdatedDocument.txt` precisam existir nos recursos do módulo — normalmente em `src/main/resources/`. Se o arquivo não for encontrado, `getResourceAsStream` pode retornar `null`. O tamanho passado para `BinaryFile` também precisa corresponder ao tamanho real do arquivo em bytes.

Neste exemplo, o `POST` envia o primeiro arquivo; o `PUT` envia um arquivo diferente para substituir o documento; e o `PATCH` envia um mapa de arquivos vazio, alterando apenas a `description`. Isso permite observar a diferença entre substituir o documento e atualizar somente um campo.

## 1.6 Pré-requisitos

Antes de executar a atividade, confirme:

- O Liferay está em execução e você consegue entrar no portal.
- A Headless Delivery aparece no API Explorer (`http://localhost:8080/o/api`).
- O módulo web possui dependências para os pacotes `com.liferay.headless.delivery.*` e `com.liferay.portal.vulcan.*`.
- O usuário que abrir a página possui permissão para criar, atualizar e excluir pastas, documentos e artigos.
- `Document.txt` e `UpdatedDocument.txt` estão em `src/main/resources/` e seus tamanhos correspondem aos valores declarados no código (`5L` e `6L`), ou esses valores foram ajustados.
- O módulo web será implantado no mesmo Liferay que fornece os Resources da Headless Delivery.

> **Atenção:** o código tenta excluir os objetos no `finally`, mas uma falha de permissão ou de rede/serviço pode impedir a limpeza. Execute em ambiente de desenvolvimento e confira os logs.

---

## Recursos adicionais

- [Document API Basics — Liferay Learn](https://learn.liferay.com/en/w/dxp/integration/headless-apis/digital-asset-management-apis/document-api-basics): upload, consulta, atualização e exclusão de documentos; inclui exemplos de `DocumentResource` e pastas.
- [Knowledge Base API Basics — Liferay Learn](https://learn.liferay.com/w/dxp/integration/headless-apis/content-management-apis/knowledge-base-api-basics): criação e consulta de artigos e operações por ID.
- [API Explorer do seu ambiente](http://localhost:8080/o/api): confira os endpoints, schemas e operações presentes na versão instalada.
- [Portlet](../theory/portlet.md), [REST](../theory/rest.md), [HTTP](../theory/http.md), [DTOs e Schemas](../theory/dtos-and-schemas.md), [API](../theory/api.md) e [JSON](../theory/json.md): conceitos já trabalhados nos guias anteriores.

---

# Etapa 2 – O código

## 2.1 Visão geral do fluxo

O método `doView` executa quatro blocos. Em cada bloco, o código cria um objeto, lista a coleção, consulta o objeto por ID, faz `PUT`, consulta novamente, faz `PATCH` e consulta uma última vez. O `finally` tenta excluir os objetos criados, na ordem inversa das dependências.

```text
Recuperar ThemeDisplay e siteId
             |
             v
Criar os quatro Resources com o usuário atual
             |
             +--> DocumentFolder: POST / GET coleção / GET ID / PUT / GET / PATCH / GET
             |       └── Document: POST multipart / GET coleção / GET ID / PUT multipart / GET / PATCH / GET
             |
             +--> KnowledgeBaseFolder: POST / GET coleção / GET ID / PUT / GET / PATCH / GET
                     └── KnowledgeBaseArticle: POST / GET coleção / GET ID / PUT / GET / PATCH / GET
             |
             v
finally: excluir artigo, pasta KB, documento e pasta de documentos
```

Os IDs são guardados em variáveis `Long` porque serão necessários nas operações seguintes. Elas começam como `null`, para que o `finally` só tente excluir um objeto cuja criação tenha retornado um ID.

**Atenção ao ciclo de vida:** o Liferay chama `doView` quando o portlet é renderizado. Como todas as operações estão nesse método, abrir ou atualizar a página executa o fluxo novamente. Isso é proposital para a atividade, não um padrão de aplicação recomendado.

## 2.2 Código completo

O código a seguir é a implementação fornecida para esta atividade. Salve-o como `R3B2Portlet.java`, no pacote `com.acme.headless.r3b2.web.internal` do módulo web.

```java
package com.acme.headless.r3b2.web.internal;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.liferay.headless.delivery.dto.v1_0.Document;
import com.liferay.headless.delivery.dto.v1_0.DocumentFolder;
import com.liferay.headless.delivery.dto.v1_0.KnowledgeBaseArticle;
import com.liferay.headless.delivery.dto.v1_0.KnowledgeBaseFolder;

import com.liferay.headless.delivery.resource.v1_0.DocumentFolderResource;
import com.liferay.headless.delivery.resource.v1_0.DocumentResource;
import com.liferay.headless.delivery.resource.v1_0.KnowledgeBaseArticleResource;
import com.liferay.headless.delivery.resource.v1_0.KnowledgeBaseFolderResource;

import com.liferay.portal.kernel.theme.ThemeDisplay;
import com.liferay.portal.kernel.util.WebKeys;
import com.liferay.portal.kernel.log.Log;
import com.liferay.portal.kernel.log.LogFactoryUtil;
import com.liferay.portal.vulcan.multipart.BinaryFile;
import com.liferay.portal.vulcan.multipart.MultipartBody;
import com.liferay.portal.vulcan.pagination.Page;
import com.liferay.portal.vulcan.pagination.Pagination;

import java.io.IOException;
import java.util.HashMap;

import javax.portlet.GenericPortlet;
import javax.portlet.Portlet;
import javax.portlet.RenderRequest;
import javax.portlet.RenderResponse;

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

		ThemeDisplay themeDisplay = (ThemeDisplay) renderRequest.getAttribute(
				WebKeys.THEME_DISPLAY);

		long siteId = themeDisplay.getScopeGroupId();

		// Não exige email/senha pois o portlet roda dentro do portal Liferay.
		// O usuário já está autenticado e seu "status" é obtido do ThemeDisplay.
		// Diferente da API REST externa que faz chamadas HTTP e precisa credenciais
		// explícitas.

		DocumentFolderResource.Builder documentFolderResourceBuilder = _documentFolderResourceFactory.create();
		DocumentFolderResource documentFolderResource = documentFolderResourceBuilder.user(
				themeDisplay.getUser())
				.build();

		DocumentResource.Builder documentResourceBuilder = _documentResourceFactory.create();
		DocumentResource documentResource = documentResourceBuilder.user(
				themeDisplay.getUser())
				.build();

		KnowledgeBaseFolderResource.Builder knowledgeBaseFolderBuilder = _knowledgeBaseFolderResourceFactory.create();
		KnowledgeBaseFolderResource knowledgeBaseFolderResource = knowledgeBaseFolderBuilder.user(
				themeDisplay.getUser())
				.build();

		KnowledgeBaseArticleResource.Builder knowledgeBaseArticleBuilder = _knowledgeBaseArticleResourceFactory
				.create();
		KnowledgeBaseArticleResource knowledgeBaseArticleResource = knowledgeBaseArticleBuilder.user(
				themeDisplay.getUser())
				.build();

		Long docFolderId = null;
		Long docId = null;

		Long kbFolderId = null;
		Long kbArticleId = null;

		try {

			/* --- DocumentFolder --- */

			docFolderId = postSiteDocumentFolder(siteId, documentFolderResource);
			getSiteDocumentFoldersPage(siteId, documentFolderResource);
			getDocumentFolder(documentFolderResource, docFolderId);
			putDocumentFolder(documentFolderResource, docFolderId);
			getDocumentFolder(documentFolderResource, docFolderId);
			patchDocumentFolder(documentFolderResource, docFolderId);
			getDocumentFolder(documentFolderResource, docFolderId);

			/* --- Document --- */

			docId = postDocumentFolderDocument(documentResource, docFolderId);
			getDocumentFolderDocumentsPage(documentResource, docFolderId);
			getDocument(documentResource, docId);
			putDocument(documentResource, docId);
			getDocument(documentResource, docId);
			patchDocument(documentResource, docId);
			getDocument(documentResource, docId);

			/* --- KnowledgeBaseFolder --- */

			kbFolderId = postSiteKnowledgeBaseFolder(siteId, knowledgeBaseFolderResource);
			getSiteKnowledgeBaseFoldersPage(siteId, knowledgeBaseFolderResource);
			getKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);
			putKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);
			getKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);
			patchKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);
			getKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);

			/* --- KnowledgeBaseArticle --- */

			kbArticleId = postKnowledgeBaseFolderKnowledgeBaseArticle(knowledgeBaseArticleResource, kbFolderId);
			getKnowledgeBaseFolderKnowledgeBaseArticlesPage(knowledgeBaseArticleResource, kbFolderId);
			getKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);
			putKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);
			getKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);
			patchKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);
			getKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);

		} catch (Exception e) {
			_log.error("Error: ", e);

		} finally {

			if (kbArticleId != null) {
				try {
					deleteKnowledgeBaseArticle(knowledgeBaseArticleResource, kbArticleId);
				} catch (Exception e) {
					_log.error("Error deleting KnowledgeBaseArticle: ", e);
				}
			}

			if (kbFolderId != null) {
				try {
					deleteKnowledgeBaseFolder(knowledgeBaseFolderResource, kbFolderId);
				} catch (Exception e) {
					_log.error("Error deleting KnowledgeBaseFolder: ", e);
				}
			}

			if (docId != null) {
				try {
					deleteDocument(documentResource, docId);
				} catch (Exception e) {
					_log.error("Error deleting Document: ", e);
				}
			}

			if (docFolderId != null) {
				try {
					deleteDocumentFolder(documentFolderResource, docFolderId);
				} catch (Exception e) {
					_log.error("Error deleting DocumentFolder: ", e);
				}
			}
		}
	}

	// =========================================================
	// DOCUMENT FOLDER
	// =========================================================

	private Long postSiteDocumentFolder(long siteId, DocumentFolderResource documentFolderResource) throws Exception {

		_log.info("=== POST Document Folder");

		DocumentFolder newDocumentFolder = new DocumentFolder();

		newDocumentFolder.setName("DocFolder");
		newDocumentFolder.setDescription("Posted DocFolder from R3B2Portlet");

		DocumentFolder createdDocumentFolder = documentFolderResource.postSiteDocumentFolder(siteId, newDocumentFolder);

		_log.debug("Created DocumentFolder: " + createdDocumentFolder);

		return createdDocumentFolder.getId();
	}

	private void getSiteDocumentFoldersPage(Long siteId, DocumentFolderResource documentFolderResource)
			throws Exception {

		_log.info("=== GET Document Folders Page ===");

		Page<DocumentFolder> documentFolderPage = documentFolderResource.getSiteDocumentFoldersPage(siteId,
				null,
				null,
				null,
				null,
				Pagination.of(1, 20),
				null);

		_log.debug("GET Document Folders Page: " + documentFolderPage);
	}

	private void getDocumentFolder(DocumentFolderResource documentFolderResource, Long docFolderId) throws Exception {

		_log.info("=== GET Document Folder ===");

		DocumentFolder documentFolder = documentFolderResource.getDocumentFolder(docFolderId);

		_log.debug("GET Document Folder: " + documentFolder);
	}

	private void putDocumentFolder(DocumentFolderResource documentFolderResource, Long docFolderId) throws Exception {

		_log.info("=== PUT Document Folder ===");

		DocumentFolder updatedDocumentFolder = new DocumentFolder();

		updatedDocumentFolder.setName("Updated DocFolder");
		updatedDocumentFolder.setDescription("Updated DocumentFolder from R3B2Portlet");

		DocumentFolder documentFolder = documentFolderResource.putDocumentFolder(docFolderId, updatedDocumentFolder);

		_log.debug("Updated Document Folder: " + documentFolder);
	}

	private void patchDocumentFolder(DocumentFolderResource documentFolderResource, Long docFolderId) throws Exception {

		_log.info("=== PATCH DocumentFolder ===");

		DocumentFolder patchedDocumentFolder = new DocumentFolder();

		patchedDocumentFolder.setDescription("Patched DocFolder Description from R3B2Portlet");

		DocumentFolder documentFolder = documentFolderResource.patchDocumentFolder(docFolderId, patchedDocumentFolder);

		_log.debug("Patched Document Folder: " + documentFolder);
	}

	private void deleteDocumentFolder(DocumentFolderResource documentFolderResource, Long docFolderId)
			throws Exception {

		_log.info("=== DELETE Document Folder ===");

		documentFolderResource.deleteDocumentFolder(docFolderId);
		_log.info("Deleted Document Folder with ID: " + docFolderId);

	}

	// =========================================================
	// DOCUMENT
	// =========================================================

	private Long postDocumentFolderDocument(DocumentResource documentResource, Long docFolderId) throws Exception {

		_log.info("=== POST Document Folder Document ===");

		Document createdDocument = documentResource.postDocumentFolderDocument(
				docFolderId,
				MultipartBody.of(

						new HashMap<String, BinaryFile>() {
							{
								put("file", new BinaryFile(
										// Define o tipo do arquivo (texto simples nesse caso)
										"text/plain",
										// Declara o nome do arquivo a ser enviado
										"Document.txt",
										// Fornece acesso ao conteúdo do arquivo por meio de um InputStream*
										// * InputStream permite ler os dados do arquivo como um fluxo de bytes
										R3B2Portlet.class.getResourceAsStream("/Document.txt"),
										// Define o tamanho do arquivo em bytes
										5L));
							}
						},

						// ObjectMapper é utilizado como forma de serialização e desserialização de dados
						// Ele pode ser usado para converter objetos Java para JSON e JSON para objetos Java
						// Nesse caso, ele não é utilizado diretamente na operação, mas precisa ser declarado
						// por ser um parâmetro obrigatório da classe MultipartBody
						new MultipartBody.ObjectMapperProvider() {
							@Override
							public ObjectMapper provide(Class<?> clazz) {
								return new ObjectMapper();
							}
						},
						// Define os campos de texto adicionais do documento
						// "title" representa o nome do campo e "Doc" representa seu valor
						new HashMap<String, String>() {
							{
								put("title", "Doc");
							}
						}));

		_log.debug("POST Document Folder Document: " + createdDocument);

		return createdDocument.getId();
	}

	private void getDocumentFolderDocumentsPage(DocumentResource documentResource, Long docFolderId) throws Exception {

		_log.info("=== GET Documents Page ===");

		Page<Document> documentsPage = documentResource.getDocumentFolderDocumentsPage(docFolderId,
			null,
			null,
			null,
			null,
			Pagination.of(1, 20), null);

		_log.debug("GET All Documents " + documentsPage);
	}

	private void getDocument(DocumentResource documentResource, Long docId) throws Exception {

		_log.info("=== GET Document ===");

		Document document = documentResource.getDocument(docId);
		_log.debug("GET Document: " + document);
	}

	private void putDocument(DocumentResource documentResource, Long docId) throws Exception {

		_log.info("=== PUT Document ===");

		Document updatedDocument = documentResource.putDocument(docId,
			MultipartBody.of(

						new HashMap<String, BinaryFile>() {
							{
								put("file", new BinaryFile(
										"text/plain",
										"UpdatedDocument.txt",
										R3B2Portlet.class.getResourceAsStream("/UpdatedDocument.txt"),
										6L));
							}
						},

						new MultipartBody.ObjectMapperProvider() {
							@Override
							public ObjectMapper provide(Class<?> clazz) {
								return new ObjectMapper();
							}
						},

						new HashMap<String, String>() {
							{
								put("title", "UpdatedDoc");
							}
						}));

		_log.debug("Updated Document " + updatedDocument);
	}

	private void patchDocument(DocumentResource documentResource, Long docId) throws Exception {

		_log.info("=== PATCH Document ===");

		Document patchedDocument = documentResource.patchDocument(docId, MultipartBody.of(

						// Nenhum novo arquivo é enviado nesta operação.
						new HashMap<String, BinaryFile>() {
							{

							}
						},

						new MultipartBody.ObjectMapperProvider() {
							@Override
							public ObjectMapper provide(Class<?> clazz) {
								return new ObjectMapper();
							}
						},

						new HashMap<String, String>() {
							{
								put("description", "PatchedDoc");
							}
						}));

		_log.debug("Patched Document " + patchedDocument);
	}

	private void deleteDocument(DocumentResource documentResource, Long docId) throws Exception {

		_log.info("=== DELETE Document ===");

		documentResource.deleteDocument(docId);
		_log.info("Deleted Document with ID: " + docId);
	}

	// =========================================================
	// KNOWLEDGE BASE FOLDER
	// =========================================================

	private Long postSiteKnowledgeBaseFolder(long siteId, KnowledgeBaseFolderResource knowledgeBaseFolderResource)
			throws Exception {

		_log.info("=== POST KnowledgeBase Folder ===");

		KnowledgeBaseFolder newKnowledgeBaseFolder = new KnowledgeBaseFolder();

		newKnowledgeBaseFolder.setName("KBFolder");
		newKnowledgeBaseFolder.setDescription("Posted KBFolder from R3B2Portlet");

		KnowledgeBaseFolder createdKnowledgeBaseFolder = knowledgeBaseFolderResource.postSiteKnowledgeBaseFolder(siteId,
				newKnowledgeBaseFolder);

		_log.debug("Created KnowledgeBase Folder: " + createdKnowledgeBaseFolder);

		return createdKnowledgeBaseFolder.getId();
	}

	private void getSiteKnowledgeBaseFoldersPage(long siteId, KnowledgeBaseFolderResource knowledgeBaseFolderResource)
			throws Exception {

		_log.info("=== GET KnowledgeBase Folders Page ===");

		Page<KnowledgeBaseFolder> knowledgeBaseFolderPage = knowledgeBaseFolderResource
				.getSiteKnowledgeBaseFoldersPage(siteId, Pagination.of(1, 20));

		_log.debug("GET KnowledgeBase Folders Page: " + knowledgeBaseFolderPage);

	}

	private void getKnowledgeBaseFolder(KnowledgeBaseFolderResource knowledgeBaseFolderResource, Long kbFolderId)
			throws Exception {

		_log.info("=== GET KnowledgeBase Folder ===");

		KnowledgeBaseFolder knowledgeBaseFolder = knowledgeBaseFolderResource.getKnowledgeBaseFolder(kbFolderId);
		_log.debug("GET KnowledgeBase Folder: " + knowledgeBaseFolder);
	}

	private void putKnowledgeBaseFolder(KnowledgeBaseFolderResource knowledgeBaseFolderResource, Long kbFolderId)
			throws Exception {

		_log.info("=== PUT KnowledgeBase Folder ===");

		KnowledgeBaseFolder updatedKBFolder = new KnowledgeBaseFolder();

		updatedKBFolder.setName("Updated KBFolder");
		updatedKBFolder.setDescription("Updated KBFolder from R3B2Portlet");

		KnowledgeBaseFolder knowledgeBaseFolder = knowledgeBaseFolderResource.putKnowledgeBaseFolder(kbFolderId,
				updatedKBFolder);
		_log.debug("Updated KnowledgeBase Folder: " + knowledgeBaseFolder);
	}

	private void patchKnowledgeBaseFolder(KnowledgeBaseFolderResource knowledgeBaseFolderResource, Long kbFolderId)
			throws Exception {

		_log.info("=== PATCH KnowledgeBase Folder ===");

		KnowledgeBaseFolder patchedKBFolder = new KnowledgeBaseFolder();

		patchedKBFolder.setDescription("Patched KBFolder from R3B2Portlet");

		KnowledgeBaseFolder knowledgeBaseFolder = knowledgeBaseFolderResource.patchKnowledgeBaseFolder(kbFolderId,
				patchedKBFolder);
		_log.debug("Patched KnowledgeBase Folder: " + knowledgeBaseFolder);
	}

	private void deleteKnowledgeBaseFolder(KnowledgeBaseFolderResource knowledgeBaseFolderResource, Long kbFolderId)
			throws Exception {

		_log.info("=== DELETE KnowledgeBase Folder ===");

		knowledgeBaseFolderResource.deleteKnowledgeBaseFolder(kbFolderId);
		_log.info("Deleted KnowledgeBase Folder with ID: " + kbFolderId);
	}

	// =========================================================
	// KNOWLEDGE BASE ARTICLE
	// =========================================================

	private Long postKnowledgeBaseFolderKnowledgeBaseArticle(KnowledgeBaseArticleResource knowledgeBaseArticleResource,
			Long kbFolderId) throws Exception {

		_log.info("=== POST KnowledgeBase Article ===");

		KnowledgeBaseArticle newKnowledgeBaseArticle = new KnowledgeBaseArticle();

		newKnowledgeBaseArticle.setTitle("KBArticle");
		newKnowledgeBaseArticle.setDescription("Posted KBFolder from R3B2Portlet");
		newKnowledgeBaseArticle.setArticleBody("<p>KBArticle Body.</p>");

		KnowledgeBaseArticle createdKBArticle = knowledgeBaseArticleResource
				.postKnowledgeBaseFolderKnowledgeBaseArticle(kbFolderId, newKnowledgeBaseArticle);

		_log.debug("Created KnowledgeBase Article " + createdKBArticle);
		return createdKBArticle.getId();
	}

	private void getKnowledgeBaseFolderKnowledgeBaseArticlesPage(
			KnowledgeBaseArticleResource knowledgeBaseArticleResource, Long kbFolderId) throws Exception {

		_log.info("=== GET KnowledgeBase Folder KnowledgeBase Articles Page ===");

		Page<KnowledgeBaseArticle> knowledgeBaseArticlePage = knowledgeBaseArticleResource
				.getKnowledgeBaseFolderKnowledgeBaseArticlesPage(kbFolderId, null,
						null,
						null,
						null,
						Pagination.of(1, 20),
						null);

		_log.debug("GET all KnowledgeBase Articles: " + knowledgeBaseArticlePage);
	}

	private void getKnowledgeBaseArticle(KnowledgeBaseArticleResource knowledgeBaseArticleResource, Long kbArticleId)
			throws Exception {

		_log.info("=== GET KnowledgeBase Article ===");

		KnowledgeBaseArticle knowledgeBaseArticle = knowledgeBaseArticleResource.getKnowledgeBaseArticle(kbArticleId);
		_log.debug("GET KnowledgeBase Article: " + knowledgeBaseArticle);
	}

	private void putKnowledgeBaseArticle(KnowledgeBaseArticleResource knowledgeBaseArticleResource, Long kbArticleId)
			throws Exception {

		_log.info("=== PUT KnowledgeBase Article ===");

		KnowledgeBaseArticle updatedKnowledgeBaseArticle = new KnowledgeBaseArticle();

		updatedKnowledgeBaseArticle.setTitle("Updated KBArticle");
		updatedKnowledgeBaseArticle.setDescription("Updated KBFolder from R3B2Portlet");
		updatedKnowledgeBaseArticle.setArticleBody("<p>Updated KBArticle Body.</p>");

		KnowledgeBaseArticle knowledgeBaseArticle = knowledgeBaseArticleResource.putKnowledgeBaseArticle(kbArticleId,
				updatedKnowledgeBaseArticle);
		_log.debug("Updated KnowledgeBase Article: " + knowledgeBaseArticle);
	}

	private void patchKnowledgeBaseArticle(KnowledgeBaseArticleResource knowledgeBaseArticleResource, Long kbArticleId)
			throws Exception {

		_log.info("=== PATCH KnowledgeBase Article ===");

		KnowledgeBaseArticle patchedKnowledgeBaseArticle = new KnowledgeBaseArticle();

		patchedKnowledgeBaseArticle.setDescription("Patched KB Article from R3B2Portlet");

		KnowledgeBaseArticle knowledgeBaseArticle = knowledgeBaseArticleResource.patchKnowledgeBaseArticle(kbArticleId,
				patchedKnowledgeBaseArticle);
		_log.debug("Patched KnowledgeBase Article: " + knowledgeBaseArticle);
	}

	private void deleteKnowledgeBaseArticle(KnowledgeBaseArticleResource knowledgeBaseArticleResource, Long kbArticleId)
			throws Exception {

		_log.info("=== DELETE KnowledgeBase Article ===");

		knowledgeBaseArticleResource.deleteKnowledgeBaseArticle(kbArticleId);

		_log.info("Deleted KnowledgeBase Article with Id: " + kbArticleId);
	}

	@Reference
	private DocumentFolderResource.Factory _documentFolderResourceFactory;

	@Reference
	private DocumentResource.Factory _documentResourceFactory;

	@Reference
	private KnowledgeBaseFolderResource.Factory _knowledgeBaseFolderResourceFactory;

	@Reference
	private KnowledgeBaseArticleResource.Factory _knowledgeBaseArticleResourceFactory;

}
```

## 2.3 Explicação por partes

### Parte 1 — DTOs e Resources da Headless Delivery

Os DTOs são importados de `com.liferay.headless.delivery.dto.v1_0`:

- `Document`: dados de um documento;
- `DocumentFolder`: dados de uma pasta de documentos;
- `KnowledgeBaseArticle`: dados de um artigo da Base de Conhecimento;
- `KnowledgeBaseFolder`: dados de uma pasta da Base de Conhecimento.

As interfaces `DocumentResource`, `DocumentFolderResource`, `KnowledgeBaseArticleResource` e `KnowledgeBaseFolderResource` vêm de `com.liferay.headless.delivery.resource.v1_0`. Em termos práticos, o DTO representa os dados; o Resource oferece as operações para consultar e modificar esses dados.

Não confunda estes tipos com os DTOs e Resources de `com.acme.headless.r3b2`: aqui estamos consumindo a API que vem com o Liferay.

### Parte 2 — O portlet, o logger e o contexto do site

`@Component` registra o widget `R3B2` na categoria `Sample`. O logger registra cada operação com `info`, os objetos retornados com `debug` e as exceções com `error`.

No início de `doView`, `ThemeDisplay` é recuperado de `RenderRequest` e usado para obter o ID do site atual e o usuário da requisição. `siteId` é passado aos `POST` que criam pastas no site; o usuário é informado aos quatro Resources por `.user(...)`.

### Parte 3 — As quatro Factories

Cada campo anotado com `@Reference` recebe do OSGi uma Factory. O código chama `create()`, configura o Builder com o usuário atual e chama `build()`.

| Factory | Instância criada |
|---|---|
| `_documentFolderResourceFactory` | `DocumentFolderResource` |
| `_documentResourceFactory` | `DocumentResource` |
| `_knowledgeBaseFolderResourceFactory` | `KnowledgeBaseFolderResource` |
| `_knowledgeBaseArticleResourceFactory` | `KnowledgeBaseArticleResource` |

Se o bundle que fornece uma Factory não estiver ativo ou o módulo web não conseguir resolver a dependência, o componente do portlet pode não ativar. Consulte os logs e o estado dos bundles antes de investigar os métodos CRUD.

### Parte 4 — `DocumentFolderResource`

`postSiteDocumentFolder` recebe o ID do site e um DTO `DocumentFolder`. O código define `name` e `description`, chama o Resource e guarda o `id` retornado.

`getSiteDocumentFoldersPage` lista as pastas do site. O tipo `Page<DocumentFolder>` encapsula os itens e os metadados de paginação. A expressão `Pagination.of(1, 20)` solicita a primeira página de até 20 resultados.

`getDocumentFolder` consulta a pasta individualmente. `putDocumentFolder` envia um DTO com `name` e `description`; `patchDocumentFolder` envia somente a descrição. O código consulta novamente após cada atualização para que as alterações possam ser conferidas no `DEBUG`.

### Parte 5 — `DocumentResource` e o corpo multipart

O documento é criado dentro da pasta por `postDocumentFolderDocument(docFolderId, MultipartBody.of(...))`. A chamada recebe o ID da pasta e um corpo multipart que contém o arquivo e os campos de texto.

O arquivo é declarado como `BinaryFile` com o tipo MIME, o nome, o stream de leitura e o tamanho em bytes. O mapa usa a chave `file`, que é o nome do campo multipart usado para o arquivo. O mapa de texto usa a chave `title`.

`getDocumentFolderDocumentsPage` lista os documentos da pasta; `getDocument` consulta um documento por ID. O `PUT` envia `UpdatedDocument.txt` e o título `UpdatedDoc`, substituindo os dados do documento conforme o contrato da API. O `PATCH` envia nenhum arquivo novo e modifica somente `description`.

**Cuidados com os arquivos:** mantenha os dois arquivos em `src/main/resources/`. Confirme o resultado de `getResourceAsStream` e o tamanho dos arquivos se a operação falhar. Um tamanho incorreto ou um stream nulo pode impedir o upload.

### Parte 6 — `KnowledgeBaseFolderResource`

A criação usa `postSiteKnowledgeBaseFolder(siteId, newKnowledgeBaseFolder)`. O ID de site indica onde a pasta é criada. O código depois lista pastas por meio de `getSiteKnowledgeBaseFoldersPage` e consulta, substitui e altera parcialmente a pasta criada.

Assim como em `DocumentFolder`, o `PUT` preenche `name` e `description`, enquanto o `PATCH` preenche apenas `description`. A pasta precisa continuar existindo até que o artigo filho seja excluído.

### Parte 7 — `KnowledgeBaseArticleResource`

O artigo é criado dentro da pasta usando `postKnowledgeBaseArticleKnowledgeBaseArticle(kbFolderId, newKnowledgeBaseArticle)`. O DTO recebe `title`, `description` e `articleBody`.

`articleBody` recebe uma string com HTML (`<p>KBArticle Body.</p>`), porque o corpo do artigo pode conter conteúdo formatado. Não confunda `articleBody` com `description`: são campos diferentes.

`getKnowledgeBaseFolderKnowledgeBaseArticlesPage` lista artigos daquela pasta, enquanto `getKnowledgeBaseArticle` consulta um artigo por ID. O `PUT` fornece os três campos definidos no exemplo; o `PATCH` fornece apenas `description`.

### Parte 8 — A ordem do CRUD e a limpeza no `finally`

O `try` executa as operações de negócio do exercício. Se uma delas lançar uma exceção, o `catch` registra o erro. Depois, o `finally` tenta limpar o que tiver sido criado.

A ordem de exclusão é importante porque os conteúdos dependem das pastas:

1. excluir o `KnowledgeBaseArticle`;
2. excluir o `KnowledgeBaseFolder`;
3. excluir o `Document`;
4. excluir o `DocumentFolder`.

Cada exclusão está em seu próprio `try/catch`. Assim, se a exclusão de um objeto falhar, o código ainda tenta excluir os outros. Isso reduz a possibilidade de sobrar conteúdo de teste, embora não garanta a limpeza se o próprio serviço rejeitar as exclusões.

### Parte 9 — `PUT` e `PATCH` nos recursos reais

O código apresenta o mesmo contraste em três tipos de conteúdo:

| Recurso | `PUT` neste exemplo | `PATCH` neste exemplo |
|---|---|---|
| `DocumentFolder` | Envia `name` e `description`. | Envia apenas `description`. |
| `Document` | Envia outro arquivo e `title`. | Não envia arquivo novo; envia apenas `description`. |
| `KnowledgeBaseFolder` | Envia `name` e `description`. | Envia apenas `description`. |
| `KnowledgeBaseArticle` | Envia `title`, `description` e `articleBody`. | Envia apenas `description`. |

A regra conceitual continua sendo a estudada anteriormente: `PUT` representa a substituição do recurso conforme seu contrato, enquanto `PATCH` permite uma atualização parcial. Campos obrigatórios e comportamentos exatos são definidos pela API correspondente; confira os schemas no API Explorer.

## 2.4 Criar o módulo, implantar e executar

### 1. Confirme a disponibilidade da API

Entre no Liferay e abra:

```text
http://localhost:8080/o/api
```

Localize a aplicação **Headless Delivery** e confira os recursos de Documents and Media e Knowledge Base. Como estamos usando serviços já fornecidos pelo portal, esta atividade não precisa gerar uma nova definição REST para eles.

### 2. Crie ou reutilize o módulo web

O arquivo deve ficar em um módulo OSGi web, por exemplo:

```text
headless-r3b2-web/
└── src/
    └── main/
        ├── java/
        │   └── com/acme/headless/r3b2/web/internal/
        │       └── R3B2Portlet.java
        └── resources/
            ├── Document.txt
            └── UpdatedDocument.txt
```

A árvore acima é ilustrativa. Se o módulo web já foi criado no PR anterior, reutilize-o e substitua ou adapte a classe de acordo com a organização dos seus PRs. Evite manter duas classes públicas de mesmo nome e pacote.

### 3. Declare as dependências necessárias

O módulo precisa compilar contra as APIs de portlet, Portal Kernel, Portal Vulcan e Headless Delivery. Em um workspace Liferay, declare as dependências `compileOnly` compatíveis com a versão/target platform do projeto. Os nomes exatos das coordenadas podem variar conforme a configuração do workspace; use os outros módulos do projeto como referência e resolva os pacotes `com.liferay.headless.delivery.*`, `com.liferay.portal.vulcan.*`, `javax.portlet.*` e `org.osgi.service.component.annotations`.

**Não adicione o client JAR apenas para obter esses imports:** este código usa as interfaces do lado do servidor, não os recursos `...client.resource...`.

### 4. Confira os arquivos enviados

Coloque `Document.txt` e `UpdatedDocument.txt` em `src/main/resources/`. Os tamanhos declarados atualmente são `5L` e `6L`. Confira os bytes reais no terminal a partir da raiz do módulo:

```bash
wc -c src/main/resources/Document.txt src/main/resources/UpdatedDocument.txt
```

Se os tamanhos não corresponderem, ajuste os valores no `new BinaryFile(...)` para os tamanhos reais.

### 5. Compile e implante

Compile o módulo web:

```bash
./gradlew headless-r3b2-web:classes
```

Depois, implante-o no container Liferay que está em execução:

```bash
./gradlew headless-r3b2-web:deploy -Ddeploy.docker.container.id=<seu-container>
```

Troque `<seu-container>` pelo nome configurado no seu ambiente. Se o projeto usa outro nome de módulo, ajuste a tarefa Gradle.

### 6. Adicione o portlet a uma página

Entre no portal com um usuário que tenha permissões para executar as operações. Edite uma página, abra o seletor de widgets e procure `R3B2` na categoria `Sample`. Adicione o widget à página.

O portlet não escreve conteúdo HTML na tela; neste exercício, o resultado é acompanhado pelo log do Liferay. Isso é esperado para o código atual.

### 7. Acompanhe os logs

Em uma instalação Docker, use:

```bash
docker logs -f <seu-container>
```

Procure mensagens como:

```text
=== POST Document Folder
=== GET Document Folders Page ===
=== POST Document Folder Document ===
=== POST KnowledgeBase Folder ===
=== POST KnowledgeBase Article ===
```

As mensagens exatas seguem os `info` do código. Para ver os objetos completos registrados com `_log.debug(...)`, habilite `DEBUG` para `com.acme.headless.r3b2.web.internal.R3B2Portlet` no sistema de logs do Liferay.

## 2.5 Pontos de atenção e melhorias

1. **Não executar CRUD dentro de `doView` em uma aplicação real.** Cada renderização cria, altera e tenta excluir conteúdos. Em uma implementação real, as escritas devem acontecer em uma ação disparada pelo usuário; `doView` deve se concentrar em preparar e exibir a interface.
2. **Validar os arquivos antes do upload.** `getResourceAsStream` pode retornar `null` se o arquivo não estiver no classpath. Também é preferível calcular o tamanho dos arquivos em vez de manter números hardcoded.
3. **Evitar mapas anônimos com inicializador duplo.** Os `new HashMap<>() {{ ... }}` facilitam o exemplo curto, mas criam subclasses anônimas. Em código de produção, prefira criar um mapa comum e preenchê-lo explicitamente.
4. **Registrar contexto nos erros.** Em vez de registrar apenas `Error:`, inclua a operação e, quando disponível, o ID do objeto que estava sendo processado.
5. **Não depender apenas do log para validar o resultado.** Depois de `PUT` e `PATCH`, confira os campos específicos retornados ou faça uma leitura e compare os valores esperados.
6. **Rever o comportamento de `PUT`.** Como a operação de substituição pode ter requisitos específicos para o tipo de recurso, verifique o schema e os campos obrigatórios no API Explorer da versão instalada.
7. **Tratar cuidadosamente a limpeza.** O `finally` tenta excluir todos os objetos, mas uma falha pode deixar itens órfãos. Revise os logs `Error deleting ...` se encontrar objetos de teste no portal.
8. **Não deixar o widget acessível a qualquer usuário em ambiente compartilhado sem revisar permissões.** O resource usa o usuário atual; as permissões de criação e exclusão precisam ser adequadas ao ambiente.

## 2.6 Erros comuns

| Sintoma | Causa provável | Como investigar ou resolver |
|---|---|---|
| `package com.liferay.headless.delivery... does not exist` | Falta dependência de compilação da API Headless Delivery ou o target platform não corresponde ao ambiente. | Ajuste as dependências do módulo web ao workspace/versão Liferay usados. |
| `package com.liferay.portal.vulcan... does not exist` | Falta dependência da API Vulcan, utilizada por `MultipartBody`, `BinaryFile`, `Page` e `Pagination`. | Confira as dependências do módulo e o target platform. |
| O portlet `R3B2` não aparece em `Sample` | O módulo não foi implantado ou o componente OSGi não foi ativado. | Verifique logs de implantação e estado dos bundles no Gogo Shell. |
| Uma Factory não foi satisfeita | O bundle que fornece o Resource não está ativo ou a dependência OSGi não foi resolvida. | Confira o bundle Headless Delivery e o estado do componente. |
| O código compila, mas a chamada falha com erro de permissão | O usuário atual não pode executar a operação. | Teste com uma conta autorizada e verifique as permissões do site e da aplicação. |
| Upload falha ou ocorre `NullPointerException` relacionado ao stream | `Document.txt`/`UpdatedDocument.txt` não foi encontrado no classpath. | Confirme o caminho `src/main/resources/` e verifique o resultado de `getResourceAsStream`. |
| O tamanho enviado não corresponde ao arquivo | O valor hardcoded no `BinaryFile` está diferente do tamanho real. | Execute `wc -c` e ajuste o valor em bytes. |
| `POST` do documento falha por causa da pasta | `docFolderId` não foi obtido, ou a pasta não foi criada. | Verifique primeiro o log de `postSiteDocumentFolder` e a permissão de criação. |
| `POST` do artigo falha por causa da pasta | `kbFolderId` não foi obtido, ou a pasta não foi criada. | Verifique `postSiteKnowledgeBaseFolder` e a permissão de criação. |
| O widget aparece vazio, mas existem logs | Esperado: `doView` não produz HTML. | Use os logs e o API Explorer para verificar os resultados. |
| As mensagens `debug` não aparecem | O nível `DEBUG` não está habilitado para a classe. | Habilite `DEBUG` para o logger do `R3B2Portlet`. |
| Pastas ou conteúdos de teste sobraram no portal | Alguma exclusão no `finally` falhou. | Localize `Error deleting ...` no log e remova os itens de teste pelo aplicativo ou API Explorer. |
| A sequência repete quando atualizo a página | O CRUD foi colocado dentro de `doView`. | É o comportamento esperado deste exercício; em produção, mover escritas para ações explícitas. |

### Para praticar

1. Altere o nome e a descrição da pasta de documentos e confirme o resultado com `getDocumentFolder`.
2. Modifique o conteúdo de `Document.txt`, confirme o tamanho em bytes e repita o upload.
3. Após o `PATCH` de `Document`, registre o `description` retornado e confira que nenhum arquivo novo foi enviado nessa operação.
4. Altere o `title`, a `description` e o `articleBody` do artigo, e compare os resultados de `PUT` e `PATCH`.
5. Consulte o API Explorer e identifique para cada método Java do código o endpoint REST, o método HTTP e o schema do corpo da requisição.
6. Como melhoria, faça o portlet apresentar um resumo dos objetos criados em vez de executar automaticamente o CRUD sempre que a página for renderizada.

---