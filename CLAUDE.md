# freela-bot — instruções para o Claude Code

Você vai construir, junto com o aluno, o robô descrito em `docs/superpowers/specs/2026-09-24-freela-bot-design.md`. Leia esse arquivo inteiro antes de qualquer coisa.

## 1. Primeiro, as regras DO ALUNO

As decisões da spec são as do Rafael (ex.: preço mínimo R$ 297, prévias em `sites.zxlab.com.br`). São exemplo, não configuração. Antes de escrever código, pergunte ao aluno, uma coisa por vez:

1. Preço mínimo por projeto.
2. Prazo prometido para a primeira versão.
3. Domínio/subdomínio das prévias (ou só `*.pages.dev`).
4. E-mail que recebe alertas e o resumo diário.
5. Escopo: manter só LP e site institucional estático (recomendado).

Grave as respostas em `config.env` (gitignored). Nada disso vai chumbado no código.

## 2. Ordem de construção

1. Estrutura do projeto + `estado` (SQLite) + `travas` (piso, escopo, filtro de contato/pagamento externo, anti-duplicado) com testes.
2. `radar` + `precificador` + `redator` em **modo `--simular`**.
3. `navegador` (Playwright, sessão salva em `storage_state.json` — o aluno loga manualmente; nunca digite senha nem resolva captcha).
4. `inbox` (e-mail de aviso do 99Freelas → abre o chat).
5. `entregador` (gera o site com `claude -p`, publica no Cloudflare Pages, CNAME ou ZIP).
6. `notificador` (e-mail ao aluno).
7. Agendamento a cada 5 min no sistema do aluno: Agendador de Tarefas (Windows), cron (Linux) ou launchd (macOS).

## 3. Regras que não se negociam

- **Modo `--simular` existe no código e é respeitado no caminho de envio** (`if simular: registrar e return` antes de qualquer clique de enviar). É o modo padrão até o aluno pedir para ligar o envio real.
- **Nunca** enviar telefone, e-mail, WhatsApp ou pedir pagamento fora da plataforma — é proibido pelos termos e dá banimento.
- **Nunca** propor abaixo do preço mínimo do aluno.
- Seletor não encontrado, captcha ou sessão expirada → **parar tudo e avisar por e-mail**, nunca clicar "no chute".
- Segredos (`storage_state.json`, `config.env`, tokens) ficam fora do git.
- Timeouts de processos longos (LLM, deploy) vêm de constante com override por variável de ambiente, default 600 s, e o tempo decorrido é logado.
