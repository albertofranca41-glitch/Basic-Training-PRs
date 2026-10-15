# Introdução ao OSGi no Liferay

> **Objetivo deste material:** entender, com linguagem simples e
> precisão técnica, o que é OSGi, por que o Liferay o utiliza e como
> módulos independentes conseguem se encontrar e colaborar.

## 1. A ideia principal

**OSGi é um conjunto de especificações e tecnologias para executar
aplicações Java de forma modular.** Ele define como partes de uma
aplicação podem ser empacotadas, declarar dependências, publicar e
consumir serviços e ter seu ciclo de vida administrado enquanto a
aplicação está em execução.

O Liferay DXP utiliza OSGi como base de sua arquitetura modular. Muitas
funcionalidades do portal e muitas customizações são executadas como
módulos OSGi, geralmente empacotados em arquivos `.jar` com metadados
adicionais.

A ideia central é dividir um sistema grande em partes menores, com
responsabilidades e relações explícitas, em vez de tratar todo o código
como um único bloco.

Documentação oficial: - [OSGi e modularidade --- Liferay
Learn](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/architecture/osgi-and-modularity) -
[Projetos de módulos --- Liferay
Learn](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/module-projects)

## 2. Uma analogia para começar

Imagine um prédio com várias empresas. Cada empresa ocupa seu espaço,
oferece alguns serviços e precisa de serviços de outras empresas. Há
regras sobre quais áreas são acessíveis e como cada empresa pode ser
substituída ou removida.

Em uma aplicação modular:

-   cada **módulo** é uma unidade com um propósito definido;
-   as **dependências** declaram de quais recursos o módulo precisa;
-   as **APIs** definem contratos públicos que outros módulos podem
    usar;
-   os **serviços OSGi** permitem que um módulo ofereça uma
    funcionalidade e outros módulos a encontrem;
-   o **framework OSGi** gerencia módulos, dependências, serviços e
    ciclo de vida.

A analogia tem limites: módulos OSGi são componentes de software
executados dentro da infraestrutura Java, não empresas reais. Mas ajuda
a entender por que modularidade exige regras claras de colaboração.

## 3. O que é um módulo OSGi?

Um **módulo OSGi**, também chamado de **bundle**, é um arquivo Java
`.jar` acompanhado de metadados que o tornam conhecido e administrável
pelo framework OSGi.

Um módulo pode conter: - classes Java; - recursos, como arquivos de
configuração e templates; - informações sobre sua identidade e versão; -
declarações dos pacotes Java que importa e exporta; - descrições de
componentes e serviços que oferece ou utiliza.

Um JAR comum contém classes e recursos Java. Um bundle OSGi contém esses
elementos **mais metadados específicos para o ambiente modular**,
geralmente no arquivo `META-INF/MANIFEST.MF` e em descritores
relacionados.

No Liferay, um projeto de módulo normalmente inclui arquivos como:

``` text
meu-modulo/
├── bnd.bnd
├── build.gradle
└── src/
    └── main/
        ├── java/
        └── resources/
```

-   `build.gradle`: descreve como construir o projeto e suas
    dependências de compilação.
-   `bnd.bnd`: configura metadados do bundle, como identidade, versão e
    aspectos de importação/exportação de pacotes.
-   `src/main/java`: contém o código Java.
-   `src/main/resources`: contém recursos do módulo.

A estrutura exata pode variar conforme o template e a versão das
ferramentas.

## 4. Por que não colocar tudo em um único JAR?

Em um sistema grande, juntar todo o código em uma única unidade pode
tornar mais difícil entender responsabilidades, controlar dependências,
substituir implementações e desenvolver funcionalidades em paralelo.

A modularidade ajuda a:

-   **separar responsabilidades:** cada módulo pode ter um objetivo mais
    claro;
-   **declarar dependências:** o módulo informa o que precisa para
    funcionar;
-   **controlar visibilidade:** nem toda classe precisa ser pública para
    todos os outros módulos;
-   **substituir implementações:** outros módulos podem consumir um
    contrato sem conhecer todos os detalhes internos;
-   **administrar o ciclo de vida:** instalar, iniciar, parar, atualizar
    ou remover módulos em tempo de execução, quando as dependências e
    condições permitirem.

Isso não significa que todo módulo seja completamente independente.
Módulos frequentemente dependem uns dos outros; o benefício vem de
declarar e gerenciar essas relações explicitamente.

## 5. Conceitos fundamentais

### 5.1 Framework OSGi

O **framework OSGi** é o ambiente de execução que administra os bundles.
No Liferay, ele coordena, entre outras coisas, a resolução de
dependências, o carregamento de classes dos módulos, o registro de
serviços e os estados do ciclo de vida.

