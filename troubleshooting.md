# 🔧 Troubleshooting - NuvexPOS

## 📋 Visão Geral

Este guia contém soluções para os problemas mais comuns encontrados durante o desenvolvimento, deploy e operação do NuvexPOS. Organize os problemas por categoria para facilitar a localização da solução.

## 🚀 Problemas de Desenvolvimento

### 1. Erro de Instalação de Dependências

#### Problema
```bash
npm ERR! peer dep missing: react@^18.0.0
```

#### Solução
```bash
# Limpar cache do npm
npm cache clean --force

# Deletar node_modules e package-lock.json
rm -rf node_modules package-lock.json

# Reinstalar dependências
npm install

# Se persistir, usar --legacy-peer-deps
npm install --legacy-peer-deps
```

### 2. Erro de TypeScript

#### Problema
```
Type 'string' is not assignable to type 'number'
```

#### Solução
```typescript
// Verificar tipos
npm run type-check

// Corrigir tipos explicitamente
const price: number = parseFloat(priceString);

// Ou usar type assertion (com cuidado)
const price = priceString as unknown as number;
```

### 3. Erro de Build Vite

#### Problema
```
[vite] Internal server error: Failed to resolve import
```

#### Solução
```bash
# Verificar se o arquivo existe
ls -la src/components/Component.tsx

# Verificar imports relativos
# ❌ Errado
import { Component } from '../../../components/Component'

# ✅ Correto
import { Component } from '@/components/Component'

# Limpar cache do Vite
rm -rf node_modules/.vite
npm run dev
```

### 4. Problemas com Tailwind CSS

#### Problema
Classes do Tailwind não funcionam

#### Solução
```bash
# Verificar configuração do Tailwind
cat tailwind.config.js

# Verificar se o CSS está sendo importado
# src/index.css deve conter:
@tailwind base;
@tailwind components;
@tailwind utilities;

# Regenerar classes
npm run build
```

## 🔐 Problemas de Autenticação

### 1. Token JWT Inválido

#### Problema
```json
{
  "error": "Invalid JWT token"
}
```

#### Solução
```typescript
// Verificar se o token está sendo enviado corretamente
const token = localStorage.getItem('auth_token');
if (!token) {
  // Redirecionar para login
  window.location.href = '/login';
}

// Verificar formato do header
const headers = {
  'Authorization': `Bearer ${token}`,
  'Content-Type': 'application/json'
};

// Verificar expiração do token
const payload = JSON.parse(atob(token.split('.')[1]));
if (payload.exp * 1000 < Date.now()) {
  // Token expirado, fazer logout
  logout();
}
```

### 2. Erro de CORS

#### Problema
```
Access to fetch at 'https://api.nuvexpos.com' from origin 'http://localhost:5173' has been blocked by CORS policy
```

#### Solução
```typescript
// No Worker, configurar CORS
const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization',
};

// Responder a OPTIONS requests
if (request.method === 'OPTIONS') {
  return new Response(null, { headers: corsHeaders });
}

// Adicionar headers CORS a todas as respostas
return new Response(JSON.stringify(data), {
  headers: { ...corsHeaders, 'Content-Type': 'application/json' }
});
```

## 🌐 Problemas de Cloudflare

### 1. Erro de Deploy Worker

#### Problema
```
Error: A request to the Cloudflare API failed
```

#### Solução
```bash
# Verificar autenticação
wrangler whoami

# Se não autenticado, fazer login
wrangler auth login

# Verificar configuração do wrangler.toml
cat wrangler.toml

# Verificar permissões do API token
# Token deve ter permissões:
# - Zone:Zone:Read
# - Zone:Zone Settings:Edit
# - Cloudflare Workers:Service Worker:Edit

# Deploy com logs detalhados
wrangler deploy --verbose
```

### 2. Erro de KV Namespace

#### Problema
```
Error: KV namespace not found
```

#### Solução
```bash
# Listar namespaces existentes
wrangler kv:namespace list

# Criar namespace se não existir
wrangler kv:namespace create "CACHE"

# Atualizar wrangler.toml com o ID correto
[[kv_namespaces]]
binding = "CACHE"
id = "seu_namespace_id_aqui"
```

### 3. Erro de D1 Database

#### Problema
```
Error: D1 database not found
```

