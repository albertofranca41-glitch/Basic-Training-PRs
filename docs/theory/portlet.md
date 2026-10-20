# Portlet: Conceito, funcionamento e aplicação no Liferay

## 1. O que é um Portlet?

Um **Portlet** é um componente de uma aplicação web que produz conteúdo ou oferece funcionalidades dentro de uma página de um portal.

Diferentemente de uma aplicação web tradicional, que normalmente controla sua própria página inteira, um portlet é projetado para funcionar dentro de uma estrutura maior, compartilhando a página com outros componentes.

No contexto do Liferay, os portlets são uma das principais formas de desenvolver interfaces e funcionalidades personalizadas para o portal.

Por exemplo, uma página do Liferay pode conter simultaneamente:

- Um portlet que apresenta notícias.
- Um portlet que exibe documentos.
- Um portlet que permite consultar informações de usuários.
- Um portlet personalizado que lista registros de uma aplicação desenvolvida pela equipe.

Cada componente possui sua própria responsabilidade, mas todos podem coexistir na mesma página.

### 1.1. A diferença entre uma página e um portlet

É importante distinguir esses dois conceitos:

- **Página:** é o espaço que organiza a navegação e os componentes apresentados ao usuário.
- **Portlet:** é um componente que apresenta conteúdo ou disponibiliza funcionalidades dentro desse espaço.

Imagine uma página do Liferay destinada ao gerenciamento de estudos:

```text
Página: Meus Estudos
│
├── Portlet de Disciplinas
│   ├── Filosofia
│   ├── Programação
│   └── Teologia
│
├── Portlet de Anotações Recentes
│   ├── Classes em Python
│   └── Metafísica
│
└── Portlet de Pesquisa
    └── Campo para pesquisar anotações
```

Nesse exemplo, a página organiza a interface, enquanto cada portlet implementa uma funcionalidade específica.

A página não precisa conhecer todos os detalhes internos de cada componente. O portlet é responsável por sua própria funcionalidade, e o Liferay gerencia a composição dos componentes na página.

Essa separação favorece a modularidade e a reutilização.

## 2. Por que os portlets existem?

Para compreender a finalidade de um portlet, considere o desenvolvimento de uma aplicação web convencional.

Em uma aplicação tradicional, o desenvolvedor pode precisar implementar ou integrar funcionalidades como:

- Autenticação de usuários.
- Navegação.
- Controle de acesso.
- Organização das páginas.
- Componentes de interface.
- Persistência de dados.

Em um portal como o Liferay, muitas dessas capacidades já são fornecidas pela própria plataforma.

O desenvolvedor pode concentrar seus esforços na funcionalidade que precisa construir, aproveitando os recursos oferecidos pelo portal.

Por exemplo, ao desenvolver um portlet para consultar documentos, você pode utilizar os mecanismos de autenticação e permissões do Liferay em vez de criar um sistema de usuários completamente independente.

Isso não significa que o portlet não precise implementar suas próprias regras de negócio ou verificações de autorização. Significa que ele pode integrar-se aos recursos da plataforma.

### 2.1. Portlets como componentes reutilizáveis

Um portlet pode ser disponibilizado para ser adicionado a diferentes páginas, desde que sua configuração e as permissões aplicáveis permitam isso.

Considere um portlet que apresenta uma lista de documentos.

Ele pode ser utilizado em uma página interna de uma organização e também em uma página destinada a uma equipe específica.

A implementação da funcionalidade pode ser reaproveitada, embora os dados exibidos, as permissões e as configurações possam variar conforme o contexto.

Essa é uma das características fundamentais dos portlets: desenvolver uma funcionalidade que possa ser integrada a diferentes páginas de um portal.

## 3. O Portlet dentro da arquitetura do Liferay

O Liferay não é apenas um servidor que hospeda páginas HTML. Ele é uma plataforma que oferece serviços, recursos administrativos, gerenciamento de conteúdo e uma arquitetura modular para aplicações.

Os portlets fazem parte dessa arquitetura.

Para entender sua posição, é útil distinguir alguns conceitos.

