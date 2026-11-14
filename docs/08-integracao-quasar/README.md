# 8 · Integração com o Quasar

O Quasar é o comercial da Demo Corporation e o Flux é a operação. Cada um tem o próprio banco; eles conversam por eventos HTTP assinados. A documentação do lado do Quasar está na [parte 7 do repositório dele](https://github.com/morigan-cybsec/quasar-demo/tree/main/docs/07-integracao-flux).

## Quasar → Flux

| Evento | O que o Flux faz |
|---|---|
| `lead.qualificado` | abre um caso em Captar |
| `proposta.aceita` | marca o caso como aguardando o fechamento da venda |
| `venda.fechada` | cria o cliente e o caso de instalação; devolve o código do cliente (`C-…`) para o Quasar guardar |
| `cliente.pediu_cancelamento` | abre o caso de encerramento, com a tentativa de retenção |

## Flux → Quasar

| Evento | Quando |
|---|---|
| `cliente.ativado` | a instalação foi concluída no Flux |
| `cliente.cancelado` | o encerramento terminou em cancelamento |
| `cobranca.recusada` | a cobrança recorrente foi recusada |
| `fidelidade.vence_60d` | faltam 60 dias para o fim da fidelidade: vira trabalho de renovação no Quasar |

Os eventos de saída vão para uma fila no banco. Um processo separado entrega a cada 30 s, com espera crescente entre tentativas e no máximo 8 tentativas. Recusa definitiva (4xx, menos 408 e 429) para de tentar e guarda o motivo, que aparece na tela de integrações.

## Contrato

- O corpo é assinado com HMAC-SHA256 e um segredo que fica só no `.env` de cada lado, no cabeçalho `x-flux-assinatura`.
- Cada evento tem identificador próprio: repetido responde 409, e quem envia trata 409 como entregue.
- Sem assinatura ou com assinatura errada: 401.
- Cada evento leva a hora de envio; mais de 5 minutos de diferença ou JSON inválido: 400.
- Os eventos levam o código do cliente e, quando existe, o lead de origem no Quasar.

## A base compartilhada

Os dois sistemas partem do mesmo seed: planos, pessoas e os 557 assinantes da Demo Corporation. Por isso um cliente que nasceu no Quasar e foi instalado no Flux é o mesmo dos dois lados.
