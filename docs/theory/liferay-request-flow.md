# Entendendo o fluxo de uma requisição no Liferay

> **Objetivo deste material:** acompanhar uma requisição do início ao
> fim, desde o momento em que um usuário clica em um botão até a
> aplicação devolver uma resposta. O foco é entender o papel de cada
> parte sem confundir HTTP, lógica de negócio, OSGi, Service Builder e
> banco de dados.

## 1. A ideia em uma frase

Quando um usuário faz algo em uma página, o navegador envia uma
requisição ao servidor. O Liferay encaminha essa requisição ao
componente responsável, que pode chamar serviços da aplicação. Esses
serviços executam a operação necessária e, se precisarem ler ou gravar
dados, usam a camada de persistência para acessar o banco. Depois, o
resultado volta ao componente de entrada, que produz a resposta enviada
ao navegador.

**Importante:** esse é um modelo mental geral. O caminho exato depende
do tipo de funcionalidade: pode ser um portlet, um endpoint REST, uma
ação AJAX ou outro mecanismo do Liferay.

## 2. Primeiro, imagine uma situação concreta

Imagine uma aplicação de biblioteca feita no Liferay. A página tem um
formulário para cadastrar um livro com título e autor.

O usuário preenche os campos e clica em **Cadastrar**.

O que parece ser uma única ação envolve várias etapas:

1.  O navegador prepara e envia uma requisição HTTP.
2.  O servidor recebe a requisição.
3.  O Liferay encaminha a requisição ao componente que sabe tratar
    aquela ação.
4.  O componente chama a camada de serviço.
5.  O serviço verifica as regras de negócio.
6.  Se for necessário, o serviço usa a persistência para acessar o
    banco.
7.  O resultado volta ao componente que recebeu a requisição.
8.  O servidor envia uma resposta HTTP ao navegador.

Vamos entender cada etapa.

## 3. O fluxo completo, de forma visual

``` text
USUÁRIO
  |
  | clica em "Cadastrar"
  v
NAVEGADOR
  |
  | 1. envia requisição HTTP
  v
SERVIDOR LIFERAY
  |
  | 2. identifica e encaminha a requisição
  v
COMPONENTE DE ENTRADA
(portlet, endpoint REST ou controlador)
  |
  | 3. chama uma operação do backend
  v
CAMADA DE SERVIÇO
  |
  | 4. valida dados e aplica regras de negócio
  v
CAMADA DE PERSISTÊNCIA
  |
  | 5. traduz a operação em acesso a dados
  v
BANCO DE DADOS
  |
  | 6. grava ou consulta os dados
  v
PERSISTÊNCIA -> SERVIÇO -> COMPONENTE DE ENTRADA
  |
  | 7. prepara o resultado e a resposta
  v
RESPOSTA HTTP
  |
  v
NAVEGADOR -> usuário vê o resultado
```

A parte de baixo do fluxo é o caminho de retorno. O banco não envia uma
resposta HTTP diretamente ao navegador: ele retorna um resultado ao
código do servidor, e esse código prepara a resposta HTTP.

## 4. Etapa 1 --- O navegador envia uma requisição HTTP

HTTP é um protocolo usado para comunicação entre clientes e servidores.

Quando o usuário clica em "Cadastrar", o navegador pode enviar os campos
do formulário ao servidor. Dependendo da implementação, isso pode ser
uma requisição `POST`, uma chamada AJAX ou uma requisição feita por
outro mecanismo web.

Uma requisição HTTP costuma incluir: - um método, como `GET` ou
`POST`; - um caminho ou endereço; - cabeçalhos; - eventualmente, um
corpo com dados.

Exemplo ilustrativo:

``` http
POST /biblioteca/livros
Content-Type: application/json

{
  "title": "Arquitetura de Software",
  "author": "Ana Silva"
}
```

Esse exemplo representa uma API HTTP genérica. O endereço e o formato
reais de uma aplicação Liferay dependem do endpoint e de como ele foi
implementado.

