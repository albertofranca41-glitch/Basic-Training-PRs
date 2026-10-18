# Introdução ao Service Builder do Liferay

> **Objetivo deste material:** entender o que é o Service Builder, quais
> problemas ele resolve e como uma requisição pode percorrer as camadas
> da aplicação até consultar ou modificar dados no banco de dados.

## 1. Visão geral

O **Service Builder** é uma ferramenta de geração de código do Liferay.
A partir de uma definição declarativa, normalmente escrita no arquivo
`service.xml`, ele gera partes importantes da estrutura de um módulo de
backend, como:

-   **Modelo (Model):** objetos Java que representam os dados da
    aplicação.
-   **Persistência (Persistence):** código que permite criar, consultar,
    atualizar e excluir dados.
-   **Serviços (Services):** pontos de entrada para operações da
    aplicação, onde podemos implementar regras de negócio.

Em vez de escrever manualmente todo o código repetitivo de acesso a
dados e integração entre camadas, o desenvolvedor descreve as entidades
e algumas configurações; o Service Builder gera a estrutura inicial. O
código gerado pode ser estendido nas classes de implementação destinadas
à customização.

Documentação oficial: [Service Builder --- Liferay
Learn](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder).

## 2. Uma analogia simples

Imagine uma aplicação que gerencia livros de uma biblioteca.

-   O **modelo** descreve o que é um livro: identificador, título e
    autor.
-   A **persistência** sabe como localizar e gravar livros no banco.
-   O **serviço** coordena uma operação, por exemplo, validar os dados e
    cadastrar um livro.
-   O **banco de dados** mantém os dados armazenados de forma
    persistente.

O Service Builder gera boa parte da estrutura que conecta essas
responsabilidades. O desenvolvedor concentra sua atenção nas regras
específicas da aplicação.

## 3. Conceitos fundamentais

### 3.1 Entidade (Entity)

Uma **entidade** representa um tipo de informação relevante para o
domínio da aplicação. Em um sistema de biblioteca, exemplos seriam
`Book`, `Author` e `Loan`.

No Service Builder, as entidades são declaradas no `service.xml`. Uma
entidade com colunas normalmente corresponde a um modelo Java e a uma
tabela no banco de dados, conforme o mapeamento configurado.

Exemplo conceitual:

  Campo      Significado
  ---------- ------------------------------
  `bookId`   Identificador único do livro
  `title`    Título
  `author`   Autor

Uma entidade não é exatamente a mesma coisa que uma tabela: a entidade é
o conceito do modelo da aplicação; a tabela é a estrutura relacional
usada para armazenar os dados. O mapeamento conecta as duas coisas.

### 3.2 Modelo (Model)

O **modelo** é a representação do dado dentro do código Java. Ele
permite que o restante da aplicação trabalhe com objetos e propriedades,
em vez de precisar manipular diretamente linhas e colunas SQL em toda
operação.

Por exemplo, uma aplicação pode receber um objeto `Book` cujo título é
`"Arquitetura de Software"`. A camada de persistência fica responsável
por mapear esse objeto para a estrutura relacional correspondente.

### 3.3 Persistência (Persistence)

**Persistência** é a capacidade de guardar dados de forma que eles
continuem disponíveis depois que a requisição termina --- e,
normalmente, depois que a aplicação é reiniciada.

No Service Builder, a camada de persistência gerada oferece operações
para armazenar e recuperar entidades. Ela encapsula detalhes do acesso
ao banco de dados. O Service Builder usa mecanismos de mapeamento
objeto-relacional (ORM), com Hibernate na implementação tradicional
documentada pelo Liferay.

Operações típicas incluem:

-   **Create:** criar um registro.
-   **Read:** consultar registros.
-   **Update:** atualizar um registro.
-   **Delete:** excluir um registro.

Esse conjunto é frequentemente chamado de **CRUD**.

### 3.4 Serviço (Service)

Um **serviço** oferece operações que outras partes do sistema podem
invocar. Em vez de fazer a tela ou o controlador acessar diretamente o
banco, eles chamam um serviço que representa uma ação da aplicação.

Exemplos:

-   `addBook(...)`: cadastrar um livro.
-   `getBook(bookId)`: buscar um livro.
-   `searchBooks(...)`: pesquisar livros.

O serviço funciona como uma camada intermediária entre quem solicita uma
operação e os detalhes de persistência.

### 3.5 Regras de negócio (Business Logic)

As **regras de negócio** definem o que é permitido ou obrigatório no
contexto da aplicação.

No exemplo da biblioteca, regras possíveis seriam:

