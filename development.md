# 🛠️ Guia de Desenvolvimento - NuvexPOS

## 📋 Pré-requisitos

### Ambiente de Desenvolvimento
- **Node.js**: 18.0.0 ou superior
- **npm**: 8.0.0 ou superior (ou yarn 1.22.0+)
- **Git**: 2.30.0 ou superior
- **VS Code**: Recomendado (com extensões)

### Conta Cloudflare
- Conta ativa no Cloudflare
- API Token com permissões adequadas
- Account ID e Zone ID configurados

### Extensões VS Code Recomendadas
```json
{
  "recommendations": [
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ms-vscode.vscode-typescript-next",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense",
    "ms-vscode.vscode-json"
  ]
}
```

## 🚀 Setup Inicial

### 1. Clone e Instalação
```bash
# Clone o repositório
git clone https://github.com/seu-usuario/nuvexpos.git
cd nuvexpos

# Instale as dependências
npm install

# Configure o ambiente
cp .env.example .env
```

### 2. Configuração do Ambiente
Edite o arquivo `.env` com suas credenciais:

```env
# Cloudflare Configuration
CLOUDFLARE_API_TOKEN=your_api_token_here
CLOUDFLARE_ACCOUNT_ID=your_account_id
CLOUDFLARE_ZONE_ID=your_zone_id

# Development Settings
NODE_ENV=development
VITE_API_URL=http://localhost:5173
VITE_APP_NAME=NuvexPOS

# Database Configuration
DATABASE_URL=your_d1_database_url
KV_NAMESPACE_ID=your_kv_namespace_id

# Authentication
JWT_SECRET=your_jwt_secret_here
AUTH_DOMAIN=your_auth_domain

# External Services
STRIPE_SECRET_KEY=your_stripe_secret
OPENAI_API_KEY=your_openai_key
```

### 3. Configuração do Cloudflare
```bash
# Login no Cloudflare
npm run cf:login

# Verificar configuração
npm run cf:whoami

# Setup inicial dos recursos
npm run setup:cloudflare
```

## 🏗️ Estrutura do Projeto

```
nuvexpos/
├── .github/                 # GitHub Actions e templates
│   ├── workflows/          # CI/CD pipelines
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/                   # Documentação
├── public/                 # Assets estáticos
├── scripts/                # Scripts de automação
├── src/                    # Código fonte
│   ├── components/         # Componentes React
│   │   ├── ui/            # Componentes base (shadcn/ui)
│   │   ├── forms/         # Componentes de formulário
│   │   ├── layout/        # Componentes de layout
│   │   └── business/      # Componentes de negócio
│   ├── config/            # Configurações
│   ├── hooks/             # Custom hooks
│   ├── lib/               # Utilitários e helpers
│   ├── pages/             # Páginas da aplicação
│   ├── services/          # Serviços e APIs
│   │   └── cloudflare/    # Serviços Cloudflare
│   ├── test/              # Configuração de testes
│   └── types/             # Definições TypeScript
├── .env.example           # Template de variáveis
├── package.json           # Dependências e scripts
├── tailwind.config.js     # Configuração Tailwind
├── tsconfig.json          # Configuração TypeScript
└── vite.config.ts         # Configuração Vite
```

## 🔧 Comandos de Desenvolvimento

### Desenvolvimento Local
```bash
# Servidor de desenvolvimento
npm run dev

# Build de produção
npm run build

# Preview do build
npm run preview

# Verificar tipos
npm run type-check
```

### Qualidade de Código
```bash
# Linting
npm run lint
npm run lint:fix

# Formatação
npm run format

# Testes
npm test
npm run test:watch
npm run test:coverage
```

### Deploy e Cloudflare
```bash
# Deploy para staging
npm run deploy:staging

# Deploy para produção
npm run deploy:production

# Gerenciar workers
npm run wrangler dev
npm run wrangler deploy
```

## 🧪 Testes

### Estrutura de Testes
```
src/
├── components/
│   └── __tests__/         # Testes de componentes
├── services/
│   └── __tests__/         # Testes de serviços
├── hooks/
│   └── __tests__/         # Testes de hooks
└── test/
    ├── setup.ts           # Configuração global
    ├── mocks/             # Mocks e fixtures
    └── utils/             # Utilitários de teste
```

### Tipos de Teste

#### Testes Unitários
```typescript
// Exemplo: components/__tests__/Button.test.tsx
import { render, screen } from '@testing-library/react';
import { Button } from '../Button';

describe('Button', () => {
  it('renders correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toBeInTheDocument();
  });
});
```

#### Testes de Integração
```typescript
// Exemplo: services/__tests__/cloudflareService.test.ts
import { CloudflareService } from '../cloudflareService';

describe('CloudflareService', () => {
  it('should setup NuvexPOS successfully', async () => {
    const result = await CloudflareService.setupNuvexPOS();
    expect(result.success).toBe(true);
  });
});
```

