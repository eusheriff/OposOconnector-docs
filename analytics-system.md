# Sistema de Analytics - NuvexPOS

## 📊 Visão Geral

O sistema de analytics do NuvexPOS é uma solução completa para tracking de conversão, análise de comportamento do usuário e otimização de performance, construído sobre a infraestrutura Cloudflare Workers.

## 🏗️ Arquitetura

### Componentes Principais

1. **Frontend Analytics** (`src/utils/analytics.ts`)
   - Hook React para tracking de eventos
   - Sistema de batching para otimização de performance
   - Tracking automático de sessões e navegação

2. **Backend Analytics** (`src/worker/analytics.ts`)
   - API REST para recebimento de eventos
   - Processamento em tempo real
   - Agregação de métricas

3. **Banco de Dados** (`src/worker/schema.sql`)
   - Schema otimizado para analytics
   - Tabelas para eventos, sessões, leads e métricas
   - Views para relatórios comuns

4. **Configuração** (`src/config/analytics.ts`)
   - Configurações centralizadas por ambiente
   - Rate limiting e sampling
   - Controles de privacidade

## 🚀 Funcionalidades

### ✅ Implementadas

- **Tracking de Eventos**: Captura automática de interações do usuário
- **Análise de Sessões**: Tracking completo de sessões de usuário
- **Métricas de Conversão**: Funil de vendas e conversões
- **Analytics em Tempo Real**: Métricas atualizadas instantaneamente
- **Heatmap**: Mapeamento de cliques e interações
- **A/B Testing**: Framework para testes de variações
- **Performance Monitoring**: Métricas de performance da aplicação

### 🔄 Em Desenvolvimento

- **Dashboard de Analytics**: Interface visual para análise de dados
- **Relatórios Automatizados**: Geração automática de relatórios
- **Alertas Inteligentes**: Notificações baseadas em métricas

## 📁 Estrutura de Arquivos

```
src/
├── components/Demo/
│   └── DemoEnvironment.tsx     # Componente com tracking integrado
├── utils/
│   └── analytics.ts            # Hook e utilitários de analytics
├── worker/
│   ├── analytics.ts            # API de analytics
│   ├── schema.sql              # Schema do banco D1
│   └── index.ts                # Worker principal
├── config/
│   └── analytics.ts            # Configurações centralizadas
└── hooks/
    └── useAnalytics.ts         # Hook React para analytics

scripts/
├── init-analytics-db.js        # Script de inicialização do banco
└── deploy-analytics.js         # Script de deploy completo

docs/
└── analytics-system.md         # Esta documentação
```

## 🔧 Configuração

### Variáveis de Ambiente

```bash
# .env
ANALYTICS_ENABLED=true
ANALYTICS_DEBUG=false
ANALYTICS_SAMPLE_RATE=1.0
ANALYTICS_BATCH_SIZE=10
ANALYTICS_FLUSH_INTERVAL=5000
```

### Cloudflare Resources

#### D1 Database
```toml
[[d1_databases]]
binding = "ANALYTICS_D1"
database_name = "nuvexpos_analytics_dev"
database_id = "analytics-dev-database-id"
```

#### KV Namespace
```toml
[[kv_namespaces]]
binding = "ANALYTICS_KV"
id = "dev-analytics-kv-id"
preview_id = "dev-analytics-kv-preview-id"
```

## 🚀 Deploy

### 1. Inicialização do Banco

```bash
# Criar banco D1
npx wrangler d1 create nuvexpos_analytics_dev

# Inicializar schema
node scripts/init-analytics-db.js development
```

### 2. Deploy Completo

```bash
# Deploy automatizado
node scripts/deploy-analytics.js development

# Deploy manual
npm run build
npx wrangler deploy --env development
```

### 3. Verificação

```bash
# Testar localmente
npx wrangler dev --env development

# Verificar banco
npx wrangler d1 execute ANALYTICS_D1 --command "SELECT COUNT(*) FROM analytics_events;" --env development
```

