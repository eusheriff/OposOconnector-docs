# 🚀 Guia de Deploy - NuvexPOS

## 📋 Visão Geral

O NuvexPOS utiliza uma estratégia de deploy serverless com Cloudflare Workers, garantindo alta disponibilidade, escalabilidade automática e distribuição global. Este guia cobre todos os aspectos do processo de deploy.

## 🏗️ Arquitetura de Deploy

### Ambientes
- **Development**: Ambiente local de desenvolvimento
- **Staging**: Ambiente de testes e homologação
- **Production**: Ambiente de produção

### Componentes Deployados
- **Frontend**: Cloudflare Pages
- **Workers**: Cloudflare Workers (API, Auth, Business Logic)
- **Database**: Cloudflare D1
- **Storage**: Cloudflare KV + R2
- **CDN**: Cloudflare CDN

## 🔧 Pré-requisitos

### Conta Cloudflare
1. Conta ativa no Cloudflare
2. Domínio configurado (opcional para staging)
3. API Token com permissões:
   - `Zone:Zone:Read`
   - `Zone:Zone Settings:Edit`
   - `Cloudflare Workers:Service Worker:Edit`
   - `Account:Cloudflare Pages:Edit`
   - `Account:D1:Edit`

### Ferramentas Locais
```bash
# Node.js 18+
node --version

# Wrangler CLI
npm install -g wrangler

# Verificar instalação
wrangler --version
```

## 🚀 Deploy Inicial

### 1. Configuração do Ambiente
```bash
# Clone o repositório
git clone https://github.com/seu-usuario/nuvexpos.git
cd nuvexpos

# Instale dependências
npm install

# Configure variáveis de ambiente
cp .env.example .env.production
```

### 2. Autenticação Cloudflare
```bash
# Login no Cloudflare
wrangler auth login

# Verificar autenticação
wrangler whoami
```

### 3. Configuração Inicial
```bash
# Criar recursos necessários
npm run setup:cloudflare

# Verificar configuração
npm run cf:whoami
```

## 🔄 Processo de Deploy

### Deploy Staging
```bash
# Build para staging
npm run build:staging

# Deploy workers para staging
npm run deploy:staging

# Verificar deploy
curl https://staging-api.nuvexpos.com/health
```

### Deploy Production
```bash
# Build para produção
npm run build:production

# Deploy workers para produção
npm run deploy:production

# Verificar deploy
curl https://api.nuvexpos.com/health
```

## 📦 Configuração de Workers

### wrangler.toml
```toml
name = "nuvexpos-api"
main = "dist/worker.js"
compatibility_date = "2024-01-15"
compatibility_flags = ["nodejs_compat"]

[env.staging]
name = "nuvexpos-api-staging"
vars = { ENVIRONMENT = "staging" }

[env.production]
name = "nuvexpos-api-production"
vars = { ENVIRONMENT = "production" }

# KV Namespaces
[[kv_namespaces]]
binding = "CACHE"
id = "your-kv-namespace-id"
preview_id = "your-preview-kv-namespace-id"

# D1 Database
[[d1_databases]]
binding = "DB"
database_name = "nuvexpos-db"
database_id = "your-database-id"

# R2 Buckets
[[r2_buckets]]
binding = "STORAGE"
bucket_name = "nuvexpos-storage"

# Durable Objects
[[durable_objects.bindings]]
name = "REALTIME"
class_name = "RealtimeHandler"

# Environment Variables
[vars]
NODE_ENV = "production"
API_VERSION = "v1"
```

### Secrets Management
```bash
# Configurar secrets
wrangler secret put JWT_SECRET
wrangler secret put STRIPE_SECRET_KEY
wrangler secret put OPENAI_API_KEY

# Listar secrets
wrangler secret list

# Deletar secret
wrangler secret delete SECRET_NAME
```

## 🗄️ Database Setup

### D1 Database
```bash
# Criar database
wrangler d1 create nuvexpos-db

# Executar migrations
wrangler d1 migrations apply nuvexpos-db

# Verificar schema
wrangler d1 execute nuvexpos-db --command "SELECT name FROM sqlite_master WHERE type='table';"
```

### Migrations
```sql
-- migrations/001_initial_schema.sql
CREATE TABLE companies (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  settings TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE stores (
  id TEXT PRIMARY KEY,
  company_id TEXT NOT NULL,
  name TEXT NOT NULL,
  address TEXT,
  settings TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (company_id) REFERENCES companies(id)
);

-- Adicionar mais tabelas...
```

## 🔧 Configuração de KV

### Namespaces
```bash
# Criar KV namespace
wrangler kv:namespace create "CACHE"
wrangler kv:namespace create "CACHE" --preview

# Listar namespaces
wrangler kv:namespace list

# Configurar dados iniciais
wrangler kv:key put --binding=CACHE "config:app" '{"version":"1.0.0"}'
```

## 📁 Configuração de R2

### Buckets
```bash
# Criar bucket
wrangler r2 bucket create nuvexpos-storage

# Listar buckets
wrangler r2 bucket list

# Upload de arquivo
wrangler r2 object put nuvexpos-storage/test.txt --file=./test.txt
```

## 🌐 Configuração de Pages