**O navegador não conversa diretamente com o banco de dados.** Ele
conversa com o servidor por HTTP.

## 5. Etapa 2 --- O Liferay recebe e encaminha a requisição

O servidor recebe a requisição e precisa determinar qual componente é
responsável por tratá-la.

Pense em uma recepção que encaminha uma solicitação para o setor
correto. A recepção não executa necessariamente o trabalho solicitado;
ela direciona a solicitação ao componente apropriado.

No Liferay, o componente de entrada pode ser, por exemplo: - um
portlet; - um endpoint REST; - um controlador web; - um mecanismo de
ação ou recurso de um portlet.

A escolha depende de como a funcionalidade foi construída.

Antes ou durante o processamento, a plataforma também pode executar
verificações de autenticação, autorização, validação, filtros e outras
etapas configuradas. Por isso, nem toda requisição percorre exatamente a
mesma sequência interna.

## 6. Etapa 3 --- O componente de entrada chama o backend

O **componente de entrada** é o código que recebe a solicitação web e
coordena o que deve acontecer com ela.

Por exemplo, ele pode: 1. ler os campos enviados; 2. converter os
valores para os tipos esperados; 3. chamar um método de serviço; 4.
interpretar o resultado; 5. decidir qual resposta devolver.

O ideal é que esse componente não concentre toda a lógica de negócio nem
implemente diretamente todos os detalhes do banco de dados.

Uma chamada conceitual poderia ser:

``` java
bookLocalService.addBook(title, author);
```

Esse nome é apenas ilustrativo; o método exato depende das entidades,
dos serviços gerados e das implementações do projeto.

## 7. Etapa 4 --- O serviço aplica as regras de negócio

A **camada de serviço** executa ou coordena a operação solicitada. É
aqui que normalmente colocamos as regras de negócio que precisam valer
independentemente de a operação ter sido iniciada por uma página, por
uma API ou por outro componente interno.

No exemplo do livro, regras possíveis seriam: - o título é
obrigatório; - o usuário precisa ter autorização para cadastrar
livros; - não se pode cadastrar um registro em uma condição proibida
pela aplicação.

O serviço verifica as regras pertinentes e decide como prosseguir. Se
uma regra não for satisfeita, a operação pode ser interrompida com um
erro ou resultado apropriado.

A validação de entrada também pode existir no formulário ou no endpoint,
mas uma regra de negócio importante não deve depender exclusivamente de
a interface ter sido usada: outros componentes podem chamar o serviço
diretamente.

## 8. Etapa 5 --- A persistência acessa os dados

Se a operação precisa consultar ou alterar dados, o serviço usa a
**camada de persistência**.

Persistência significa guardar dados de forma que eles permaneçam
disponíveis depois que a requisição termina. A camada de persistência
oferece mecanismos para criar, consultar, atualizar ou excluir registros
sem obrigar cada componente a construir manualmente todo o acesso ao
banco.

Em projetos que usam Service Builder, parte importante desse código é
gerada pela ferramenta.

Exemplos de operações: - buscar um livro por identificador; - inserir um
novo livro; - atualizar o título de um livro; - excluir um registro.

Essas operações são conhecidas como CRUD: Create, Read, Update e Delete.

## 9. Etapa 6 --- O banco de dados executa a operação

A camada de acesso a dados utiliza a tecnologia de persistência
configurada para consultar ou modificar o banco.

Para uma inclusão, o banco pode executar uma instrução SQL `INSERT`.
Para uma consulta, pode executar um `SELECT`.

O banco devolve um resultado ao código do servidor. Esse resultado pode
conter dados consultados, a confirmação da operação ou informações de
erro.

Nem toda chamada ao serviço gera obrigatoriamente uma consulta SQL
imediata: caches, transações e outros mecanismos podem influenciar o
comportamento.