Ele não é o código de negócio da aplicação. É a infraestrutura que
permite que os módulos sejam organizados e executados.

### 5.2 Bundle

**Bundle** é o nome OSGi para um módulo instalável e administrável pelo
framework. Em conversas sobre Liferay, "módulo OSGi" e "bundle"
geralmente se referem à mesma unidade de implantação.

### 5.3 Pacote Java (package)

Um **pacote Java** organiza classes por nome, por exemplo:

``` java
package com.exemplo.biblioteca.api;
```

No OSGi, os pacotes também são importantes para as dependências entre
módulos. Um módulo pode **importar** um pacote que outro módulo
**exporta**.

-   **Exportar um pacote:** declarar que classes daquele pacote estão
    disponíveis para outros módulos.
-   **Importar um pacote:** declarar que o módulo precisa de classes
    daquele pacote fornecidas por outro bundle ou pela plataforma.

Por padrão, as classes de um bundle não se tornam automaticamente
acessíveis a todos os outros módulos. OSGi controla a visibilidade por
meio dos pacotes exportados/importados e de seu sistema de carregamento
de classes.

### 5.4 Dependência

Uma **dependência** é algo de que um módulo precisa para compilar ou
funcionar. Pode ser um pacote Java, uma API ou outra biblioteca.

Há uma distinção importante:

-   uma dependência declarada no `build.gradle` informa ao sistema de
    build o que é necessário para compilar ou montar o projeto;
-   os metadados OSGi do bundle descrevem requisitos e capacidades
    necessários em tempo de execução.

As ferramentas do Liferay/Bnd ajudam a gerar metadados OSGi a partir do
código e da configuração. Uma dependência de compilação, sozinha, não
garante que todos os requisitos de execução estejam satisfeitos no
servidor.

### 5.5 API e implementação

Uma **API** define o contrato: quais operações existem e como podem ser
chamadas. A **implementação** contém o código que realiza essas
operações.

Por exemplo, uma API pode definir uma interface `SaudacaoService` com um
método `saudar(String nome)`. Uma implementação concreta decide como
produzir a saudação.

Separar API e implementação permite que um consumidor dependa do
contrato sem precisar conhecer os detalhes internos da implementação.

### 5.6 Serviço OSGi

Um **serviço OSGi** é um objeto Java disponibilizado no registro de
serviços do framework sob um ou mais tipos (normalmente interfaces).
Outros componentes podem solicitar um serviço compatível sem precisar
criar diretamente a classe concreta que o implementa.

O registro de serviços é gerenciado pelo framework. Isso permite que o
fornecedor e o consumidor fiquem menos acoplados à classe concreta.

Atenção: "serviço OSGi" não significa necessariamente um microserviço de
rede. Em geral, é um objeto Java acessado dentro da mesma instância da
aplicação, por meio do framework OSGi. Não é automaticamente uma API
HTTP ou REST.

### 5.7 Componente e Declarative Services (DS)

Um **componente** é uma unidade de código gerenciada pelo runtime de
componentes. No Liferay, é comum usar OSGi Declarative Services (DS),
frequentemente com anotações como `@Component` e `@Reference`.

-   `@Component` declara uma classe como componente gerenciado e pode
    declarar qual serviço ela fornece.
-   `@Reference` declara uma dependência de serviço que o runtime deve
    satisfazer.

Exemplo ilustrativo:

``` java
@Component(service = SaudacaoService.class)
public class SaudacaoServiceImpl implements SaudacaoService {

    @Override
    public String saudar(String nome) {
        return "Olá, " + nome;
    }
}
```

Um consumidor pode receber esse serviço por referência:

``` java
@Component(service = SaudacaoPainel.class)
public class SaudacaoPainel {

    @Reference
    private SaudacaoService _saudacaoService;

    public String gerarMensagem(String nome) {
        return _saudacaoService.saudar(nome);
    }
}
```

O exemplo é conceitual: para compilar, as interfaces, imports e
dependências do projeto precisam estar definidos. O runtime só consegue
ativar o componente consumidor quando suas referências obrigatórias
puderem ser satisfeitas.

### 5.8 Injeção de dependência

**Injeção de dependência** significa que uma classe recebe de fora uma
dependência de que precisa, em vez de instanciá-la diretamente.

Sem injeção, uma classe poderia executar `new SaudacaoServiceImpl()`,
ficando diretamente ligada à implementação concreta. Com `@Reference`,
ela declara que precisa de um serviço do tipo `SaudacaoService`, e o
runtime OSGi DS procura e conecta um serviço compatível.