### Deploy Frontend
```bash
# Build do frontend
npm run build

# Deploy para Pages
wrangler pages deploy dist --project-name=nuvexpos-frontend

# Configurar domínio customizado
wrangler pages domain add nuvexpos-frontend app.nuvexpos.com
```

### pages.toml
```toml
[build]
command = "npm run build"
destination = "dist"

[build.environment]
NODE_VERSION = "18"

[[redirects]]
from = "/api/*"
to = "https://api.nuvexpos.com/:splat"
status = 200

[[headers]]
for = "/*"
[headers.values]
X-Frame-Options = "DENY"
X-Content-Type-Options = "nosniff"
```

## 🔄 CI/CD Pipeline

### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloudflare

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test
      - run: npm run lint
      - run: npm run type-check

  deploy-staging:
    needs: test
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          environment: staging

  deploy-production:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          environment: production
```

## 🔍 Monitoramento de Deploy

### Health Checks
```typescript
// workers/health.ts
export default {
  async fetch(request: Request): Promise<Response> {
    const health = {
      status: 'healthy',
      timestamp: new Date().toISOString(),
      version: process.env.APP_VERSION,
      environment: process.env.ENVIRONMENT,
      services: {
        database: await checkDatabase(),
        kv: await checkKV(),
        r2: await checkR2()
      }
    };

    return new Response(JSON.stringify(health), {
      headers: { 'Content-Type': 'application/json' }
    });
  }
};
```

### Logs e Métricas
```bash
# Visualizar logs em tempo real
wrangler tail nuvexpos-api

# Logs com filtros
wrangler tail nuvexpos-api --format=pretty --status=error

# Métricas de performance
wrangler analytics --since=1h
```

## 🚨 Rollback e Recovery

### Rollback Rápido
```bash
# Listar deployments
wrangler deployments list

# Rollback para versão anterior
wrangler rollback --deployment-id=<deployment-id>

# Verificar rollback
curl https://api.nuvexpos.com/health
```

### Backup de Dados
```bash
# Backup do D1
wrangler d1 export nuvexpos-db --output=backup-$(date +%Y%m%d).sql

# Backup do KV
wrangler kv:bulk download --binding=CACHE --output=kv-backup.json

# Backup do R2
wrangler r2 object list nuvexpos-storage > r2-inventory.txt
```

## 🔧 Troubleshooting

### Problemas Comuns

#### 1. Erro de Autenticação
```bash
# Reautenticar
wrangler auth login

# Verificar token
wrangler whoami
```

#### 2. Erro de Build
```bash
# Limpar cache
npm run clean
rm -rf node_modules
npm install

# Build local
npm run build
```

#### 3. Erro de Deploy
```bash
# Verificar configuração
wrangler whoami
cat wrangler.toml

# Deploy com logs detalhados
wrangler deploy --verbose
```

#### 4. Erro de Database
```bash
# Verificar conexão
wrangler d1 execute nuvexpos-db --command "SELECT 1"

# Aplicar migrations
wrangler d1 migrations apply nuvexpos-db
```

### Logs de Debug
```bash
# Logs detalhados
wrangler tail --format=pretty

# Filtrar por status
wrangler tail --status=error

# Filtrar por método
wrangler tail --method=POST
```

## 📊 Performance e Otimização

### Bundle Optimization
```bash
# Analisar bundle
npm run build:analyze

# Verificar tamanho
wrangler deploy --dry-run
```

### Cache Strategy
```typescript
// Configuração de cache
const cacheConfig = {
  browser: {
    maxAge: 3600, // 1 hora
  },
  edge: {
    maxAge: 86400, // 24 horas
    staleWhileRevalidate: 3600,
  }
};
```

### Database Optimization
```sql
-- Índices para performance
CREATE INDEX idx_sales_store_date ON sales(store_id, created_at);
CREATE INDEX idx_products_store_active ON products(store_id, active);
CREATE INDEX idx_inventory_product ON inventory(product_id);
```

## 🔐 Segurança

### Environment Variables
```bash
# Produção - nunca commitar
CLOUDFLARE_API_TOKEN=your_production_token
JWT_SECRET=your_production_jwt_secret
STRIPE_SECRET_KEY=your_production_stripe_key

# Staging
CLOUDFLARE_API_TOKEN=your_staging_token
JWT_SECRET=your_staging_jwt_secret
STRIPE_SECRET_KEY=your_staging_stripe_key
```

### Security Headers
```typescript
// Configurar headers de segurança
const securityHeaders = {
  'X-Frame-Options': 'DENY',
  'X-Content-Type-Options': 'nosniff',
  'X-XSS-Protection': '1; mode=block',
  'Strict-Transport-Security': 'max-age=31536000',
  'Content-Security-Policy': "default-src 'self'"
};
```

## 📚 Recursos Adicionais

### Documentação Oficial
- [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/)
- [Cloudflare Pages](https://developers.cloudflare.com/pages/)
- [D1 Database](https://developers.cloudflare.com/d1/)

### Ferramentas Úteis
- [Cloudflare Dashboard](https://dash.cloudflare.com)
- [Workers Analytics](https://dash.cloudflare.com/analytics)
- [Pages Analytics](https://dash.cloudflare.com/pages)

### Suporte
- [Cloudflare Community](https://community.cloudflare.com)
- [Discord NuvexPOS](https://discord.gg/nuvexpos)
- [GitHub Issues](https://github.com/seu-usuario/nuvexpos/issues)