-   o título do livro não pode estar vazio;
-   um empréstimo só pode ser criado se o livro estiver disponível;
-   uma devolução não pode ser registrada duas vezes para o mesmo
    empréstimo.

Essas regras devem ser implementadas na camada de serviço apropriada,
normalmente na implementação do serviço local, como
`BookLocalServiceImpl`, quando a regra pertence à lógica interna da
aplicação.

A persistência responde principalmente a perguntas como "como salvar?" e
"como consultar?". A regra de negócio responde a "essa operação é
válida?" e "quais passos devem acontecer?".

### 3.6 Serviço local e serviço remoto

O Service Builder pode gerar diferentes tipos de serviço:

-   **Local service:** chamado por código que roda dentro da mesma JVM
    do Liferay. É frequentemente usado por portlets, módulos e outros
    componentes do servidor.
-   **Remote service:** quando configurado, oferece uma camada destinada
    a chamadas remotas e pode incluir verificações de permissões. Não
    deve ser confundido automaticamente com uma API REST criada com REST
    Builder.

Nem toda aplicação precisa expor serviços remotos. É possível usar
serviços locais para organizar a lógica interna.

### 3.7 Transação

Uma **transação** agrupa operações de banco de dados para que sejam
tratadas como uma unidade. Por exemplo, ao registrar um empréstimo, o
sistema pode precisar criar o empréstimo e atualizar a disponibilidade
do livro.

Se a operação falhar no meio, a transação pode permitir que as
alterações sejam revertidas, conforme a configuração e os limites
transacionais envolvidos. Isso ajuda a evitar estados inconsistentes.

### 3.8 API e implementação

É útil distinguir:

-   **Interface/API:** define quais operações podem ser chamadas.
-   **Implementação:** contém o código que executa essas operações.

No Service Builder, muitas interfaces e classes-base são geradas
automaticamente. As classes `*LocalServiceImpl` e, quando aplicável,
`*ServiceImpl` são pontos comuns para adicionar lógica personalizada.
Depois de alterar a implementação e adicionar operações, normalmente é
necessário executar novamente a geração do Service Builder para propagar
as mudanças às interfaces geradas.

**Importante:** não altere indiscriminadamente arquivos gerados. Prefira
as classes de implementação destinadas à customização e siga as
convenções da versão do Liferay utilizada.

## 4. O papel do `service.xml`

O arquivo `service.xml` descreve o modelo que o Service Builder deve
gerar. Nele, normalmente definimos informações como:

-   o nome da entidade;
-   suas colunas e tipos;
-   qual coluna é a chave primária;
-   se serviços locais ou remotos devem ser gerados;
-   configurações adicionais, como métodos de busca (*finders*).

Exemplo simplificado e ilustrativo:

``` xml
<?xml version="1.0"?>
<!DOCTYPE service-builder PUBLIC
    "-//Liferay//DTD Service Builder 7.4.0//EN"
    "http://www.liferay.com/dtd/liferay-service-builder_7_4_0.dtd">

<service-builder
    dependency-injector="ds"
    package-path="com.exemplo.biblioteca">

    <namespace>BIBLIO</namespace>

    <entity
        name="Book"
        local-service="true"
        remote-service="false">

        <column name="bookId" type="long" primary="true" />
        <column name="title" type="String" />
        <column name="author" type="String" />
    </entity>
</service-builder>
```

Este exemplo ilustra a ideia, não um projeto completo pronto para
produção. Dependências, metadados do módulo e convenções podem variar
conforme a versão e a estrutura do projeto.

Em termos simples, essa definição informa ao Service Builder que existe
uma entidade `Book` com identificador, título e autor, e que deve ser
gerado um serviço local. A partir dessa definição, a ferramenta gera
código de modelo, serviço e persistência.

## 5. Como uma requisição HTTP chega ao banco de dados

Considere um usuário que clica em "Cadastrar livro" numa interface web.
O navegador envia uma requisição HTTP ao servidor Liferay.

O fluxo exato depende da tecnologia de entrada utilizada --- por
exemplo, um portlet MVC, uma API JAX-RS ou uma API construída com REST
Builder. O diagrama abaixo mostra um fluxo conceitual comum, não uma
sequência obrigatória para todos os projetos.

``` text
Navegador / Cliente
        |
        | 1. Requisição HTTP
        v
Liferay: ponto de entrada HTTP
        |
        | 2. Roteamento para o componente responsável
        v
Portlet, endpoint ou controlador
        |
        | 3. Chamada a uma operação de serviço
        v
Camada de serviço
(LocalService / lógica de negócio)
        |
        | 4. Validar dados e aplicar regras
        v
Camada de persistência
        |
        | 5. Mapear operação para acesso a dados
        v
ORM / acesso ao banco
        |
        | 6. SQL, quando necessário
        v
Banco de dados
        |
        | 7. Resultado da operação
        v
Persistência -> Serviço -> Controlador/endpoint
        |
        | 8. Resposta HTTP
        v
Navegador / Cliente
```

