# Guia de Troubleshooting NuvexPOS
## Resolução de Problemas e Diagnósticos - Janeiro 2025

---

## 🎯 **OBJETIVO**

Este guia fornece soluções práticas para os problemas mais comuns encontrados na plataforma NuvexPOS, incluindo diagnósticos, logs, métricas e procedimentos de resolução.

---

## 🚨 **PROBLEMAS CRÍTICOS**

### **1. Sistema Fora do Ar**

#### **Sintomas:**
- API retorna erro 503 (Service Unavailable)
- Interface não carrega
- Timeout em todas as requisições

#### **Diagnóstico:**
```bash
# Verificar status dos serviços
curl -I https://api.nuvexpos.com/v2/health

# Verificar CDN
curl -I https://cdn.nuvexpos.com/status

# Verificar banco de dados
curl -X GET https://api.nuvexpos.com/v2/system/db-status \
  -H "X-Admin-Key: admin_key"
```

#### **Soluções:**

**1. Verificar Status Page:**
```
https://status.nuvexpos.com
```

**2. Contato de Emergência:**
```
WhatsApp: +55 11 99999-9999
Email: emergency@nuvexpos.com
Slack: #incidents
```

**3. Fallback Local:**
```javascript
// Ativar modo offline
localStorage.setItem('nuvexpos_offline_mode', 'true');

// Usar cache local
const offlineData = localStorage.getItem('nuvexpos_cache');
```

### **2. Perda de Dados**

#### **Sintomas:**
- Vendas não aparecem no sistema
- Produtos desapareceram
- Relatórios zerados

#### **Diagnóstico:**
```bash
# Verificar logs de auditoria
curl -X GET https://api.nuvexpos.com/v2/audit/logs \
  -H "Authorization: Bearer {token}" \
  -G -d "action=delete" -d "date_from=2025-01-20"

# Verificar backups
curl -X GET https://api.nuvexpos.com/v2/backups/list \
  -H "Authorization: Bearer {token}"
```

#### **Soluções:**

**1. Restaurar do Backup:**
```bash
# Listar backups disponíveis
curl -X GET https://api.nuvexpos.com/v2/backups \
  -H "Authorization: Bearer {token}"

# Restaurar backup específico
curl -X POST https://api.nuvexpos.com/v2/backups/restore \
  -H "Authorization: Bearer {token}" \
  -d '{"backup_id": "backup_123", "confirm": true}'
```

**2. Recuperação Parcial:**
```bash
# Recuperar vendas específicas
curl -X POST https://api.nuvexpos.com/v2/recovery/sales \
  -H "Authorization: Bearer {token}" \
  -d '{"date_from": "2025-01-20", "date_to": "2025-01-21"}'
```

### **3. Performance Degradada**

#### **Sintomas:**
- Requisições lentas (>5 segundos)
- Interface travando
- Timeouts frequentes

#### **Diagnóstico:**
```bash
# Verificar métricas de performance
curl -X GET https://api.nuvexpos.com/v2/metrics/performance \
  -H "Authorization: Bearer {token}"

# Verificar uso de recursos
curl -X GET https://api.nuvexpos.com/v2/system/resources \
  -H "X-Admin-Key: admin_key"
```

#### **Soluções:**

**1. Otimização de Queries:**
```javascript
// Usar paginação
const products = await client.products.list({
  page: 1,
  limit: 50  // Reduzir de 100 para 50
});

// Usar campos específicos
const sales = await client.sales.list({
  fields: ['id', 'total', 'created_at']  // Apenas campos necessários
});
```

**2. Cache Local:**
```javascript
// Implementar cache
const cache = new Map();

async function getCachedProducts() {
  if (cache.has('products')) {
    return cache.get('products');
  }
  
  const products = await client.products.list();
  cache.set('products', products);
  
  // Expirar cache em 5 minutos
  setTimeout(() => cache.delete('products'), 5 * 60 * 1000);
  
  return products;
}
```

---

## 🔐 **PROBLEMAS DE AUTENTICAÇÃO**

### **1. Token Expirado**

#### **Sintomas:**
```json
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "Token de acesso expirado"
  }
}
```

#### **Soluções:**

