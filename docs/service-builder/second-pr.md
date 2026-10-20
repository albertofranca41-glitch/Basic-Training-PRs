# Guia: Segundo Pull Request de Service Builder no Liferay — DynamicQuery

Este guia continua o guia do primeiro PR. Lá você implementou consultas para `H7G5Entry` usando os **finders** gerados pelo Service Builder, dividiu a lógica entre Local Service e Remote Service e testou tudo com um portlet e com `curl`.

Nesta segunda parte, a atividade pede que você **explore `DynamicQuery` para substituir o uso dos finders** nas consultas. A motivação é entender uma alternativa mais flexível: os finders são declarados previamente para consultas conhecidas, enquanto as dynamic queries permitem montar combinações de filtros em tempo de execução. O desempenho de ambas depende da consulta resultante, dos índices disponíveis e do plano escolhido pelo banco; nenhum dos dois mecanismos é sempre mais rápido.

Como a arquitetura, o `service.xml`, o Remote Service e o JSONWS já foram explicados no primeiro guia, aqui o foco é **a teoria de DynamicQuery** e **o código que mudou**.

| Etapa | O que você vai aprender |
|---|---|
| **Etapa 1 – Teoria** | O que é DynamicQuery, como funciona, comparação com finders, pontos positivos e negativos |
| **Etapa 2 – O código e os testes** | Quais arquivos enviar, explicação do `H7G5EntryLocalServiceImpl.java`, execução dos testes e erros comuns |

---

## Sumário