## 10. Etapa 7 --- O resultado volta pelas camadas

Depois do acesso ao banco, o resultado retorna pelo código do servidor:

``` text
Banco de dados
   |
   v
Persistência
   |
   v
Serviço
   |
   v
Componente de entrada
```

O serviço pode devolver a entidade criada, um identificador, uma lista
de resultados ou outro valor. Se ocorrer uma falha, pode lançar uma
exceção ou produzir um resultado de erro, conforme o projeto.

O componente de entrada interpreta esse resultado e decide o que o
cliente deve receber.

## 11. Etapa 8 --- O servidor devolve uma resposta HTTP

O componente web transforma o resultado em uma resposta HTTP.

Dependendo da aplicação, a resposta pode ser: - uma página HTML; - um
redirecionamento para outra página; - um objeto JSON; - um código HTTP
de sucesso ou erro, possivelmente acompanhado de conteúdo.

O navegador recebe a resposta e atualiza a interface. Por exemplo, pode
exibir "Livro cadastrado com sucesso" ou apresentar uma mensagem
explicando por que o cadastro falhou.

**O banco de dados não responde diretamente ao navegador por HTTP.** A
resposta HTTP é produzida pelo servidor web com base no resultado do
processamento.

## 12. Onde entram OSGi e Service Builder?

Esses conceitos se relacionam com o fluxo, mas têm papéis diferentes.

### OSGi

OSGi é a infraestrutura modular usada pelo Liferay para organizar e
conectar muitos componentes Java. Um componente pode receber um serviço
OSGi por meio de mecanismos como `@Reference`.

OSGi ajuda os componentes do servidor a colaborar. Ele não é o protocolo
HTTP e não significa que cada serviço seja uma API de rede.

### Service Builder

Service Builder gera código relacionado a entidades, persistência e
serviços a partir de uma definição como `service.xml`.

Quando uma requisição chega a um componente web, esse componente pode
chamar um serviço gerado ou personalizado com Service Builder. O serviço
pode aplicar regras de negócio e usar a persistência gerada para acessar
os dados.

### Banco de dados

O banco é responsável pelo armazenamento e pela consulta dos dados. Ele
não decide sozinho todas as regras de negócio da aplicação e não
substitui as camadas de serviço e persistência.

Resumo da relação:

  -----------------------------------------------------------------------
  Conceito                            Papel no fluxo
  ----------------------------------- -----------------------------------
  HTTP                                Transporta a requisição e a
                                      resposta entre cliente e servidor

  Liferay / componente web            Recebe ou trata a requisição

  OSGi                                Organiza módulos e ajuda a conectar
                                      componentes e serviços Java

  Serviço                             Executa ou coordena a operação e as
                                      regras de negócio

  Service Builder                     Gera partes do modelo, serviços e
                                      persistência

  Persistência                        Implementa o acesso aos dados

  Banco de dados                      Armazena e consulta os dados
  -----------------------------------------------------------------------

## 13. Um exemplo completo: cadastro de livro

Vamos percorrer o cenário de ponta a ponta.

**Situação:** o usuário envia o título `Arquitetura de Software` e o
autor `Ana Silva`.

1.  **Navegador:** envia os dados ao servidor.
2.  **Liferay:** encaminha a requisição ao portlet ou endpoint
    responsável.
3.  **Componente de entrada:** lê os campos e chama o serviço de
    cadastro.
4.  **Serviço:** verifica se o título foi informado e aplica as regras
    de negócio relevantes.
5.  **Persistência:** prepara a operação de gravação usando os
    mecanismos disponíveis.
6.  **Banco:** armazena os dados e devolve o resultado.
7.  **Serviço:** recebe o resultado e devolve os dados relevantes ao
    chamador.
8.  **Componente de entrada:** prepara uma mensagem ou resposta JSON.
9.  **Navegador:** recebe a resposta e mostra o resultado ao usuário.

