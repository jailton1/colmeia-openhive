# 🐝 COLMEIA — Web App Autônoma de Conversão

**Padrão Hub-and-Spoke | Orquestrador Central + Interface Serena**

---

## Arquitetura

```
TikTok (tráfego) ──→ Site Replit
                           │
              ┌────────────▼────────────┐
              │   ABELHA RAINHA         │
              │   server.js             │
              │   (Orquestrador)        │
              │                         │
              │  ┌─────────────────┐   │
              │  │  Sanitizer      │   │  ← Anti-injection
              │  │  Rate Limiter   │   │  ← Abuse protection
              │  │  Orchestrator   │   │  ← knowledge-base.json
              │  └────────┬────────┘   │
              │           │             │
              │    ┌──────▼──────┐     │
              │    │ Claude API  │     │  ← Serena responde
              │    └──────┬──────┘     │
              │           │             │
              │    ┌──────▼──────┐     │
              │    │ n8n Webhook │     │  ← Lead tracking
              │    └─────────────┘     │
              └─────────────────────────┘
                           │
              ┌────────────▼────────────┐
              │   SERENA (Frontend)     │
              │   public/index.html     │
              │   Dark Gold Editorial   │
              └─────────────────────────┘
```

---

## Estrutura de Arquivos

```
colmeia/
├── server.js                 # Entry point — Abelha Rainha
├── package.json
├── .env.example              # Modelo de variáveis (não commitar .env real)
│
├── config/
│   └── orchestrator.js       # Monta system prompt + payload n8n
│
├── data/
│   └── knowledge-base.json   # ← PREENCHA ANTES DE SUBIR
│
├── middleware/
│   └── sanitizer.js          # Anti-injection + rate limit por sessão
│
├── routes/
│   ├── chat.js               # POST /api/chat — core do chat
│   └── config-public.js      # GET /api/config — dados públicos
│
└── public/                   # Serena (Frontend estático)
    ├── index.html
    ├── css/serena.css
    └── js/serena.js
```

---

## Deploy no Replit — Checklist Obrigatório

### 1. Preencher knowledge-base.json
```json
"price": {
  "current": 297,          // ← preço real
  "installments": {
    "max": 12,
    "value": 29.70         // ← current / max
  }
},
"checkout_url": "https://pay.hotmart.com/SEU_PRODUTO"
```

### 2. Configurar Replit Secrets
Acesse: **Tools → Secrets** no painel do Replit

| Secret | Valor |
|--------|-------|
| `ANTHROPIC_API_KEY` | `sk-ant-...` |
| `N8N_WEBHOOK_URL` | URL do seu webhook |
| `ALLOWED_ORIGINS` | URL do seu Repl |

### 3. Configurar Run Command
No `.replit` ou painel de configuração:
```
run = "npm start"
```

### 4. Instalar dependências
O Replit instala automaticamente via `package.json`.
Se não instalar, rode no Shell: `npm install`

---

## Integração n8n

O webhook recebe este payload a cada interação:
```json
{
  "event": "chat_interaction",
  "timestamp": "2025-01-01T00:00:00.000Z",
  "session_id": "sess_...",
  "product": "Nome do Produto",
  "user_message": "mensagem do usuário",
  "serena_response": "resposta da Serena",
  "checkout_url": "https://...",
  "metadata": {
    "closing_intent": false,
    "response_time_ms": 843,
    "history_length": 4
  }
}
```

**Sugestão de automação n8n:**
- `closing_intent: true` → notificação no Telegram/WhatsApp
- Salvar todos os `user_message` para análise de objeções

---

## Segurança Implementada

| Camada | Mecanismo |
|--------|-----------|
| Headers HTTP | Helmet.js (CSP, HSTS, X-Frame) |
| CORS | Whitelist de origens |
| Rate Limit (IP) | 200 req/15min global |
| Rate Limit (sessão) | 15 msg/min por sessão |
| Payload | Limite 10KB, input 600 chars |
| Prompt Injection | 15 patterns regex + log |
| Dados sensíveis | Apenas via Replit Secrets |
| Alucinação de preços | Dados servidos só do KB JSON |

---

## Próximos Passos Sugeridos

1. **Analytics**: Adicionar evento de PageView e CTA click via GTM ou Plausible
2. **n8n Flow**: Criar fluxo que captura `closing_intent: true` e notifica em tempo real
3. **A/B de headline**: Testar 2 versões do `hero-title` por UTM param