| Conceito | Responsabilidade |
|---|---|
| Liferay Portal | Fornece a plataforma, as páginas, a navegação, os usuários e os recursos compartilhados. |
| Portlet | Implementa uma funcionalidade de interface integrada a uma página do portal. |
| Módulo OSGi | Empacota e disponibiliza código e componentes no ambiente modular do Liferay. |
| Controller | Coordena o processamento de uma solicitação recebida pelo portlet. |
| View | Apresenta o conteúdo ao usuário. |
| Service | Executa operações e regras de negócio da aplicação. |
| Repository ou camada de persistência | Interage com o armazenamento de dados, conforme a arquitetura adotada. |

Esses conceitos estão relacionados, mas não são sinônimos.

Um portlet pode utilizar serviços da aplicação, e esses serviços podem acessar uma camada de persistência. A interface, por sua vez, apresenta os resultados ao usuário.

### 3.1. Portlet e módulo OSGi

No Liferay 7.4, os módulos desenvolvidos para a plataforma são normalmente executados no ambiente modular OSGi.

**OSGi** é uma tecnologia que permite organizar aplicações Java em módulos, chamados *bundles*, com metadados, dependências e serviços.

Um módulo web pode conter uma classe que implementa um portlet, além de JSPs, arquivos JavaScript, CSS e outros recursos.

Por exemplo:

```text
modules/
└── meu-portlet-web/
    ├── bnd.bnd
    ├── build.gradle
    └── src/
        └── main/
            ├── java/
            │   └── com/example/
            │       └── MeuPortlet.java
            └── resources/
                └── META-INF/
                    └── resources/
                        └── view.jsp
```

Essa estrutura é ilustrativa. A organização exata pode variar conforme o projeto e o tipo de módulo criado.

Nesse exemplo:

- `MeuPortlet.java` contém a classe Java que registra e implementa o portlet.
- `view.jsp` contém uma possível interface renderizada pelo servidor.
- `bnd.bnd` participa da configuração e dos metadados OSGi do bundle.
- `build.gradle` define aspectos da construção do módulo.

Portanto, **o portlet é um componente da aplicação, enquanto o módulo OSGi é uma unidade de empacotamento e execução modular**. Eles estão relacionados, mas representam conceitos diferentes.

## 4. Como um portlet é registrado no Liferay?

Para que um portlet personalizado possa ser reconhecido pela plataforma, ele precisa ser registrado como um componente apropriado.

Em um portlet tradicional baseado no framework MVC do Liferay, esse registro pode ser realizado por meio da anotação `@Component`, do Declarative Services do OSGi.

Considere o seguinte exemplo:

```java
import com.liferay.portal.kernel.portlet.bridges.mvc.MVCPortlet;

import javax.portlet.Portlet;

import org.osgi.service.component.annotations.Component;

@Component(
    property = {
        "com.liferay.portlet.display-category=category.sample",
        "javax.portlet.display-name=Meu Portlet",
        "javax.portlet.name=com_example_MeuPortlet"
    },
    service = Portlet.class
)
public class MeuPortlet extends MVCPortlet {
}
```

O exemplo ilustra o registro de um portlet MVC tradicional. Em projetos Liferay 7.4, o uso de APIs `javax.portlet` ou `jakarta.portlet` depende da versão e das dependências efetivamente utilizadas; não se deve misturar essas APIs indiscriminadamente.

Vamos analisar os principais elementos.

### 4.1. `@Component`

A anotação `@Component` declara um componente gerenciado pelo ambiente OSGi.

O Liferay utiliza os metadados desse componente para identificá-lo e integrá-lo à plataforma.

A anotação, isoladamente, não implementa a funcionalidade de negócio do portlet. Ela participa do registro do componente.

### 4.2. `property`

O atributo `property` define propriedades do componente.

No exemplo:

- `com.liferay.portlet.display-category=category.sample` indica a categoria em que o portlet pode aparecer no seletor de aplicações.
- `javax.portlet.display-name=Meu Portlet` define seu nome de apresentação.
- `javax.portlet.name=com_example_MeuPortlet` define seu nome técnico.

O nome técnico é particularmente importante porque identifica o portlet para a plataforma e pode ser utilizado na associação de URLs, comandos MVC e outras configurações.

Essas propriedades não são meramente decorativas: elas participam da integração do componente ao ambiente do Liferay.

### 4.3. `service = Portlet.class`

Essa declaração indica que o componente fornece um serviço do tipo `Portlet`.