Se o título estiver vazio, o serviço pode rejeitar a operação antes da
gravação. Nesse caso, o fluxo segue um caminho de erro: o componente de
entrada transforma a falha em uma resposta apropriada, e o navegador
apresenta a mensagem ao usuário.

## 14. O que pode mudar no fluxo?

O diagrama é um modelo mental, não uma regra de que todo pedido precisa
passar pelas mesmas classes.

-   Uma requisição de leitura pode retornar dados sem modificar o banco.
-   Um endpoint pode chamar vários serviços.
-   Um serviço pode chamar outro serviço.
-   Uma operação pode não precisar de banco de dados.
-   A autenticação e a autorização podem ser verificadas em diferentes
    pontos, conforme o mecanismo utilizado.
-   Uma falha de validação pode interromper o fluxo antes da
    persistência.
-   Uma transação pode agrupar várias operações de banco para que sejam
    tratadas como uma unidade.

O princípio que continua útil é separar a comunicação HTTP, a
coordenação da operação, as regras de negócio e o acesso aos dados.

## 15. Confusões comuns

### "O clique vai direto para o banco de dados."

Não. O navegador envia uma requisição ao servidor. O código do servidor
decide como processá-la e, se necessário, acessa o banco.

### "O controlador deve conter todas as regras."

Não é uma boa regra geral. O controlador deve lidar com a entrada e a
resposta. Regras de negócio reutilizáveis normalmente pertencem à camada
de serviço apropriada.

### "Service Builder recebe a requisição HTTP."

Não por si só. O endpoint ou componente web recebe a requisição. Ele
pode chamar serviços gerados pelo Service Builder.

### "OSGi é o banco de dados ou o protocolo HTTP."

Não. OSGi é uma infraestrutura de modularidade e serviços para
aplicações Java.

### "Toda chamada de serviço executa SQL imediatamente."

Não necessariamente. O comportamento depende da persistência, do cache,
das transações e da operação executada.

### "Se a interface validou os dados, o backend não precisa validar."

A validação no cliente melhora a experiência do usuário, mas não
substitui as verificações necessárias no servidor. O backend pode ser
chamado por outros clientes ou componentes.

## 16. Resumo para memorizar

Pense nesta sequência:

1.  **O usuário faz uma ação.**
2.  **O navegador envia uma requisição HTTP.**
3.  **O Liferay encaminha a requisição ao componente responsável.**
4.  **O componente chama um serviço.**
5.  **O serviço aplica as regras de negócio.**
6.  **A persistência acessa o banco, se necessário.**
7.  **O resultado volta ao componente web.**
8.  **O servidor envia uma resposta HTTP ao navegador.**

> **Frase para lembrar:** o navegador conversa com o servidor; o
> componente web coordena a requisição; o serviço executa a operação; a
> persistência acessa os dados; e o servidor devolve a resposta.

## 17. Perguntas para revisar

1.  O navegador se conecta diretamente ao banco de dados?
2.  Qual é a responsabilidade do componente de entrada?
3.  Em que camada normalmente ficam as regras de negócio?
4.  Qual é o papel da persistência?
5.  Qual é a diferença entre OSGi e Service Builder?
6.  O que acontece se uma regra de negócio falhar antes da gravação?
7.  Quem prepara a resposta HTTP enviada ao navegador?

## 18. Documentação para continuar estudando

-   [Arquitetura e modularidade no
    Liferay](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/fundamentals/architecture/osgi-and-modularity)
-   [Service Builder --- visão
    geral](https://learn.liferay.com/w/dxp/development/traditional-java-based-development/data-frameworks/service-builder)
-   [Liferay REST
    Builder](https://learn.liferay.com/w/dxp/integration/rest-builder)

------------------------------------------------------------------------

**Nota:** este documento descreve um fluxo conceitual comum. Os detalhes
concretos dependem da versão do Liferay e de como o projeto implementa a
entrada HTTP, os serviços, a autorização e a persistência.
