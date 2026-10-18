# 🔀 Guia Completo: Como Enviar um Pull Request (PR)

> Um passo a passo prático para manter seu fork sincronizado e enviar suas atividades corretamente.

Este guia mostra o fluxo completo para enviar uma atividade via **Pull Request (PR)**: desde sincronizar seu fork com o repositório original até abrir o PR no GitHub. Seguir esses passos na ordem certa evita conflitos, mantém seu histórico limpo e garante que cada atividade comece sempre a partir da versão mais atual do projeto.

---

## 📑 O que você vai encontrar aqui

1. Entendendo os remotes: `origin` vs `upstream`
2. Etapa 1 — Sincronizar o fork com o upstream
3. Etapa 2 — Criar uma nova branch
4. Etapa 3 — Fazer as alterações da atividade
5. Etapa 4 — Adicionar as alterações (`git add`)
6. Etapa 5 — Commit
7. Etapa 6 — Enviar (`push`) a branch
8. Etapa 7 — Abrir o Pull Request
9. Checklist rápido com todos os comandos
10. Erros comuns e cuidados

---

## 🧭 Entendendo os remotes: `origin` vs `upstream`

Antes de tudo, vale entender os dois remotes envolvidos nesse fluxo:

| Remote | O que é | Papel no fluxo |
|---|---|---|
| **`upstream`** | O repositório original, de onde as atividades partem | Fonte da verdade — nunca recebe push direto |
| **`origin`** | O seu fork (cópia pessoal no seu GitHub) | Onde você envia (`push`) suas branches e de onde abre o PR |

> 💡 **No seu caso:** o `upstream` costuma apontar para o repositório da atividade (por exemplo, sob a conta **brianchandotcom**), enquanto o `origin` é o seu fork pessoal desse repositório.

Se ainda não tiver o `upstream` configurado localmente, adicione uma única vez com:

```bash
git remote add upstream <URL-do-repositorio-original>
```

Você pode conferir os remotes atuais a qualquer momento com `git remote -v`.

---

## 🔄 Etapa 1 — Sincronizar o fork (`origin`) com o `upstream`

Esse é o passo mais importante e deve ser feito **sempre antes de começar uma nova atividade**. O objetivo é garantir que sua branch `master` fique idêntica à do repositório original, evitando conflitos e histórico bagunçado.

```bash
git fetch upstream
git checkout master
git reset --hard upstream/master
git push --force origin master
```

### O que cada comando faz

| Comando | O que faz |
|---|---|
| `git fetch upstream` | Baixa todo o histórico mais recente do repositório original, **sem** aplicá-lo ainda |
| `git checkout master` | Garante que você está posicionado na branch `master` local |
| `git reset --hard upstream/master` | Reescreve sua `master` local para ficar **idêntica** à `master` do upstream, descartando qualquer diferença |
| `git push --force origin master` | Sobrescreve a `master` do seu fork no GitHub com essa versão sincronizada |

> ⚠️ **Atenção:** `reset --hard` e `push --force` são comandos destrutivos — eles descartam alterações locais e sobrescrevem o histórico remoto. Por isso:
> - Só rode essa sequência estando na `master` — **nunca** em uma branch com trabalho em andamento que você queira manter.
> - Trate sua `master` sempre como um espelho do `upstream`; todo o seu trabalho real acontece em branches separadas (próxima etapa).

---

## 🌿 Etapa 2 — Criar uma nova branch

Com a `master` sincronizada, crie uma branch nova para a atividade que você vai desenvolver:

```bash
git checkout -b nome-da-sua-branch
```

> 💡 **Dica de nomenclatura:** use um nome curto e descritivo, relacionado à atividade — por exemplo `h7g5-dynamic-query` ou `r3b2-crud-goo`. Isso facilita identificar o conteúdo da branch (e do PR) só pelo nome.

Essa branch nasce a partir da `master` já atualizada, então você sempre começa do ponto mais recente do projeto.

---

## ✏️ Etapa 3 — Fazer as alterações da atividade

Agora é a parte "de verdade": implemente o código, os arquivos ou as mudanças que a atividade pede, normalmente dentro do seu editor/IDE.

> 💡 Use `git status` a qualquer momento para ver quais arquivos foram modificados, criados ou removidos.

