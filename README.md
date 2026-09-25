# Robô de Freelas — 99Freelas no automático com Claude Code

Robô que opera a sua conta no 99Freelas: acha projetos de **landing page e site institucional**, envia proposta com preço competitivo, conversa e negocia no chat da plataforma e, quando o cliente fecha, **produz, publica e entrega o site**.

Este repositório é a **planta completa do projeto**. Você clona, abre o Claude Code e ele constrói o robô com você, seguindo o desenho que está em [`docs/superpowers/specs/`](docs/superpowers/specs/2026-09-24-freela-bot-design.md).

## Como baixar

```bash
gh repo clone zxmarketingdigital/freela-bot
cd freela-bot
claude
```

Sem o `gh`? Baixe o ZIP: https://github.com/zxmarketingdigital/freela-bot/archive/refs/heads/main.zip

Funciona no computador que você já usa (Windows, Linux ou macOS) — tudo roda via Claude Code.

## O que acontece quando você abre o Claude Code aqui

1. Ele pergunta **as suas regras**: preço mínimo, prazo da primeira versão, domínio das prévias, e-mail para alertas.
2. Constrói o robô em fases, começando pelo **modo simulado** (faz tudo menos clicar em "enviar").
3. Você lê o que ele *teria* enviado. Só depois liga o envio real.

## Como o robô funciona

| Parte | O que faz |
|---|---|
| Radar | Olha os projetos novos a cada 5 min e filtra só site estático e LP |
| Proposta | Calcula o preço (nunca abaixo do seu mínimo) e escreve uma proposta que cita o pedido do cliente |
| Conversa | Lê o aviso de mensagem no seu e-mail, abre o chat e responde/negocia |
| Entrega | Cliente fechou e pagou pela plataforma → gera o site, publica uma prévia, faz os ajustes e entrega no domínio do cliente (CNAME) ou em ZIP |

## Regras da plataforma que o robô respeita

- **Nada de contato ou pagamento fora da plataforma** — proibido pelos termos do 99Freelas.
- **Spam dá punição** (rebaixamento → bloqueio → banimento, inclusive de outras contas). O robô respeita o limite de propostas do seu plano.
- **Login e captcha são seus.** Se aparecer captcha ou a sessão cair, o robô para e te avisa.

## Arquivos importantes

- [`docs/superpowers/specs/2026-09-24-freela-bot-design.md`](docs/superpowers/specs/2026-09-24-freela-bot-design.md) — o desenho completo (as decisões ali são as do Rafael, use como exemplo)
- [`CLAUDE.md`](CLAUDE.md) — as instruções que o Claude Code segue para construir o robô

Feito na ZX LAB.
