# 10 · Segurança

O que foi fechado antes de abrir a demo para qualquer pessoa.

## Servidor

- Docker rootless, num usuário próprio sem sudo.
- Containers só de leitura, sem capabilities, sem novos privilégios, com limite de memória.
- nginx com política de conteúdo rígida, `nosniff`, sem referrer, sem câmera, microfone ou localização, e `noindex`.
- Só `/` e `/api` respondem; qualquer outro caminho dá 404. Corpo de requisição limitado a 32 KB.

## Dados

- Tudo fictício. Nenhum dado de cliente nem credencial do sistema real entra na demo, no seed ou neste repositório.
- O IP do visitante nunca é guardado: só um hash com segredo.
- Chaves de API e o segredo dos eventos só no `.env` do servidor.

## Abuso

- Limite de mudanças em sequência por endereço, porque a base é compartilhada.
- Limite de empresas novas por endereço por dia.
- Limites da IA por visitante, por dia e por verba (ver [Algory](../07-algory/#gestão-dos-tokens)).
- Reset de hora em hora: qualquer estrago dura no máximo uma hora.

## Integração

- Eventos sem assinatura HMAC válida são recusados com 401; repetidos, com 409.
- O segredo é copiado à mão entre os servidores, nunca por chat ou repositório.

## IA

- A pergunta do visitante vai marcada e as regras mandam ignorar instruções dentro dela.
- A IA não executa nada e não tem ferramentas; rascunhos de fluxo passam por conferência no servidor.