#### Testes E2E (Planejados)
```typescript
// Exemplo: e2e/sales.spec.ts
import { test, expect } from '@playwright/test';

test('complete sale flow', async ({ page }) => {
  await page.goto('/pos');
  // ... teste completo de venda
});
```

## 🎨 Padrões de Código

### Convenções de Nomenclatura
- **Componentes**: PascalCase (`ProductCard.tsx`)
- **Hooks**: camelCase com prefixo `use` (`useProducts.ts`)
- **Utilitários**: camelCase (`formatCurrency.ts`)
- **Constantes**: UPPER_SNAKE_CASE (`API_ENDPOINTS`)

### Estrutura de Componentes
```typescript
// Exemplo: components/ProductCard.tsx
import { FC } from 'react';
import { cn } from '@/lib/utils';

interface ProductCardProps {
  product: Product;
  onSelect?: (product: Product) => void;
  className?: string;
}

export const ProductCard: FC<ProductCardProps> = ({
  product,
  onSelect,
  className
}) => {
  return (
    <div className={cn('product-card', className)}>
      {/* Conteúdo do componente */}
    </div>
  );
};
```

### Custom Hooks
```typescript
// Exemplo: hooks/useProducts.ts
import { useQuery } from '@tanstack/react-query';
import { productService } from '@/services/productService';

export const useProducts = (storeId: string) => {
  return useQuery({
    queryKey: ['products', storeId],
    queryFn: () => productService.getByStore(storeId),
    staleTime: 5 * 60 * 1000, // 5 minutos
  });
};
```

### Serviços
```typescript
// Exemplo: services/productService.ts
import { apiClient } from '@/lib/apiClient';
import { Product, CreateProductData } from '@/types/product';

export const productService = {
  async getByStore(storeId: string): Promise<Product[]> {
    const response = await apiClient.get(`/stores/${storeId}/products`);
    return response.data;
  },

  async create(data: CreateProductData): Promise<Product> {
    const response = await apiClient.post('/products', data);
    return response.data;
  },
};
```

## 🔄 Workflow de Desenvolvimento

### 1. Feature Development
```bash
# Criar branch para feature
git checkout -b feature/nova-funcionalidade

# Desenvolver e testar
npm run dev
npm test

# Commit com conventional commits
git commit -m "feat: adiciona nova funcionalidade"

# Push e criar PR
git push origin feature/nova-funcionalidade
```

### 2. Code Review
- Todos os PRs devem ser revisados
- Testes devem passar no CI
- Cobertura de código deve ser mantida
- Documentação deve ser atualizada

### 3. Deploy
- **Staging**: Deploy automático em PRs
- **Production**: Deploy manual após merge

## 🐛 Debugging

### Frontend Debugging
```typescript
// Usar React DevTools
// Logs estruturados
console.log('🔍 Debug:', { data, state });

// Breakpoints condicionais
if (process.env.NODE_ENV === 'development') {
  debugger;
}
```

### Backend Debugging
```typescript
// Workers debugging
console.log('🔧 Worker Debug:', {
  request: request.url,
  method: request.method,
  timestamp: new Date().toISOString()
});
```

### Performance Debugging
```bash
# Bundle analyzer
npm run build:analyze

# Performance profiling
npm run dev:profile
```

## 📊 Monitoramento Local

### Logs de Desenvolvimento
```typescript
// lib/logger.ts
export const logger = {
  info: (message: string, data?: any) => {
    if (process.env.NODE_ENV === 'development') {
      console.log(`ℹ️ ${message}`, data);
    }
  },
  error: (message: string, error?: Error) => {
    console.error(`❌ ${message}`, error);
  }
};
```

### Métricas de Performance
```typescript
// hooks/usePerformance.ts
export const usePerformance = () => {
  useEffect(() => {
    const observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        console.log('⚡ Performance:', entry);
      });
    });
    observer.observe({ entryTypes: ['measure'] });
  }, []);
};
```

## 🔧 Troubleshooting

### Problemas Comuns

#### 1. Erro de Build
```bash
# Limpar cache
npm run clean
npm install

# Verificar tipos
npm run type-check
```

#### 2. Problemas de Cloudflare
```bash
# Verificar configuração
npm run cf:whoami

# Reautenticar
npm run cf:login
```

#### 3. Testes Falhando
```bash
# Executar testes específicos
npm test -- --testNamePattern="nome do teste"

# Modo debug
npm test -- --verbose
```

## 📚 Recursos Adicionais

### Documentação Externa
- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs)
- [Cloudflare Workers](https://developers.cloudflare.com/workers)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Shadcn/ui](https://ui.shadcn.com)

### Ferramentas Úteis
- [React DevTools](https://chrome.google.com/webstore/detail/react-developer-tools)
- [Cloudflare Dashboard](https://dash.cloudflare.com)
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler)

### Comunidade
- [Discord NuvexPOS](https://discord.gg/nuvexpos)
- [GitHub Discussions](https://github.com/seu-usuario/nuvexpos/discussions)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/nuvexpos)