- [Etapa 1 – Teoria](#etapa-1--teoria)
  - [1.1 O que é DynamicQuery](#11-o-que-é-dynamicquery)
  - [1.2 Como uma DynamicQuery funciona](#12-como-uma-dynamicquery-funciona)
  - [1.3 Finder × DynamicQuery](#13-finder--dynamicquery)
  - [1.4 Pontos positivos e negativos](#14-pontos-positivos-e-negativos)
- [Etapa 2 – O código e os testes](#etapa-2--o-código-e-os-testes)
  - [2.1 Quais arquivos enviar neste PR](#21-quais-arquivos-enviar-neste-pr)
  - [2.2 O código completo](#22-o-código-completo)
  - [2.3 Explicando o código](#23-explicando-o-código)
  - [2.4 Como executar e testar](#24-como-executar-e-testar)
  - [2.5 Erros comuns](#25-erros-comuns)
  - [Para praticar](#para-praticar)

---

# Etapa 1 – Teoria

## 1.1 O que é DynamicQuery

**DynamicQuery** é uma API do Liferay para montar consultas ao banco **em tempo de execução**, usando objetos Java em vez de SQL escrito à mão. Você parte de uma consulta vazia da entidade e vai acrescentando condições (filtros, ordenação, limites). Só no final a consulta é executada.

A proposta é dar flexibilidade. Com um finder, a consulta precisa ser declarada antecipadamente no `service.xml`, e qualquer combinação nova de filtros exige alterar o `service.xml` e rodar o `buildService` outra vez. Com DynamicQuery, a combinação de filtros é decidida no próprio código Java.

Os principais componentes usados nesta atividade são:

| Componente | Papel |
|---|---|
| `DynamicQuery` | Representa a consulta que está sendo montada |
| `RestrictionsFactoryUtil` | Fábrica de condições (`eq`, `like`, `in`, etc.) e de agrupadores (`conjunction`, `disjunction`) |
| `Conjunction` | Agrupa várias condições ligadas por **AND** |
| `h7g5EntryPersistence.findWithDynamicQuery(...)` | Executa a consulta e devolve a lista de resultados |

## 1.2 Como uma DynamicQuery funciona

O fluxo tem sempre três passos:

```text
dynamicQuery()                          cria a consulta vazia da entidade
        │
        ▼
dynamicQuery.add(restrição)             acrescenta filtros (cláusula WHERE)
        │
        ▼
persistence.findWithDynamicQuery(...)   executa a consulta e devolve a lista
```

Três pontos merecem atenção:

1. **A consulta só roda no último passo.** Chamar `add(...)` apenas monta a consulta; nada é enviado ao banco até a persistência receber a `DynamicQuery`.
2. **O resultado é sempre uma `List`.** Mesmo quando você espera um único registro, a consulta devolve uma lista (possivelmente vazia). Quem decide o que fazer com lista vazia ou com vários itens é o seu código.
3. **Os nomes usados nos filtros são nomes de propriedade da entidade**, os mesmos declarados no `service.xml` (por exemplo, `"h7g5FolderId"` e `"key"`), e não os nomes das colunas SQL.

Algumas restrições comuns de `RestrictionsFactoryUtil`:

| Método | Equivale a |
|---|---|
| `eq("name", valor)` | `name = valor` |
| `ne("name", valor)` | `name <> valor` |
| `like("name", "abc%")` | `name LIKE 'abc%'` |
| `in("h7g5FolderId", lista)` | `h7g5FolderId IN (...)` |
| `gt` / `lt` | maior que / menor que |
| `isNull("description")` | `description IS NULL` |
| `conjunction()` | agrupa condições com **AND** |
| `disjunction()` | agrupa condições com **OR** |

Nesta atividade só usamos `eq` e `conjunction`.

## 1.3 Finder × DynamicQuery

| Aspecto | Finder (1º PR) | DynamicQuery (2º PR) |
|---|---|---|
| Onde é definido | No `service.xml`, antes do `buildService` | No código Java, em tempo de execução |
| Consulta | O finder é declarado no `service.xml` e sua implementação é gerada | A estrutura da consulta é montada no código em tempo de execução |
| Índices | A disponibilidade de índices depende do esquema e da configuração do banco | A mesma regra se aplica; o banco pode usar índices compatíveis com a consulta |
| Resultado único | `findBy...` devolve a entidade ou lança `NoSuchH7G5EntryException` | Devolve sempre uma lista; a exceção precisa ser lançada manualmente |
| Valores nulos | Trata `null` e texto vazio de forma especial (`IS NULL`) | `eq` com `null` não corresponde a nenhum registro |
| Ordenação | Segue a ordenação padrão definida para a entidade, se houver | Nenhuma, a menos que você adicione uma ordem |
| Flexibilidade | Só as combinações declaradas no `service.xml` | Qualquer combinação de filtros |

> **Sobre desempenho:** índices pertencem ao banco de dados e não são exclusivos de finders. O otimizador pode utilizá-los para uma consulta criada por finder ou por `DynamicQuery`, se forem adequados ao SQL resultante. Para comparar desempenho, é preciso considerar a consulta gerada, os índices disponíveis e o plano de execução.

## 1.4 Pontos positivos e negativos

**Pontos positivos**

- Permite combinar filtros livremente (AND, OR, `like`, intervalos) sem alterar o `service.xml`.
- Não exige rodar o `buildService` quando surge uma consulta nova.
- Funciona para consultas condicionais, em que os filtros dependem dos parâmetros recebidos.
- É uma API padrão do Liferay, usada em muitos módulos.

**Pontos negativos**

- Em consultas complexas ou sobre tabelas grandes, pode ser necessário inspecionar o SQL e o plano de execução para avaliar o desempenho e os índices adequados.
- O tratamento de "não encontrado" é responsabilidade do desenvolvedor.
- Os nomes de propriedade são strings: um erro de digitação só aparece em tempo de execução, não na compilação.
- Casos como valores nulos e ordenação precisam ser tratados explicitamente.
- É necessário montar explicitamente as condições no código, o que pode aumentar a complexidade em consultas com muitos filtros.

Por isso, a regra prática é: **use finders para consultas frequentes e previsíveis; use DynamicQuery quando a consulta precisa de mais flexibilidade.**

---

# Etapa 2 – O código e os testes

## 2.1 Quais arquivos enviar neste PR

Apenas **um** arquivo foi alterado em relação ao primeiro PR, mas o PR deve conter os **quatro**:

```text
H7G5EntryLocalServiceImpl.java  # ALTERADO: consultas agora usam DynamicQuery
H7G5EntryServiceImpl.java       # igual ao 1º PR: permissão + chamadas ao Local Service
H7G5Portlet.java                # igual ao 1º PR: chama o Remote Service
<seu-script-de-testes>.sh       # igual ao 1º PR: chamadas JSONWS via curl
```

Por que enviar os quatro, se três não mudaram?

- **A atividade pede isso em cada PR:** os arquivos Java personalizados e o script com todos os `curl`. O segundo PR é uma entrega completa.
- **Os três arquivos provam que a troca não quebrou nada.** Como as assinaturas dos métodos continuam as mesmas, o Remote Service, o portlet e o script funcionam sem alterações. Quem revisa vê o conjunto inteiro funcionando com a nova implementação.
- **O script de `curl` é o teste do comportamento.** Ele deve produzir os mesmos resultados de antes, o que mostra que a substituição de finder por DynamicQuery foi equivalente.

Continua valendo a regra do primeiro PR: **não inclua o `service.xml` nem os arquivos gerados pelo Service Builder.** Antes de abrir o PR, confira com `git status` que os quatro arquivos estão na branch e que nenhum arquivo gerado foi junto.

## 2.2 O código completo

```java
/**
 * SPDX-FileCopyrightText: (c) 2026 Liferay, Inc. https://liferay.com
 * SPDX-License-Identifier: LGPL-2.1-or-later OR LicenseRef-Liferay-DXP-EULA-2.0.0-2023-06
 */

package com.liferay.h7g5.service.impl;

import com.liferay.h7g5.exception.NoSuchH7G5EntryException;
import com.liferay.h7g5.model.H7G5Entry;
import com.liferay.h7g5.service.base.H7G5EntryLocalServiceBaseImpl;
import com.liferay.portal.aop.AopService;
import com.liferay.portal.kernel.dao.orm.Conjunction;
import com.liferay.portal.kernel.dao.orm.DynamicQuery;
import com.liferay.portal.kernel.dao.orm.RestrictionsFactoryUtil;
import com.liferay.portal.kernel.exception.PortalException;

import java.util.List;

import org.osgi.service.component.annotations.Component;

/**
 * Implementação do Local Service da entidade {@link H7G5Entry}.
 *
 * <p>
 * Os métodos de consulta desta classe usam <code>DynamicQuery</code> em vez dos
 * finders gerados pelo Service Builder (<code>findBy...</code>). A assinatura
 * pública de cada método é a mesma da versão com finders; por isso, o Remote
 * Service, o portlet e o script de <code>curl</code> continuam funcionando sem
 * alterações.
 * </p>
 *
 * <p>
 * Diferenças importantes em relação aos finders:
 * </p>
 *
 * <ul>
 * <li>
 * O finder é declarado previamente, enquanto a <code>DynamicQuery</code> é
 * montada em tempo de execução. O desempenho de ambas depende da consulta
 * resultante, dos índices disponíveis e do plano escolhido pelo banco.
 * </li>
 * <li>
 * Nos finders de retorno único, o Service Builder lança
 * <code>NoSuchH7G5EntryException</code> quando não há resultado. A
 * <code>DynamicQuery</code> sempre devolve uma lista, então essa exceção é
 * lançada manualmente.
 * </li>
 * <li>
 * A <code>DynamicQuery</code> não aplica nenhuma ordenação por conta própria.
 * </li>
 * </ul>
 *
 * @author Alberto
 */
@Component(
    property = "model.class.name=com.liferay.h7g5.model.H7G5Entry",
    service = AopService.class
)
public class H7G5EntryLocalServiceImpl extends H7G5EntryLocalServiceBaseImpl {

    /**
     * Lista todos os H7G5Entry que pertencem a uma determinada pasta. Se a
     * pasta não tiver nenhum, a lista retorna vazia.
     *
     * @param  h7g5FolderId o id da pasta
     * @return os H7G5Entry da pasta
     */
    public List<H7G5Entry> getByH7G5FolderId(long h7g5FolderId) {

        // Cria uma consulta vazia já associada à entidade H7G5Entry. O método
        // dynamicQuery() é herdado da classe-base gerada pelo Service Builder.

        DynamicQuery dynamicQuery = dynamicQuery();

        // O método add() acrescenta um filtro (h7g5FolderId = ?). O nome usado
        // é o da propriedade da entidade, declarado no service.xml, e não o
        // nome da coluna no banco.

        dynamicQuery.add(
            RestrictionsFactoryUtil.eq("h7g5FolderId", h7g5FolderId));

        // Até aqui nada foi consultado no banco: a consulta só é executada
        // quando a persistência recebe a DynamicQuery.

        return h7g5EntryPersistence.findWithDynamicQuery(dynamicQuery);
    }

    /**
     * Busca um único H7G5Entry pelos 3 atributos: id da pasta, descrição e
     * nome. Substitui o finder H_D_N gerado pelo Service Builder.
     *
     * <p>
     * Atenção: o finder gerado tratava uma descrição nula ou vazia de forma
     * especial (<code>IS NULL</code>). Com <code>eq</code>, um valor nulo não
     * corresponde a nenhum registro.
     * </p>
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

        DynamicQuery dynamicQuery = dynamicQuery();

        // Uma Conjunction agrupa várias condições ligadas por AND, ou seja,
        // o registro só é devolvido se TODAS elas forem verdadeiras.

        Conjunction conjunction = RestrictionsFactoryUtil.conjunction();

        conjunction.add(
            RestrictionsFactoryUtil.eq("h7g5FolderId", h7g5FolderId));
        conjunction.add(
            RestrictionsFactoryUtil.eq("description", description));
        conjunction.add(RestrictionsFactoryUtil.eq("name", name));

        // A Conjunction inteira é adicionada à consulta como um único filtro.

        dynamicQuery.add(conjunction);

        List<H7G5Entry> h7g5Entries = h7g5EntryPersistence.findWithDynamicQuery(
            dynamicQuery);

        // O finder gerado lançava NoSuchH7G5EntryException quando não
        // encontrava nada. A DynamicQuery apenas devolve uma lista vazia,
        // então a exceção precisa ser lançada manualmente.

        if (h7g5Entries.isEmpty()) {
            throw new NoSuchH7G5EntryException(
                "No H7G5Entry exists with the given h7g5FolderId, " +
                    "description and name");
        }

        return h7g5Entries.get(0);
    }

    /**
     * Busca um único H7G5Entry pela chave. Substitui o finder Key gerado pelo
     * Service Builder.
     *
     * <p>
     * O finder Key foi declarado com <code>return-type="H7G5Entry"</code> no
     * service.xml, ou seja, a chave identifica uma única entrada. Por isso o
     * método devolve um objeto e não uma lista.
     * </p>
     *
     * @param  key a chave do H7G5Entry
     * @return o H7G5Entry que possui essa chave
     * @throws PortalException se nenhum H7G5Entry com essa chave for encontrado
     */
    public H7G5Entry getByKey(String key) throws PortalException {
        DynamicQuery dynamicQuery = dynamicQuery();

        dynamicQuery.add(RestrictionsFactoryUtil.eq("key", key));

        List<H7G5Entry> h7g5Entries = h7g5EntryPersistence.findWithDynamicQuery(
            dynamicQuery);

        // Lista vazia significa que nenhuma entrada possui essa chave.

        if (h7g5Entries.isEmpty()) {
            throw new NoSuchH7G5EntryException(
                "No H7G5Entry exists with key " + key);
        }

        return h7g5Entries.get(0);
    }

    /**
     * Lista os H7G5Entry que possuem um determinado nome. O nome não é
     * necessariamente único, por isso o método retorna uma lista. Se nenhum
     * for encontrado, a lista retorna vazia.
     *
     * @param  name o nome dos H7G5Entry
     * @return os H7G5Entry que possuem esse nome
     */
    public List<H7G5Entry> getByName(String name) {
        DynamicQuery dynamicQuery = dynamicQuery();

        dynamicQuery.add(RestrictionsFactoryUtil.eq("name", name));

        return h7g5EntryPersistence.findWithDynamicQuery(dynamicQuery);
    }

}
```

## 2.3 Explicando o código

### O que mudou nos imports

Em relação ao primeiro PR, entraram quatro imports novos:

| Import | Para quê |
|---|---|
| `NoSuchH7G5EntryException` | Exceção lançada manualmente quando uma busca de resultado único não encontra nada. É gerada pelo Service Builder no módulo API |
| `DynamicQuery` | O tipo da consulta que está sendo montada |
| `RestrictionsFactoryUtil` | Cria as condições (`eq`) e o agrupador `conjunction` |
| `Conjunction` | O tipo do agrupador de condições ligadas por AND |

A classe continua estendendo `H7G5EntryLocalServiceBaseImpl`. É dessa classe-base que vêm tanto o `h7g5EntryPersistence` quanto o método `dynamicQuery()`, usado em todos os métodos.

### `getByH7G5FolderId`: o padrão básico

```java
DynamicQuery dynamicQuery = dynamicQuery();

dynamicQuery.add(
    RestrictionsFactoryUtil.eq("h7g5FolderId", h7g5FolderId));

return h7g5EntryPersistence.findWithDynamicQuery(dynamicQuery);
```

Este método mostra o padrão que todos os outros seguem:

1. `dynamicQuery()` cria a consulta vazia da entidade `H7G5Entry`.
2. `RestrictionsFactoryUtil.eq("h7g5FolderId", h7g5FolderId)` cria a condição `h7g5FolderId = ?`.
3. `add(...)` acrescenta a condição à consulta.
4. `findWithDynamicQuery(...)` executa a consulta e devolve a lista.

O retorno é `List<H7G5Entry>` e o método não declara `throws`, porque uma consulta que não encontra nada devolve uma lista vazia, sem erro. É exatamente o que o finder de coleção fazia.

### `getByName`: o mesmo padrão com outro filtro

```java
dynamicQuery.add(RestrictionsFactoryUtil.eq("name", name));

return h7g5EntryPersistence.findWithDynamicQuery(dynamicQuery);
```

Só muda a propriedade filtrada. Como o nome pode se repetir entre entradas, o retorno continua sendo uma lista.

### `getByKey`: tratando "não encontrado"

```java
List<H7G5Entry> h7g5Entries = h7g5EntryPersistence.findWithDynamicQuery(
    dynamicQuery);

if (h7g5Entries.isEmpty()) {
    throw new NoSuchH7G5EntryException(
        "No H7G5Entry exists with key " + key);
}

return h7g5Entries.get(0);
```

Este é o ponto mais importante do PR. O finder `findByKey` devolvia um `H7G5Entry` ou lançava `NoSuchH7G5EntryException`. A DynamicQuery devolve sempre uma lista, então o método:

1. recebe a lista;
2. se ela estiver vazia, lança a exceção manualmente, reproduzindo o comportamento do finder;
3. caso contrário, devolve o primeiro elemento com `get(0)`.

`NoSuchH7G5EntryException` é uma subclasse de `PortalException`. Por isso o `throws PortalException` da assinatura continua suficiente, e o Remote Service e o portlet, que já tratavam `PortalException`, não precisam de nenhuma alteração. A mensagem segue o estilo das exceções geradas pelo Service Builder.

### `getByH_D_N`: combinando condições com `Conjunction`

```java
Conjunction conjunction = RestrictionsFactoryUtil.conjunction();

conjunction.add(
    RestrictionsFactoryUtil.eq("h7g5FolderId", h7g5FolderId));
conjunction.add(
    RestrictionsFactoryUtil.eq("description", description));
conjunction.add(RestrictionsFactoryUtil.eq("name", name));

dynamicQuery.add(conjunction);
```

O finder `H_D_N` usava três colunas ao mesmo tempo. Aqui, a mesma ideia é construída com uma `Conjunction`: um grupo de condições ligadas por **AND**. O registro só é devolvido se a pasta, a descrição e o nome corresponderem simultaneamente. O grupo inteiro é adicionado à consulta com um único `dynamicQuery.add(conjunction)`.

Depois, o tratamento do resultado é idêntico ao de `getByKey`: lista vazia vira `NoSuchH7G5EntryException`, e um resultado válido é devolvido com `get(0)`.

### Pontos de atenção

- **Valores nulos.** O finder gerado tratava `description` nula ou vazia de forma especial. Com `eq`, um valor `null` não corresponde a nenhum registro. Se o seu teste usar descrição nula ou vazia, o resultado pode ser diferente do primeiro PR.
- **Ordenação.** A DynamicQuery não ordena os resultados. Se a ordem importar, ela precisa ser adicionada explicitamente.
- **Vários resultados em consultas de resultado único.** `getByKey` e `getByH_D_N` carregam a lista completa e devolvem apenas o primeiro elemento. Para um conjunto pequeno isso não é problema, mas é uma diferença em relação ao finder.

### E os outros três arquivos?

`H7G5EntryServiceImpl.java`, `H7G5Portlet.java` e o script de `curl` **não mudaram**. O Remote Service continua chamando `h7g5EntryLocalService.getBy...(...)` com os mesmos parâmetros e tipos de retorno, e o portlet continua chamando o Remote Service. Para eles, a troca de finder por DynamicQuery é invisível.

## 2.4 Como executar e testar

Execute os comandos a partir da raiz do projeto.

### 1. Compilar

```bash
./gradlew classes
```

Se aparecer erro de símbolo não encontrado, veja a seção de erros comuns.

### 2. Implantar

```bash
./gradlew deploy -Ddeploy.docker.container.id=<nome-do-seu-container>
```

Substitua o nome do container pelo do seu ambiente e confirme no console do Liferay que os módulos iniciaram sem erros. Como as assinaturas dos métodos não mudaram e o `service.xml` não foi alterado, não é necessário rodar o `buildService` outra vez.

### 3. Testar o portlet

Adicione o portlet **H7G5** a uma página e atualize. O console deve mostrar as mesmas mensagens do primeiro PR, incluindo `Entries in folder: ...`.

### 4. Testar com o script de `curl`

Use o mesmo script do primeiro PR, confirmando os caminhos na página `http://localhost:8080/api/jsonws`. Execute um comando por vez e compare com o comportamento esperado:

| Chamada | Resultado esperado |
|---|---|
| `get-by-name` com um nome existente | Lista com as entradas que têm esse nome |
| `get-by-name` com um nome inexistente | Lista vazia |
| `get-by-h7-g5-folder-id` com uma pasta com entradas | Lista com as entradas da pasta |
| `get-by-h7-g5-folder-id` com uma pasta sem entradas | Lista vazia |
| `get-by-key` com uma chave existente | A entrada correspondente em JSON |
| `get-by-key` com uma chave inexistente | Erro com a mensagem `No H7G5Entry exists with key ...` |
| `get-by-h_-d_-n` com a combinação existente | A entrada correspondente em JSON |
| `get-by-h_-d_-n` com uma combinação inexistente | Erro com a mensagem `No H7G5Entry exists with the given h7g5FolderId, description and name` |
| Qualquer chamada com usuário fora de `@liferay.com` | Exceção de permissão, vinda do Remote Service |

A comparação importante é com o primeiro PR: mesmos parâmetros, mesmos resultados. As duas chamadas de "não encontrado" agora retornam a exceção lançada manualmente no Local Service.

## 2.5 Erros comuns

| Sintoma | Causa provável | Como resolver |
|---|---|---|
| `cannot find symbol` para `NoSuchH7G5EntryException` | Import ausente ou incorreto, ou classe gerada ainda não existe | Confira o import (`com.liferay.h7g5.exception`) e, se necessário, rode `./gradlew h7g5-service:buildService` |
| `cannot find symbol` para `dynamicQuery()` ou `h7g5EntryPersistence` | A classe não estende `H7G5EntryLocalServiceBaseImpl` | Confirme a herança da classe |
| Erro de propriedade não encontrada (`could not resolve property`) | Nome da propriedade digitado errado no `eq`, ou nome de coluna usado no lugar do nome da propriedade | Use exatamente o nome declarado no `service.xml`, como `"h7g5FolderId"` e `"key"` |
| `get-by-h_-d_-n` não encontra entrada que existe | Descrição nula ou vazia, que o finder tratava de forma especial | Teste com uma descrição preenchida e compare com o comportamento do primeiro PR |
| Consulta de chave inexistente devolve erro, mas o portlet continua funcionando | Comportamento esperado: o portlet só usa `getByH7G5FolderId`, que não lança erro de "não encontrado" | Teste a chave pelo `curl` |
| `get-by-key` retorna lista em vez de objeto | O método foi alterado para devolver lista | Mantenha o retorno `H7G5Entry` e use `get(0)` |
| O Remote Service ou o portlet deixou de compilar | A assinatura de algum método do Local Service foi alterada | Restaure os tipos de parâmetro e de retorno originais |

## Para praticar

1. O que muda na prática entre `h7g5EntryPersistence.findByKey(key)` e `findWithDynamicQuery(dynamicQuery)` quando nenhuma entrada é encontrada?
2. Por que `getByH7G5FolderId` e `getByName` não precisam declarar `throws PortalException`, enquanto `getByKey` e `getByH_D_N` precisam?
3. Para que serve a `Conjunction` em `getByH_D_N`? O que mudaria se você usasse uma `Disjunction`?
4. Por que o Remote Service, o portlet e o script de `curl` não precisaram de alteração?
5. Cite uma situação em que você preferiria um finder e outra em que preferiria uma DynamicQuery.
6. Execute `get-by-key` com uma chave inexistente e compare a mensagem de erro com a do primeiro PR.

---

## Referência principal

Este guia complementa o guia do primeiro PR de Service Builder e foi elaborado com base no tutorial de Service Builder do Liferay fornecido para a atividade e na nova versão do `H7G5EntryLocalServiceImpl.java`, que substitui os finders por `DynamicQuery`.
