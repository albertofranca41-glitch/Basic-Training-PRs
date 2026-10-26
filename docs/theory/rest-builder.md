# REST Builder no Liferay

## 1. O que é o REST Builder?

O **REST Builder** é uma ferramenta do Liferay que gera código para a criação de APIs REST a partir de um contrato definido com a especificação **OpenAPI**.

Em vez de escrever manualmente toda a estrutura necessária para expor uma API, você descreve seus endpoints, parâmetros, modelos de dados e respostas. O REST Builder gera parte da estrutura, enquanto o desenvolvedor implementa a lógica necessária.

```text
Contrato OpenAPI
       ↓
 REST Builder
       ↓
Código gerado
       ↓
Implementação
       ↓
API disponível
```

A geração reduz o trabalho repetitivo, mas **não implementa automaticamente todas as regras de negócio da aplicação**.

---

## 2. O ciclo de geração

Em um projeto REST Builder, o arquivo `rest-openapi.yaml` descreve a API, incluindo caminhos, métodos HTTP, parâmetros, modelos de dados e respostas.

O arquivo `rest-config.yaml`, por sua vez, configura aspectos da geração, como pacotes e diretórios dos módulos.

Depois de configurar os arquivos, execute a tarefa Gradle `buildREST` a partir do módulo `impl`, ajustando o caminho do Gradle Wrapper à estrutura do seu workspace.

```bash
../gradlew buildREST
```

O processo pode ser resumido assim:

```text
rest-openapi.yaml + rest-config.yaml
                 ↓
             buildREST
                 ↓
       Classes e estruturas geradas
                 ↓
       Desenvolvedor implementa
                 ↓
       Compila e implanta os módulos
                 ↓
          API pode ser consumida
```

A tarefa `buildREST` **gera código**. Ela não deve ser confundida com a compilação, a implantação dos módulos ou a garantia de que a API esteja funcionando corretamente.

---

## 3. Os módulos `api`, `impl` e `client`

Um projeto REST Builder pode separar os componentes da API em módulos diferentes. A configuração determina quais componentes são gerados e onde são colocados.

| Módulo | Responsabilidade geral |
|---|---|
| `api` | Reúne interfaces, DTOs e outras definições compartilhadas pela API. |
| `impl` | Contém a estrutura de exposição da API e os pontos em que o desenvolvedor implementa o comportamento dos recursos. |
| `client` | Quando configurado, contém classes para facilitar o consumo da API por aplicações Java. |

### `api`

O módulo `api` contém tipos e contratos Java utilizados pelos componentes da API, como DTOs (*Data Transfer Objects*) e interfaces de recursos.

Um DTO representa dados que podem ser recebidos ou devolvidos pela API. Por exemplo, um DTO `Foo` pode ter propriedades como `id`, `name` e `description`.

### `impl`

O módulo `impl` contém a implementação dos recursos. É nele que normalmente ficam classes como `FooResourceImpl` e `GooResourceImpl`, nas quais você escreve a lógica associada às operações definidas no contrato.

Por exemplo, o contrato pode declarar uma operação para criar um `Foo`, mas é a implementação que determina como essa criação será realizada.

### `client`

O módulo `client`, quando configurado no projeto, fornece classes que facilitam o consumo da API a partir de Java. Ele pode incluir Resources, DTOs e código responsável por construir e enviar requisições HTTP.

Essa distinção é importante: **o cliente consome a API; não é ele que disponibiliza o endpoint no servidor**.

---

## 4. Resources OSGi e client JAR

É importante distinguir o Resource que participa da exposição da API no servidor do Resource cliente gerado para chamadas HTTP. Também há Resources da API disponibilizados no servidor como serviços OSGi, que podem ser obtidos por código executado dentro do próprio Liferay.

### Resource no servidor

No módulo `impl`, classes como `FooResourceImpl` implementam o comportamento dos endpoints. A estrutura gerada pelo REST Builder integra esses componentes ao ambiente do Liferay, que utiliza OSGi para modularizar e disponibilizar componentes.