---

## 📦 Etapa 4 — Adicionar as alterações (`git add`)

Com as mudanças prontas, adicione todos os arquivos alterados à área de staging:

```bash
git add .
```

> 💡 O `.` adiciona **todos** os arquivos alterados no diretório atual e subdiretórios. Para adicionar apenas arquivos específicos, use `git add caminho/do/arquivo` no lugar.

---

## 💾 Etapa 5 — Commit

Registre as alterações com uma mensagem clara sobre o que foi feito:

```bash
git commit -m "Descreva aqui o que foi implementado"
```

> 💡 **Boas mensagens de commit** costumam resumir o "o quê" em vez do "como" — por exemplo, `"Implementa CRUD de Goo no R3B2Portlet"` em vez de apenas `"mudanças"` ou `"fix"`.

---

## 🚀 Etapa 6 — Enviar (`push`) a branch para o seu fork

```bash
git push origin nome-da-sua-branch
```

Isso envia sua branch (com os commits) para o seu fork no GitHub — ainda **não** para o `upstream`.

---

## 🔁 Etapa 7 — Abrir o Pull Request no GitHub

> ⚠️ **Importante:** o PR precisa ser aberto **no repositório `upstream`** (o repositório do Basic-Traininig que Brian fez o fork, não o seu). É lá que o Brian revisa as atividades — enviar o PR contra o seu próprio fork faz com que ele nunca chegue até quem vai avaliar.

1. Depois do push, o GitHub costuma mostrar um banner automático **"Compare & pull request"** tanto no seu fork, como no do Brian, clique no banner do fork dele. Ou se preferir, acesse diretamente o **repositório `upstream`** no GitHub e vá em **Pull Requests → New Pull Request**.
2. Confira **com atenção** os dois lados da comparação antes de criar o PR:
   - **base repository:** `upstream` / branch `master` ← é para cá que o PR precisa apontar
   - **head repository:** `origin` (seu fork) / sua branch nova ← de onde vêm suas alterações
3. Escreva um título e uma descrição explicando o que a atividade implementa.
4. Clique em **Create Pull Request**.
5. Confirme, na página do PR já criado, que a URL e o cabeçalho mostram o repositório `upstream` (não o seu fork) como destino.

---

## ✅ Checklist rápido — todos os comandos em sequência

Para copiar e colar quando já souber o fluxo de cor:

```bash
# 1. Sincronizar o fork com o upstream
git fetch upstream
git checkout master
git reset --hard upstream/master
git push --force origin master

# 2. Criar a branch da atividade
git checkout -b nome-da-sua-branch

# 3. (fazer as alterações no código)

# 4. Adicionar as alterações
git add .

# 5. Commit
git commit -m "mensagem descritiva"

# 6. Push da branch para o fork
git push origin nome-da-sua-branch

# 7. Abrir o PR no GitHub (via interface web)
```

---

## ⚠️ Erros comuns e cuidados

- **Rodar o `reset --hard` na branch errada:** confirme com `git branch` que está na `master` antes da Etapa 1 — se estiver em outra branch com trabalho não commitado, ele será perdido.
- **Esquecer de sincronizar antes de começar:** pular a Etapa 1 pode fazer sua branch nascer desatualizada, gerando conflitos na hora do PR.
- **Commitar direto na `master`:** trate a `master` sempre como somente leitura/espelho — todo trabalho deve estar em uma branch própria.
- **Abrir o PR contra a branch errada:** confira sempre se o `base` do PR aponta para `upstream/master`.
- **Abrir o PR no repositório errado:** o PR precisa ir para o repositório `upstream`, não para o seu próprio fork — é lá que ele fica visível para quem vai revisar a atividade.

---

## 🎯 Resumo visual do fluxo

```
upstream (repositório original)
      │
      │  git fetch + reset --hard
      ▼
origin/master (seu fork, sincronizado)
      │
      │  git checkout -b nome-da-branch
      ▼
sua branch local  →  você faz as alterações
      │
      │  git add . → git commit → git push origin nome-da-branch
      ▼
origin/nome-da-branch (no GitHub)
      │
      │  abrir Pull Request
      ▼
upstream (aguardando revisão / merge)

```
---