# Cliente e servidor

**Cliente-servidor** é um modelo de comunicação em que um componente, o cliente, solicita um recurso ou uma operação a outro componente, o servidor. O servidor recebe a solicitação, processa-a e devolve uma resposta.

```text
Cliente ─── requisição ───> Servidor
Cliente <──── resposta ──── Servidor
```

Cliente e servidor são papéis, não necessariamente máquinas diferentes. Um navegador, um aplicativo JavaScript, um programa Java ou outro serviço podem atuar como clientes. O servidor pode ser um único processo ou envolver vários componentes, como aplicação, banco de dados e cache.

Na Web, esse diálogo costuma usar HTTP. O cliente inicia a requisição; o servidor a processa e envia a resposta. Os detalhes da estrutura dessas mensagens, dos métodos e dos códigos de status estão em [HTTP](./http.md). Para entender como APIs oferecem operações aos clientes, consulte [API](./api.md).

Por exemplo, ao acessar `https://github.com`, o navegador atua como cliente. A infraestrutura que recebe e processa a solicitação atua como servidor. Em termos simplificados:

```text
Navegador → servidor/aplicação → resposta → navegador
                         ↕
                    banco de dados
```
