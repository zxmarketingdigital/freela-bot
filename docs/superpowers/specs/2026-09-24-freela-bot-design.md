# freela-bot — Design

> Data: 24/Set/2026 · Dono: Rafael Castro · Status: design aprovado, aguardando plano de implementação

## Objetivo

Robô 100% automático que opera a conta do Rafael no 99Freelas: acha projetos de landing page e site institucional, envia proposta com preço competitivo, conversa e negocia no chat, e — ao fechar — produz, publica e entrega o site sem depender de aprovação humana em nenhum ponto.

## Decisões travadas (todas do Rafael, 24/Set/26)

| Tema | Decisão |
|---|---|
| Abordagem | Robô próprio (Python + Playwright) rodando no Mac do Rafael. Quem envia é o programa, não uma sessão do Claude no navegador. |
| Aprovação humana | Nenhuma. As 4 partes (prospecção, monitor, negociação, entrega) são automáticas. |
| Escopo aceito | **Somente** landing page e site institucional **estático** (HTML). Recusar: WordPress, sistema, área logada, integração, formulário com backend. |
| Preço mínimo | **R$ 297** — nunca propor nem aceitar abaixo. |
| Prazo prometido | **Primeira versão em até 24 horas úteis** (dias úteis; entrega real assim que a sessão de produção terminar). |
| Limite de propostas | O limite do plano da conta — o robô lê o saldo restante e para ao zerar. |
| Negociação | Pode descer até R$ 297. |
| Ajustes | Ilimitados até o cliente aprovar, desde que dentro do escopo estático. |
| Início da produção | Só depois que o cliente pagar pela plataforma (valor em custódia). |
| Domínio das prévias | `<cliente>.sites.zxlab.com.br` (verificado livre em 24/Set/26 com `zx-dominio-livre`). |
| Publicação final | Domínio do cliente via **CNAME** apontando pro Cloudflare Pages da conta do Rafael (sem pedir senha de DNS/hospedagem). Alternativa: ZIP + instrução no chat. |
| Notificação ao Rafael | E-mail: resumo diário + aviso a cada projeto fechado + alerta de falha. |
| Fila de produção | Uma entrega por vez. |
| Varredura de reserva do chat | A cada 1 hora, abre a lista de conversas da plataforma. |
| Portfólio | Etapa 0: 6 LPs demo por nicho numa vitrine em `portfolio.sites.zxlab.com.br` + perfil do 99Freelas preenchido pelo robô na 1ª execução. |

## Regras da plataforma (lidas em `99freelas.com.br/termos`, 24/Set/26)

- Proibido solicitar/compartilhar dados de contato em proposta, pergunta ou chat.
- Proibido solicitar/aceitar pagamento fora da plataforma.
- Spam gera violação → penalização → **banimento, inclusive de outras contas**.
- Automação não é proibida explicitamente.
- Taxa de 5–20% (mín. R$ 10) é somada à oferta e paga pelo contratante.

## Arquitetura

Repositório `~/projetos/freela-bot` (GitHub privado). Rodada disparada por LaunchAgent a cada 5 min. Sessão logada do 99Freelas salva em `storage_state.json` (gitignored, `chmod 600`) — login e captcha são sempre do Rafael.

| Módulo | Responsabilidade | Depende de |
|---|---|---|
| `radar` | Lê a lista pública de projetos, filtra por escopo, registra motivo de descarte | `estado` |
| `precificador` | Define valor a partir do orçamento informado pelo cliente e nº de concorrentes; piso R$ 297; aprende com propostas ganhas/perdidas | `estado` |
| `redator` | Gera proposta, resposta e negociação via `zx-claude-headless` (assinatura); cita o pedido e o portfólio | `travas` |
| `inbox` | Lê Gmail (`gws`) a cada 5 min; e-mail do 99Freelas → abre o chat; reserva de 1 em 1 h pela lista de conversas | `navegador`, `estado` |
| `navegador` | Único módulo que toca o site (Playwright): enviar proposta, ler/enviar chat, anexar arquivo, marcar concluído, preencher perfil | `storage_state` |
| `entregador` | Monta o briefing, abre sessão headless de desenvolvimento, faz a checagem de celular e links quebrados, publica no CF Pages, liga o domínio via CNAME, gera o ZIP | `estado`, `navegador` |
| `travas` | Filtro de contato/pagamento externo; piso de preço; escopo estático; bloqueio de envio duplicado | — |
| `estado` | SQLite: projetos, propostas, conversas, mensagens, entregas, eventos | — |
| `notificador` | E-mail ao Rafael (resumo diário, fechamento, falha) | — |