#### Solução
```bash
# Listar databases
wrangler d1 list

# Criar database se não existir
wrangler d1 create nuvexpos-db

# Aplicar migrations
wrangler d1 migrations apply nuvexpos-db

# Verificar conexão
wrangler d1 execute nuvexpos-db --command "SELECT 1"
```

## 🗄️ Problemas de Banco de Dados

### 1. Erro de Migration

#### Problema
```
Error applying migration: table already exists
```

#### Solução
```sql
-- Verificar estado atual
SELECT name FROM sqlite_master WHERE type='table';

-- Criar migration condicional
CREATE TABLE IF NOT EXISTS products (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL
);

-- Ou usar DROP TABLE IF EXISTS (cuidado!)
DROP TABLE IF EXISTS temp_table;
```

### 2. Erro de Query

#### Problema
```
Error: no such column: created_at
```

#### Solução
```sql
-- Verificar schema da tabela
PRAGMA table_info(products);

-- Adicionar coluna se não existir
ALTER TABLE products ADD COLUMN created_at DATETIME DEFAULT CURRENT_TIMESTAMP;

-- Verificar novamente
PRAGMA table_info(products);
```

### 3. Performance de Query

#### Problema
Queries muito lentas

#### Solução
```sql
-- Adicionar índices
CREATE INDEX IF NOT EXISTS idx_products_store_id ON products(store_id);
CREATE INDEX IF NOT EXISTS idx_sales_date ON sales(created_at);

-- Otimizar queries
-- ❌ Evitar
SELECT * FROM products WHERE name LIKE '%termo%';

-- ✅ Preferir
SELECT id, name, price FROM products 
WHERE store_id = ? AND active = 1 
ORDER BY name LIMIT 20;

-- Usar EXPLAIN QUERY PLAN para analisar
EXPLAIN QUERY PLAN SELECT * FROM products WHERE store_id = ?;
```

## 🧪 Problemas de Testes

### 1. Testes Falhando

#### Problema
```
TypeError: Cannot read property 'mockReturnValue' of undefined
```

#### Solução
```typescript
// Verificar se o mock está configurado corretamente
jest.mock('@/services/api', () => ({
  apiClient: {
    get: jest.fn(),
    post: jest.fn(),
    put: jest.fn(),
    delete: jest.fn(),
  }
}));

// Usar beforeEach para limpar mocks
beforeEach(() => {
  jest.clearAllMocks();
});

// Verificar imports
import { apiClient } from '@/services/api';
const mockApiClient = apiClient as jest.Mocked<typeof apiClient>;
```

### 2. Erro de Timeout

#### Problema
```
Timeout - Async callback was not invoked within the 5000 ms timeout
```

#### Solução
```typescript
// Aumentar timeout para testes específicos
it('should handle long operation', async () => {
  // Teste que demora mais tempo
}, 10000); // 10 segundos

// Ou configurar globalmente no jest.config.js
module.exports = {
  testTimeout: 10000,
};

// Verificar se async/await está correto
// ❌ Errado
it('should work', () => {
  someAsyncFunction(); // Sem await
});

// ✅ Correto
it('should work', async () => {
  await someAsyncFunction();
});
```

## 🎨 Problemas de UI/UX

### 1. Componentes não Renderizam

#### Problema
Componente aparece em branco

#### Solução
```typescript
// Verificar se há erros no console
console.error('Component error:', error);

// Adicionar Error Boundary
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({error}: {error: Error}) {
  return (
    <div role="alert">
      <h2>Algo deu errado:</h2>
      <pre>{error.message}</pre>
    </div>
  );
}

// Usar Error Boundary
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <MyComponent />
</ErrorBoundary>
```

### 2. Estilos não Aplicam

#### Problema
CSS não funciona como esperado

#### Solução
```typescript
// Verificar especificidade CSS
// ❌ Baixa especificidade
.button { color: red; }

// ✅ Maior especificidade
.pos-interface .button { color: red; }

// Usar !important com parcimônia
.button { color: red !important; }

// Verificar se classes Tailwind estão sendo purgadas
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  // ...
};
```

### 3. Responsividade

#### Problema
Layout quebra em mobile