## 📊 API Endpoints

### Eventos

```http
POST /analytics/api/analytics
Content-Type: application/json

{
  "event": "page_view",
  "properties": {
    "page": "/demo",
    "title": "Demo Page"
  },
  "userId": "user123",
  "sessionId": "session456"
}
```

### Batch de Eventos

```http
POST /analytics/api/analytics/batch
Content-Type: application/json

{
  "events": [
    {
      "event": "click",
      "properties": { "element": "button" }
    }
  ]
}
```

### Métricas em Tempo Real

```http
GET /analytics/api/realtime?timeframe=1h
```

### Funil de Conversão

```http
GET /analytics/api/funnel?steps=page_view,demo_start,lead_capture
```

## 🎯 Uso no Frontend

### Hook useAnalytics

```tsx
import { useAnalytics } from '@/hooks/useAnalytics';

function MyComponent() {
  const analytics = useAnalytics();

  const handleClick = () => {
    analytics.trackEvent('button_click', {
      button: 'cta',
      page: 'demo'
    });
  };

  return <button onClick={handleClick}>Click me</button>;
}
```

### Tracking Automático

```tsx
// Tracking automático de demo
const analytics = useAnalytics();

useEffect(() => {
  analytics.trackDemoStart({
    demoType: 'interactive',
    userType: 'prospect'
  });
}, []);
```

## 📈 Métricas Principais

### Conversão
- **Taxa de Conversão**: Visitantes → Leads
- **Funil de Vendas**: Etapas do processo de venda
- **Abandono**: Pontos de saída do usuário

### Engagement
- **Tempo na Página**: Duração média das sessões
- **Interações**: Cliques, scrolls, hovers
- **Jornada do Usuário**: Fluxo de navegação

### Performance
- **Tempo de Carregamento**: Performance da aplicação
- **Erros**: Tracking de erros JavaScript
- **Disponibilidade**: Uptime da aplicação

## 🔒 Privacidade e Segurança

### Conformidade LGPD/GDPR
- **Anonimização de IPs**: IPs são hasheados
- **Consentimento**: Respeita preferências do usuário
- **Retenção de Dados**: Configurável por ambiente
- **Do Not Track**: Respeita header DNT

### Segurança
- **Rate Limiting**: Proteção contra spam
- **Validação**: Sanitização de dados de entrada
- **Criptografia**: Dados sensíveis criptografados
- **Auditoria**: Logs de acesso e modificações

## 🔧 Manutenção

### Monitoramento

```bash
# Verificar saúde do sistema
curl https://your-worker.workers.dev/analytics/api/health

# Métricas de performance
npx wrangler tail --env production
```

### Backup

```bash
# Backup do banco D1
npx wrangler d1 export ANALYTICS_D1 --output backup.sql --env production
```

### Limpeza de Dados

```sql
-- Remover dados antigos (configurável)
DELETE FROM analytics_events 
WHERE created_at < datetime('now', '-365 days');
```

## 🚀 Próximos Passos

### Roadmap

1. **Dashboard Visual** (Q1 2024)
   - Interface para visualização de métricas
   - Relatórios interativos
   - Alertas personalizados

2. **Machine Learning** (Q2 2024)
   - Predição de conversão
   - Segmentação automática
   - Recomendações personalizadas

3. **Integrações** (Q3 2024)
   - CRM integration
   - Email marketing
   - Social media tracking

### Melhorias Técnicas

- **Otimização de Performance**: Caching avançado
- **Escalabilidade**: Sharding de dados
- **Observabilidade**: Métricas detalhadas

## 📞 Suporte

Para dúvidas ou problemas:

1. Verificar logs: `npx wrangler tail`
2. Consultar documentação da API
3. Revisar configurações de ambiente
4. Contatar equipe de desenvolvimento

---

**Última atualização**: Janeiro 2024  
**Versão**: 1.0.0  
**Autor**: Equipe NuvexPOS