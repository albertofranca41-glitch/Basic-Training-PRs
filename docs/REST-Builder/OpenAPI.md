# OpenAPI

## 1. O que é OpenAPI?

OpenAPI é uma especificação utilizada para descrever APIs HTTP de maneira estruturada e padronizada. Ela permite documentar como uma API pode ser utilizada sem que seja necessário analisar toda a implementação do servidor.

Uma descrição OpenAPI pode especificar:

- Os caminhos (*paths*) disponíveis na API.
- As operações HTTP aceitas em cada caminho, como `GET`, `POST`, `PUT`, `PATCH` e `DELETE`.
- Os parâmetros necessários para realizar uma requisição.
- A estrutura dos dados enviados no corpo da requisição.
- As respostas que a API pode retornar.
- Os códigos de status HTTP esperados.
- Os modelos de dados utilizados pela API.

Por exemplo, uma API pode disponibilizar uma operação para consultar um objeto `Foo` pelo seu identificador. A descrição OpenAPI pode informar o caminho dessa operação, o método HTTP utilizado, o parâmetro necessário e o formato da resposta.

Assim, OpenAPI funciona como uma descrição formal do contrato de uma API.

## 2. O que é um contrato de API?

Um contrato de API descreve como um cliente deve interagir com um servidor.

Considere uma API que disponibiliza a seguinte operação:

```http
GET /o/headless-r3b2/v1.0/foo/{fooId}
```

Esse caminho representa uma operação para consultar um `Foo` específico.

O contrato precisa esclarecer questões como:

- Qual é o tipo de requisição utilizado?
- Qual parâmetro identifica o `Foo`?
- Qual estrutura de dados será retornada?
- O que acontece quando o objeto não é encontrado?
- Quais códigos de status HTTP podem ser retornados?

Essas informações permitem que o cliente utilize a API sem precisar conhecer os detalhes internos de sua implementação.

O contrato também ajuda a manter o cliente e o servidor alinhados. Se a estrutura esperada pelo cliente não corresponder ao que o servidor disponibiliza, a integração poderá falhar.

## 3. Como uma descrição OpenAPI é organizada?

Uma descrição OpenAPI costuma ser escrita em YAML ou JSON. Esses formatos permitem representar a estrutura da API de maneira legível por pessoas e interpretável por ferramentas.

Veja um exemplo simplificado em YAML:

```yaml
openapi: 3.0.1

info:
  title: Foo API
  version: v1.0

paths:
  /foo/{fooId}:
    get:
      summary: Consulta um Foo pelo identificador
      parameters:
        - name: fooId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Foo encontrado
        '404':
          description: Foo não encontrado
```

Esse exemplo é ilustrativo. Ele demonstra a organização de uma descrição OpenAPI, não necessariamente o conteúdo exato do contrato gerado no projeto Liferay.

Os principais elementos são:

- **`openapi`**: indica a versão da especificação OpenAPI utilizada.
- **`info`**: contém informações gerais sobre a API, como título e versão.
- **`paths`**: define os caminhos disponibilizados pela API.
- **`get`**: descreve a operação HTTP disponível naquele caminho.
- **`parameters`**: descreve os parâmetros necessários para executar a operação.
- **`responses`**: documenta as respostas possíveis.

O campo `schema` descreve a estrutura ou o tipo de um dado. Nesse exemplo, o parâmetro `fooId` é uma string.

Observe que o contrato descreve o comportamento esperado da interface HTTP, mas não mostra como o servidor consulta o banco de dados ou executa suas regras internas.

## 4. O que são paths, operações e schemas?

Esses três conceitos são fundamentais para compreender uma descrição OpenAPI.

### Paths

Um *path* é o caminho utilizado para acessar um recurso ou uma funcionalidade da API.

Por exemplo:

```text
/foo
/foo/{fooId}
/foo/{fooId}/goos
/goo/{gooId}
```

Esses caminhos correspondem aos exemplos de endpoints utilizados no projeto REST Builder. O trecho `{fooId}` representa uma variável do caminho, cujo valor será fornecido pelo cliente.

### Operações

Uma operação representa uma ação HTTP disponibilizada em determinado caminho.

Por exemplo, o caminho `/foo/{fooId}` pode disponibilizar uma operação `GET` para consultar um objeto e uma operação `DELETE` para removê-lo.

O caminho identifica a parte da API que será acessada; o método HTTP identifica a operação solicitada.

### Schemas

Um *schema* descreve a estrutura de um dado.

Uma API pode, por exemplo, definir um schema para representar um `Foo`, especificando seus campos e os tipos desses campos. Esse schema ajuda a documentar o formato dos dados que o cliente envia ou recebe.

É importante distinguir o schema do objeto utilizado internamente pelo servidor: a estrutura exposta pela API não precisa ser idêntica à estrutura das entidades persistidas no banco de dados.

## 5. Qual é a relação entre OpenAPI e REST Builder?

No Liferay, o REST Builder utiliza descrições OpenAPI como parte do processo de definição e geração de uma API REST.

No módulo do projeto, o arquivo `rest-openapi.yaml` descreve aspectos do contrato da API. O REST Builder utiliza essa definição, junto à configuração do projeto, para gerar classes relacionadas à API.

Entre os artefatos que podem fazer parte desse processo estão:

- Interfaces e classes-base para a implementação dos recursos REST.
- DTOs (*Data Transfer Objects*), que representam dados transferidos pela API.
- Classes utilizadas pelo cliente para consumir a API.
- Outros artefatos necessários à estrutura da API gerada.

A geração reduz a necessidade de escrever manualmente todo o código estrutural e ajuda a manter diferentes partes da API consistentes com o contrato definido.

Entretanto, **OpenAPI e REST Builder não são a mesma coisa**:

- **OpenAPI** é a especificação utilizada para descrever o contrato da API.
- **REST Builder** é a ferramenta do Liferay que utiliza essa descrição e outras configurações para gerar artefatos relacionados à API.

A descrição OpenAPI, por si só, não implementa as regras de negócio da aplicação. O comportamento efetivo depende também do código implementado no servidor e dos demais componentes envolvidos.

## 6. OpenAPI, API Explorer e Client JAR

Esses elementos estão relacionados, mas têm funções diferentes.

**OpenAPI** descreve formalmente a API: caminhos, operações, parâmetros, schemas e respostas.

**API Explorer** é uma interface disponibilizada pelo Liferay que permite visualizar e, conforme a configuração e as permissões, experimentar operações das APIs disponíveis. No ambiente local, ela pode ser acessada em:

```text
http://localhost:8080/o/api
```

**Client JAR** é uma biblioteca Java que permite que um programa Java consuma a API por meio de classes e métodos gerados, abstraindo parte do trabalho de construir requisições HTTP e processar respostas.

No contexto do REST Builder, a descrição OpenAPI contribui para definir o contrato que serve de base para os artefatos gerados. O API Explorer apresenta as operações disponíveis, enquanto o Client JAR oferece uma forma de consumi-las a partir de código Java.

## 7. Resumo

OpenAPI é uma especificação para descrever APIs HTTP de maneira padronizada.

Seu papel é definir o contrato da API: quais caminhos existem, quais operações podem ser executadas, quais parâmetros são necessários, quais dados são aceitos e quais respostas são possíveis.

No Liferay REST Builder, essa descrição participa do processo de geração de classes relacionadas à API. Isso ajuda a estabelecer uma estrutura consistente para o servidor e para as bibliotecas clientes, mas não substitui a implementação do comportamento da aplicação.

Em outras palavras, OpenAPI descreve **como a API deve ser consumida**; o código do servidor determina **como as operações são efetivamente executadas**.