**1. Renovação Automática:**
```javascript
class NuvexPOSClient {
  async request(endpoint, options = {}) {
    try {
      return await this.makeRequest(endpoint, options);
    } catch (error) {
      if (error.code === 'TOKEN_EXPIRED') {
        await this.refreshToken();
        return await this.makeRequest(endpoint, options);
      }
      throw error;
    }
  }
  
  async refreshToken() {
    const response = await fetch('/v2/auth/refresh', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${this.refreshToken}` }
    });
    
    const { access_token } = await response.json();
    this.accessToken = access_token;
  }
}
```

**2. Verificação de Expiração:**
```javascript
function isTokenExpired(token) {
  try {
    const payload = JSON.parse(atob(token.split('.')[1]));
    return Date.now() >= payload.exp * 1000;
  } catch {
    return true;
  }
}

// Usar antes de fazer requisições
if (isTokenExpired(accessToken)) {
  await refreshToken();
}
```

### **2. API Key Inválida**

#### **Sintomas:**
```json
{
  "error": {
    "code": "INVALID_API_KEY",
    "message": "API key inválida ou não encontrada"
  }
}
```

#### **Soluções:**

**1. Verificar Configuração:**
```javascript
// Verificar se API key está definida
if (!process.env.NUVEXPOS_API_KEY) {
  throw new Error('NUVEXPOS_API_KEY não definida');
}

// Verificar formato
const apiKeyPattern = /^nvx_(live|test)_[a-zA-Z0-9]{32}$/;
if (!apiKeyPattern.test(apiKey)) {
  throw new Error('Formato de API key inválido');
}
```

**2. Testar API Key:**
```bash
# Testar API key
curl -X GET https://api.nuvexpos.com/v2/auth/verify \
  -H "X-API-Key: nvx_live_1234567890abcdef" \
  -H "X-Store-ID: store_123"
```

### **3. Permissões Insuficientes**

#### **Sintomas:**
```json
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "Permissões insuficientes para esta operação"
  }
}
```

#### **Soluções:**

**1. Verificar Escopos:**
```bash
# Verificar permissões do token
curl -X GET https://api.nuvexpos.com/v2/auth/permissions \
  -H "Authorization: Bearer {token}"
```

**2. Solicitar Permissões:**
```javascript
// Verificar permissões necessárias
const requiredScopes = ['write_products', 'read_sales'];
const userScopes = await client.auth.getPermissions();

const missingScopes = requiredScopes.filter(
  scope => !userScopes.includes(scope)
);

if (missingScopes.length > 0) {
  throw new Error(`Permissões necessárias: ${missingScopes.join(', ')}`);
}
```

---

## 💾 **PROBLEMAS DE DADOS**

### **1. Produto Não Encontrado**

#### **Sintomas:**
```json
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "Produto não encontrado",
    "details": { "product_id": "prod_123" }
  }
}
```

#### **Diagnóstico:**
```bash
# Buscar produto por diferentes critérios
curl -X GET "https://api.nuvexpos.com/v2/products/search?q=prod_123&field=id" \
  -H "Authorization: Bearer {token}"

# Verificar se foi deletado
curl -X GET "https://api.nuvexpos.com/v2/audit/logs?entity=product&entity_id=prod_123" \
  -H "Authorization: Bearer {token}"
