# JSON (JavaScript Object Notation)

O **JSON (JavaScript Object Notation)** é um formato textual utilizado para representar e transportar dados estruturados entre sistemas.

Ele é muito utilizado em APIs HTTP porque permite que aplicações desenvolvidas em linguagens diferentes troquem informações utilizando uma estrutura padronizada.

Apesar de seu nome fazer referência ao JavaScript, JSON não é exclusivo dessa linguagem. Java, Python, JavaScript e diversas outras linguagens possuem bibliotecas capazes de interpretar e gerar JSON.

## 1. Por que o JSON existe?

Imagine que um programa Java precise enviar informações sobre um usuário para um servidor desenvolvido em Python.

Cada linguagem possui suas próprias estruturas de dados e suas próprias regras de representação. Um objeto Java não pode simplesmente ser enviado pela rede como se fosse automaticamente um objeto Python.

Para resolver esse problema, os sistemas podem utilizar um **formato de intercâmbio de dados**: uma representação que ambos conseguem interpretar.

O JSON cumpre esse papel.

Por exemplo, um programa pode representar um usuário desta maneira:

```json
{
  "name": "Ana",
  "age": 25,
  "active": true
}
```

Esse conteúdo é texto. O sistema que recebe o JSON interpreta sua estrutura e pode convertê-lo para os tipos de dados apropriados à linguagem utilizada.

O JSON, portanto, não é o próprio objeto da aplicação. É uma representação dos dados que podem ser transmitidos e reconstruídos pelo sistema destinatário.

## 2. A estrutura do JSON

O JSON possui seis categorias de valores:

- Objetos.
- Arrays.
- Strings.
- Números.
- Valores booleanos (`true` e `false`).
- O valor `null`.

Essas categorias podem ser combinadas para representar estruturas simples ou complexas.

### 2.1 Objetos

Um objeto JSON é uma coleção de pares formados por **chave e valor**.

```json
{
  "name": "Ana",
  "age": 25,
  "active": true
}
```

Nesse exemplo:

- `"name"` é uma chave cujo valor é `"Ana"`.
- `"age"` é uma chave cujo valor é `25`.
- `"active"` é uma chave cujo valor é `true`.

As chaves de um objeto JSON precisam ser strings delimitadas por aspas duplas.

Os valores podem ser strings, números, booleanos, `null`, outros objetos ou arrays.

### 2.2 Arrays

Um array JSON representa uma sequência ordenada de valores.

```json
{
  "subjects": [
    "Filosofia",
    "Programação",
    "Teologia"
  ]
}
```

O campo `"subjects"` contém um array com três strings.

Um array também pode conter objetos:

```json
{
  "subjects": [
    {
      "id": 1,
      "name": "Filosofia"
    },
    {
      "id": 2,
      "name": "Programação"
    }
  ]
}
```

Essa estrutura é útil para representar listas de registros retornados por uma API.

### 2.3 Strings

Strings representam valores textuais e precisam ser delimitadas por aspas duplas.

```json
{
  "name": "Ana",
  "description": "Estudante de programação"
}
```

JSON não permite utilizar aspas simples para delimitar strings.

Por exemplo, isto não é um JSON válido:

```javascript
{
  'name': 'Ana'
}
```

Para representar aspas duplas ou determinados caracteres especiais dentro de uma string, é necessário utilizar as regras de escape do JSON.

### 2.4 Números

JSON permite representar números inteiros e números com parte fracionária.

```json
{
  "age": 25,
  "price": 19.90,
  "temperature": -2.5
}
```

O separador decimal é o ponto, e não a vírgula.

JSON não possui um tipo específico para datas. Uma data pode ser representada como string, utilizando um formato convencionado pela aplicação.

Por exemplo:

```json
{
  "createdAt": "2026-10-17T18:30:00Z"
}
```

Nesse caso, o valor é uma string. Interpretá-lo como data e horário depende do sistema que consome o conteúdo e das convenções definidas pela API.

### 2.5 Booleanos e `null`

Os valores booleanos são escritos em letras minúsculas:

```json
{
  "active": true,
  "deleted": false
}
```

O valor `null` representa a ausência de um valor naquele campo, conforme o significado definido pela aplicação.

