# 9 · Documentação técnica

Estrutura, decisões e testes da versão fx032 (13/11/2026), sem o código-fonte.

## 1. Visão geral

```
visitante ──HTTPS──> nginx da VPS ──> web (nginx, sem root) ──/api──> app (FastAPI) ──> PostgreSQL
                                                                          │
                                                       worker: reset de hora em hora, fila de eventos, planilha
                                                                          │
                                                Quasar (eventos assinados) · Claude / Groq / Gemini (Algory)
```

Tudo roda em Docker rootless, num usuário próprio da VPS, sem sudo.

## 2. Por que essas ferramentas

| Ferramenta | Por quê |
|---|---|
| FastAPI | API pequena, tipada e rápida de escrever; as regras ficam no servidor, não na página |
| PostgreSQL 16 | dados em JSON onde a forma muda (tickets, casos) e tabelas normais onde precisa de conta (uso da IA, eventos); um esquema separado sobrevive ao reset |
| HTML/CSS/JS sem framework | uma página só, sem build pesado, que carrega rápido e roda numa política de conteúdo rígida |
| Docker rootless | o app não tem como virar root na VPS, que é compartilhada com outros projetos |
| nginx | serve a página, repassa só `/api` e devolve 404 para todo o resto |
| Claude Haiku, Groq, Gemini | um modelo bom e barato como principal e dois gratuitos como reserva, todos chamados pelo mesmo formato |
| Playwright | testes de ponta a ponta na página real, com cliques e a IA simulada |

## 3. Estrutura

| Parte | O que faz |
|---|---|
| `app` | API: leitura do estado, mudanças com regra, eventos, Algory, integrações |
| `motor` | medição: SLA, tempos, conversão, metas e caixa, calculados na hora; não escreve nada |
| `carga` | prepara o banco e recarrega o seed a cada reset |
| `worker` | espera a hora cheia e reseta; entrega a fila de eventos; lê a planilha |
| `ia` | cadeia de modelos, limites, custo e detecção de idioma |
| `saida` | fila de eventos para o Quasar, com assinatura e novas tentativas |
| `web` | a página única montada a partir de módulos |

## 4. Serviços e configuração

Quatro serviços no Compose: banco, app, worker e web. App e web rodam com sistema de arquivos só de leitura, sem nenhuma capability, sem novos privilégios e com limite de memória.

As chaves (Anthropic, Groq, e-mail) e o segredo dos eventos ficam só no `.env` da VPS, nunca no repositório e nunca na página.

## 5. Sessão e papéis

Sem senha na demo: três papéis (dono, supervisora, consultora). A troca de papel muda o que a API aceita, não só o que a tela mostra. O administrador real não existe na demo.

## 6. Dados

O seed tem a empresa, 7 equipes, 18 pessoas, 557 clientes, o catálogo de processos, as regras de cobrança, os tickets e os casos. Tempos ficam em minutos relativos ao reset: a cada hora a base volta com as mesmas idades, então a demo sempre parece "agora".

## 7. Testes

| | Resultado |
|---|---|
| API (regras do quadro, papéis, eventos, limites) | 25 de 25 |
| Página (navegação, missões, construtor, idiomas, Algory) | 3 suítes, sem erros |
| IA com provedores simulados (cadeia, limites, verba, falha, cabeçalho de teste) | todos os casos |
| IA com o modelo real, no servidor | 4 de 4 (PT, EN, ES e rascunho de fluxo) |

## 8. Operação

Cada versão é um pacote único que confere os próprios arquivos por hash, guarda a versão anterior, sobe os containers e roda uma conferência no fim: página certa, cabeçalhos de segurança, estado da base, medição, IA, evento sem assinatura recusado e 404 fora das rotas.
