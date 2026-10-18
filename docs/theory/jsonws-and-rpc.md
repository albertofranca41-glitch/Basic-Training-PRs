# JSONWS e RPC

## 1. O que é RPC?

RPC (*Remote Procedure Call*, ou chamada de procedimento remoto) é um modelo de comunicação no qual um programa solicita a execução de uma operação disponibilizada por outro programa, possivelmente executado em outra máquina.

A ideia é permitir que o cliente invoque uma operação remota de maneira semelhante à chamada de uma função ou método local, embora a execução ocorra em outro processo ou sistema.

Por exemplo, imagine um sistema que disponibiliza uma operação chamada `getUser` para consultar um usuário pelo seu identificador. Um cliente poderia solicitar essa operação enviando o identificador e recebendo os dados correspondentes.

Conceitualmente, o fluxo é:

1. O cliente solicita a execução de uma operação.
2. A solicitação é enviada ao sistema remoto.
3. O sistema remoto executa a operação.
4. O resultado é enviado de volta ao cliente.

Essa abstração não elimina a comunicação pela rede. A chamada ainda depende de mecanismos de transporte, serialização dos dados, autenticação e tratamento de erros.

## 2. O que é JSONWS no Liferay?

JSONWS é um mecanismo do Liferay que permite acessar remotamente operações disponibilizadas por serviços da plataforma, utilizando requisições HTTP e dados representados em JSON.

Seu nome vem de *JSON Web Services*. Nesse contexto, o mecanismo permite que aplicações clientes invoquem métodos expostos pelo servidor Liferay sem precisarem executar esses métodos no mesmo processo.

Por exemplo, um serviço do Liferay pode disponibilizar uma operação para consultar determinado conteúdo. Um cliente autorizado pode enviar uma requisição ao endpoint correspondente e receber o resultado serializado em JSON.

O JSONWS tem, portanto, uma abordagem orientada a chamadas de operações remotas, característica associada ao modelo RPC.

É importante observar que nem todo método interno de uma classe Java fica automaticamente acessível remotamente. A operação precisa estar exposta de acordo com os mecanismos e as regras de acesso do Liferay.

## 3. Como uma chamada remota funciona?

Embora RPC procure abstrair a chamada remota como se fosse uma chamada de método, existem etapas de comunicação entre o cliente e o servidor.

Considere um exemplo conceitual de chamada a uma operação `getUser`.

O cliente precisa:

1. Identificar o endpoint que disponibiliza a operação.
2. Enviar a requisição HTTP com os parâmetros necessários.
3. Receber a resposta do servidor.
4. Interpretar os dados retornados.

Uma representação simplificada seria:

```text
Cliente
   |
   | Requisição HTTP com a operação e seus parâmetros
   v
Servidor Liferay
   |
   | Localiza e executa a operação disponibilizada
   v
Resultado da operação
   |
   | Resposta HTTP com os dados serializados em JSON
   v
Cliente
```

O JSON é utilizado para representar os dados transferidos, enquanto o HTTP transporta a requisição e a resposta.

Esses conceitos cumprem funções diferentes: JSON é um formato de representação de dados, HTTP é um protocolo de comunicação e RPC é um modelo de interação entre sistemas.

## 4. Por que JSONWS está associado ao modelo RPC?

A distinção está principalmente na forma como a interação é organizada.

Em um modelo RPC, o cliente solicita a execução de uma operação remota. A operação é o centro da interação.

Por exemplo, o cliente pode solicitar algo conceitualmente equivalente a:

```text
getUser(userId)
```

O interesse principal é executar a operação `getUser` e receber seu resultado.

No JSONWS, esse estilo de interação aparece na exposição de métodos de serviços do Liferay para consumo remoto.

Isso não significa que RPC seja necessariamente limitado a JSON ou HTTP. Existem implementações de RPC que utilizam outros protocolos e formatos de dados.

Da mesma forma, utilizar HTTP e JSON não torna automaticamente uma API REST ou RPC. A classificação depende do modelo de interação e de como a interface é projetada.