```json
{
  "middleName": null
}
```

É importante não confundir `null` com uma string vazia (`""`), o número `0` ou o booleano `false`. São valores diferentes.

Além disso, omitir uma propriedade de um objeto não é necessariamente equivalente a incluí-la com valor `null`. O significado depende do contrato e do comportamento da aplicação.

## 3. Regras de sintaxe

Um JSON válido precisa respeitar regras específicas.

### Aspas duplas

Chaves e strings precisam utilizar aspas duplas.

```json
{
  "name": "Ana"
}
```

### Separação dos elementos

Vírgulas separam os pares de um objeto e os elementos de um array.

Não se utiliza uma vírgula adicional depois do último elemento.

Válido:

```json
{
  "name": "Ana",
  "age": 25
}
```

Inválido:

```javascript
{
  "name": "Ana",
  "age": 25,
}
```

### Valores permitidos

JSON não aceita expressões de programação, comentários ou valores específicos de linguagens que não pertençam à sua sintaxe.

Por exemplo, `undefined`, `NaN` e `Infinity` não são valores JSON válidos.

O conteúdo abaixo também é inválido:

```javascript
{
  name: "Ana",
  // Nome do usuário
  age: 25
}
```

O problema não está nos dados em si, mas no fato de que a estrutura não respeita a sintaxe JSON.

## 4. JSON e objetos JavaScript

JSON e objetos JavaScript são conceitos relacionados, mas diferentes.

Um **objeto JavaScript** é uma estrutura de dados que existe durante a execução de um programa JavaScript.

O **JSON** é uma representação textual que segue uma especificação própria.

Considere um objeto JavaScript:

```javascript
const user = {
  name: "Ana",
  age: 25,
  active: true
};
```

O programa pode converter esse objeto para uma string JSON:

```javascript
const json = JSON.stringify(user);

console.log(json);
```

O resultado é uma string semelhante a:

```json
{"name":"Ana","age":25,"active":true}
```

A função `JSON.stringify()` realiza a **serialização** do objeto para uma representação textual JSON.

O caminho inverso pode ser realizado com `JSON.parse()`:

```javascript
const json = '{"name":"Ana","age":25,"active":true}';

const user = JSON.parse(json);

console.log(user.name);
```

O resultado será:

```text
Ana
```

Nesse caso, `JSON.parse()` interpreta a string JSON e cria uma estrutura de dados JavaScript correspondente.

Em resumo:

| Operação | Função | Resultado |
|---|---|---|
| Serialização | `JSON.stringify()` | String JSON |
| Análise do JSON | `JSON.parse()` | Valor JavaScript correspondente |

A conversão não significa que qualquer valor JavaScript possa ser representado fielmente em JSON. Por exemplo, funções e propriedades com determinados valores podem ser omitidas ou transformadas durante a serialização.

## 5. JSON e Java

JSON não depende do JavaScript. Um programa Java também pode representar dados por meio de classes e convertê-los para JSON utilizando bibliotecas apropriadas.

Imagine uma classe Java:

```java
public class User {

    private String name;
    private int age;
    private boolean active;

    // Construtores, getters e setters
}
```

Uma biblioteca de serialização pode converter uma instância dessa classe para uma representação JSON, como:

```json
{
  "name": "Ana",
  "age": 25,
  "active": true
}
```

O processo inverso também pode ocorrer: uma biblioteca interpreta o JSON e preenche um objeto Java compatível com os dados recebidos.

A conversão depende da biblioteca, da configuração utilizada e das regras aplicadas aos campos.

Portanto, **JSON não é uma classe Java nem um tipo de objeto Java**. É o formato de dados utilizado na comunicação.

## 6. JSON em uma API HTTP

JSON é frequentemente utilizado no corpo das requisições e respostas de APIs HTTP.

Imagine que um cliente queira criar um recurso `Foo` na API do Liferay.

Uma requisição ilustrativa poderia ser:

```http
POST /o/headless-r3b2/v1.0/foo HTTP/1.1
Host: localhost:8080
Content-Type: application/json
```

Corpo da requisição:

```json
{
  "name": "Primeiro Foo",
  "description": "Um recurso de exemplo"
}
```

O cabeçalho `Content-Type: application/json` informa que o corpo enviado está no formato JSON.

O servidor recebe a requisição, interpreta os dados de acordo com o contrato da API e executa o comportamento implementado para a operação.

A resposta pode conter o recurso criado, incluindo um identificador atribuído pelo servidor. O formato e o código de status exatos dependem do contrato da API.

### E o JSON nas respostas?

O servidor também pode retornar dados em JSON.

Uma resposta ilustrativa seria:

```http
HTTP/1.1 200 OK
Content-Type: application/json
```

Corpo:

```json
{
  "id": 123,
  "name": "Primeiro Foo",
  "description": "Um recurso de exemplo"
}
```

O cliente pode interpretar o corpo e utilizar os dados retornados.

O código de status, os campos e o formato efetivo da resposta devem ser consultados na documentação da API.

## 7. O papel do JSON no client JAR do REST Builder

Na atividade de REST Builder, você utiliza classes Java como `Foo`, `Goo`, `FooResource` e `GooResource`.

Por exemplo:

```java
Foo foo = new Foo();

foo.setName("Primeiro Foo");
foo.setDescription("Um recurso de exemplo");

fooResource.postFoo(foo);
```

O código trabalha com um objeto Java, não com uma string JSON escrita manualmente.

O cliente gerado pelo REST Builder abstrai parte do trabalho de comunicação: de acordo com a implementação gerada, ele prepara a requisição HTTP, converte os dados para o formato esperado e interpreta a resposta.

O fluxo conceitual é:

```text
Objeto Java: Foo
       |
       v
Client JAR
       |
       v
Serialização para JSON
       |
       v
Requisição HTTP
       |
       v
API REST do Liferay
       |
       v
Processamento no servidor
       |
       v
Resposta HTTP com JSON, quando aplicável
       |
       v
Client JAR interpreta a resposta
       |
       v
Objeto Java: Foo
```

Esse diagrama representa o fluxo geral. Os detalhes de serialização, tratamento de erros e conversão dependem do código gerado e da operação utilizada.

A vantagem é que o programa cliente trabalha com tipos e métodos Java, sem precisar montar manualmente cada corpo JSON.

Entretanto, o JSON continua fazendo parte da comunicação entre o cliente e o servidor. O client JAR apenas abstrai os detalhes necessários para o consumidor.

## 8. JSON não define as regras de negócio

JSON define como os dados são representados, mas não determina o que a aplicação deve fazer com eles.

Considere:

```json
{
  "name": "",
  "age": -5
}
```

Esse conteúdo pode ser sintaticamente válido como JSON, mesmo que uma aplicação decida rejeitá-lo por causa de regras de validação.

É necessário distinguir:

- **Validade sintática:** o conteúdo respeita a especificação JSON?
- **Validade estrutural:** o conteúdo corresponde ao schema esperado pela API?
- **Validade de negócio:** os dados são permitidos pelas regras da aplicação?

Uma requisição pode conter JSON válido e, ainda assim, ser rejeitada pelo servidor.

Da mesma maneira, o schema da API pode definir campos e tipos permitidos sem descrever sozinho todas as regras de negócio envolvidas.

## 9. Resumo

- JSON é um formato textual para representar e transportar dados estruturados.
- Ele é independente da linguagem de programação utilizada.
- Seus valores incluem objetos, arrays, strings, números, booleanos e `null`.
- Objetos JSON utilizam pares de chave e valor; arrays representam sequências ordenadas.
- JSON possui regras de sintaxe próprias, como o uso de aspas duplas para strings.
- Em JavaScript, `JSON.stringify()` serializa dados para uma string JSON e `JSON.parse()` interpreta uma string JSON.
- Em Java, bibliotecas podem converter objetos para JSON e fazer a conversão inversa.
- Em APIs HTTP, JSON costuma ser utilizado no corpo das requisições e respostas.
- No REST Builder, o client JAR permite trabalhar com objetos Java enquanto abstrai parte da comunicação HTTP e da conversão dos dados.
- JSON não define, por si só, a estrutura completa da API nem as regras de negócio da aplicação.
