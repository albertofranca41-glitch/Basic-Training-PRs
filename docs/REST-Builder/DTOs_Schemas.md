# DTOs e Schemas

## 1. O que é um DTO?

DTO significa *Data Transfer Object*, ou objeto de transferência de dados.

É um objeto utilizado para transportar dados entre diferentes partes de um sistema ou entre sistemas distintos. Sua finalidade principal é agrupar informações em uma estrutura que possa ser transmitida ou recebida por uma interface.

Por exemplo, imagine uma API que disponibiliza informações sobre um objeto `Foo`. O DTO correspondente poderia conter um identificador e um nome:

```java
public class Foo {

    private String id;
    private String name;

    // Getters e setters
}
```

Nesse exemplo, `Foo` é uma classe Java que representa um conjunto de dados. O cliente pode preencher um objeto dessa classe e enviá-lo ao servidor, ou receber um objeto preenchido como resultado de uma consulta.

DTOs não precisam representar diretamente entidades do banco de dados. Uma entidade persistida pode conter campos internos que não devem ser expostos pela API, enquanto um DTO pode apresentar apenas os dados necessários para determinada operação.

## 2. Por que utilizar DTOs?

DTOs ajudam a separar a estrutura de dados exposta por uma interface das estruturas internas utilizadas pela aplicação.

Considere um sistema que armazena um usuário com os seguintes dados:

```text
Entidade interna:
- id
- name
- email
- passwordHash
- createdAt
```

Uma API pública poderia precisar retornar somente:

```text
DTO de resposta:
- id
- name
- email
```

O DTO permite definir quais informações serão transferidas, sem exigir que toda a estrutura interna seja exposta ao cliente.

Essa separação pode facilitar a evolução da API, reduzir a exposição de informações internas e permitir que diferentes operações utilizem diferentes representações dos dados.

É importante observar que criar um DTO não garante, por si só, a segurança da aplicação. O código ainda precisa controlar quais dados são preenchidos, retornados e aceitos nas requisições.

## 3. O que é um schema?

Um *schema* é uma descrição formal da estrutura de um conjunto de dados.

No contexto de uma API, um schema pode especificar quais propriedades um objeto possui, quais são os tipos dessas propriedades e quais restrições se aplicam a elas.

Por exemplo, um schema OpenAPI poderia descrever o objeto `Foo` assim:

```yaml
Foo:
  type: object
  properties:
    id:
      type: string
    name:
      type: string
```

Essa descrição informa que `Foo` é um objeto com as propriedades `id` e `name`, ambas do tipo string.

O schema não é, nesse exemplo, uma classe Java que transporta os dados. Ele é uma descrição do formato esperado para esses dados.

Schemas também podem expressar outras características, como propriedades obrigatórias, valores permitidos e restrições de tamanho, dependendo de como forem definidos.

## 4. Qual é a diferença entre DTO e schema?

DTO e schema estão relacionados, mas não são a mesma coisa.

**DTO** é uma estrutura utilizada para representar e transportar dados no código de uma aplicação.

**Schema** é uma descrição das propriedades, dos tipos e das restrições que definem uma estrutura de dados.

Considere a seguinte comparação:

| DTO | Schema |
|---|---|
| Pode ser implementado como uma classe Java. | Pode ser descrito em YAML ou JSON. |
| É utilizado pelo código para representar dados. | Descreve o formato esperado dos dados. |
| Pode conter propriedades, métodos e outros comportamentos definidos pela implementação. | Declara a estrutura e as restrições previstas pela especificação utilizada. |
| Pode ser instanciado e preenchido durante a execução. | Serve como definição para documentação, validação ou geração de código, conforme as ferramentas utilizadas. |

Um schema e um DTO podem representar o mesmo conceito, mas isso não significa que sejam equivalentes ou que precisem ter exatamente a mesma estrutura.