## Ciclo de vida de um projeto

```
NOVO → FILTRADO → PROPOSTA_ENVIADA → EM_CONVERSA → FECHADO (pago em custódia)
  → EM_PRODUCAO → V1_ENTREGUE → AJUSTES* → APROVADO → PUBLICADO → FINALIZADO
DESCARTADO (com motivo) · PERDIDO (outro freelancer escolhido)
```

1. **Radar**: projetos novos a cada 5 min; descartes registrados com motivo.
2. **Proposta**: precificador → redator → travas → navegador envia; para ao zerar o saldo do plano.
3. **Conversa**: e-mail chega → chat aberto → resposta gerada → travas → envio. Negocia até R$ 297.
4. **Fechado**: cliente aceita e paga na plataforma → e-mail ao Rafael.
5. **Produção**: briefing = descrição do projeto + todo o chat + anexos. Falta de conteúdo → sessão redige o texto e usa imagens geradas ou de banco. Checagem de celular e links antes de sair.
6. **V1**: prévia em `<cliente>.sites.zxlab.com.br`, link no chat.
7. **Ajustes**: cada pedido = nova rodada da sessão; pedido fora do escopo estático é recusado com educação.
8. **Publicação**: instrução de CNAME → robô confere o DNS → liga o domínio customizado via API → confere HTTP 200. Ou ZIP anexado.
9. **Finalizado**: projeto marcado como concluído na plataforma pra liberar o pagamento.

## Tratamento de falhas

| Situação | Reação |
|---|---|
| Layout do site mudou (seletor não encontrado) | Para todos os envios + e-mail. Nunca clica "no chute". |
| Captcha / sessão expirada / logout | Para + e-mail pedindo novo login. |
| Rodadas sobrepostas | Trava de arquivo. |
| Envio duplicado | `estado` registra cada envio; idempotência por projeto/mensagem. |
| Texto com telefone, e-mail, "WhatsApp" ou pagamento externo | `travas` bloqueia antes do envio e o redator reescreve. |
| Sessão de produção falhou ou site reprovou na checagem | E-mail na hora, com o prazo prometido ao cliente. |
| Gmail não trouxe o aviso | Varredura de reserva de 1 em 1 h na lista de conversas. |
| Timeouts de processo longo | Constante no topo + override por env, default 600 s, tempo decorrido logado. |

## Testes e lançamento

- **Modo `--simular`**, confirmado no código: executa tudo menos o clique de envio e grava o que teria sido enviado.
- **Testes automatizados** com fixtures HTML reais do 99Freelas: filtro de escopo, piso de preço, filtro de contato, idempotência.
- **Lançamento**: 1 dia em `--simular` → Rafael lê os envios simulados → liga o envio real.
- Review do código em loop até 0 ALTO (`luna-review`).

## Pré-requisitos do Rafael

- Logar uma vez no 99Freelas na sessão do robô (e sempre que expirar).
- Ativar no 99Freelas os e-mails de aviso de nova mensagem e de proposta respondida.

## Pendências a verificar na conta logada (não confirmadas pelas páginas públicas)

- Limite de propostas do plano grátis e como o saldo aparece.
- Se o valor das propostas dos concorrentes é visível (afeta o precificador).
- Fluxo exato de aceite, pagamento em custódia e conclusão do projeto.

## Fora de escopo

WordPress, sistemas, área logada, integrações, backend de formulário, hospedagem gerenciada pro cliente depois da entrega.