```

#### **Soluções:**

**1. Busca Alternativa:**
```javascript
async function findProduct(identifier) {
  // Tentar por ID
  try {
    return await client.products.get(identifier);
  } catch (error) {
    if (error.code !== 'PRODUCT_NOT_FOUND') throw error;
  }
  
  // Tentar por SKU
  try {
    const results = await client.products.search({ sku: identifier });
    if (results.data.length > 0) return results.data[0];
  } catch (error) {
    console.warn('Busca por SKU falhou:', error);
  }
  
  // Tentar por código de barras
  try {
    const results = await client.products.search({ barcode: identifier });
    if (results.data.length > 0) return results.data[0];
  } catch (error) {
    console.warn('Busca por código de barras falhou:', error);
  }
  
  throw new Error(`Produto não encontrado: ${identifier}`);
}
```

### **2. Estoque Insuficiente**

#### **Sintomas:**
```json
{
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Estoque insuficiente",
    "details": {
      "product_id": "prod_123",
      "requested": 10,
      "available": 5
    }
  }
}
```

#### **Soluções:**

**1. Verificação Prévia:**
```javascript
async function validateStock(items) {
  const stockChecks = await Promise.all(
    items.map(async item => {
      const product = await client.products.get(item.product_id);
      return {
        product_id: item.product_id,
        requested: item.quantity,
        available: product.stock.quantity,
        sufficient: product.stock.quantity >= item.quantity
      };
    })
  );
  
  const insufficient = stockChecks.filter(check => !check.sufficient);
  
  if (insufficient.length > 0) {
    throw new Error(`Estoque insuficiente: ${JSON.stringify(insufficient)}`);
  }
  
  return true;
}
```

**2. Venda Parcial:**
```javascript
async function createPartialSale(items) {
  const availableItems = [];
  const unavailableItems = [];
  
  for (const item of items) {
    const product = await client.products.get(item.product_id);
    
    if (product.stock.quantity >= item.quantity) {
      availableItems.push(item);
    } else if (product.stock.quantity > 0) {
      // Vender quantidade disponível
      availableItems.push({
        ...item,
        quantity: product.stock.quantity
      });
      
      // Registrar quantidade não disponível
      unavailableItems.push({
        ...item,
        quantity: item.quantity - product.stock.quantity
      });
    } else {
      unavailableItems.push(item);
    }
  }
  
  const sale = await client.sales.create({
    items: availableItems,
    notes: `Venda parcial. Itens não disponíveis: ${JSON.stringify(unavailableItems)}`
  });
  
  return { sale, unavailableItems };
}
```

### **3. Dados Corrompidos**

#### **Sintomas:**
- Preços negativos
- Quantidades inválidas
- Datas futuras

#### **Diagnóstico:**
```bash
# Verificar integridade dos dados
curl -X GET https://api.nuvexpos.com/v2/system/data-integrity \
  -H "Authorization: Bearer {token}"

# Relatório de anomalias
curl -X GET https://api.nuvexpos.com/v2/analytics/anomalies \
  -H "Authorization: Bearer {token}"
```

#### **Soluções:**

**1. Validação de Dados:**
```javascript
function validateProduct(product) {
  const errors = [];
  
  if (!product.name || product.name.trim().length === 0) {
    errors.push('Nome é obrigatório');
  }
  
  if (product.price < 0) {
    errors.push('Preço não pode ser negativo');
  }
  
  if (product.stock && product.stock.quantity < 0) {
    errors.push('Estoque não pode ser negativo');
  }
  
  if (product.created_at && new Date(product.created_at) > new Date()) {
    errors.push('Data de criação não pode ser futura');
  }
  
  if (errors.length > 0) {
    throw new Error(`Dados inválidos: ${errors.join(', ')}`);
  }
  
  return true;
}
```

**2. Limpeza de Dados:**
```javascript
async function cleanupData() {
  // Corrigir preços negativos
  const negativePrice = await client.products.search({
    filters: { price_range: { max: 0 } }
  });
  
  for (const product of negativePrice.data) {
    await client.products.update(product.id, {
      price: Math.abs(product.price)
    });
  }
  
  // Corrigir estoque negativo
  const negativeStock = await client.products.search({
    filters: { stock_range: { max: 0 } }
  });
  
  for (const product of negativeStock.data) {
    await client.products.update(product.id, {
      stock: { quantity: 0 }
    });
  }
}
```

---

## 🌐 **PROBLEMAS DE CONECTIVIDADE**

### **1. Timeout de Requisições**

#### **Sintomas:**
- Requisições demoram mais de 30 segundos
- Erro "Request timeout"

#### **Soluções:**

**1. Configurar Timeout:**
```javascript
const client = new NuvexPOS({
  apiKey: 'sua_api_key',
  timeout: 10000,  // 10 segundos
  retries: 3,
  retryDelay: 1000  // 1 segundo entre tentativas
});
```

**2. Implementar Retry:**
```javascript
async function requestWithRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;  // Backoff exponencial
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Uso
const products = await requestWithRetry(() => 
  client.products.list()
);
```

### **2. Rate Limiting**

#### **Sintomas:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Limite de requisições excedido",
    "retry_after": 60
  }
}
```

#### **Soluções:**