```text
Requisição HTTP
      ↓
Endpoint da API no Liferay
      ↓
Estrutura de Resource
      ↓
FooResourceImpl
      ↓
Lógica da aplicação
```

### Resource cliente (client JAR)

No código consumidor, uma classe como `FooResource` pode oferecer métodos Java que representam operações da API:

```java
FooResource fooResource = FooResource.builder()
    .authentication("usuario", "senha")
    .build();

Foo createdFoo = fooResource.postFoo(newFoo);
```

Nesse exemplo, `fooResource` é o Resource cliente do client JAR. `fooResource.postFoo(newFoo)` prepara uma chamada HTTP à API; a operação é processada pelo servidor. Esse padrão é usado por programas que consomem a API pela rede.

### Resource OSGi no servidor

Em um portlet ou outro componente executado dentro do Liferay, uma Factory de Resource pode ser injetada pelo OSGi e usada para construir o Resource local:

```java
FooResource fooResource = _fooResourceFactory.create()
    .user(themeDisplay.getUser())
    .build();

Foo createdFoo = fooResource.postFoo(newFoo);
```

Aqui, `fooResource` é um Resource da API obtido via OSGi, chamado dentro da mesma JVM. A chamada de método não atravessa HTTP; o usuário é fornecido como contexto local. Essa Factory não é o cliente HTTP do client JAR.

Em resumo:

- **Resource de implementação:** participa do processamento da API no servidor.
- **Resource cliente:** usa HTTP para chamar a API, geralmente a partir de um programa externo.
- **Resource OSGi:** permite que código dentro do Liferay invoque a API localmente, na mesma JVM.
- **OSGi:** fornece a infraestrutura modular na qual os bundles e componentes do Liferay são registrados e utilizados.
- **Client JAR:** empacota classes que facilitam o consumo da API por HTTP; não substitui a API implantada no servidor.

---

## 5. Contrato OpenAPI e comportamento implementado

O contrato OpenAPI descreve a interface esperada da API. Ele pode especificar:

- caminhos e métodos HTTP;
- parâmetros de caminho, query e cabeçalhos;
- estrutura dos dados de entrada e saída;
- códigos de resposta e formatos de conteúdo.

Por exemplo, um contrato pode declarar uma operação `POST` em `/foo`, que recebe um objeto `Foo` e retorna uma resposta definida no documento.

Isso permite que o REST Builder gere estruturas coerentes com a descrição. Porém, há uma diferença entre **descrever uma operação** e **implementar corretamente seu comportamento**.

```text
OpenAPI
   │
   ├── descreve o endpoint
   ├── define parâmetros e schemas
   └── declara respostas esperadas
             ↓
       REST Builder
             ↓
       Estrutura gerada
             ↓
       Implementação do desenvolvedor
             ↓
       Comportamento efetivo
```

O contrato não garante, por si só, que a lógica esteja correta, que os dados sejam persistidos, que as permissões estejam adequadas ou que todas as respostas documentadas ocorram como esperado. Esses aspectos dependem da implementação e da configuração da aplicação, além dos mecanismos fornecidos pela plataforma.

Por isso, após gerar o código, ainda é necessário implementar, compilar, implantar e testar a API.

---

## 6. O que guardar sobre REST Builder

> **REST Builder é uma ferramenta do Liferay que gera a estrutura de APIs a partir de um contrato OpenAPI. Os módulos separam definições compartilhadas, implementação no servidor e, quando configurado, código cliente. A geração reduz código repetitivo, mas não substitui a implementação, a implantação nem os testes.**

Guarde especialmente estas distinções:

- **OpenAPI:** descreve o contrato da API.
- **`buildREST`:** executa a geração de código.
- **`api`:** reúne tipos e interfaces compartilhados.
- **`impl`:** contém a implementação dos recursos no servidor.
- **`client`:** facilita o consumo da API por código cliente.
- **Resource no servidor e Resource cliente:** têm funções diferentes.
- **Código gerado e comportamento real:** não são a mesma coisa.

## Referência

- [Documentação oficial do Liferay — REST Builder](https://learn.liferay.com/w/dxp/integration/rest-builder)