Em conjunto com a implementação da classe e os metadados declarados, isso permite que o ambiente reconheça o componente como um portlet.

### 4.4. `extends MVCPortlet`

A classe herda de `MVCPortlet`, uma implementação do Liferay que facilita o desenvolvimento de portlets utilizando o padrão MVC.

Isso evita que o desenvolvedor precise implementar manualmente diversos aspectos do processamento de um portlet tradicional.

O framework fornece mecanismos para encaminhar solicitações às visualizações e para organizar operações de controle.

É importante observar que nem todo portlet precisa utilizar `MVCPortlet`. Ele é uma opção comum quando se utiliza o framework MVC tradicional do Liferay.

## 5. O que é o MVC Portlet?

**MVC Portlet** é o framework do Liferay que permite organizar a implementação de um portlet seguindo o padrão arquitetural Model-View-Controller.

O objetivo é separar responsabilidades que, se misturadas, tornariam o código mais difícil de compreender e manter.

O padrão MVC possui três elementos fundamentais.

### 5.1. Model

O Model representa os dados e as operações necessárias para trabalhar com eles.

Em uma aplicação Liferay, essa responsabilidade pode envolver entidades, serviços e mecanismos de persistência.

Por exemplo, um sistema de disciplinas pode possuir um serviço responsável por consultar as disciplinas cadastradas.

O portlet não precisa implementar diretamente toda a lógica de acesso ao banco de dados.

### 5.2. View

A View é responsável pela apresentação das informações.

Em um portlet MVC tradicional, é comum utilizar arquivos JSP para gerar HTML no servidor.

Por exemplo, uma JSP pode receber uma lista de disciplinas e apresentá-la em uma tabela.

A View deve se concentrar na apresentação, evitando concentrar nela toda a lógica de negócio.

### 5.3. Controller

O Controller coordena o processamento das solicitações recebidas pelo portlet.

Ele pode interpretar parâmetros, chamar serviços e preparar os dados necessários para a apresentação.

Em aplicações menores, parte desse trabalho pode estar na própria classe que estende `MVCPortlet`.

Em aplicações maiores, o Liferay permite distribuir as responsabilidades entre classes especializadas, como `MVCActionCommand`, `MVCRenderCommand` e `MVCResourceCommand`.

A separação não exige que todas as classes existam em qualquer projeto. A estrutura depende da complexidade da aplicação.

### 5.4. Como as três partes se relacionam?

Considere uma funcionalidade que lista as disciplinas cadastradas.

```text
Usuário
   |
   | Acessa a página
   v
Portlet / Controller
   |
   | Solicita as disciplinas
   v
Service
   |
   | Consulta os dados
   v
Repository / Persistência
   |
   | Retorna os dados
   v
Service
   |
   | Entrega o resultado
   v
Controller
   |
   | Disponibiliza os dados para a View
   v
JSP / View
   |
   | Gera HTML
   v
Página do Liferay
```

Esse fluxo é uma representação conceitual. A arquitetura concreta pode variar, e o Liferay também fornece serviços próprios que a aplicação pode consumir.

O ponto central é que o portlet coordena a funcionalidade apresentada ao usuário, enquanto outras camadas podem cuidar das regras de negócio e da persistência.

## 6. O ciclo de processamento de um portlet

Uma característica importante dos portlets é que seu processamento é organizado em fases.

Isso significa que uma solicitação recebida por um portlet não deve ser compreendida simplesmente como uma chamada genérica a um método de um controller web tradicional.

No modelo de portlets, há fases com finalidades distintas.

### 6.1. Fase de renderização

A fase de renderização produz a apresentação do portlet.

Por exemplo, quando o usuário abre uma página que contém um portlet de disciplinas, o componente pode consultar os dados necessários e gerar a interface correspondente.

No framework MVC do Liferay, a classe `MVCPortlet` fornece mecanismos para essa renderização.

### 6.2. Fase de ação

A fase de ação processa operações que alteram o estado da aplicação.

Por exemplo:

- Criar uma disciplina.
- Atualizar o nome de um tópico.
- Excluir uma anotação.

Uma URL de ação pode encaminhar uma solicitação do formulário para o processamento correspondente.

No MVC Portlet, essas operações podem ser implementadas na própria classe do portlet ou em classes `MVCActionCommand`.

