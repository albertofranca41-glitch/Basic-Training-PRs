# Guia: Primeiro Pull Request de Service Builder no Liferay

Este guia explica, do zero, a atividade em que você implementa métodos de consulta para a entidade `H7G5Entry` usando **Service Builder**, divide a lógica entre **Local Service** e **Remote Service** e testa os métodos por meio de um portlet e de requisições `curl`.

A ideia é entender não apenas *o que* escrever, mas também **por que cada arquivo existe e como os componentes se comunicam**. Como Service Builder gera boa parte do código automaticamente, é importante distinguir o código que você implementa daquele que a ferramenta gera.

O guia está dividido em duas etapas principais:

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | Arquitetura do Liferay, Service Builder, `service.xml`, finders, persistência, Local Service, Remote Service, permissões, portlets e JSON Web Services |
| **Etapa 2 – O código e os testes** | Os arquivos personalizados da atividade, explicação parte por parte e execução dos testes |

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 O que é o Liferay e como ele organiza o código](#11-o-que-é-o-liferay-e-como-ele-organiza-o-código)
  - [1.2 O que é Service Builder](#12-o-que-é-service-builder)
  - [1.3 Entendendo o `service.xml` usado na atividade](#13-entendendo-o-servicexml-usado-na-atividade)
  - [1.4 O que são finders](#14-o-que-são-finders)
  - [1.5 Código gerado × código personalizado](#15-código-gerado--código-personalizado)
  - [1.6 A arquitetura API, Service e Web](#16-a-arquitetura-api-service-e-web)
  - [1.7 Local Service × Remote Service](#17-local-service--remote-service)
  - [1.8 Persistência, herança e injeção de dependências](#18-persistência-herança-e-injeção-de-dependências)
  - [1.9 Autenticação e verificação de permissão](#19-autenticação-e-verificação-de-permissão)
  - [1.10 Portlets e JSON Web Services](#110-portlets-e-json-web-services)
  - [1.11 Pré-requisitos e visão geral do fluxo](#111-pré-requisitos-e-visão-geral-do-fluxo)
  - [1.12 Recursos adicionais](#112-recursos-adicionais)
- [Etapa 2 – O código e os testes](#etapa-2--o-código-e-os-testes)
  - [2.1 Visão geral dos arquivos](#21-visão-geral-dos-arquivos)
  - [2.2 `H7G5EntryLocalServiceImpl.java`](#22-h7g5entrylocalserviceimpljava)
  - [2.3 `H7G5EntryServiceImpl.java`](#23-h7g5entryserviceimpljava)
  - [2.4 `H7G5Portlet.java`](#24-h7g5portletjava)
  - [2.5 Script `curl`](#25-script-curl)
  - [2.6 Como executar e testar](#26-como-executar-e-testar)
  - [2.7 Erros comuns](#27-erros-comuns)
  - [Para praticar](#para-praticar)

---

# Etapa 1 – Teoria

Antes de analisar o Java, vamos entender as peças que já existem no projeto. Se você nunca trabalhou com Liferay, pense no Service Builder como uma ferramenta que gera a estrutura repetitiva de acesso ao banco de dados e deixa você implementar a lógica específica da aplicação.

## 1.1 O que é o Liferay e como ele organiza o código

O **Liferay DXP/Portal** é uma plataforma Java para criar portais e aplicações corporativas. Além da interface web, ele oferece infraestrutura para usuários, permissões, sites, conteúdo, serviços e módulos extensíveis.

Uma aplicação Liferay não precisa ser um único programa monolítico com todas as classes juntas. Ela pode ser dividida em módulos que são instalados e executados dentro do ambiente do portal. Esses módulos usam **OSGi**, uma tecnologia de modularização para aplicações Java.

Neste exercício, os nomes ajudam a distinguir as responsabilidades:

| Módulo/camada | Responsabilidade nesta atividade |
|---|---|
| `h7g5-api` | Expõe interfaces e modelos que outros módulos precisam conhecer, como `H7G5Entry`, `H7G5EntryLocalService` e `H7G5EntryService` |
| `h7g5-service` | Contém implementações dos serviços e a lógica de acesso aos dados |
| `h7g5-web` | Contém o portlet `H7G5Portlet`, que demonstra a chamada de um serviço a partir da interface do portal |

Os nomes exatos dos diretórios podem variar em outro projeto, mas a separação entre API, implementação de serviços e interface web é uma organização comum em projetos Liferay.

### O que é OSGi neste contexto?

Para esta atividade, você não precisa dominar OSGi por completo. O essencial é saber que cada módulo é publicado como um *bundle* e que os bundles podem disponibilizar e consumir serviços e pacotes Java.

Por isso, o Liferay precisa conseguir resolver as dependências entre os módulos. Por exemplo, o módulo Web depende das interfaces do módulo API. Se um bundle não exportar o pacote necessário ou se a dependência estiver ausente, o módulo pode não iniciar corretamente.

As anotações `@Component` e `@Reference`, usadas no código, participam desse sistema de componentes e dependências.

## 1.2 O que é Service Builder

**Service Builder é um gerador de código do Liferay.** Sua principal entrada é o arquivo `service.xml`. A partir dele, a ferramenta gera classes Java para representar entidades, acessar o banco de dados e disponibilizar serviços.

Sem Service Builder, você teria de escrever manualmente muito código repetitivo: classes de modelo, operações de persistência, interfaces de serviço, classes-base, métodos de consulta e outros elementos de integração com o portal.

O fluxo simplificado é:

```text
                 service.xml
                     │
                     ▼
        Gradle: buildService
                     │
                     ▼
      Código Java e recursos gerados
      ┌──────────────┼───────────────┐
      ▼              ▼               ▼
    Modelos      Persistência     Serviços-base
      │              │               │
      └──────────────┴───────────────┘
                     │
                     ▼
      Implementações personalizadas
      (LocalServiceImpl / ServiceImpl)
                     │
                     ▼
       Portlet ou outro consumidor
```

O comando que dispara a geração no tutorial é:

```bash
./gradlew h7g5-service:buildService
```

O caminho `h7g5-service:buildService` diz ao Gradle para executar a tarefa `buildService` do módulo `h7g5-service`. A tarefa lê o `service.xml` e gera ou atualiza as classes relacionadas às entidades definidas nele.

> **Ideia central:** você declara a estrutura das entidades e dos métodos de consulta no `service.xml`; o Service Builder gera a infraestrutura; então você implementa a lógica personalizada nos arquivos `*LocalServiceImpl.java` e `*ServiceImpl.java`.

## 1.3 Entendendo o `service.xml` usado na atividade

O arquivo `service.xml` será usado como **contexto técnico**, mas não será incluído neste Pull Request. Ele é essencial para compreender de onde vêm as classes e métodos que o código utiliza.

A declaração inicial contém:

```xml
<service-builder
    auto-namespace-tables="true"
    dependency-injector="ds"
    package-path="com.liferay.h7g5">
    <namespace>OHQIWTSFHL</namespace>
```

- `package-path="com.liferay.h7g5"` define o prefixo dos pacotes Java gerados.
- `<namespace>OHQIWTSFHL</namespace>` define o namespace das tabelas relacionadas ao Service Builder. Com `auto-namespace-tables="true"`, as tabelas geradas recebem esse prefixo; por isso o tutorial consulta tabelas como `OHQIWTSFHL_H7G5Entry`.
- `dependency-injector="ds"` informa que o projeto utiliza Declarative Services para a infraestrutura de injeção de dependências.

A entidade que interessa diretamente ao PR está declarada assim:

```xml
<entity local-service="true" name="H7G5Entry" remote-service="true">
```

O nome `H7G5Entry` identifica a entidade. Os atributos `local-service="true"` e `remote-service="true"` pedem ao Service Builder que gere os contratos e a infraestrutura de Local Service e Remote Service para ela.

### Os campos da entidade

A entidade possui, entre outros, estes campos:

```xml
<column name="h7g5EntryId" primary="true" type="long" />
<column name="h7g5FolderId" type="long" />
<column name="description" type="String" />
<column name="key" type="String" />
<column name="name" type="String" />
```

`h7g5EntryId` é a chave primária da entidade. Os campos `h7g5FolderId`, `description`, `key` e `name` são relevantes para os métodos de consulta que serão implementados.

A coluna `h7g5FolderId` permite relacionar uma entrada (`H7G5Entry`) a uma pasta (`H7G5Folder`). Nesta atividade, o portlet cria uma pasta e consulta as entradas associadas ao ID dessa pasta.

### Por que olhar o `service.xml` se ele não vai no PR?

Porque o código personalizado chama métodos como `findByKey`, `findByName` e `findByH7G5FolderId`. Esses métodos não foram escritos nos arquivos personalizados desta atividade: são gerados a partir dos finders declarados no `service.xml`.

Sem conhecer o `service.xml`, pareceria que os métodos de persistência surgiram do nada. O arquivo explica de onde eles vêm e o tipo de resultado que se espera de cada consulta.

## 1.4 O que são finders

Um **finder** é uma definição de consulta que o Service Builder usa para gerar métodos de busca e a infraestrutura correspondente. Na atividade, o `service.xml` declara quatro finders:

```xml
<finder name="H7G5FolderId" return-type="Collection">
    <finder-column name="h7g5FolderId" />
</finder>
<finder name="Key" return-type="H7G5Entry">
    <finder-column name="key" />
</finder>
<finder name="Name" return-type="Collection">
    <finder-column name="name" />
</finder>
<finder name="H_D_N" return-type="H7G5Entry">
    <finder-column name="h7g5FolderId" />
    <finder-column name="description" />
    <finder-column name="name" />
</finder>
```

O atributo `name` do finder determina o sufixo usado nos métodos gerados. Por exemplo, o finder `H7G5FolderId` gera um método como `findByH7G5FolderId(...)` na persistência.

O atributo `return-type` indica se a busca deve devolver uma coleção ou uma única entidade:

| Finder | Campos usados para pesquisar | Tipo de resultado | Consequência para o código |
|---|---|---|---|
| `H7G5FolderId` | `h7g5FolderId` | `Collection` | Uma pasta pode ter várias entradas; o método devolve uma coleção/lista |
| `Key` | `key` | `H7G5Entry` | O finder representa uma busca por uma entrada individual |
| `Name` | `name` | `Collection` | Entradas diferentes podem compartilhar o mesmo nome; o resultado pode conter várias entradas |
| `H_D_N` | pasta + descrição + nome | `H7G5Entry` | Busca uma entrada usando a combinação dos três campos |

Os nomes das consultas personalizadas (`getBy...`) serão diferentes dos nomes de persistência (`findBy...`). Essa diferença é intencional: os métodos `getBy...` formarão a interface de serviço que a atividade oferece; dentro deles, chamaremos os métodos `findBy...` que o Service Builder gerou.

> **Atenção:** o tipo declarado no finder orienta o contrato de retorno. No caso dos finders de retorno individual, o método de busca pode lançar uma exceção de entidade inexistente caso não encontre resultado. No caso dos finders de coleção, normalmente uma busca sem correspondências resulta em uma coleção vazia, e não em uma exceção por ausência de elemento.

## 1.5 Código gerado × código personalizado

Esta é uma das distinções mais importantes para enviar o PR corretamente.

### Código gerado

São arquivos que o Service Builder cria ou atualiza a partir do `service.xml`. Entre eles estão, por exemplo:

- `H7G5Entry.java` e `H7G5EntryModel.java`;
- `H7G5EntryPersistence.java` e `H7G5EntryUtil.java`;
- `H7G5EntryLocalService.java` e `H7G5EntryService.java`;
- `H7G5EntryLocalServiceBaseImpl.java` e `H7G5EntryServiceBaseImpl.java`;
- classes de implementação do modelo, recursos SQL e arquivos de configuração gerados.

Esses arquivos contêm infraestrutura padronizada. Não devemos editar manualmente um arquivo marcado com `@generated`, porque a próxima execução de `buildService` pode sobrescrever as alterações.

### Código personalizado

Os arquivos `H7G5EntryLocalServiceImpl.java` e `H7G5EntryServiceImpl.java` são pontos de extensão para lógica escrita pelo desenvolvedor. Embora o Service Builder possa criá-los inicialmente se eles ainda não existirem, eles não são recriados integralmente em toda execução como os contratos gerados. É neles que adicionamos os métodos personalizados.

O arquivo `H7G5Portlet.java` é código próprio do módulo Web. O script `.sh` também é um arquivo criado para executar os testes via `curl`.

### Quais arquivos entram neste PR?

Para esta atividade, o conjunto enviado é:

```text
H7G5EntryLocalServiceImpl.java  # consultas usando a persistência
H7G5EntryServiceImpl.java       # verificação de permissão + chamadas locais
H7G5Portlet.java                # demonstra chamada ao Remote Service
<seu-script-de-testes>.sh       # chamadas JSONWS via curl
```

**Não inclua neste PR** o `service.xml` nem os arquivos Java gerados pelo Service Builder. O `service.xml` é referência para entender a atividade, mas a instrução deste exercício é enviar as implementações personalizadas e o script de shell, não a saída gerada.

## 1.6 A arquitetura API, Service e Web

No projeto do tutorial, os módulos têm responsabilidades distintas:

```text
h7g5-web
└── H7G5Portlet
       │ chama a interface do serviço
       ▼
h7g5-api
├── H7G5Entry
├── H7G5EntryLocalService
└── H7G5EntryService
       ▲                         ▲
       │ implementados por       │ implementados por
       │                         │
h7g5-service
├── H7G5EntryLocalServiceImpl
├── H7G5EntryServiceImpl
└── classes de persistência geradas
```

A API define os contratos que outros módulos podem utilizar. O módulo Service fornece as implementações. O módulo Web consome os serviços por meio das interfaces expostas pela API.

Esse desenho ajuda a separar as responsabilidades: o portlet não precisa conhecer diretamente os detalhes de SQL ou da persistência. Ele chama um serviço; o serviço faz a consulta necessária.

## 1.7 Local Service × Remote Service

O Service Builder pode gerar dois tipos de serviço para uma entidade. Eles têm propósitos diferentes.

### Local Service

O **Local Service** contém a lógica principal da operação e pode ser chamado por outros módulos OSGi dentro do portal. No modelo deste exercício, ele consulta a persistência para encontrar `H7G5Entry` por diferentes critérios.

Uma regra didática importante do tutorial é que o Local Service **não deve ser confundido com a camada que decide se o usuário tem permissão para realizar uma operação**. A verificação de acesso será feita no Remote Service deste exemplo.

### Remote Service

O **Remote Service** expõe operações que podem ser chamadas remotamente — no caso deste exercício, por JSON Web Services (JSONWS) — e pode executar verificações de permissão antes de delegar o trabalho ao Local Service.

No código deste PR, o Remote Service segue esta sequência:

1. Verifica o e-mail do usuário autenticado.
2. Se o endereço terminar com `@liferay.com`, permite continuar.
3. Se não terminar, lança `PrincipalException`.
4. Se a verificação passar, chama o método equivalente no Local Service.

O objetivo pedagógico é manter a consulta em um lugar e a verificação de acesso no outro.

| Responsabilidade | Local Service | Remote Service |
|---|---|---|
| Buscar dados pela persistência | Sim | Não diretamente; delega ao Local Service |
| Executar a verificação simples de e-mail deste exercício | Não | Sim |
| Ser utilizado por outro módulo OSGi | Sim | Sim |
| Ser exposto via JSONWS neste exercício | Não é o ponto de entrada usado pelo script | Sim, após a configuração do Service Builder e do módulo |

> A checagem por sufixo de e-mail é uma regra simplificada de treinamento, não um modelo de autorização adequado para copiar sem alterações para um sistema de produção. Aplicações reais devem usar o mecanismo de permissões do Liferay e validar as regras exigidas pelo domínio.

## 1.8 Persistência, herança e injeção de dependências

O termo **persistência** designa a camada que realiza operações de leitura e escrita no armazenamento de dados. Neste exercício, o Service Builder gera uma interface de persistência e uma implementação para `H7G5Entry`.

A classe personalizada herda de uma classe-base gerada:

```java
public class H7G5EntryLocalServiceImpl
    extends H7G5EntryLocalServiceBaseImpl {
```

A classe-base fornece infraestrutura comum, incluindo referências a componentes necessários para a implementação do serviço. Por isso o código pode acessar:

```java
h7g5EntryPersistence.findByKey(key);
```

`h7g5EntryPersistence` não é declarada no código personalizado mostrado: ela é fornecida pela classe-base gerada pelo Service Builder.

A mesma ideia aparece no Remote Service:

```java
public class H7G5EntryServiceImpl extends H7G5EntryServiceBaseImpl {
```

O serviço remoto herda infraestrutura e referências que permitem chamar o Local Service, como `h7g5EntryLocalService`.

### O que significam `@Component` e `@Reference`?

- `@Component` registra a classe como componente OSGi e declara propriedades usadas pelo ambiente para identificá-la e configurá-la.
- `@Reference` indica que uma dependência deve ser fornecida pelo mecanismo de serviços OSGi. No portlet, por exemplo, a referência a `H7G5EntryService` permite que o Liferay injete uma implementação compatível.

A injeção evita que a classe tenha de instanciar manualmente todos os serviços dos quais depende.

### Por que não consultar o banco diretamente no portlet?

Porque isso misturaria interface, lógica de negócio e acesso a dados. O portlet deve chamar os serviços. O Local Service concentra a consulta e o Remote Service acrescenta a verificação de acesso do exemplo.

## 1.9 Autenticação e verificação de permissão

**Autenticação** responde à pergunta: *quem é o usuário?* **Autorização** responde: *esse usuário pode executar esta operação?*

No script da atividade, o `curl` envia credenciais com `-u usuario:senha`. O Liferay autentica o usuário; depois, o método remoto consulta o usuário atual usando `getUser()` e verifica seu e-mail.

A regra personalizada é:

```java
if (!emailAddress.endsWith("@liferay.com")) {
    throw new PrincipalException(
        "O usuário não tem permissão para acessar este serviço");
}
```

- `endsWith("@liferay.com")` verifica se o e-mail termina com esse texto.
- `!` inverte o resultado: o bloco é executado quando o e-mail **não** termina com o sufixo permitido.
- `PrincipalException` interrompe a operação e sinaliza que o usuário não tem permissão segundo a regra do exercício.

O método declara `throws PortalException`. Isso é suficiente porque `PrincipalException` é uma subclasse de `PortalException`.

**Importante:** o exemplo não está verificando se a conta tem um papel administrativo nem se pertence a um grupo específico. A única condição codificada é o final do e-mail. Além disso, as credenciais usadas nos exemplos precisam existir no seu Liferay local; não presuma que `test@liferay.com:test` funcionará se essa conta não estiver configurada na sua instância.

## 1.10 Portlets e JSON Web Services

### O que é um portlet?

Um **portlet** é um componente de aplicação que pode ser adicionado a uma página do portal. O `H7G5Portlet` deste exercício escreve uma mensagem, cria uma pasta de teste e consulta o Remote Service para contar as entradas associadas à pasta.

O método `doView(...)` é chamado quando o Liferay renderiza a visualização do portlet. Isso significa que o código dentro dele pode ser executado novamente quando a página é atualizada.

> Neste exemplo, o portlet cria uma pasta a cada renderização. Isso é útil para demonstrar as chamadas ao serviço, mas tem um efeito colateral: atualizar a página cria novas pastas. Não trate esse comportamento de demonstração como uma implementação de produção.

### O que é JSONWS?

**JSON Web Services (JSONWS)** é um mecanismo do Liferay que permite invocar métodos de serviços publicados por meio de requisições HTTP, recebendo uma representação JSON da resposta quando aplicável.

O tutorial usa o endereço:

```text
http://localhost:8080/api/jsonws
```

Essa página permite explorar os serviços JSONWS disponíveis na instância. É também o local indicado para confirmar o nome exato de uma operação e os parâmetros que ela espera.

A classe remota declara estas propriedades:

```java
@Component(
    property = {
        "json.web.service.context.name=ohqiwtsfhl",
        "json.web.service.context.path=H7G5Entry"
    },
    service = AopService.class
)
```

Essas propriedades configuram o contexto JSONWS do componente. No script, elas aparecem no caminho `ohqiwtsfhl.h7g5entry`.

O nome da URL é derivado das convenções JSONWS do Liferay e pode ser surpreendente, especialmente quando o método contém underscores. Por isso, **confirme a URL real na página `/api/jsonws`**, em vez de confiar cegamente no nome inferido.

### O que é `curl`?

`curl` é uma ferramenta de linha de comando para fazer requisições HTTP. Ela permite testar o serviço sem criar uma interface gráfica.

Por exemplo:

```bash
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-name \
    -u test@liferay.com:test \
    -d name='Exemplo'
```

- A URL identifica o método remoto.
- `-u test@liferay.com:test` fornece usuário e senha para autenticação HTTP.
- `-d name='Exemplo'` envia o parâmetro `name`. Ao usar `-d`, o `curl` faz uma requisição POST por padrão.

## 1.11 Pré-requisitos e visão geral do fluxo

Antes de testar a atividade, você precisa de:

| Recurso | Para quê |
|---|---|
| Instância Liferay do tutorial em execução | Hospeda os módulos, os serviços e o JSONWS |
| Banco configurado conforme o Basic Training | Armazena as entidades e os registros usados pela atividade |
| Projeto Gradle com os módulos `h7g5-api`, `h7g5-service` e `h7g5-web` | Contém os contratos, implementações e portlet |
| Java/Gradle compatíveis com a versão do projeto | Compilam e empacotam os módulos |
| Usuário válido no Liferay | Autentica as chamadas `curl` e permite testar a regra de e-mail |
| Git e acesso ao repositório do exercício | Permitem acompanhar as alterações nos arquivos da atividade |

O fluxo da consulta feita pelo script é:

```text
script curl
    │ HTTP + credenciais + parâmetros
    ▼
Liferay JSONWS
    │ identifica o método remoto
    ▼
H7G5EntryServiceImpl
    │ _checkPermission()
    │ se permitido, delega
    ▼
H7G5EntryLocalServiceImpl
    │ chama o finder gerado
    ▼
h7g5EntryPersistence
    │ consulta os registros
    ▼
Banco de dados
```

O caminho do portlet é parecido, mas começa na renderização da página e na chamada do serviço por injeção OSGi.

---

## 1.12 Recursos adicionais

- [Entendendo o fluxo de requisição no Liferay](./entendendo-fluxo-requisicao-liferay.md)
- [Introdução ao OSGi no Liferay](./introducao-osgi-liferay.md)
- [Introdução ao Service Builder do Liferay](./introducao-service-builder-liferay.md)

# Etapa 2 – O código e os testes

## 2.1 Visão geral dos arquivos

O PR implementa quatro partes que trabalham juntas:

```text
H7G5EntryLocalServiceImpl.java
    └── métodos de consulta que delegam aos finders gerados

H7G5EntryServiceImpl.java
    └── verifica o e-mail do usuário e chama o Local Service

H7G5Portlet.java
    └── chama o Remote Service a partir do portlet

script .sh
    └── testa os métodos expostos via JSONWS com curl
```

O ponto essencial é que o Remote Service **não repete a consulta**. Ele verifica a permissão e delega ao Local Service. O Local Service, por sua vez, usa os métodos de persistência que o Service Builder gerou a partir dos finders do `service.xml`.

## 2.2 `H7G5EntryLocalServiceImpl.java`

### Código completo

```java
/**
 * SPDX-FileCopyrightText: (c) 2026 Liferay, Inc. https://liferay.com
 * SPDX-License-Identifier: LGPL-2.1-or-later OR LicenseRef-Liferay-DXP-EULA-2.0.0-2023-06
 */

package com.liferay.h7g5.service.impl;

import com.liferay.h7g5.model.H7G5Entry;
import com.liferay.h7g5.service.base.H7G5EntryLocalServiceBaseImpl;
import com.liferay.portal.aop.AopService;
import com.liferay.portal.kernel.exception.PortalException;

import java.util.List;

import org.osgi.service.component.annotations.Component;

/**
 * @author Alberto
 */
@Component(
    property = "model.class.name=com.liferay.h7g5.model.H7G5Entry",
    service = AopService.class
)
public class H7G5EntryLocalServiceImpl extends H7G5EntryLocalServiceBaseImpl {

    /**
     * Esse método busca as informações de um H7G5Entry único no banco de dados
     * pelo Persistence por meio de 3 atributos: id da pasta, descrição e nome.
     *
     * @param  h7g5FolderId o id da pasta
     * @param  description a descrição do H7G5Entry
     * @param  name o nome do H7G5Entry
     * @return o H7G5Entry que corresponde aos 3 atributos
     * @throws PortalException se nenhum H7G5Entry for encontrado
     */
    public H7G5Entry getByH_D_N(
            long h7g5FolderId, String description, String name)
        throws PortalException {

        return h7g5EntryPersistence.findByH_D_N(
            h7g5FolderId, description, name);
    }

    /**
     * Esse método lista todos os H7G5Entry que pertencem a uma determinada
     * pasta. Se a pasta não tiver nenhum, a lista retorna vazia.
     *
     * @param  h7g5FolderId o id da pasta
     * @return os H7G5Entry da pasta
     */
    public List<H7G5Entry> getByH7G5FolderId(long h7g5FolderId) {
        return h7g5EntryPersistence.findByH7G5FolderId(h7g5FolderId);
    }

    /**
     * O atributo "key" retorna apenas um objeto. Podemos perceber isso pelo
     * service.xml não ter definido o finder de key como "Collection", diferente
     * de h7g5FolderId e name.
     *
     * @param  key a chave do H7G5Entry
     * @return o H7G5Entry que possui essa chave
     * @throws PortalException se nenhum H7G5Entry com essa chave for encontrado
     */
    public H7G5Entry getByKey(String key) throws PortalException {
        return h7g5EntryPersistence.findByKey(key);
    }

    /**
     * O atributo "name" não necessariamente é único, então o método retorna uma
     * lista de H7G5Entry que possuem o mesmo nome. Se nenhum for encontrado, a
     * lista retorna vazia.
     *
     * @param  name o nome dos H7G5Entry
     * @return os H7G5Entry que possuem esse nome
     */
    public List<H7G5Entry> getByName(String name) {
        return h7g5EntryPersistence.findByName(name);
    }

}
```

### O papel da classe e da anotação

```java
public class H7G5EntryLocalServiceImpl
    extends H7G5EntryLocalServiceBaseImpl {
```

- `H7G5EntryLocalServiceImpl` é a implementação personalizada do Local Service da entidade `H7G5Entry`.
- `extends H7G5EntryLocalServiceBaseImpl` reaproveita a infraestrutura gerada pelo Service Builder. É por isso que a classe consegue usar `h7g5EntryPersistence` sem declarar esse campo no próprio arquivo.

A anotação:

```java
@Component(
    property = "model.class.name=com.liferay.h7g5.model.H7G5Entry",
    service = AopService.class
)
```

registra o componente no ambiente OSGi e associa sua implementação à entidade `com.liferay.h7g5.model.H7G5Entry` para a infraestrutura de serviço/AOP do Liferay. A anotação não realiza a consulta; ela participa do registro e da integração do componente com a plataforma.

### Método 1 — `getByH_D_N`

```java
public H7G5Entry getByH_D_N(
        long h7g5FolderId, String description, String name)
    throws PortalException {

    return h7g5EntryPersistence.findByH_D_N(
        h7g5FolderId, description, name);
}
```

Esse método recebe três argumentos e os repassa ao finder correspondente:

| Parâmetro | Tipo Java | Significado |
|---|---|---|
| `h7g5FolderId` | `long` | ID da pasta à qual a entrada pertence |
| `description` | `String` | Descrição da entrada |
| `name` | `String` | Nome da entrada |

Como o finder `H_D_N` no `service.xml` usa os três campos e declara `return-type="H7G5Entry"`, o objetivo é encontrar uma entrada individual correspondente à combinação.

O método declara `throws PortalException` porque a operação de persistência de retorno individual pode sinalizar que a entrada não foi encontrada. O método não captura essa exceção; ele permite que ela seja propagada a quem chamou o Local Service.

Observe que o método personalizado se chama `getByH_D_N`, enquanto o finder gerado se chama `findByH_D_N`. O primeiro define uma operação na camada de serviço; o segundo é a operação de persistência que efetivamente realiza a consulta.

### Método 2 — `getByH7G5FolderId`

```java
public List<H7G5Entry> getByH7G5FolderId(long h7g5FolderId) {
    return h7g5EntryPersistence.findByH7G5FolderId(h7g5FolderId);
}
```

Esse método busca as entradas de uma pasta. Como um mesmo ID de pasta pode estar associado a várias entradas, ele devolve `List<H7G5Entry>`.

- `List` é uma coleção ordenada de elementos.
- `H7G5Entry` é o tipo de cada elemento da lista.
- Se não houver resultados, a consulta de coleção normalmente devolve uma lista vazia.

Esse é o método usado pelo portlet para consultar as entradas vinculadas à pasta que acabou de criar.

### Método 3 — `getByKey`

```java
public H7G5Entry getByKey(String key) throws PortalException {
    return h7g5EntryPersistence.findByKey(key);
}
```

A consulta é feita pela coluna `key`. O finder foi definido com `return-type="H7G5Entry"`, em vez de `Collection`, então a assinatura retorna um único `H7G5Entry` e declara a possibilidade de `PortalException`.

Isso reflete o contrato que foi escolhido no `service.xml`: a chave é usada como critério de uma busca individual. O código não transforma a resposta em lista, pois isso contrariaria o tipo definido para esse finder.

### Método 4 — `getByName`

```java
public List<H7G5Entry> getByName(String name) {
    return h7g5EntryPersistence.findByName(name);
}
```

O nome pode aparecer em várias entradas. Por isso, o finder `Name` foi declarado como `Collection`, e o método personalizado devolve `List<H7G5Entry>`.

Os métodos `getByName` e `getByH7G5FolderId` seguem o mesmo padrão: recebem um critério, chamam um finder de coleção e devolvem os resultados. Os outros dois usam finders de retorno individual e podem propagar uma exceção quando não há entidade correspondente.

### O que esta classe não faz?

Ela não monta SQL manualmente, não cria conexão com o banco e não verifica o e-mail do usuário. O Service Builder já forneceu os métodos de persistência, e a validação de acesso fica no Remote Service. Essa separação torna cada camada mais simples de entender e testar.

## 2.3 `H7G5EntryServiceImpl.java`

### Código completo

```java
/**
 * SPDX-FileCopyrightText: (c) 2026 Liferay, Inc. https://liferay.com
 * SPDX-License-Identifier: LGPL-2.1-or-later OR LicenseRef-Liferay-DXP-EULA-2.0.0-2023-06
 */

package com.liferay.h7g5.service.impl;

import com.liferay.h7g5.model.H7G5Entry;
import com.liferay.h7g5.service.base.H7G5EntryServiceBaseImpl;
import com.liferay.portal.aop.AopService;
import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.model.User;
import com.liferay.portal.kernel.security.auth.PrincipalException;

import java.util.List;

import org.osgi.service.component.annotations.Component;

/**
 * @author Alberto
 */
@Component(
    property = {
        "json.web.service.context.name=ohqiwtsfhl",
        "json.web.service.context.path=H7G5Entry"
    },
    service = AopService.class
)
public class H7G5EntryServiceImpl extends H7G5EntryServiceBaseImpl {

    /**
     * Chama o método do local service que busca um H7G5Entry único pelos 3
     * atributos: id da pasta, descrição e nome. Antes, verifica a permissão.
     *
     * @param  h7g5FolderId o id da pasta
     * @param  description a descrição do H7G5Entry
     * @param  name o nome do H7G5Entry
     * @return o H7G5Entry que corresponde aos 3 atributos
     * @throws PortalException se o usuário não tiver permissão ou se nenhum
     *         H7G5Entry for encontrado
     */
    public H7G5Entry getByH_D_N(
            long h7g5FolderId, String description, String name)
        throws PortalException {

        _checkPermission();

        return h7g5EntryLocalService.getByH_D_N(
            h7g5FolderId, description, name);
    }

    /**
     * Lista os H7G5Entry de uma pasta, depois de verificar a permissão.
     *
     * @param  h7g5FolderId o id da pasta
     * @return os H7G5Entry da pasta
     * @throws PortalException se o usuário não tiver permissão
     */
    public List<H7G5Entry> getByH7G5FolderId(long h7g5FolderId)
        throws PortalException {

        _checkPermission();

        return h7g5EntryLocalService.getByH7G5FolderId(h7g5FolderId);
    }

    /**
     * Busca um H7G5Entry pela chave, depois de verificar a permissão.
     *
     * @param  key a chave do H7G5Entry
     * @return o H7G5Entry que possui essa chave
     * @throws PortalException se o usuário não tiver permissão ou se nenhum
     *         H7G5Entry com essa chave for encontrado
     */
    public H7G5Entry getByKey(String key) throws PortalException {
        _checkPermission();

        return h7g5EntryLocalService.getByKey(key);
    }

    /**
     * Lista os H7G5Entry com um determinado nome, depois de verificar a
     * permissão.
     *
     * @param  name o nome dos H7G5Entry
     * @return os H7G5Entry que possuem esse nome
     * @throws PortalException se o usuário não tiver permissão
     */
    public List<H7G5Entry> getByName(String name) throws PortalException {
        _checkPermission();

        return h7g5EntryLocalService.getByName(name);
    }

    /**
     * Apenas usuários cujo e-mail termina com "@liferay.com" podem acessar os
     * métodos deste Remote Service. O método getUser() é herdado de
     * H7G5EntryServiceBaseImpl (que o herda de BaseServiceImpl) e retorna o
     * usuário logado no portal. PrincipalException é subclasse de
     * PortalException, então declarar PortalException já é suficiente.
     */
    private void _checkPermission() throws PortalException {
        User user = getUser();

        String emailAddress = user.getEmailAddress();

        if (!emailAddress.endsWith("@liferay.com")) {
            throw new PrincipalException(
                "O usuário não tem permissão para acessar este serviço");
        }
    }

}
```

### Por que herdar de `H7G5EntryServiceBaseImpl`?

```java
public class H7G5EntryServiceImpl extends H7G5EntryServiceBaseImpl {
```

A classe-base é gerada pelo Service Builder e oferece a infraestrutura necessária ao Remote Service. Entre outras coisas, ela disponibiliza referências a serviços que a implementação precisa utilizar, como `h7g5EntryLocalService`, e métodos herdados, como `getUser()`.

### A anotação `@Component`

```java
@Component(
    property = {
        "json.web.service.context.name=ohqiwtsfhl",
        "json.web.service.context.path=H7G5Entry"
    },
    service = AopService.class
)
```

Além de registrar o componente de serviço no ambiente OSGi, as propriedades configuram o contexto usado para localizar os métodos no JSONWS do Liferay. Por isso, o script procura caminhos que começam por `ohqiwtsfhl.h7g5entry`.

### O padrão dos métodos remotos

Veja o método `getByName`:

```java
public List<H7G5Entry> getByName(String name) throws PortalException {
    _checkPermission();

    return h7g5EntryLocalService.getByName(name);
}
```

Ele segue duas etapas:

1. `_checkPermission()` verifica se o usuário pode continuar.
2. `h7g5EntryLocalService.getByName(name)` delega a consulta à implementação local.

Os outros três métodos repetem essa organização, alterando apenas os parâmetros e o tipo de resultado. Assim, a consulta fica concentrada no Local Service e a verificação de e-mail é aplicada em todas as entradas públicas do Remote Service.

### Por que o Remote Service não chama `h7g5EntryPersistence` diretamente?

Porque a atividade separa responsabilidades. O Remote Service decide se a chamada é permitida e encaminha o pedido ao Local Service. O Local Service sabe qual finder de persistência utilizar. Se os dois serviços repetissem a consulta, haveria duplicação de lógica.

### Entendendo `_checkPermission()` linha por linha

```java
private void _checkPermission() throws PortalException {
    User user = getUser();

    String emailAddress = user.getEmailAddress();

    if (!emailAddress.endsWith("@liferay.com")) {
        throw new PrincipalException(
            "O usuário não tem permissão para acessar este serviço");
    }
}
```

| Linha/expressão | Significado |
|---|---|
| `private` | O método é usado internamente pela própria classe; não é uma operação pública do serviço |
| `getUser()` | Recupera o usuário associado ao contexto de execução autenticado do serviço |
| `getEmailAddress()` | Obtém o endereço de e-mail desse usuário |
| `endsWith("@liferay.com")` | Verifica se o e-mail termina com o sufixo permitido |
| `!` | Inverte a condição; o bloco de erro roda se o sufixo não corresponder |
| `throw new PrincipalException(...)` | Interrompe o fluxo lançando uma exceção de permissão |

Declarar `throws PortalException` é suficiente porque `PrincipalException` herda de `PortalException`. Assim, os métodos públicos não precisam declarar especificamente `PrincipalException` para propagar esse erro.

## 2.4 `H7G5Portlet.java`

> **Nota sobre a implementação:** a estrutura inicial deste portlet já fazia parte do projeto; boa parte do código é de apoio e utiliza serviços disponibilizados pelo Service Builder. Nesta atividade, a alteração feita neste arquivo foi a inclusão do bloco `try/catch` ao final de `doView(...)`, para tratar possíveis erros na chamada ao serviço.

### Código completo

```java
package com.liferay.h7g5.web.internal.portlet;

import com.liferay.h7g5.model.H7G5Entry;
import com.liferay.h7g5.model.H7G5Folder;
import com.liferay.h7g5.service.H7G5EntryService;
import com.liferay.h7g5.service.H7G5FolderLocalService;
import com.liferay.portal.kernel.exception.PortalException;
import com.liferay.portal.kernel.util.StringUtil;

import java.io.IOException;
import java.io.PrintWriter;

import java.util.List;

import javax.portlet.GenericPortlet;
import javax.portlet.Portlet;
import javax.portlet.RenderRequest;
import javax.portlet.RenderResponse;

import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;

@Component(
    property = {
        "com.liferay.portlet.display-category=category.sample",
        "javax.portlet.display-name=H7G5"
    },
    service = Portlet.class
)
public class H7G5Portlet extends GenericPortlet {

    public H7G5Portlet() {
        System.out.println("Constructing H7G5Portlet");
    }

    @Override
    protected void doView(
            RenderRequest renderRequest, RenderResponse renderResponse)
        throws IOException {

        System.out.println("Invoking H7G5Portlet#doView");

        PrintWriter printWriter = renderResponse.getWriter();

        printWriter.println("Hello, H7G5!");

        System.out.println(
            "There are " + _h7G5FolderLocalService.getH7G5FoldersCount() +
                " folders.");

        H7G5Folder h7g5Folder = _h7G5FolderLocalService.createH7G5Folder(
            System.currentTimeMillis());

        h7g5Folder.setDescription(StringUtil.randomString());
        h7g5Folder.setName(StringUtil.randomString());

        _h7G5FolderLocalService.addH7G5Folder(h7g5Folder);

        System.out.println(
            "After adding a new folder, there are now " +
                _h7G5FolderLocalService.getH7G5FoldersCount() + " folders.");

        try {
            List<H7G5Entry> h7g5Entries = _h7g5EntryService.getByH7G5FolderId(
                h7g5Folder.getH7G5FolderId());

            System.out.println("Entries in folder: " + h7g5Entries.size());
        }
        catch (PortalException portalException) {
            System.out.println(
                "Error retrieving entries: " + portalException.getMessage());
        }
    }

    @Reference
    private H7G5EntryService _h7g5EntryService;

    @Reference
    private H7G5FolderLocalService _h7G5FolderLocalService;

}
```

### O que o portlet demonstra?

O portlet é a parte web que chama o serviço. Ele não substitui o Local Service nem o Remote Service; ele é apenas outro consumidor do serviço.

A anotação `@Component` registra o portlet e define propriedades de apresentação:

- `com.liferay.portlet.display-category=category.sample` coloca o portlet na categoria de widgets de exemplo;
- `javax.portlet.display-name=H7G5` define o nome mostrado no portal;
- `service = Portlet.class` registra o componente como portlet.

A classe herda de `GenericPortlet`, que fornece a estrutura básica para um portlet Java.

### O método `doView(...)`

```java
@Override
protected void doView(
        RenderRequest renderRequest, RenderResponse renderResponse)
    throws IOException {
```

`doView` é executado quando o Liferay precisa renderizar a visualização do portlet. O método obtém um `PrintWriter` a partir de `renderResponse` e escreve a mensagem `Hello, H7G5!` na saída da página.

O restante do método é uma demonstração de chamadas entre serviços.

### A consulta de quantidade de pastas

```java
System.out.println(
    "There are " + _h7G5FolderLocalService.getH7G5FoldersCount() +
        " folders.");
```

O portlet chama o Local Service de `H7G5Folder` para consultar quantas pastas existem. Esse serviço foi gerado a partir da entidade `H7G5Folder` no `service.xml`.

### Criação e gravação de uma pasta

```java
H7G5Folder h7g5Folder = _h7G5FolderLocalService.createH7G5Folder(
    System.currentTimeMillis());

h7g5Folder.setDescription(StringUtil.randomString());
h7g5Folder.setName(StringUtil.randomString());

_h7G5FolderLocalService.addH7G5Folder(h7g5Folder);
```

1. `createH7G5Folder(...)` cria em memória um objeto de modelo `H7G5Folder` com o ID fornecido.
2. `System.currentTimeMillis()` usa o horário atual em milissegundos como valor do ID nesta demonstração.
3. `StringUtil.randomString()` cria valores aleatórios para descrição e nome.
4. `addH7G5Folder(...)` entrega o objeto ao serviço para persistência no banco.

O portlet imprime a quantidade antes e depois de salvar a nova pasta para mostrar o efeito da gravação.

### Chamada ao Remote Service

```java
try {
    List<H7G5Entry> h7g5Entries = _h7g5EntryService.getByH7G5FolderId(
        h7g5Folder.getH7G5FolderId());

    System.out.println("Entries in folder: " + h7g5Entries.size());
}
catch (PortalException portalException) {
    System.out.println(
        "Error retrieving entries: " + portalException.getMessage());
}
```

O código chama `getByH7G5FolderId` pelo serviço remoto, passando o ID da pasta recém-criada. O método devolve uma lista de entradas; `size()` informa quantas foram encontradas.

O bloco `try/catch` é necessário porque o Remote Service declara `throws PortalException`. Se ocorrer um erro — por exemplo, se a verificação de e-mail não for satisfeita — a exceção é capturada e sua mensagem é exibida no console. Como a pasta foi criada imediatamente antes, ela pode continuar existindo mesmo se a consulta falhar.

### As referências `@Reference`

```java
@Reference
private H7G5EntryService _h7g5EntryService;

@Reference
private H7G5FolderLocalService _h7G5FolderLocalService;
```

O Liferay injeta as implementações desses serviços no portlet. O código não executa `new H7G5EntryServiceImpl()` nem `new H7G5FolderLocalServiceImpl()`: a resolução dos componentes fica a cargo do ambiente OSGi.

O uso de `H7G5EntryService`, em vez de `H7G5EntryLocalService`, demonstra especificamente a chamada ao **Remote Service** construída nesta atividade.

## 2.5 Script `curl`

### Código completo

Salve os comandos em um arquivo, por exemplo `test-h7g5-entry.sh`:

```bash
#!/bin/bash

# Substitua os valores de exemplo por dados que existam em OHQIWTSFHL_H7G5Entry.

# getByName
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-name \
    -u test@liferay.com:test \
    -d name='Exemplo'

# getByH7G5FolderId
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-h7-g5-folder-id \
    -u test@liferay.com:test \
    -d h7g5FolderId=1616680001273

# getByH_D_N (confirme o nome da URL em /api/jsonws)
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-h_-d_-n \
    -u test@liferay.com:test \
    -d h7g5FolderId=1616680001273 \
    -d description='Descricao de exemplo' \
    -d name='Exemplo'

# getByKey
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-key \
    -u test@liferay.com:test \
    -d key='chave-exemplo'

# Teste de permissão: usuário sem e-mail @liferay.com deve receber a exceção
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-name \
    -u outro@exemplo.com:senha \
    -d name='Exemplo'
```

### Como ler o comando

Veja a primeira chamada:

```bash
curl http://localhost:8080/api/jsonws/ohqiwtsfhl.h7g5entry/get-by-name \
    -u test@liferay.com:test \
    -d name='Exemplo'
```

- `curl` executa a requisição HTTP.
- `http://localhost:8080` aponta para a instância local do Liferay.
- `/api/jsonws/` indica o caminho da interface JSONWS.
- `ohqiwtsfhl.h7g5entry/get-by-name` identifica o contexto e o método remoto.
- `-u test@liferay.com:test` envia as credenciais.
- `-d name='Exemplo'` fornece o argumento `name` esperado pelo método `getByName(String name)`.

O script testa os quatro métodos públicos e inclui uma quinta chamada para testar a regra de permissão com um usuário cujo e-mail não termina em `@liferay.com`.

### Antes de executar: confirme três coisas

1. **Credenciais:** substitua os usuários e senhas por contas que realmente existam no seu Liferay local.
2. **Dados:** troque os nomes, IDs, descrições e chaves de exemplo por valores que existam na tabela `OHQIWTSFHL_H7G5Entry`, se quiser receber resultados. Os valores de ID do tutorial são apenas exemplos e podem não existir no seu banco.
3. **URL JSONWS:** acesse `http://localhost:8080/api/jsonws` e confirme o caminho exato de cada método. Em particular, confirme o caminho gerado para `getByH_D_N`, pois o nome da URL com underscores pode não ser intuitivo.

Se o objetivo for testar a regra de permissão, a conta da última chamada também precisa existir e ter a senha correta. Caso contrário, a autenticação poderá falhar antes que `_checkPermission()` seja executado.

## 2.6 Como executar e testar

Os comandos abaixo seguem o fluxo do tutorial de Service Builder. Execute-os a partir da raiz do projeto que contém os módulos.

### 1. Gerar o código

```bash
./gradlew h7g5-service:buildService
```

Isso gera os contratos, classes-base, métodos de persistência e demais arquivos derivados do `service.xml`. Como a atividade parte de um projeto já montado, não é necessário reconstruir manualmente essas classes.

### 2. Compilar

```bash
./gradlew classes
```

O objetivo é confirmar que o código Java compila. Se houver erros, leia a mensagem do compilador e confira imports, assinaturas, tipos de retorno e dependências.

### 3. Implantar os módulos no container do tutorial

O tutorial anterior usa este comando:

```bash
./gradlew deploy -Ddeploy.docker.container.id=ephesians-liferay
```

O nome `ephesians-liferay` é o nome do container usado naquele ambiente. Se o seu container tiver outro nome, substitua-o pelo nome real do seu ambiente. O comando de implantação precisa ser compatível com a configuração Docker/Gradle do projeto.

Confirme no console do Liferay que os módulos necessários iniciaram sem erros. Se o bundle de Service não conseguir resolver um pacote, confira a configuração do módulo API e as dependências, conforme o tutorial completo de Service Builder.

### 4. Testar o portlet

Abra o Liferay local, adicione o portlet **H7G5** a uma página e atualize a página. Observe o console do servidor.

Você deve ver mensagens relacionadas à renderização do portlet, à contagem de pastas e à quantidade de entradas encontradas. Como o código cria uma pasta a cada renderização, a quantidade de pastas aumenta a cada atualização.

Se aparecer uma mensagem de erro na consulta remota, verifique a causa impressa depois de `Error retrieving entries:`. Uma falha de permissão não significa necessariamente que o módulo está quebrado; pode ser a regra de e-mail do exercício funcionando como previsto.

### 5. Testar JSONWS com `curl`

Primeiro, confirme no navegador quais operações estão registradas em `http://localhost:8080/api/jsonws`. Depois, execute um comando por vez para saber qual método falhou.

Você também pode tornar o script executável:

```bash
chmod +x test-h7g5-entry.sh
./test-h7g5-entry.sh
```

Rodar cada `curl` separadamente costuma facilitar o diagnóstico, especialmente enquanto você está aprendendo a diferenciar erros de URL, autenticação, parâmetros e permissões.

### 6. Interpretar os resultados

| Resultado | O que verificar |
|---|---|
| Uma ou mais entradas em JSON | A chamada foi aceita e encontrou registros |
| Lista vazia | A consulta pode ter funcionado, mas nenhum registro correspondeu aos parâmetros |
| Erro de autenticação | Usuário/senha ou configuração de autenticação |
| Método/URL não encontrado | Nome do contexto ou caminho JSONWS incorreto; confirme em `/api/jsonws` |
| Parâmetro ausente ou inválido | Nome e tipo dos parâmetros enviados com `-d` |
| Mensagem de permissão | A conta autenticada não passa na regra de e-mail do exercício |
| Erro de entidade inexistente | Uma consulta individual por `key` ou pela combinação de campos não encontrou resultado |
| Erro de conexão | O Liferay não está rodando ou a porta não é `8080` |

## 2.7 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `cannot find symbol` para `findBy...` | O `service.xml` não contém o finder esperado, ou o código gerado está desatualizado | Confira o nome do finder e execute `./gradlew h7g5-service:buildService` |
| `cannot find symbol` para `h7g5EntryPersistence` | A classe não herda da classe-base correta ou a geração/dependências estão incorretas | Confirme `extends H7G5EntryLocalServiceBaseImpl` e a estrutura do módulo |
| `cannot find symbol` para `h7g5EntryLocalService` | A classe-base remota não corresponde ao serviço ou o Service Builder não foi executado corretamente | Confira a herança de `H7G5EntryServiceBaseImpl`, a entidade e a geração |
| O Remote Service não aparece em `/api/jsonws` | Módulo não implantado, bundle não iniciado ou configuração do serviço remoto incorreta | Confira `remote-service="true"`, os logs do Liferay e as propriedades JSONWS |
| JSONWS retorna método não encontrado | Caminho digitado incorretamente | Localize o método na página `/api/jsonws` e copie a URL exibida |
| A chamada retorna lista vazia | Não há registro que corresponda ao nome ou ID enviado | Use dados que existam no banco ou crie registros para testar |
| A chamada individual gera exceção de entidade inexistente | O `key` ou a combinação de campos não corresponde a uma entrada | Confirme os valores no banco e teste com registros reais |
| A chamada termina em `401` ou erro de autenticação | Credenciais inválidas ou usuário inexistente | Confirme a conta e a senha no Liferay local |
| A regra de permissão bloqueia um teste | O e-mail não termina com `@liferay.com` | Para testar sucesso, use uma conta de treinamento com o sufixo permitido; para testar bloqueio, use uma conta válida com outro domínio |
| O portlet cria pastas repetidamente | O método `doView` cria uma pasta em toda renderização | É o comportamento demonstrativo do código; evite atualizar a página desnecessariamente durante o teste |

## Para praticar

1. Explique, sem consultar o código, a diferença entre `findByName` e `getByName`.
2. Por que `getByH7G5FolderId` devolve uma lista, enquanto `getByKey` devolve um único `H7G5Entry`?
3. O que aconteceria se `_checkPermission()` fosse retirado do Remote Service? Qual requisito de acesso deixaria de ser aplicado?
4. Por que o portlet chama `H7G5EntryService` em vez de consultar `h7g5EntryPersistence` diretamente?
5. Execute um dos comandos `curl` com um nome que não existe e compare o resultado com uma consulta que retorna registros.
6. Compare uma chamada que retorna registros com outra que produz uma lista vazia e explique a diferença entre os resultados.

---

## Referência principal

Este guia foi elaborado com base no tutorial de Service Builder do Liferay fornecido para a atividade e nos quatro arquivos de referência do primeiro PR: `H7G5EntryLocalServiceImpl.java`, `H7G5EntryServiceImpl.java`, `H7G5Portlet.java` e o script de testes `curl`. O `service.xml` é apresentado como contexto para explicar a origem dos finders e dos serviços gerados.