#### Solução
```typescript
// Usar breakpoints do Tailwind
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
  {/* Conteúdo */}
</div>

// Testar em diferentes tamanhos
// Chrome DevTools > Toggle device toolbar

// Usar hook para detectar tamanho da tela
const useBreakpoint = () => {
  const [breakpoint, setBreakpoint] = useState('sm');
  
  useEffect(() => {
    const updateBreakpoint = () => {
      if (window.innerWidth >= 1024) setBreakpoint('lg');
      else if (window.innerWidth >= 768) setBreakpoint('md');
      else setBreakpoint('sm');
    };
    
    updateBreakpoint();
    window.addEventListener('resize', updateBreakpoint);
    return () => window.removeEventListener('resize', updateBreakpoint);
  }, []);
  
  return breakpoint;
};
```

## 🔄 Problemas de Performance

### 1. Aplicação Lenta

#### Problema
Interface travando ou lenta

#### Solução
```typescript
// Usar React.memo para componentes pesados
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* Renderização pesada */}</div>;
});

// Usar useMemo para cálculos pesados
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// Usar useCallback para funções
const handleClick = useCallback(() => {
  // Handler
}, [dependency]);

// Lazy loading de componentes
const LazyComponent = lazy(() => import('./LazyComponent'));

// Virtualização para listas grandes
import { FixedSizeList as List } from 'react-window';
```

### 2. Bundle Muito Grande

#### Problema
```
Warning: asset size limit: The following asset(s) exceed the recommended size limit
```

#### Solução
```typescript
// Analisar bundle
npm run build:analyze

// Code splitting por rota
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Products = lazy(() => import('./pages/Products'));

// Tree shaking - importar apenas o necessário
// ❌ Importa toda a biblioteca
import * as _ from 'lodash';

// ✅ Importa apenas a função necessária
import { debounce } from 'lodash';

// Ou melhor ainda
import debounce from 'lodash/debounce';
```

## 🔍 Debugging e Logs

### 1. Debug no Desenvolvimento

```typescript
// Logger estruturado
const logger = {
  debug: (message: string, data?: any) => {
    if (process.env.NODE_ENV === 'development') {
      console.log(`🔍 ${message}`, data);
    }
  },
  error: (message: string, error?: Error) => {
    console.error(`❌ ${message}`, error);
  }
};

// React DevTools
// Instalar extensão do Chrome
// Usar React Developer Tools

// Redux DevTools (se usar Redux)
// Instalar extensão do Chrome
```

### 2. Debug no Worker

```typescript
// Logs no Worker
console.log('Worker Debug:', {
  url: request.url,
  method: request.method,
  timestamp: new Date().toISOString()
});

// Visualizar logs
wrangler tail nuvexpos-api --format=pretty

// Filtrar logs
wrangler tail nuvexpos-api --status=error
```

## 🆘 Quando Pedir Ajuda

### Informações para Incluir
1. **Versão do Node.js**: `node --version`
2. **Versão do npm**: `npm --version`
3. **Sistema Operacional**: macOS, Windows, Linux
4. **Mensagem de erro completa**
5. **Passos para reproduzir**
6. **Código relevante**
7. **Configurações (sem secrets)**

### Onde Pedir Ajuda
- [GitHub Issues](https://github.com/seu-usuario/nuvexpos/issues)
- [Discord NuvexPOS](https://discord.gg/nuvexpos)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/nuvexpos)

### Template de Issue
```markdown
## Descrição do Problema
Descrição clara do que está acontecendo.

## Passos para Reproduzir
1. Vá para '...'
2. Clique em '....'
3. Veja o erro

## Comportamento Esperado
O que deveria acontecer.

## Screenshots
Se aplicável, adicione screenshots.

## Ambiente
- OS: [e.g. macOS 14.0]
- Node.js: [e.g. 18.17.0]
- npm: [e.g. 9.6.7]
- Browser: [e.g. Chrome 120.0]

## Informações Adicionais
Qualquer outra informação relevante.
```

## 📚 Recursos Úteis

### Ferramentas de Debug
- [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools)
- [Cloudflare Workers Debugger](https://developers.cloudflare.com/workers/learning/debugging-workers/)
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/)

### Documentação
- [React Docs](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs)
- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers)
- [Tailwind CSS Docs](https://tailwindcss.com/docs)

### Comunidades
- [React Community](https://reactjs.org/community/support.html)
- [Cloudflare Community](https://community.cloudflare.com)
- [TypeScript Community](https://www.typescriptlang.org/community)