### 6.3. Fase de recurso

A fase de recurso permite fornecer um recurso ou conteúdo específico, sem necessariamente executar o fluxo completo de renderização da página.

Ela pode ser utilizada, por exemplo, para responder a uma solicitação AJAX com dados em JSON.

No framework MVC do Liferay, a classe `MVCResourceCommand` permite organizar esse tipo de processamento.

### 6.4. Comparação das fases

| Fase | Finalidade | Exemplo |
|---|---|---|
| Render | Produzir a apresentação. | Exibir uma lista de disciplinas. |
| Action | Processar uma operação que altera o estado. | Salvar uma nova disciplina. |
| Resource | Fornecer um recurso específico. | Retornar dados JSON para uma solicitação AJAX. |

Essas fases são distintas, embora possam participar de uma mesma interação do usuário.

Por exemplo, o usuário preenche um formulário, o portlet processa a ação de salvar e, depois, a página pode ser renderizada novamente para apresentar o resultado.

## 7. O que são as classes MVC do Liferay?

O framework MVC do Liferay oferece classes e interfaces que permitem organizar o processamento de cada fase.

Entre as mais importantes estão:

### 7.1. `MVCPortlet`

É a classe-base utilizada para implementar um portlet com o framework MVC tradicional.

Ela fornece mecanismos para processar solicitações e encaminhá-las às visualizações e aos comandos apropriados.

É comum encontrar uma classe com o sufixo `Portlet`, como `MeuPortlet`, estendendo `MVCPortlet`.

### 7.2. `MVCActionCommand`

Uma classe que implementa `MVCActionCommand` concentra uma operação executada na fase de ação.

Por exemplo, um comando pode receber os dados de um formulário e solicitar ao serviço que crie um novo tópico.

Isso permite separar a lógica de cada operação em classes distintas, em vez de concentrar todos os métodos em uma única classe de portlet.

### 7.3. `MVCRenderCommand`

Uma classe que implementa `MVCRenderCommand` concentra o processamento associado à renderização de uma determinada visualização.

Ela pode preparar os dados necessários e indicar qual JSP deve ser apresentada.

É útil quando um portlet possui diversas telas ou quando a lógica de renderização começa a ficar extensa.

### 7.4. `MVCResourceCommand`

Uma classe que implementa `MVCResourceCommand` processa solicitações da fase de recurso.

É uma alternativa útil para operações que precisam retornar dados ou disponibilizar conteúdo específico sem depender da renderização completa da página.

### 7.5. Como essas classes se relacionam?

Uma classe `MVCPortlet` pode servir como ponto de entrada e integrar comandos especializados.

```text
MeuPortlet
    |
    ├── MVCActionCommand
    |      └── Processa operações
    |
    ├── MVCRenderCommand
    |      └── Prepara a visualização
    |
    └── MVCResourceCommand
           └── Fornece recursos
```

O desenho representa uma relação funcional, não uma hierarquia obrigatória de herança.

Esses comandos são componentes independentes que podem ser registrados no OSGi e associados ao portlet correspondente.

## 8. Portlet e JSP

JSP (*JavaServer Pages*) é uma tecnologia que permite gerar conteúdo dinâmico, normalmente HTML, no servidor.

No desenvolvimento tradicional de portlets do Liferay, as JSPs são frequentemente utilizadas como Views.

Por exemplo, uma JSP pode apresentar um formulário para cadastrar uma disciplina:

```jsp
<%@ taglib uri="http://java.sun.com/portlet_2_0" prefix="portlet" %>

<portlet:defineObjects />

<h2>Cadastro de disciplina</h2>

<form>
    <label for="name">Nome:</label>

    <input type="text" id="name" name="name" />

    <button type="submit">
        Salvar
    </button>
</form>
```

Esse exemplo é apenas ilustrativo: o formulário ainda não está associado a uma URL de ação e, portanto, não implementa o cadastro.

A JSP fornece a estrutura de apresentação. O processamento do formulário precisa ser conectado ao portlet e à lógica responsável por salvar os dados.

### 8.1. O que é `portlet:defineObjects`?

A diretiva `<portlet:defineObjects />` disponibiliza objetos relacionados ao ambiente de portlets para utilização na JSP.