**1. Implementar Rate Limiting:**
```javascript
class RateLimiter {
  constructor(maxRequests = 100, windowMs = 60000) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
    this.requests = [];
  }
  
  async waitIfNeeded() {
    const now = Date.now();
    
    // Remover requisições antigas
    this.requests = this.requests.filter(
      time => now - time < this.windowMs
    );
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = Math.min(...this.requests);
      const waitTime = this.windowMs - (now - oldestRequest);
      
      if (waitTime > 0) {
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
    
    this.requests.push(now);
  }
}

const rateLimiter = new RateLimiter();

// Usar antes de cada requisição
await rateLimiter.waitIfNeeded();
const products = await client.products.list();
```

**2. Batch Requests:**
```javascript
// Em vez de múltiplas requisições individuais
const products = [];
for (const id of productIds) {
  products.push(await client.products.get(id));  // ❌ Muitas requisições
}

// Usar batch request
const products = await client.products.getBatch(productIds);  // ✅ Uma requisição
```

### **3. Problemas de DNS/CDN**

#### **Sintomas:**
- Erro "DNS resolution failed"
- Recursos não carregam

#### **Diagnóstico:**
```bash
# Verificar DNS
nslookup api.nuvexpos.com

# Verificar conectividade
ping api.nuvexpos.com

# Verificar CDN
curl -I https://cdn.nuvexpos.com/health
```

#### **Soluções:**

**1. Usar IPs Alternativos:**
```javascript
const client = new NuvexPOS({
  apiKey: 'sua_api_key',
  baseURL: 'https://backup-api.nuvexpos.com/v2'  // URL de backup
});
```

**2. Configurar Proxy:**
```javascript
const client = new NuvexPOS({
  apiKey: 'sua_api_key',
  proxy: {
    host: 'proxy.empresa.com',
    port: 8080,
    auth: {
      username: 'user',
      password: 'pass'
    }
  }
});
```

---

## 🤖 **PROBLEMAS DE IA**

### **1. Predições Incorretas**

#### **Sintomas:**
- IA prevê demanda muito alta/baixa
- Recomendações não fazem sentido

#### **Diagnóstico:**
```bash
# Verificar qualidade dos dados
curl -X GET https://api.nuvexpos.com/v2/ai/data-quality \
  -H "Authorization: Bearer {token}"

# Verificar modelo atual
curl -X GET https://api.nuvexpos.com/v2/ai/model-info \
  -H "Authorization: Bearer {token}"
```

#### **Soluções:**

**1. Melhorar Dados de Treinamento:**
```javascript
// Fornecer feedback sobre predições
await client.ai.feedback({
  prediction_id: 'pred_123',
  actual_value: 150,
  predicted_value: 200,
  accuracy: 'low',
  notes: 'Não considerou promoção concorrente'
});

// Adicionar contexto
await client.ai.addContext({
  type: 'promotion',
  start_date: '2025-01-20',
  end_date: '2025-01-25',
  discount: 0.20,
  affected_products: ['prod_123', 'prod_456']
});
```

**2. Ajustar Parâmetros:**
```javascript
// Configurar sensibilidade
const prediction = await client.ai.predictDemand('prod_123', {
  sensitivity: 'conservative',  // conservative, balanced, aggressive
  factors: ['seasonality', 'promotions'],
  exclude_factors: ['weather']  // Excluir fatores não relevantes
});
```

### **2. Comandos de Voz Não Reconhecidos**

#### **Sintomas:**
- "Comando não compreendido"
- Transcrição incorreta

#### **Soluções:**

**1. Melhorar Qualidade do Áudio:**
```javascript
// Configurar parâmetros de áudio
const voiceConfig = {
  sampleRate: 16000,
  channels: 1,
  bitDepth: 16,
  noiseReduction: true,
  autoGainControl: true
};

// Processar comando com configurações otimizadas
const result = await client.voice.process({
  audio_url: audioUrl,
  language: 'pt-BR',
  context: 'sales',
  config: voiceConfig
});
```

**2. Treinar Vocabulário Personalizado:**
```javascript
// Adicionar termos específicos da loja
await client.voice.addVocabulary([
  { term: 'coca-cola', alternatives: ['coca', 'refrigerante'] },
  { term: 'guaraná', alternatives: ['guarana', 'guaraná antarctica'] },
  { term: 'desconto', alternatives: ['desc', 'promoção'] }
]);
```