### Etapa 1 --- O cliente envia a requisição

O navegador envia uma requisição HTTP com os dados necessários. Em um
formulário de cadastro, por exemplo, os dados podem incluir o título e o
autor do livro.

HTTP é o protocolo de comunicação entre cliente e servidor. Ele não
determina, por si só, como o código Java será organizado internamente.

### Etapa 2 --- O Liferay encaminha a requisição

O servidor recebe a requisição e a encaminha ao componente responsável,
de acordo com o tipo de endpoint e a configuração da aplicação.

Esse componente pode ser um portlet, um endpoint REST ou outra entrada
da aplicação. Autenticação, autorização, validação de entrada e outros
filtros podem participar do fluxo, dependendo da configuração.

### Etapa 3 --- O componente chama um serviço

O controlador ou endpoint chama uma operação do backend, por exemplo, um
método de serviço para cadastrar o livro.

A ideia é evitar que o componente HTTP concentre todas as regras de
negócio e os detalhes do banco. Ele coordena a entrada e a resposta; o
serviço executa a operação da aplicação.

### Etapa 4 --- O serviço aplica as regras de negócio

O serviço verifica se os dados e a operação respeitam as regras
definidas. Pode validar campos, conferir permissões ou consultar outras
informações necessárias.

Se uma regra falhar, a operação pode terminar com uma exceção ou um
resultado de erro, conforme a implementação. Nesse caso, a gravação pode
nem chegar a acontecer.

### Etapa 5 --- A persistência executa a operação de dados

Se a operação for válida, o serviço utiliza a camada de persistência
para criar ou atualizar a entidade.

A persistência abstrai boa parte do trabalho de acesso a dados. Em vez
de escrever SQL manual para toda operação simples, o código pode usar os
métodos gerados pelo Service Builder.

### Etapa 6 --- O banco de dados armazena ou consulta os dados

A camada de acesso a dados/ORM traduz a operação para comandos adequados
ao banco configurado. Para uma inserção, por exemplo, o banco pode
executar um `INSERT`; para uma consulta, um `SELECT`.

A persistência não significa necessariamente que cada chamada resulte
imediatamente em um comando SQL: caches, contexto transacional e outras
otimizações podem afetar o comportamento observado.

### Etapa 7 --- O resultado retorna pelas camadas

O resultado volta da persistência para o serviço. O serviço pode
retornar a entidade criada, um identificador ou outro resultado, ou
lançar uma exceção caso ocorra um problema.

O componente de entrada trata esse resultado e decide como representá-lo
para o cliente.

### Etapa 8 --- O servidor responde por HTTP

Por fim, o endpoint ou controlador transforma o resultado em uma
resposta HTTP, como uma página, um redirecionamento ou um documento
JSON. O navegador recebe a resposta e atualiza a interface conforme o
caso.

**Ideia essencial:** HTTP é a comunicação entre o cliente e o servidor.
Service Builder organiza e gera partes do backend que podem ser usadas
durante o processamento dessa requisição. O Service Builder não é, por
si só, o protocolo HTTP nem necessariamente o endpoint que o cliente
chama.

## 6. Onde cada responsabilidade deve ficar?

  -----------------------------------------------------------------------
  Componente              Responsabilidade        Exemplo
                          principal               
  ----------------------- ----------------------- -----------------------
  Interface / cliente     Coletar dados e         Formulário de livro
                          apresentar resultados   

  Controlador / endpoint  Receber a entrada e     Interpretar o pedido
                          preparar a resposta     HTTP

  Serviço                 Coordenar a operação e  Impedir cadastro sem
                          aplicar regras de       título
                          negócio                 

  Modelo / entidade       Representar os dados da `Book` com título e
                          aplicação               autor

  Persistência            Consultar e gravar      Buscar ou atualizar um
                          entidades               livro

  Banco de dados          Armazenar os dados      Tabela de livros
  -----------------------------------------------------------------------

Essa separação é conhecida como **separação de responsabilidades**. Ela
facilita a manutenção: uma alteração na interface não precisa exigir
reescrever a persistência, e uma regra de negócio pode ser testada sem
ficar misturada ao código que recebe HTTP.

## 7. Um exemplo conceitual de regra de negócio

Imagine que a aplicação não permita cadastrar um livro sem título. De
forma ilustrativa, a lógica poderia ser semelhante a esta:

``` java
public Book addBook(String title, String author)
    throws Exception {

    if (title == null || title.trim().isEmpty()) {
        throw new IllegalArgumentException(
            "O título é obrigatório"
        );
    }

    // A persistência é usada depois da validação.
    // Os métodos reais dependem do código gerado
    // e da versão/configuração do projeto.

    // Criar/preencher a entidade e persistir o livro.
    // ...

    return book;
}
```

O trecho não é uma implementação completa para copiar e executar. Ele
serve para demonstrar a ordem do raciocínio:

1.  Receber os dados.
2.  Validar a regra.
3.  Criar ou obter a entidade.
4.  Persistir os dados.
5.  Retornar o resultado.

Em um módulo real, a implementação deve usar os métodos e as convenções
gerados pelo Service Builder. Não basta escrever uma classe com esse
método para que ela se integre automaticamente ao módulo.

## 8. O que o Service Builder gera e o que o desenvolvedor escreve?

### Normalmente gerado

-   classes e interfaces do modelo;
-   interfaces e implementações-base de serviços;
-   classes de persistência e métodos comuns de consulta;
-   infraestrutura de mapeamento e integração com o ambiente Liferay.

### Normalmente implementado ou personalizado pelo desenvolvedor

-   regras de negócio específicas;
-   operações próprias da aplicação;
-   validações e coordenação de chamadas a outros serviços;
-   consultas personalizadas quando os métodos gerados não são
    suficientes.

O conjunto exato de classes e arquivos depende da versão do Liferay e
das opções definidas no `service.xml`.

## 9. Erros comuns de entendimento

-   **"Service Builder é o banco de dados."** Não. Ele gera código que
    ajuda a aplicação a trabalhar com os dados.
-   **"A entidade e a tabela são a mesma coisa."** São conceitos
    relacionados, mas diferentes: a entidade pertence ao modelo da
    aplicação; a tabela é uma estrutura de armazenamento relacional.
-   **"Toda regra de negócio deve ficar no controlador HTTP."** Em
    geral, regras de negócio reutilizáveis devem ficar na camada de
    serviço apropriada, e não presas a uma única forma de entrada.
-   **"A persistência decide se uma operação faz sentido para o
    negócio."** A persistência cuida principalmente do acesso a dados;
    regras como disponibilidade, elegibilidade e validações de domínio
    pertencem à lógica de negócio.
-   **"Serviço remoto é sinônimo de REST Builder."** Não. São mecanismos
    diferentes de exposição de serviços.
-   **"Tudo sempre passa por todas as camadas exatamente como no
    diagrama."** O diagrama é conceitual. O fluxo real varia com o tipo
    de endpoint, os serviços chamados, as transações e as configurações
    do projeto.

## 10. Resumo mental

Guarde esta sequência:

1.  **`service.xml` descreve** entidades e configurações.
2.  **Service Builder gera** partes do modelo, da persistência e dos
    serviços.
3.  **O endpoint recebe** a requisição e chama o backend.
4.  **O serviço aplica** regras de negócio e coordena a operação.
5.  **A persistência acessa** os dados.
6.  **O banco armazena ou retorna** informações.
7.  **O resultado volta** até o endpoint, que responde ao cliente.

Uma frase para lembrar:

> **O serviço decide e coordena a operação de negócio; a persistência
> cuida do acesso aos dados; o banco mantém os dados armazenados.**

## 11. Perguntas para revisar

1.  Qual é o papel do arquivo `service.xml`?
2.  Qual é a diferença entre entidade e tabela?
3.  O que significa persistência?
4.  Por que colocar regras de negócio na camada de serviço?
5.  Qual é a diferença entre um serviço local e um serviço remoto?
6.  O Service Builder é responsável por receber diretamente toda
    requisição HTTP?
7.  Em qual camada você esperaria encontrar a operação de gravação no
    banco?

## 12. Fontes oficiais para continuar estudando

-   [Service Builder --- visão
    geral](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder)
-   [Gerando código de modelo, persistência e
    serviço](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder/service-builder-basics/generating-model-persistence-and-service-code)
-   [Definindo
    entidades](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder/defining-entities)
-   [Lógica de negócio com Service
    Builder](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder/business-logic-with-service-builder)

------------------------------------------------------------------------

**Nota de versão:** os nomes, comandos de build, classes geradas e
opções de configuração podem variar entre versões do Liferay. Este
material apresenta os conceitos gerais e usa um exemplo de configuração
da família Liferay DXP 7.4. Consulte a documentação correspondente à
versão do seu projeto antes de aplicar os exemplos.