Isso permite trabalhar com objetos como a solicitação de renderização e a resposta de renderização, conforme o ambiente e as APIs utilizadas.

### 8.2. Por que utilizar URLs específicas de portlet?

Um portlet não deve ser tratado como se fosse simplesmente uma página HTML independente.

O Liferay precisa identificar qual portlet receberá a solicitação e qual fase de processamento deverá ser executada.

Por isso, as JSPs podem utilizar tags como:

- `portlet:renderURL`, para gerar URLs de renderização.
- `portlet:actionURL`, para gerar URLs de ação.
- `portlet:resourceURL`, para gerar URLs de recurso.

Essas URLs são geradas pelo ambiente de portlets e permitem que as solicitações sejam encaminhadas corretamente.

## 9. Portlet e Service Builder

O Service Builder é uma ferramenta do Liferay que gera componentes de persistência e serviços a partir de definições de entidades.

Ele é frequentemente utilizado em aplicações que precisam armazenar e manipular dados próprios.

Considere um módulo de gerenciamento de disciplinas que utilize Service Builder.

O projeto pode possuir módulos como:

```text
modules/
└── disciplinas/
    ├── disciplinas-api/
    ├── disciplinas-service/
    └── disciplinas-web/
```

Essa estrutura é ilustrativa de uma organização comum em projetos baseados em Service Builder.

- `disciplinas-api`: disponibiliza interfaces, modelos e contratos gerados.
- `disciplinas-service`: contém implementações e componentes de serviço, incluindo código gerado e extensões personalizadas.
- `disciplinas-web`: contém o portlet e a interface web.

O portlet pode utilizar um serviço para criar, consultar, atualizar ou excluir disciplinas.

Por exemplo, conceitualmente:

```text
Usuário
   |
   v
Portlet
   |
   v
Serviço do Service Builder
   |
   v
Persistência
   |
   v
Banco de dados
```

O portlet não precisa executar diretamente comandos SQL para cada operação. Ele pode delegar esse trabalho aos serviços e componentes de persistência apropriados.

Essa separação favorece a reutilização dos serviços por outras partes da aplicação.

**Atenção:** Service Builder não é um requisito para todo portlet. Um portlet também pode consumir serviços existentes do Liferay, acessar APIs externas ou utilizar outras estratégias de persistência.

## 10. Portlet e APIs REST

É importante distinguir um portlet de uma API REST.

Um portlet é um componente de interface integrado a uma página do portal.

Uma API REST disponibiliza operações e recursos por meio de endpoints HTTP.

Embora sejam conceitos diferentes, eles podem trabalhar juntos.

Considere uma aplicação em que o portlet apresenta uma lista de documentos.

Existem pelo menos duas possibilidades:

**Possibilidade A — processamento no servidor**

O portlet recebe a solicitação, chama os serviços necessários e gera a interface HTML.

```text
Navegador
   |
   v
Portlet
   |
   v
Serviços
   |
   v
Dados
```

**Possibilidade B — interface consumindo uma API**

O portlet apresenta uma interface JavaScript que realiza solicitações HTTP para uma API REST.

```text
Navegador
   |
   v
Portlet / Interface
   |
   | HTTP
   v
API REST
   |
   v
Serviços
   |
   v
Dados
```

A segunda possibilidade permite separar a interface do endpoint que fornece os dados.

No Liferay, por exemplo, um portlet pode consumir uma API Headless já existente, enquanto um módulo desenvolvido com REST Builder pode disponibilizar endpoints próprios.

O REST Builder não é um tipo de portlet: é uma ferramenta para desenvolver APIs REST no ecossistema Liferay.

Da mesma forma, um módulo que implementa uma API REST não precisa necessariamente conter um portlet de interface.

## 11. O que acontece quando um portlet é implantado?

Durante o desenvolvimento, o código de um portlet é construído e empacotado como parte de um módulo.

Em um projeto Gradle típico do Liferay, o comando a seguir constrói e pode implantar o módulo, conforme a configuração do projeto:

```bash
./gradlew deploy
```

O processo de implantação disponibiliza o módulo ao ambiente Liferay.

Em uma instalação com o mecanismo de implantação configurado, o Liferay identifica o bundle e processa seu registro e suas dependências.