Por exemplo, um schema pode declarar que uma propriedade é obrigatória, enquanto a classe Java correspondente pode não conseguir impor essa exigência sozinha. A validação dependerá dos mecanismos implementados ou gerados para esse propósito.

## 5. Como DTOs e schemas se relacionam com JSON?

JSON é um formato de representação de dados. DTOs e schemas cumprem funções diferentes ao trabalhar com esse formato.

Considere o seguinte JSON:

```json
{
  "id": "123",
  "name": "Exemplo"
}
```

Esse documento representa um objeto com duas propriedades: `id` e `name`.

Uma classe Java pode representar esses dados por meio de um DTO:

```java
Foo foo = new Foo();

foo.setId("123");
foo.setName("Exemplo");
```

Um schema pode descrever quais propriedades esse objeto JSON deve conter e quais tipos são aceitos.

Portanto, o fluxo conceitual pode ser representado assim:

```text
Schema
  |
  | Descreve a estrutura esperada
  v
DTO Java <----> JSON
```

O schema descreve a estrutura; o DTO representa os dados no código Java; o JSON representa os dados em um formato utilizado na comunicação.

A conversão entre um objeto Java e JSON é chamada de serialização quando os dados são transformados em uma representação transmissível. O processo inverso é chamado de desserialização.

Essas conversões dependem das bibliotecas e dos mecanismos utilizados pela aplicação.

## 6. DTOs e schemas no Liferay REST Builder

No REST Builder do Liferay, a descrição da API inclui schemas que definem os modelos de dados expostos pela interface.

O arquivo `rest-openapi.yaml` pode declarar esses modelos, suas propriedades e seus tipos. O REST Builder utiliza essa descrição para gerar artefatos relacionados à API, incluindo DTOs e classes utilizadas pelo cliente, conforme a configuração e a estrutura do projeto.

Isso significa que, em vez de escrever manualmente todas as classes de transferência de dados, o desenvolvedor pode definir os modelos no contrato e gerar parte do código necessário.

No projeto, é importante distinguir dois elementos:

- **Schema no contrato:** descreve a estrutura dos dados que a API aceita ou retorna.
- **DTO gerado:** é uma classe Java utilizada pelos componentes da API ou pelo cliente para representar esses dados.

Por exemplo, se o contrato define um modelo `Foo`, o REST Builder pode gerar uma classe Java correspondente para que o código utilize esse modelo de maneira tipada.

O nome exato, o pacote e os detalhes da classe gerada dependem da definição e da configuração do projeto.

## 7. DTOs não são necessariamente entidades do banco de dados

Uma distinção importante é que DTOs, schemas e entidades persistidas têm finalidades diferentes.

Uma **entidade de persistência** representa dados gerenciados pela camada de persistência. Um **DTO** transporta dados entre componentes ou através de uma interface. Um **schema** descreve a estrutura de dados prevista por um contrato.

Esses três elementos podem representar informações relacionadas, mas não precisam possuir os mesmos campos ou a mesma organização.

Por exemplo, um sistema pode ter uma entidade interna com informações adicionais, um DTO que expõe apenas parte delas e um schema que documenta a estrutura desse DTO na API.

Essa separação permite que a interface HTTP evolua sem exigir que a estrutura do banco de dados seja idêntica à estrutura exposta aos clientes.

## 8. Resumo

DTOs e schemas participam da definição e da transferência de dados, mas desempenham papéis diferentes.

- **DTO:** objeto utilizado pelo código para representar e transportar dados.
- **Schema:** descrição formal da estrutura e das restrições desses dados.
- **JSON:** formato utilizado para representar os dados durante a comunicação.
- **Entidade de persistência:** estrutura utilizada para representar dados gerenciados pela camada de persistência.

No REST Builder do Liferay, os schemas definidos no contrato da API podem servir de base para a geração de DTOs e de outros artefatos. Compreender essa relação ajuda a entender como uma definição declarativa da API se transforma em classes Java utilizáveis pelo servidor e pelo cliente.