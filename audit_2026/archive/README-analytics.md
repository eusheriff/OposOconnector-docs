# 📊 Sistema de Analytics - Guia Rápido

## 🚀 Início Rápido

### 1. Configuração Inicial

```bash
# Instalar dependências
npm install

# Inicializar banco de analytics
npm run analytics:init

# Deploy do sistema
npm run analytics:deploy
```

### 2. Desenvolvimento Local

```bash
# Iniciar servidor de desenvolvimento
npm run dev

# Em outro terminal, iniciar worker
npm run dev:worker
```

### 3. Testar Analytics

```bash
# Verificar eventos no banco
npm run analytics:test

# Monitorar logs em tempo real
npx wrangler tail
```

## 📋 Scripts Disponíveis

| Script | Descrição |
|--------|-----------|
| `npm run analytics:init` | Inicializa banco D1 (dev) |
| `npm run analytics:init:staging` | Inicializa banco D1 (staging) |
| `npm run analytics:init:production` | Inicializa banco D1 (produção) |
| `npm run analytics:deploy` | Deploy completo (dev) |
| `npm run analytics:deploy:staging` | Deploy completo (staging) |
| `npm run analytics:deploy:production` | Deploy completo (produção) |
| `npm run analytics:test` | Testa conexão com banco |

## 🔧 Configuração

### Variáveis de Ambiente (.env)

```bash
# Analytics
ANALYTICS_ENABLED=true
ANALYTICS_DEBUG=true
ANALYTICS_SAMPLE_RATE=1.0

# Cloudflare
CLOUDFLARE_ACCOUNT_ID=your-account-id
CLOUDFLARE_API_TOKEN=your-api-token
```

### Recursos Cloudflare Necessários

1. **D1 Database**: `nuvexpos_analytics_dev`
2. **KV Namespace**: `ANALYTICS_KV`
3. **Worker**: Configurado no `wrangler.toml`

## 📊 Uso no Frontend

### Hook useAnalytics

```tsx
import { useAnalytics } from '@/hooks/useAnalytics';

function MyComponent() {
  const analytics = useAnalytics();

  // Tracking de evento simples
  const handleClick = () => {
    analytics.trackEvent('button_click', {
      button: 'cta',
      page: 'demo'
    });
  };

  // Tracking de conversão
  const handleLead = () => {
    analytics.trackConversion('lead_capture', {
      source: 'demo',
      value: 100
    });
  };

  return (
    <div>
      <button onClick={handleClick}>Click me</button>
      <button onClick={handleLead}>Convert</button>
    </div>
  );
}
```

### Tracking Automático

O sistema já inclui tracking automático para:

- ✅ **Page Views**: Navegação entre páginas
- ✅ **Demo Interactions**: Interações na demonstração
- ✅ **Form Submissions**: Envio de formulários
- ✅ **Button Clicks**: Cliques em botões importantes
- ✅ **Session Tracking**: Sessões de usuário

## 📈 Endpoints da API

### Eventos

```http
POST /analytics/api/analytics
{
  "event": "page_view",
  "properties": { "page": "/demo" }
}
```

### Métricas

```http
GET /analytics/api/metrics?timeframe=24h
GET /analytics/api/funnel?steps=page_view,demo_start,lead_capture
GET /analytics/api/realtime
```

## 🔍 Debugging

### Verificar Logs

```bash
# Logs do worker
npx wrangler tail

# Logs específicos de analytics
npx wrangler tail --format=pretty | grep analytics
```

### Verificar Banco

```bash
# Contar eventos
npx wrangler d1 execute ANALYTICS_D1 --command "SELECT COUNT(*) FROM analytics_events;"

# Ver últimos eventos
npx wrangler d1 execute ANALYTICS_D1 --command "SELECT * FROM analytics_events ORDER BY created_at DESC LIMIT 10;"
```

### Verificar KV

```bash
# Listar chaves
npx wrangler kv:key list --binding=ANALYTICS_KV

# Ver valor específico
npx wrangler kv:key get "session:123" --binding=ANALYTICS_KV
```

## 🚨 Troubleshooting

### Problema: Eventos não aparecem no banco

1. Verificar se o worker está rodando: `npx wrangler tail`
2. Verificar configuração do D1: `wrangler.toml`
3. Verificar logs de erro: `npx wrangler tail --format=pretty`

### Problema: Rate limiting

1. Verificar configuração de rate limit no worker
2. Ajustar `ANALYTICS_SAMPLE_RATE` no `.env`
3. Verificar logs para mensagens de rate limit

### Problema: Performance lenta

1. Verificar `ANALYTICS_BATCH_SIZE` no `.env`
2. Ajustar `ANALYTICS_FLUSH_INTERVAL`
3. Verificar métricas do worker no dashboard Cloudflare

## 📚 Documentação Completa

Para documentação detalhada, consulte:

- [Sistema de Analytics](./docs/analytics-system.md)
- [API Reference](./docs/api-reference.md)
- [Configuração Avançada](./docs/advanced-config.md)

## 🆘 Suporte

Em caso de problemas:

1. Verificar [troubleshooting](#-troubleshooting)
2. Consultar logs do sistema
3. Revisar configurações
4. Contatar equipe de desenvolvimento

---

**Última atualização**: Janeiro 2024  
**Versão**: 1.0.0