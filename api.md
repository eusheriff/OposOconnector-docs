# 🔌 API Reference - NuvexPOS

## 📋 Visão Geral

A API do NuvexPOS é construída sobre Cloudflare Workers, oferecendo endpoints RESTful com autenticação JWT e rate limiting. Todas as respostas seguem um padrão consistente e incluem códigos de status HTTP apropriados.

## 🔐 Autenticação

### JWT Token
Todas as requisições (exceto login) devem incluir o token JWT no header:

```http
Authorization: Bearer <jwt_token>
```

### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "usuario@exemplo.com",
  "password": "senha123"
}
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "user_123",
      "email": "usuario@exemplo.com",
      "name": "João Silva",
      "role": "manager",
      "storeId": "store_456"
    }
  }
}
```

## 📊 Estrutura de Resposta

### Resposta de Sucesso
```json
{
  "success": true,
  "data": {
    // Dados da resposta
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

### Resposta de Erro
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dados inválidos fornecidos",
    "details": {
      "field": "email",
      "reason": "Email é obrigatório"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_abc123"
  }
}
```

## 🏪 Endpoints - Produtos

### Listar Produtos
```http
GET /api/products?storeId={storeId}&page=1&limit=20&search=termo
```

**Parâmetros de Query:**
- `storeId` (obrigatório): ID da loja
- `page` (opcional): Página (padrão: 1)
- `limit` (opcional): Itens por página (padrão: 20, máx: 100)
- `search` (opcional): Termo de busca
- `categoryId` (opcional): Filtrar por categoria

**Resposta:**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "prod_123",
        "name": "Produto Exemplo",
        "description": "Descrição do produto",
        "price": 29.99,
        "categoryId": "cat_456",
        "category": {
          "id": "cat_456",
          "name": "Eletrônicos"
        },
        "inventory": {
          "quantity": 50,
          "minStock": 10
        },
        "images": [
          "https://r2.example.com/products/prod_123_1.jpg"
        ],
        "active": true,
        "createdAt": "2024-01-15T10:30:00Z",
        "updatedAt": "2024-01-15T10:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8
    }
  }
}
```

### Criar Produto
```http
POST /api/products
Content-Type: application/json

{
  "name": "Novo Produto",
  "description": "Descrição detalhada",
  "price": 49.99,
  "categoryId": "cat_456",
  "storeId": "store_789",
  "inventory": {
    "quantity": 100,
    "minStock": 20
  },
  "images": ["base64_image_data"]
}
```

### Atualizar Produto
```http
PUT /api/products/{productId}
Content-Type: application/json

{
  "name": "Produto Atualizado",
  "price": 59.99
}
```

### Deletar Produto
```http
DELETE /api/products/{productId}
```

## 🛒 Endpoints - Vendas

### Criar Venda
```http
POST /api/sales
Content-Type: application/json

{
  "storeId": "store_789",
  "items": [
    {
      "productId": "prod_123",
      "quantity": 2,
      "price": 29.99
    }
  ],
  "payments": [
    {
      "method": "credit_card",
      "amount": 59.98
    }
  ],
  "customer": {
    "name": "Cliente Exemplo",
    "email": "cliente@exemplo.com",
    "phone": "+5511999999999"
  }
}
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "sale": {
      "id": "sale_456",
      "number": "VND-2024-001",
      "storeId": "store_789",
      "total": 59.98,
      "status": "completed",
      "items": [
        {
          "id": "item_789",
          "productId": "prod_123",
          "productName": "Produto Exemplo",
          "quantity": 2,
          "unitPrice": 29.99,
          "totalPrice": 59.98
        }
      ],
      "payments": [
        {
          "id": "pay_101",
          "method": "credit_card",
          "amount": 59.98,
          "status": "approved"
        }
      ],
      "customer": {
        "name": "Cliente Exemplo",
        "email": "cliente@exemplo.com"
      },
      "createdAt": "2024-01-15T10:30:00Z"
    }
  }
}
```

### Listar Vendas
```http
GET /api/sales?storeId={storeId}&startDate=2024-01-01&endDate=2024-01-31
```

### Detalhes da Venda
```http
GET /api/sales/{saleId}
```

### Cancelar Venda
```http
POST /api/sales/{saleId}/cancel
Content-Type: application/json

{
  "reason": "Produto defeituoso"
}
```

## 📦 Endpoints - Estoque

### Consultar Estoque
```http
GET /api/inventory?storeId={storeId}&lowStock=true
```

### Atualizar Estoque
```http
PUT /api/inventory/{productId}
Content-Type: application/json

{
  "quantity": 75,
  "operation": "set", // "set", "add", "subtract"
  "reason": "Reposição de estoque"
}
```

### Histórico de Movimentação
```http
GET /api/inventory/{productId}/movements
```

## 👥 Endpoints - Clientes

### Listar Clientes
```http
GET /api/customers?storeId={storeId}&search=nome
```

### Criar Cliente
```http
POST /api/customers
Content-Type: application/json

{
  "name": "João Silva",
  "email": "joao@exemplo.com",
  "phone": "+5511999999999",
  "address": {
    "street": "Rua Exemplo, 123",
    "city": "São Paulo",
    "state": "SP",
    "zipCode": "01234-567"
  },
  "storeId": "store_789"
}
```

### Histórico de Compras
```http
GET /api/customers/{customerId}/purchases
```

## 📊 Endpoints - Relatórios

### Dashboard
```http
GET /api/reports/dashboard?storeId={storeId}&period=today
```

**Resposta:**
```json
{
  "success": true,
  "data": {
    "sales": {
      "total": 1250.50,
      "count": 25,
      "average": 50.02
    },
    "products": {
      "topSelling": [
        {
          "productId": "prod_123",
          "name": "Produto Top",
          "quantity": 15,
          "revenue": 449.85
        }
      ],
      "lowStock": [
        {
          "productId": "prod_456",
          "name": "Produto Baixo",
          "currentStock": 5,
          "minStock": 10
        }
      ]
    },
    "period": {
      "start": "2024-01-15T00:00:00Z",
      "end": "2024-01-15T23:59:59Z"
    }
  }
}
```

### Relatório de Vendas
```http
GET /api/reports/sales?storeId={storeId}&startDate=2024-01-01&endDate=2024-01-31&groupBy=day
```

### Relatório de Produtos
```http
GET /api/reports/products?storeId={storeId}&period=month&sortBy=revenue
```

## 🏢 Endpoints - Configurações

### Configurações da Loja
```http
GET /api/stores/{storeId}/settings
```

```http
PUT /api/stores/{storeId}/settings
Content-Type: application/json

{
  "name": "Loja Atualizada",
  "timezone": "America/Sao_Paulo",
  "currency": "BRL",
  "taxRate": 0.18,
  "receiptSettings": {
    "showLogo": true,
    "footerMessage": "Obrigado pela preferência!"
  }
}
```

### Usuários da Loja
```http
GET /api/stores/{storeId}/users
```

```http
POST /api/stores/{storeId}/users
Content-Type: application/json

{
  "email": "novo@usuario.com",
  "name": "Novo Usuário",
  "role": "cashier",
  "permissions": ["sales.create", "products.read"]
}
```

## 🔄 Webhooks

### Configurar Webhook
```http
POST /api/webhooks
Content-Type: application/json

{
  "url": "https://seu-sistema.com/webhook",
  "events": ["sale.created", "inventory.low"],
  "secret": "webhook_secret_123"
}
```

### Eventos Disponíveis
- `sale.created` - Nova venda criada
- `sale.cancelled` - Venda cancelada
- `inventory.low` - Estoque baixo
- `product.created` - Produto criado
- `customer.created` - Cliente criado

## ⚡ Rate Limiting

### Limites por Endpoint
- **Autenticação**: 5 tentativas por minuto por IP
- **Vendas**: 100 requisições por minuto por usuário
- **Produtos**: 200 requisições por minuto por usuário
- **Relatórios**: 50 requisições por minuto por usuário

### Headers de Rate Limit
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642248000
```

## 🚨 Códigos de Erro

### Códigos HTTP
- `200` - Sucesso
- `201` - Criado com sucesso
- `400` - Requisição inválida
- `401` - Não autorizado
- `403` - Acesso negado
- `404` - Não encontrado
- `409` - Conflito
- `422` - Dados inválidos
- `429` - Rate limit excedido
- `500` - Erro interno do servidor

### Códigos de Erro Customizados
- `VALIDATION_ERROR` - Erro de validação
- `AUTHENTICATION_FAILED` - Falha na autenticação
- `INSUFFICIENT_PERMISSIONS` - Permissões insuficientes
- `RESOURCE_NOT_FOUND` - Recurso não encontrado
- `DUPLICATE_RESOURCE` - Recurso duplicado
- `BUSINESS_RULE_VIOLATION` - Violação de regra de negócio
- `EXTERNAL_SERVICE_ERROR` - Erro em serviço externo

## 🔧 Exemplos de Uso

### JavaScript/TypeScript
```typescript
// Cliente API
class NuvexPOSClient {
  private baseURL = 'https://api.nuvexpos.com';
  private token: string;

  constructor(token: string) {
    this.token = token;
  }

  async request(endpoint: string, options: RequestInit = {}) {
    const response = await fetch(`${this.baseURL}${endpoint}`, {
      ...options,
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json',
        ...options.headers,
      },
    });

    const data = await response.json();
    
    if (!data.success) {
      throw new Error(data.error.message);
    }

    return data.data;
  }

  async getProducts(storeId: string) {
    return this.request(`/api/products?storeId=${storeId}`);
  }

  async createSale(saleData: any) {
    return this.request('/api/sales', {
      method: 'POST',
      body: JSON.stringify(saleData),
    });
  }
}
```

### Python
```python
import requests

class NuvexPOSClient:
    def __init__(self, token):
        self.base_url = 'https://api.nuvexpos.com'
        self.token = token
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {token}',
            'Content-Type': 'application/json'
        })

    def get_products(self, store_id):
        response = self.session.get(f'{self.base_url}/api/products', 
                                  params={'storeId': store_id})
        return response.json()['data']

    def create_sale(self, sale_data):
        response = self.session.post(f'{self.base_url}/api/sales', 
                                   json=sale_data)
        return response.json()['data']
```

## 📚 SDKs Oficiais

### JavaScript/TypeScript
```bash
npm install @nuvexpos/sdk
```

### Python
```bash
pip install nuvexpos-sdk
```

### PHP
```bash
composer require nuvexpos/sdk
```

## 🔍 Debugging

### Headers de Debug
```http
X-Debug-Mode: true
X-Trace-Id: custom-trace-id
```

### Logs de Requisição
Todas as requisições são logadas com:
- Request ID único
- Timestamp
- Método e endpoint
- Status de resposta
- Tempo de processamento