### **3. Computer Vision Falha**

#### **Sintomas:**
- Produtos não reconhecidos
- Confiança baixa (<70%)

#### **Soluções:**

**1. Melhorar Qualidade da Imagem:**
```javascript
// Verificar qualidade da imagem
function validateImage(imageUrl) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    
    img.onload = () => {
      if (img.width < 300 || img.height < 300) {
        reject(new Error('Imagem muito pequena (mín. 300x300)'));
      }
      
      if (img.width / img.height > 3 || img.height / img.width > 3) {
        reject(new Error('Proporção da imagem inadequada'));
      }
      
      resolve(true);
    };
    
    img.onerror = () => reject(new Error('Erro ao carregar imagem'));
    img.src = imageUrl;
  });
}

// Usar antes do reconhecimento
await validateImage(imageUrl);
const recognition = await client.vision.recognizeProduct(imageUrl);
```

**2. Treinar com Novas Imagens:**
```javascript
// Adicionar imagens de treinamento
await client.vision.addTrainingImage({
  product_id: 'prod_123',
  image_url: 'https://example.com/product.jpg',
  annotations: {
    bounding_box: { x: 100, y: 150, width: 200, height: 300 },
    quality: 'high',
    lighting: 'good'
  }
});
```

---

## 📊 **PROBLEMAS DE PERFORMANCE**

### **1. Consultas Lentas**

#### **Diagnóstico:**
```bash
# Verificar performance de queries
curl -X GET https://api.nuvexpos.com/v2/analytics/query-performance \
  -H "Authorization: Bearer {token}"

# Verificar índices
curl -X GET https://api.nuvexpos.com/v2/system/database-indexes \
  -H "X-Admin-Key: admin_key"
```

#### **Soluções:**

**1. Otimizar Consultas:**
```javascript
// ❌ Consulta ineficiente
const allProducts = await client.products.list({ limit: 10000 });
const activeProducts = allProducts.data.filter(p => p.active);

// ✅ Consulta otimizada
const activeProducts = await client.products.list({
  active: true,
  limit: 100,
  fields: ['id', 'name', 'price']  // Apenas campos necessários
});
```

**2. Usar Paginação:**
```javascript
async function getAllProducts() {
  const products = [];
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await client.products.list({
      page,
      limit: 100
    });
    
    products.push(...response.data);
    hasMore = response.pagination.has_next;
    page++;
    
    // Evitar sobrecarga
    if (page > 100) break;
  }
  
  return products;
}
```

### **2. Memória Insuficiente**

#### **Sintomas:**
- Erro "Out of memory"
- Aplicação trava

#### **Soluções:**

**1. Streaming de Dados:**
```javascript
// ❌ Carregar tudo na memória
const allSales = await client.sales.list({ limit: 100000 });

// ✅ Processar em chunks
async function processSalesStream(callback) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const response = await client.sales.list({
      page,
      limit: 100
    });
    
    // Processar chunk
    await callback(response.data);
    
    hasMore = response.pagination.has_next;
    page++;
  }
}

// Uso
await processSalesStream(async (sales) => {
  for (const sale of sales) {
    await processIndividualSale(sale);
  }
});
```

**2. Limpeza de Memória:**
```javascript
// Limpar cache periodicamente
setInterval(() => {
  if (global.gc) {
    global.gc();
  }
}, 60000);  // A cada minuto

// Limpar variáveis grandes
let largeData = await fetchLargeDataset();
await processData(largeData);
largeData = null;  // Liberar memória
```

---

## 🔧 **FERRAMENTAS DE DIAGNÓSTICO**

### **1. Health Check**

```bash
# Verificar saúde geral do sistema
curl -X GET https://api.nuvexpos.com/v2/health \
  -H "Authorization: Bearer {token}"
```

**Resposta:**
```json
{
  "status": "healthy",
  "timestamp": "2025-01-20T15:30:00Z",
  "services": {
    "api": "healthy",
    "database": "healthy",
    "cache": "healthy",
    "ai": "healthy",
    "storage": "degraded"
  },
  "metrics": {
    "response_time": 150,
    "cpu_usage": 45.2,
    "memory_usage": 67.8,
    "disk_usage": 23.1
  }
}
```

### **2. Logs de Sistema**

