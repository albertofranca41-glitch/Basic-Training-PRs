<h1 align="center">Basic Training Liferay</h1>

<p align="center">
  <sub>Guias de estudo para o Projeto Soli Deo Gloria</sub>
</p>

---

## Sobre

Este repositório foi criado para os futuros alunos do **Projeto Soli Deo Gloria** que vão estudar o **Basic Training da Liferay**.

O objetivo é oferecer uma documentação direta e fácil de acompanhar, que sirva de apoio para entender os conceitos do treinamento e realizar as atividades e Pull Requests.

## Como usar

Os guias podem ser lidos de duas formas:

- **Para estudar:** leia os conceitos antes de começar uma atividade.
- **Para acompanhar:** consulte o guia correspondente enquanto desenvolve o Pull Request.

Um bom ponto de partida é o guia de início, que também explica como trabalhar com Pull Requests.

## Organização

Todo o material fica na pasta `docs/`, dividido por tema. Cada pasta tem um `README.md` que funciona como índice.

```text
docs/
├── Beginning/         início e Pull Requests
├── JavaScript/        conceitos e PRs de JavaScript
└── Service-Builder/   conceitos e PRs de Service Builder
```

Dentro de JavaScript e Service Builder, os guias seguem a mesma lógica: primeiro os conceitos, depois os Pull Requests do treinamento.

---

## Navegação

### Início

- [Guia de início e Pull Requests](docs/Beginning/README.md): como começar e como trabalhar com Pull Requests.

### JavaScript

Conceitos

- [Índice dos guias](docs/JavaScript/README.md): visão geral dos materiais de JavaScript.
- [API](docs/JavaScript/API.md): conceitos de API.
- [HTTP](docs/JavaScript/HTTP.md): conceitos de HTTP.
- [Cliente e servidor](docs/JavaScript/Cliente%20e%20Servidor.md): como cliente e servidor se comunicam.

Pull Requests

- [Primeiro PR: CRUD de blogs](docs/JavaScript/First_PR.md)
- [Segundo PR: upload de documentos](docs/JavaScript/Second_PR.md)

### Service Builder

Conceitos

- [Índice dos guias](docs/Service-Builder/README.md): visão geral dos materiais de Service Builder.
- [Fluxo de requisição no Liferay](docs/Service-Builder/entendendo-fluxo-requisicao-liferay.md): como uma requisição é tratada.
- [Introdução ao OSGi no Liferay](docs/Service-Builder/introducao-osgi-liferay.md): primeiros passos com OSGi.
- [Introdução ao Service Builder](docs/Service-Builder/introducao-service-builder-liferay.md): o que é e como funciona.

Pull Requests

- [Primeiro PR: Finder Methods](docs/Service-Builder/First_PR.md)
- [Segundo PR: Dynamic Query](docs/Service-Builder/Second_PR.md)