Isso reduz o acoplamento e permite que a infraestrutura gerencie o
vínculo entre consumidor e fornecedor.

## 6. Como dois módulos colaboram?

Imagine três módulos:

1.  **API:** define a interface `SaudacaoService`.
2.  **Implementação:** implementa essa interface e registra o serviço.
3.  **Consumidor:** usa `@Reference` para receber o serviço e chamar seu
    método.

Fluxo simplificado:

``` text
Módulo API
  define SaudacaoService
          |
          | contrato conhecido pelos módulos
          v
Módulo de implementação
  implementa SaudacaoService
  registra o serviço no framework
          |
          | serviço disponível no registro OSGi
          v
Framework OSGi / Declarative Services
  encontra um serviço compatível
  satisfaz a referência do consumidor
          |
          v
Módulo consumidor
  recebe SaudacaoService via @Reference
  chama saudar("Ana")
```

A API não precisa conhecer o consumidor. O consumidor depende do
contrato da API, e o framework conecta a referência ao serviço
disponível. Isso é um exemplo de **baixo acoplamento**.

## 7. Ciclo de vida de um módulo

O framework administra o ciclo de vida dos bundles. Os estados mais
importantes para uma introdução são:

-   **Installed:** o bundle foi instalado no framework.
-   **Resolved:** os requisitos necessários, como pacotes importados,
    foram resolvidos.
-   **Starting:** o framework está iniciando o bundle.
-   **Active:** o bundle está ativo.
-   **Stopping:** o framework está parando o bundle.
-   **Uninstalled:** o bundle foi removido do framework.

Em termos simples, um módulo não fica pronto para uso apenas porque seu
arquivo JAR existe no disco. Ele precisa ser instalado no framework e
ter seus requisitos resolvidos. Os componentes DS dentro dele também
dependem de suas próprias referências e condições de ativação.

O caminho comum é:

``` text
JAR implantado
    |
    v
Installed
    |
    | dependências e requisitos resolvidos
    v
Resolved
    |
    | inicialização do bundle
    v
Active
```

Essa sequência é uma simplificação didática; atualizações, falhas de
resolução, política de ativação e outros fatores podem mudar o percurso
observado.

### Implantação dinâmica: o que isso realmente significa?

Uma característica importante do OSGi é permitir instalar, atualizar,
iniciar, parar e remover módulos sem necessariamente reiniciar toda a
plataforma. Isso é frequentemente chamado de **implantação dinâmica**.

Mas isso não significa que toda alteração pode ser aplicada sem impacto
ou que qualquer módulo pode ser removido a qualquer momento. Outros
módulos podem depender dele; uma atualização pode exigir que componentes
sejam reiniciados, e uma mudança incompatível pode impedir a resolução
de dependências.

## 8. Como o OSGi se relaciona com o Liferay?

O Liferay usa OSGi para organizar muitas de suas funcionalidades e
extensões como módulos.

Uma aplicação Liferay pode, por exemplo, ter:

-   um módulo de API;
-   um módulo de serviço/implementação;
-   um módulo web ou consumidor;
-   módulos de teste.

Nem todo projeto precisa dessa divisão exata, mas ela ilustra uma
arquitetura comum.

O **Service Builder** e o **OSGi** não são a mesma coisa:

-   **Service Builder** gera código de modelo, persistência e serviços a
    partir de uma definição como `service.xml`.
-   **OSGi** fornece a infraestrutura modular em que muitos desses
    módulos Liferay são empacotados, implantados e conectados.

Assim, um módulo gerado pelo Service Builder pode fazer parte de uma
arquitetura OSGi, mas o Service Builder não é o framework OSGi.

## 9. OSGi e o fluxo de uma requisição HTTP

OSGi não é, por si só, o protocolo que recebe requisições HTTP. Uma
requisição normalmente chega ao Liferay por um endpoint, portlet ou
outro componente web. Esse componente pode usar serviços OSGi para
executar o trabalho.

Exemplo conceitual:

``` text
Navegador
   |
   | requisição HTTP
   v
Endpoint / Portlet no Liferay
   |
   | chama um serviço
   v
Componente consumidor
   |
   | @Reference
   v
Serviço OSGi disponível
   |
   | executa lógica de aplicação
   v
Persistência / banco de dados (se necessário)
   |
   v
Resultado -> endpoint -> resposta HTTP
```

O OSGi participa da organização e conexão dos componentes dentro do
servidor. Ele não substitui HTTP, não implica que cada serviço seja
remoto e não obriga todas as requisições a seguir exatamente esse
desenho.