Quando o componente está ativo e devidamente registrado, o portlet pode ser disponibilizado para uso nas páginas, de acordo com sua configuração e com as permissões aplicáveis.

O fluxo conceitual é:

```text
Código Java, JSP, JavaScript e CSS
                |
                v
         Construção Gradle
                |
                v
          Módulo OSGi
                |
                v
       Implantação no Liferay
                |
                v
      Registro do componente
                |
                v
     Disponibilização do portlet
                |
                v
       Adição a uma página
```

A construção bem-sucedida do projeto, por si só, não garante que o portlet esteja disponível. É necessário que o módulo seja implantado corretamente e que o componente esteja ativo, sem problemas de dependências ou de configuração.

## 12. Portlet não é sinônimo de aplicação web inteira

Essa distinção ajuda a evitar confusões ao estudar o Liferay.

| Conceito | O que representa |
|---|---|
| Aplicação web | Um sistema web ou conjunto de funcionalidades. |
| Portal | Uma plataforma que integra páginas, componentes e serviços. |
| Portlet | Um componente que funciona dentro de um portal. |
| Módulo OSGi | Uma unidade modular de código e componentes. |
| MVC Portlet | Um framework para desenvolver portlets utilizando MVC. |
| Service Builder | Uma ferramenta para gerar modelos, persistência e serviços. |
| REST Builder | Uma ferramenta para desenvolver APIs REST. |
| JSP | Uma tecnologia para gerar conteúdo dinâmico no servidor. |

Esses elementos podem fazer parte da mesma aplicação, mas cada um possui uma função específica.

Um módulo web pode conter um portlet MVC e suas JSPs. Esse portlet pode utilizar serviços gerados pelo Service Builder. Outro módulo pode disponibilizar uma API REST por meio do REST Builder.

A arquitetura final depende dos requisitos e das escolhas do projeto.

## 13. Resumo dos conceitos fundamentais

- **Portlet:** componente que implementa uma funcionalidade dentro de uma página de portal.
- **Liferay:** plataforma que integra portlets, páginas, autenticação, permissões e outros serviços.
- **OSGi:** ambiente de modularidade utilizado para organizar e executar componentes no Liferay.
- **MVC Portlet:** framework do Liferay para separar responsabilidades de Model, View e Controller.
- **`MVCPortlet`:** classe-base utilizada na implementação de portlets MVC tradicionais.
- **`MVCActionCommand`:** componente especializado no processamento de ações.
- **`MVCRenderCommand`:** componente especializado na preparação da renderização.
- **`MVCResourceCommand`:** componente especializado no fornecimento de recursos.
- **JSP:** tecnologia frequentemente utilizada para construir as Views dos portlets tradicionais.
- **Service Builder:** ferramenta que gera modelos, componentes de persistência e serviços.
- **REST Builder:** ferramenta utilizada para desenvolver APIs REST no Liferay.

A principal ideia a reter é que **o portlet é o componente que integra uma funcionalidade à interface do portal, mas não precisa concentrar toda a lógica da aplicação**. Ele pode atuar como camada de interface e controle, delegando o processamento a serviços e utilizando outros módulos do Liferay conforme a necessidade.

## 14. Documentação complementar

Os materiais abaixo aprofundam os conceitos apresentados neste documento.

1. [Portlets — Liferay Learn](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/core-frameworks/portlets): visão geral dos portlets e sua integração com o Liferay.
2. [Using MVC — Liferay Learn](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/developing-a-web-application/using-mvc): funcionamento do framework MVC Portlet, seus componentes e sua modularidade.
3. [Using a JSP and MVC Portlet — Liferay Learn](https://learn.liferay.com/w//dxp/development/traditional-java-based-development/developing-a-web-application/using-mvc/using-a-jsp-and-mvc-portlet): exemplo de criação de um portlet MVC com JSP.
4. [Rendering Views with MVC Portlet — Liferay Learn](https://learn.liferay.com/pt/w/dxp/development/traditional-java-based-development/developing-a-web-application/using-mvc/rendering-views-with-mvc-portlet): navegação entre visualizações.
5. [Invoking Actions with MVC Portlet — Liferay Learn](https://learn.lxc.liferay.com/w/dxp/development/traditional-java-based-development/developing-a-web-application/using-mvc/invoking-actions-with-mvc-portlet): processamento de ações a partir da interface.

