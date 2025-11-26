# 🏗️ Arquitetura do Projeto

## Visão Geral

O Analytics Dashboard - Upay é construído seguindo uma arquitetura moderna, escalável e baseada em componentes, utilizando o Next.js 14 com App Router e TypeScript para garantir type-safety em todo o código.

## Arquitetura em Camadas

### 1. Camada de Apresentação (Presentation Layer)

**Localização:** `src/app/` e `src/components/`

- **Next.js App Router**: Sistema de roteamento baseado em arquivos
- **Server Components**: Componentes renderizados no servidor para melhor performance
- **Client Components**: Componentes interativos com `'use client'`
- **Componentes UI**: Biblioteca baseada em Radix UI e shadcn/ui

**Características:**
- Separação clara entre Server e Client Components
- Componentes reutilizáveis e modulares
- Design system consistente

### 2. Camada de Lógica de Negócio (Business Logic Layer)

**Localização:** `src/hooks/`, `src/requests/`, `src/schemas/`

- **Custom Hooks**: Lógica reutilizável encapsulada
  - `use-profit-per-period.tsx`
  - `useRefundTransactions.ts`
  - `useCompanies.ts`
  - `useRecursiveRequest.ts`

- **Schemas de Validação**: Definições de tipos e estruturas de dados
  - `company.ts`
  - `transaction.ts`
  - `profit.ts`
  - `dateRange.ts`

- **Request Functions**: Funções especializadas para requisições
  - `requests/mongo/` - Queries para MongoDB
  - `requests/hopy/` - Integrações com APIs externas

### 3. Camada de Serviços (Service Layer)

**Localização:** `src/services/`

- **MongoDB Service** (`mongo.ts`): 
  - Singleton pattern para conexão única
  - Gerenciamento de conexão e pool
  - Reutilização de conexões

- **API Services**:
  - `alpaApi.ts` - Cliente Axios para API Alpa
  - `summitApi.ts` - Cliente Axios para API Summit
  - `api.ts` - Factory pattern para seleção dinâmica de API

- **Query Client** (`queryClient.ts`):
  - Configuração do TanStack Query
  - Cache management
  - Retry policies

### 4. Camada de Dados (Data Layer)

**Localização:** `src/requests/mongo/`, `src/data/`

- **MongoDB Collections**: Acesso direto ao banco de dados
- **Data Mappers** (`mappers.ts`): Transformação de dados entre camadas
- **Default Values** (`defaultValues.ts`): Valores padrão do sistema

## Padrões Arquiteturais Utilizados

### 1. Factory Pattern
```typescript
// src/lib/api.ts
export const getApi = () => {
  const selectedApi = cookieStore.get('selectedApi')
  return selectedApi?.value === 'summit' ? summitApi : alpaApi
}
```

### 2. Singleton Pattern
```typescript
// src/services/mongo.ts
class MongoService {
  private static instance: MongoService
  public static getInstance(): MongoService
}
```

### 3. Repository Pattern
- Funções em `requests/mongo/` atuam como repositórios
- Abstração do acesso aos dados
- Facilita testes e manutenção

### 4. Custom Hooks Pattern
- Encapsulamento de lógica de negócio
- Reutilização de código
- Separação de responsabilidades

## Fluxo de Dados

```
┌─────────────────┐
│   Componentes   │
│   (UI Layer)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Custom Hooks   │
│ (Business Logic)│
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Request Funcs  │
│  (Data Access)  │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌──────────┐
│ MongoDB│ │ APIs Ext │
│        │ │ (Alpa/   │
│        │ │ Summit)  │
└────────┘ └──────────┘
```

## Estrutura de Módulos

### Módulo de Faturamento
- **Rota:** `/app/(main)/`
- **Componentes:** Dashboard, gráficos de vendas, análise por método
- **Dados:** Transações, vendas, métodos de pagamento

### Módulo de Reembolsos
- **Rota:** `/app/reembolsos/`
- **Submódulos:**
  - Dashboard geral
  - Análise por empresa
  - Análise por período
  - Dívidas
  - Últimos reembolsos

### Módulo de Empresas
- **Rota:** `/app/empresas/`
- **Funcionalidades:**
  - Dashboard de empresas
  - Listagem e filtros
  - Análise de crescimento
  - Seção de presentes

## Gerenciamento de Estado

### TanStack Query (React Query)
- **Cache automático**: Reduz requisições desnecessárias
- **Sincronização em background**: Dados sempre atualizados
- **Otimistic updates**: UI responsiva
- **Error handling**: Tratamento centralizado de erros

### Context API
- `QueryProviderWrapper`: Provedor global do React Query
- `CurrentAppContext`: Contexto da aplicação atual

## Autenticação e Segurança

### Middleware de Autenticação
- **Localização:** `src/middleware.ts`
- **Proteção de rotas**: Verificação de token via cookies
- **Redirecionamento automático**: Usuários não autenticados

### Cookies
- Armazenamento seguro de tokens
- Seleção de API (Alpa/Summit)
- Sessão do usuário

## Performance e Otimização

### Server-Side Rendering (SSR)
- Páginas renderizadas no servidor
- Melhor SEO e performance inicial

### Code Splitting
- Next.js automaticamente divide o código
- Carregamento sob demanda de rotas

### Image Optimization
- Next.js Image component
- Lazy loading automático

### Caching Strategy
- React Query cache
- MongoDB connection pooling
- HTTP request caching

## Escalabilidade

### Horizontal Scaling
- Stateless architecture
- Pode rodar múltiplas instâncias
- Load balancing ready

### Vertical Scaling
- Connection pooling no MongoDB
- Otimização de queries
- Lazy loading de componentes

## Testabilidade

### Separação de Responsabilidades
- Lógica de negócio isolada em hooks
- Serviços testáveis independentemente
- Componentes puros quando possível

### Type Safety
- TypeScript em todo o projeto
- Schemas de validação
- Interfaces bem definidas

## Tecnologias e Ferramentas

### Frontend
- **Next.js 14**: Framework React
- **TypeScript**: Type safety
- **Tailwind CSS**: Estilização
- **Radix UI**: Componentes acessíveis
- **Recharts**: Visualização de dados

### Backend/Data
- **MongoDB**: Banco de dados NoSQL
- **Axios**: Cliente HTTP
- **TanStack Query**: Gerenciamento de estado

### Dev Tools
- **ESLint**: Linting
- **TypeScript**: Compilação e type checking
- **pnpm**: Gerenciador de pacotes

## Conclusão

A arquitetura do projeto foi projetada para ser:
- ✅ **Modular**: Fácil manutenção e extensão
- ✅ **Escalável**: Suporta crescimento
- ✅ **Performática**: Otimizada para velocidade
- ✅ **Testável**: Código organizado para testes
- ✅ **Type-safe**: TypeScript em todo o projeto
- ✅ **Moderno**: Utiliza as melhores práticas atuais