```bash
# Logs de erro
curl -X GET "https://api.nuvexpos.com/v2/logs?level=error&limit=50" \
  -H "Authorization: Bearer {token}"

# Logs de uma requisição específica
curl -X GET "https://api.nuvexpos.com/v2/logs?request_id=req_123" \
  -H "Authorization: Bearer {token}"
```

### **3. Métricas de Performance**

```javascript
// Monitorar performance em tempo real
const metrics = await client.system.getMetrics({
  metrics: ['response_time', 'throughput', 'error_rate'],
  period: 'last_hour',
  interval: '5m'
});

console.log('Tempo de resposta médio:', metrics.response_time.avg);
console.log('Taxa de erro:', metrics.error_rate.current);
```

### **4. Trace de Requisições**

```javascript
// Habilitar tracing detalhado
const client = new NuvexPOS({
  apiKey: 'sua_api_key',
  debug: true,
  tracing: {
    enabled: true,
    sampleRate: 0.1,  // 10% das requisições
    includeHeaders: true,
    includeBody: false
  }
});

// Analisar trace
const trace = await client.system.getTrace('req_123');
console.log('Duração total:', trace.duration);
console.log('Etapas:', trace.spans);
```

---

## 📋 **CHECKLIST DE DIAGNÓSTICO**

### **Problema Reportado:**
- [ ] **Reproduzir** - Conseguir replicar o problema
- [ ] **Logs** - Verificar logs de erro relevantes
- [ ] **Métricas** - Analisar métricas de performance
- [ ] **Ambiente** - Identificar ambiente (dev/staging/prod)

### **Investigação:**
- [ ] **Timeline** - Quando o problema começou
- [ ] **Escopo** - Quantos usuários afetados
- [ ] **Padrão** - Problema intermitente ou constante
- [ ] **Correlação** - Mudanças recentes no sistema

### **Resolução:**
- [ ] **Solução** - Implementar correção
- [ ] **Teste** - Validar que problema foi resolvido
- [ ] **Monitoramento** - Acompanhar por 24h
- [ ] **Documentação** - Registrar solução para futuros casos

### **Prevenção:**
- [ ] **Root Cause** - Identificar causa raiz
- [ ] **Melhorias** - Implementar melhorias preventivas
- [ ] **Alertas** - Configurar alertas para detecção precoce
- [ ] **Treinamento** - Treinar equipe se necessário

---

## 🚨 **CONTATOS DE EMERGÊNCIA**

### **Suporte Técnico 24/7:**
- **WhatsApp:** +55 11 99999-9999
- **Email:** emergency@nuvexpos.com
- **Slack:** #incidents
- **Telefone:** 0800-123-4567

### **Escalação:**

| **Nível** | **Tempo** | **Contato** |
|-----------|-----------|-------------|
| **L1** | 0-2h | Suporte Técnico |
| **L2** | 2-4h | Engenharia |
| **L3** | 4-8h | Arquitetura |
| **L4** | 8h+ | CTO |

### **Status Page:**
- **URL:** https://status.nuvexpos.com
- **Twitter:** @NuvexPOSStatus
- **RSS:** https://status.nuvexpos.com/rss

---

## 📚 **RECURSOS ADICIONAIS**

### **Documentação:**
- **API Docs:** https://docs.nuvexpos.com
- **Tutoriais:** https://learn.nuvexpos.com
- **FAQ:** https://help.nuvexpos.com

### **Comunidade:**
- **Discord:** https://discord.gg/nuvexpos
- **GitHub:** https://github.com/nuvexpos
- **Stack Overflow:** Tag `nuvexpos`

### **Treinamento:**
- **Webinars:** Quintas-feiras 14h
- **Workshops:** Mensais
- **Certificação:** https://cert.nuvexpos.com

---

## ✅ **STATUS**

**Data de Criação:** 20 Janeiro 2025  
**Versão:** 1.0  
**Responsável:** Equipe de Suporte Técnico  
**Status:** 🎯 **GUIA ATIVO**

**Próxima Revisão:** 1 Março 2025  
**Feedback:** support-feedback@nuvexpos.com

---

*Guia criado em: Janeiro 2025*  
*Responsável: Equipe de Suporte NuvexPOS*  
*Objetivo: Resolução rápida de problemas técnicos*