## 10. O que acontece quando uma dependência está ausente?

Suponha que um módulo importe um pacote que nenhum bundle disponível
exporta, ou que um componente precise de um serviço obrigatório que não
foi registrado.

Podem ocorrer problemas diferentes:

-   **Falha de resolução do bundle:** o framework não consegue
    satisfazer um requisito de pacote ou capacidade; o módulo pode não
    chegar ao estado `Resolved`.
-   **Referência DS não satisfeita:** o bundle pode estar ativo, mas um
    componente que exige aquele serviço não consegue ser ativado
    enquanto a referência obrigatória não for satisfeita.
-   **Erro de execução:** se a falha ocorrer durante uma operação, pode
    haver uma exceção ou outro erro em tempo de execução.

Esses problemas são relacionados, mas não são idênticos. Um bundle pode
estar resolvido e ativo sem que todos os componentes dentro dele estejam
ativos.

## 11. Erros comuns de entendimento

-   **"OSGi é uma linguagem de programação."** Não. É um conjunto de
    especificações e tecnologias para modularidade e execução de
    aplicações Java.
-   **"Todo JAR é automaticamente um bundle OSGi."** Não. Um bundle
    precisa de metadados OSGi apropriados e deve ser instalado em um
    framework compatível.
-   **"Um serviço OSGi é sempre uma API REST."** Não. Normalmente é um
    objeto Java registrado no framework e consumido dentro da aplicação.
-   **"`@Reference` cria uma instância da implementação."** Não
    diretamente. Ela declara uma referência que o runtime tenta
    satisfazer com um serviço registrado compatível.
-   **"Todas as classes de um módulo ficam disponíveis para todos os
    outros."** Não. A visibilidade dos pacotes é controlada; pacotes
    precisam ser exportados e importados conforme necessário.
-   **"Se o bundle está ativo, todos os seus componentes estão
    prontos."** Não necessariamente. Componentes DS podem depender de
    serviços e outras condições.
-   **"OSGi elimina todas as dependências."** Não. Ele torna
    dependências e relações mais explícitas e gerenciáveis; módulos
    continuam precisando uns dos outros.
-   **"OSGi e Service Builder são a mesma ferramenta."** Não. Um
    gerencia modularidade e serviços em tempo de execução; o outro gera
    código relacionado ao modelo, persistência e serviços no Liferay.

## 12. Resumo mental

Guarde esta sequência:

1.  **Módulo/bundle:** unidade implantável de código Java com metadados
    OSGi.
2.  **Pacotes importados/exportados:** definem quais classes podem ser
    usadas entre módulos.
3.  **API:** contrato que descreve operações.
4.  **Serviço OSGi:** objeto disponibilizado para consumidores pelo
    registro do framework.
5.  **`@Component`:** declara um componente gerenciado.
6.  **`@Reference`:** declara um serviço necessário ao componente.
7.  **Framework OSGi:** resolve dependências, conecta serviços e
    administra o ciclo de vida.
8.  **Liferay:** usa essa infraestrutura para organizar e estender
    muitas de suas funcionalidades.

> **Frase para lembrar:** OSGi é a infraestrutura modular que ajuda
> partes Java a declarar dependências, oferecer e consumir serviços e
> ser administradas durante a execução.

## 13. Perguntas para revisar

1.  Qual é a diferença entre um JAR comum e um bundle OSGi?
2.  O que significa exportar um pacote Java?
3.  Qual é a diferença entre API e implementação?
4.  O que o framework OSGi faz com os serviços registrados?
5.  Para que servem `@Component` e `@Reference`?
6.  Um serviço OSGi é necessariamente uma API HTTP? Por quê?
7.  Qual é a diferença entre o papel do OSGi e o do Service Builder?
8.  Por que um bundle instalado pode não conseguir funcionar
    corretamente?

## 14. Documentação oficial para continuar estudando

-   [OSGi e
    modularidade](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/architecture/osgi-and-modularity)
-   [Projetos de
    módulos](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/module-projects)
-   [Ciclo de vida dos
    módulos](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/architecture/module-lifecycle)
-   [APIs como serviços
    OSGi](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/apis-as-osgi-services)
-   [Como usar um serviço
    OSGi](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/using-an-osgi-service)

------------------------------------------------------------------------

**Nota de versão:** este material apresenta os conceitos gerais do OSGi
usados pelo Liferay DXP. A estrutura do projeto, as ferramentas de
build, os metadados e alguns detalhes de implementação podem variar
conforme a versão do Liferay e as convenções adotadas no projeto.
Consulte a documentação correspondente à versão utilizada.
