# Gerador Gauntlet Loop

Ferramenta web que transforma três campos (Tarefa, Método, Padrão de Parada) em um prompt no formato **gauntlet loop** — pronto pra colar no Claude Code, Codex ou qualquer outro agente de codificação. Site estático, sem build, sem dependências — só `index.html` + `gaundltet.css` + `agente.js`.

O prompt gerado segue a metodologia original do [gauntlet loop de Matt Shumer](https://github.com/mshumer/Claude-of-Duty), endurecida pela skill [`gauntlet-loop-seguro`](https://github.com/criatedavid-tech/skills/tree/main/skills/gauntlet-loop-seguro): defesa contra prompt injection vinda da referência, teto de rodadas obrigatório, limite de subagentes em paralelo e proibição de efeitos colaterais silenciosos (publicar, enviar dado, agir por conta própria).

## Como usar

Abra `index.html` (local ou publicado), preencha:

1. **A Tarefa (o quê)** — o que construir e qual é a referência de elite a ser igualada.
2. **O Método (como)** — em quais frentes os subagentes devem trabalhar em paralelo.
3. **O Padrão de Parada (quando)** — contra qual referência real o crítico deve comparar às cegas antes de aprovar.

Escolhe o harness de destino (Claude Code ou Codex/outro), ajusta os tetos de segurança se quiser, e clica em **Gerar Prompt Gauntlet**. O resultado sai pronto pra copiar.

Os botões de preset (FPS ThreeJS, Dashboard SaaS, API Rust) preenchem os três campos com um exemplo completo, só pra testar rápido.

## Dois modos de geração

O site tenta, nessa ordem:

1. **Via n8n** — se o campo avançado "Endpoint do Webhook n8n" apontar pra uma URL real (não o placeholder padrão), o site envia os campos pra lá via `POST`. O workflow do n8n roda um agente de IA (OpenRouter) que compila o prompt com mais nuance na redação.
2. **Compilador local (fallback automático)** — se o webhook estiver offline, não configurado, ou responder erro, o próprio JavaScript do site monta o prompt com um template fixo, sem depender de nenhum backend. **É o modo padrão de fábrica** — o site funciona sozinho, sem precisar de n8n nenhum.

### Configurando o n8n (opcional)

Se você tem o workflow "Gauntlet Loop Generator" rodando no n8n:

1. Abra "Avançado — Tetos de Segurança & Webhook" no site.
2. Cole a URL do webhook em **Endpoint do Webhook n8n**.
3. Cole a chave em **Chave secreta (header X-App-Secret)** — precisa ser a mesma configurada no node "Check Secret" do workflow. Esse campo **fica só no seu navegador**, nunca é salvo no código-fonte nem commitado.
4. Sem a chave certa, o node "Check Secret" do n8n responde 401 e o site cai automaticamente pro compilador local — nada quebra.

## Arquitetura do workflow n8n (referência)

```
Webhook (POST) → Check Secret (If) → AI Agent (OpenRouter + memória Postgres) → Respond to Webhook
                        ↳ falha → Respond Unauthorized (401)
```

- **Check Secret**: rejeita qualquer chamada sem o header `X-App-Secret` correto, antes de gastar créditos do modelo pago.
- **AI Agent**: recebe tarefa/método/parada/harness/tetos, aplica o mesmo template estrutural da skill `gauntlet-loop-seguro`, devolve só o texto final do prompt.
- **Postgres Chat Memory**: usa `{{ $execution.id }}` como chave de sessão — cada chamada é isolada, nenhuma conversa vaza entre usuários diferentes.

Isso é proteção "levanta a régua", não autenticação real — como o site é estático e público, qualquer segredo que ele envie fica visível pra quem inspecionar a aba de rede do navegador. Proteção de verdade exigiria um backend intermediário segurando a chave.

## Estrutura de arquivos

```
gauntlet-loop-gerador/
├── index.html      # marcação + formulário
├── gaundltet.css    # tema OLED dark, tudo em CSS puro
├── agente.js        # lógica: presets, geração local, integração n8n, copiar
└── README.md
```

## Deploy

Site 100% estático — funciona em qualquer host que sirva arquivos, sem passo de build. Pra Vercel:

```bash
vercel --prod
```

## Créditos

Metodologia gauntlet loop original por [Matt Shumer](https://github.com/mshumer). Versão endurecida contra prompt injection e loops sem teto: skill [`gauntlet-loop-seguro`](https://github.com/criatedavid-tech/skills).