## 5. JSONWS e REST: qual é a diferença?

JSONWS e REST podem ser utilizados para disponibilizar funcionalidades de um servidor a clientes remotos, mas representam abordagens distintas de exposição e organização das operações.

No estilo RPC, a interação tende a ser organizada em torno de operações que o cliente solicita ao servidor.

No estilo REST, a interface é organizada em torno de recursos identificados por URLs, utilizando as semânticas dos métodos HTTP para interagir com esses recursos.

Considere um exemplo conceitual de consulta de usuário.

**Abordagem orientada a RPC:**

```text
Executar a operação getUser, informando o identificador.
```

**Abordagem orientada a recursos:**

```http
GET /users/123
```

No segundo exemplo, `/users/123` identifica um recurso específico, e `GET` expressa a intenção de consultá-lo.

A diferença não está simplesmente em utilizar nomes de métodos ou URLs. Uma API REST também possui operações definidas, e uma chamada RPC também pode utilizar URLs. A distinção está no modelo de interação que a interface adota.

| Aspecto | JSONWS / RPC | REST |
|---|---|---|
| Organização conceitual | Operações remotas | Recursos |
| Intenção da requisição | Solicitar a execução de uma operação | Interagir com um recurso por meio de uma operação HTTP |
| Interface | Frequentemente centrada em métodos expostos | Centrada em recursos e semântica HTTP |
| Representação de dados | Pode utilizar JSON | Pode utilizar JSON ou outras representações |
| Transporte | JSONWS utiliza HTTP | APIs REST frequentemente utilizam HTTP |

Essas diferenças são conceituais. Não se deve concluir que uma abordagem é sempre superior à outra sem considerar as necessidades e a arquitetura da aplicação.

## 6. JSONWS, serviços internos e persistência

Uma chamada JSONWS pode resultar na execução de um método de serviço do Liferay. Esse serviço pode, por sua vez, consultar ou modificar dados persistidos, executar validações ou delegar o trabalho a outros componentes.

Um fluxo possível é:

```text
Cliente externo
      |
      | Requisição HTTP
      v
Endpoint JSONWS
      |
      v
Método de serviço exposto
      |
      v
Camada de persistência
      |
      v
Banco de dados
```

Esse diagrama representa um fluxo possível, não uma regra de que toda chamada JSONWS necessariamente acessa o banco de dados.

Uma operação pode realizar apenas um cálculo, consultar outro serviço ou executar qualquer comportamento previsto pela implementação.

O ponto importante é distinguir a interface remota da implementação interna. O cliente conhece a operação disponibilizada e o formato da comunicação, mas não precisa conhecer todos os detalhes internos do servidor.

## 7. Relação com o desenvolvimento de APIs no Liferay

No Liferay, JSONWS e REST Builder estão relacionados à possibilidade de disponibilizar funcionalidades para outros sistemas, mas seguem modelos diferentes.

O JSONWS permite expor operações de serviços remotamente. Já o REST Builder é uma ferramenta utilizada para desenvolver APIs REST a partir de uma descrição de contrato, gerando classes relacionadas à API e ao seu consumo.

Em uma API desenvolvida com REST Builder, o contrato descreve caminhos, operações HTTP, parâmetros, dados e respostas. A implementação do servidor define o comportamento efetivo de cada operação.

Essa distinção ajuda a compreender por que uma API REST não é apenas uma maneira diferente de chamar métodos Java: ela possui uma interface HTTP organizada em torno de recursos e suas representações.

## 8. Resumo

RPC é um modelo de comunicação em que um cliente solicita a execução de uma operação em um sistema remoto.

JSONWS é um mecanismo do Liferay que permite expor operações remotamente, utilizando HTTP e JSON, e está associado a esse estilo de interação.

REST, por outro lado, organiza a interface em torno de recursos e utiliza a semântica dos métodos HTTP para definir como interagir com eles.

A distinção fundamental é: **RPC enfatiza a chamada de operações remotas; REST enfatiza a interação com recursos por meio de uma interface padronizada.**