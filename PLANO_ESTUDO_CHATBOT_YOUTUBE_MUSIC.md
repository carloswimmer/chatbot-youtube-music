# 🎵 Plano de Estudos: Chatbot YouTube Music com RAG

**Projeto:** Music Bot - Chatbot de Informações Musicais do YouTube Music  
**LLM:** Google Gemini  
**Autor:** Carlos Wimmer  
**GitHub:** https://github.com/carloswimmer  
**Data:** Outubro 2025

---

## 📚 Índice

1. [Introdução e Conceitos Fundamentais](#1-introdução-e-conceitos-fundamentais)
2. [Arquitetura do Sistema](#2-arquitetura-do-sistema)
3. [Setup Inicial do Projeto](#3-setup-inicial-do-projeto)
4. [Banco de Dados com PostgreSQL e pgvector](#4-banco-de-dados-com-postgresql-e-pgvector)
5. [Sistema de Embeddings](#5-sistema-de-embeddings)
6. [API de Ingestão de Dados](#6-api-de-ingestão-de-dados)
7. [API de Chat com RAG](#7-api-de-chat-com-rag)
8. [Interface do Usuário](#8-interface-do-usuário)
9. [Integração com Google Gemini](#9-integração-com-google-gemini)
10. [Deploy e Próximos Passos](#10-deploy-e-próximos-passos)

---

## 1. Introdução e Conceitos Fundamentais

### 1.1 O que você vai aprender?

Este projeto vai te ensinar a construir um **chatbot RAG (Retrieval-Augmented Generation)** do zero. Você vai dominar:

- ✅ **RAG (Retrieval-Augmented Generation)**: Como fazer um LLM responder usando apenas informações específicas
- ✅ **Vector Embeddings**: Como transformar texto em vetores numéricos para busca semântica
- ✅ **Similarity Search**: Como encontrar informações relevantes usando distância de cosseno
- ✅ **Next.js 14 App Router**: Framework React moderno com Server Actions
- ✅ **Drizzle ORM**: ORM TypeScript-first para PostgreSQL
- ✅ **Google Gemini API**: LLM da Google para geração de texto
- ✅ **AI SDK (Vercel)**: Biblioteca para trabalhar com LLMs de forma padronizada
- ✅ **Docker**: Containerização da aplicação e banco de dados

### 1.2 O que é RAG?

**RAG (Retrieval-Augmented Generation)** é uma técnica que combina:

1. **Retrieval (Recuperação)**: Buscar informações relevantes de uma base de conhecimento
2. **Generation (Geração)**: Usar um LLM para gerar respostas baseadas nas informações recuperadas

**Por que usar RAG?**
- ❌ **Sem RAG**: O LLM pode "alucinar" (inventar) informações
- ✅ **Com RAG**: O LLM responde apenas com base em dados verificados

**Exemplo prático:**

```
Usuário: "Qual é o álbum mais recente do Taylor Swift?"

Sem RAG: 
→ LLM pode inventar ou usar dados desatualizados do treinamento

Com RAG:
→ Sistema busca no banco de dados → Encontra "The Tortured Poets Department (2024)"
→ LLM usa essa informação para responder → Resposta precisa e atualizada
```

### 1.3 Como funciona a busca vetorial?

**Vector Embeddings** são representações numéricas de texto:

```
Texto: "música pop animada" 
→ Embedding: [0.234, -0.567, 0.891, ... ] (1536 dimensões)

Texto: "canção alegre pop"
→ Embedding: [0.221, -0.543, 0.876, ... ] (muito similar!)
```

**Busca Semântica:**
- Calcula a **distância de cosseno** entre vetores
- Quanto menor a distância, mais similar o significado
- Threshold típico: 0.5 (50% de similaridade)

### 1.4 Fluxo Completo do Sistema

```
┌─────────────────────────────────────────────────────────────┐
│  FASE 1: INGESTÃO DE DADOS (Executa 1 vez)                  │
└─────────────────────────────────────────────────────────────┘
    
    1. Dados musicais (JSON/TXT)
          ↓
    2. Processa em chunks de texto
          ↓
    3. Gera embeddings com Gemini
          ↓
    4. Salva no PostgreSQL (pgvector)


┌─────────────────────────────────────────────────────────────┐
│  FASE 2: CHAT (Executa a cada pergunta)                     │
└─────────────────────────────────────────────────────────────┘
    
    1. Usuário faz pergunta
          ↓
    2. Gera embedding da pergunta
          ↓
    3. Busca embeddings similares (cosine distance)
          ↓
    4. Retorna top 4 músicas/informações mais relevantes
          ↓
    5. Gemini gera resposta usando o contexto
          ↓
    6. Usuário recebe resposta precisa
```

---

## 2. Arquitetura do Sistema

### 2.1 Stack Tecnológica

| Camada | Tecnologia | Por que usar? |
|--------|-----------|---------------|
| **Frontend** | Next.js 14 + React | Framework full-stack moderno, SEO-friendly |
| **Styling** | TailwindCSS + shadcn-ui | Design system rápido e consistente |
| **Backend API** | Next.js API Routes | APIs integradas no mesmo projeto |
| **Database** | PostgreSQL 16 | Banco relacional robusto |
| **Vector Search** | pgvector | Extensão para busca vetorial no Postgres |
| **ORM** | Drizzle ORM | TypeScript-first, type-safe, performático |
| **LLM** | Google Gemini | API gratuita/acessível da Google |
| **Embeddings** | text-embedding-004 | Modelo de embeddings do Gemini |
| **AI SDK** | Vercel AI SDK | Abstração padronizada para LLMs |
| **Validation** | Zod | Validação de schemas TypeScript |
| **Package Manager** | pnpm | Mais rápido e eficiente que npm |

### 2.2 Estrutura de Pastas

```
music-bot/
├── app/                          # Next.js 14 App Router
│   ├── api/
│   │   ├── chat/
│   │   │   └── route.ts         # API de chat com RAG
│   │   └── populate/
│   │       └── route.ts         # API de ingestão de dados
│   ├── layout.tsx               # Layout principal
│   ├── page.tsx                 # Interface de chat
│   └── globals.css              # Estilos globais
├── components/
│   └── ui/                      # Componentes shadcn-ui
├── lib/
│   ├── ai/
│   │   └── embedding.ts         # Lógica de embeddings e busca
│   ├── db/
│   │   ├── index.ts            # Conexão com database
│   │   ├── migrate.ts          # Script de migração
│   │   └── schema/
│   │       ├── resources.ts    # Schema de recursos (músicas)
│   │       └── embeddings.ts   # Schema de embeddings
│   └── actions/
│       └── resources.ts         # Server Actions para CRUD
├── scripts/
│   └── populate-sample-data.ts  # Script para popular banco
├── docker-compose.yml           # Docker para Postgres
├── drizzle.config.ts           # Configuração do Drizzle
├── package.json
├── tsconfig.json
└── .env                        # Variáveis de ambiente
```

### 2.3 Esquema do Banco de Dados

```sql
-- Tabela de recursos (informações sobre músicas)
CREATE TABLE resources (
  id VARCHAR(191) PRIMARY KEY,
  content TEXT NOT NULL,              -- Texto completo sobre a música
  song_id VARCHAR(191) NOT NULL,      -- ID da música no YouTube Music
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Tabela de embeddings (vetores para busca)
CREATE TABLE embeddings (
  id VARCHAR(191) PRIMARY KEY,
  resource_id VARCHAR(191) REFERENCES resources(id) ON DELETE CASCADE,
  embedding VECTOR(768) NOT NULL,     -- Vetor de 768 dimensões (Gemini)
);

-- Índice HNSW para busca vetorial rápida
CREATE INDEX embedding_index ON embeddings 
  USING hnsw (embedding vector_cosine_ops);
```

**Por que duas tabelas?**
- `resources`: Armazena dados originais (fácil de ler/editar)
- `embeddings`: Armazena vetores (otimizado para busca)
- Separação permite gerenciar dados e vetores independentemente

---

## 3. Setup Inicial do Projeto

### 3.1 Pré-requisitos

```bash
# Verificar versões instaladas
node --version    # >= 18.17.0
pnpm --version    # >= 8.0.0
docker --version  # >= 20.10.0
```

### 3.2 Criar Projeto Next.js

```bash
# 1. Criar diretório do projeto
mkdir music-bot
cd music-bot

# 2. Inicializar Next.js com TypeScript
pnpm create next-app@latest . --typescript --tailwind --app --src-dir=false

# Escolher as opções:
# ✔ Would you like to use TypeScript? → Yes
# ✔ Would you like to use ESLint? → Yes
# ✔ Would you like to use Tailwind CSS? → Yes
# ✔ Would you like to use `src/` directory? → No
# ✔ Would you like to use App Router? → Yes
# ✔ Would you like to customize the default import alias? → No
```

### 3.3 Instalar Dependências

```bash
# AI e LLM
pnpm add ai @ai-sdk/google

# Database e ORM
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit

# Validação
pnpm add zod drizzle-zod

# UI Components
pnpm add @radix-ui/react-slot @radix-ui/react-avatar
pnpm add class-variance-authority clsx tailwind-merge lucide-react

# Utilities
pnpm add nanoid

# Dev tools
pnpm add -D tsx dotenv @types/pg
```

**Por que cada biblioteca?**

| Biblioteca | Propósito |
|-----------|-----------|
| `ai` | Vercel AI SDK - abstração para LLMs |
| `@ai-sdk/google` | Provider do Gemini para AI SDK |
| `drizzle-orm` | ORM TypeScript-first |
| `postgres` | Driver PostgreSQL para Node.js |
| `zod` | Validação de schemas |
| `nanoid` | Gerador de IDs únicos |
| `lucide-react` | Ícones SVG |

### 3.4 Configurar shadcn-ui

```bash
# Instalar CLI do shadcn-ui
pnpm dlx shadcn-ui@latest init

# Configurações:
# ✔ Which style would you like to use? → Default
# ✔ Which color would you like to use as base color? → Slate
# ✔ Would you like to use CSS variables for colors? → Yes

# Adicionar componentes necessários
pnpm dlx shadcn-ui@latest add button
pnpm dlx shadcn-ui@latest add input
pnpm dlx shadcn-ui@latest add avatar
```

### 3.5 Configurar Variáveis de Ambiente

Criar arquivo `.env.local`:

```bash
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/musicbot

# Google Gemini API
GOOGLE_GENERATIVE_AI_API_KEY=sua-chave-aqui

# Next.js
NODE_ENV=development
```

**Como obter a API Key do Gemini:**

1. Acesse: https://makersuite.google.com/app/apikey
2. Clique em "Create API Key"
3. Copie a chave gerada
4. Cole no `.env.local`

### 3.6 Configurar TypeScript

Atualizar `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

## 4. Banco de Dados com PostgreSQL e pgvector

### 4.1 Por que PostgreSQL + pgvector?

**PostgreSQL:**
- ✅ Open-source e maduro (30+ anos)
- ✅ Suporta JSON, arrays, tipos customizados
- ✅ ACID compliant (transações confiáveis)
- ✅ Excelente performance

**pgvector:**
- ✅ Extensão que adiciona suporte a vetores
- ✅ Índice HNSW (Hierarchical Navigable Small World) para busca rápida
- ✅ Operações de similaridade otimizadas
- ✅ Integração nativa com PostgreSQL

**Alternativas consideradas:**
- Pinecone: Pago, vendor lock-in
- Weaviate: Complexo para começar
- ChromaDB: Menos maduro
- ✅ **pgvector**: Simples, open-source, integrado

### 4.2 Setup Docker para PostgreSQL

Criar `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: pgvector/pgvector:pg16
    container_name: musicbot-db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: musicbot
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

Criar `init-db.sql`:

```sql
-- Habilitar extensão pgvector
CREATE EXTENSION IF NOT EXISTS vector;
```

**Iniciar o banco:**

```bash
docker-compose up -d

# Verificar se está rodando
docker-compose ps
docker-compose logs postgres
```

### 4.3 Configurar Drizzle ORM

Criar `drizzle.config.ts`:

```typescript
import type { Config } from 'drizzle-kit';
import * as dotenv from 'dotenv';

dotenv.config({ path: '.env.local' });

export default {
  schema: './lib/db/schema/*',
  out: './lib/db/migrations',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

**Por que Drizzle?**
- ✅ TypeScript-first (type safety completo)
- ✅ SQL-like syntax (fácil migração de SQL raw)
- ✅ Migrations automáticas
- ✅ Zero runtime overhead
- ✅ Suporte nativo a pgvector

### 4.4 Criar Schemas do Banco

Criar `lib/db/schema/resources.ts`:

```typescript
import { pgTable, varchar, text, timestamp } from 'drizzle-orm/pg-core';
import { createSelectSchema } from 'drizzle-zod';
import { sql } from 'drizzle-orm';
import { z } from 'zod';
import { nanoid } from 'nanoid';

export const resources = pgTable('resources', {
  id: varchar('id', { length: 191 })
    .primaryKey()
    .$defaultFn(() => nanoid()),
  content: text('content').notNull(),
  song_id: varchar('song_id', { length: 191 }).notNull(),
  created_at: timestamp('created_at').notNull().default(sql`now()`),
  updated_at: timestamp('updated_at').notNull().default(sql`now()`),
});

// Schema de validação para inserção
export const insertResourceSchema = createSelectSchema(resources)
  .omit({
    id: true,
    created_at: true,
    updated_at: true,
  });

export type NewResourceParams = z.infer<typeof insertResourceSchema>;
```

**Explicação do código:**

- `pgTable`: Define tabela no Drizzle
- `nanoid()`: Gera IDs únicos e curtos (melhor que UUID)
- `createSelectSchema`: Gera validação Zod automaticamente
- `omit`: Remove campos auto-gerados do schema de inserção

Criar `lib/db/schema/embeddings.ts`:

```typescript
import { pgTable, varchar, vector, index } from 'drizzle-orm/pg-core';
import { nanoid } from 'nanoid';
import { resources } from './resources';

export const embeddings = pgTable(
  'embeddings',
  {
    id: varchar('id', { length: 191 })
      .primaryKey()
      .$defaultFn(() => nanoid()),
    resource_id: varchar('resource_id', { length: 191 })
      .references(() => resources.id, { onDelete: 'cascade' }),
    embedding: vector('embedding', { dimensions: 768 }).notNull(),
  },
  (table) => ({
    embeddingIndex: index('embedding_index').using(
      'hnsw',
      table.embedding.op('vector_cosine_ops')
    ),
  })
);
```

**Detalhes importantes:**

- `vector('embedding', { dimensions: 768 })`: 
  - Gemini text-embedding-004 gera vetores de 768 dimensões
  - OpenAI usa 1536 dimensões (por isso a diferença)
- `references(..., { onDelete: 'cascade' })`:
  - Se deletar uma música, deleta o embedding também
- `hnsw`: Algoritmo de índice para busca vetorial rápida
- `vector_cosine_ops`: Operador para calcular distância de cosseno

### 4.5 Criar Conexão com Database

Criar `lib/db/index.ts`:

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema/resources';
import * as embeddingsSchema from './schema/embeddings';

if (!process.env.DATABASE_URL) {
  throw new Error('DATABASE_URL environment variable is not set');
}

// Conexão para queries
const connectionString = process.env.DATABASE_URL;
const client = postgres(connectionString, { prepare: false });

export const db = drizzle(client, {
  schema: { ...schema, ...embeddingsSchema },
});
```

**Por que `prepare: false`?**
- Evita problemas com prepared statements em serverless
- Necessário para Vercel/Next.js

### 4.6 Criar Script de Migração

Criar `lib/db/migrate.ts`:

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import { migrate } from 'drizzle-orm/postgres-js/migrator';
import postgres from 'postgres';
import * as dotenv from 'dotenv';

dotenv.config({ path: '.env.local' });

const runMigrate = async () => {
  if (!process.env.DATABASE_URL) {
    throw new Error('DATABASE_URL is not defined');
  }

  const connection = postgres(process.env.DATABASE_URL, { max: 1 });
  const db = drizzle(connection);

  console.log('⏳ Running migrations...');

  const start = Date.now();
  await migrate(db, { migrationsFolder: './lib/db/migrations' });
  const end = Date.now();

  console.log(`✅ Migrations completed in ${end - start}ms`);

  await connection.end();
  process.exit(0);
};

runMigrate().catch((err) => {
  console.error('❌ Migration failed');
  console.error(err);
  process.exit(1);
});
```

### 4.7 Adicionar Scripts no package.json

```json
{
  "scripts": {
    "dev": "next dev -p 3000",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "tsx lib/db/migrate.ts",
    "db:push": "drizzle-kit push",
    "db:studio": "drizzle-kit studio",
    "populate": "tsx scripts/populate-sample-data.ts"
  }
}
```

### 4.8 Executar Migrações

```bash
# 1. Gerar arquivos de migração
pnpm db:generate

# 2. Aplicar migrações no banco
pnpm db:migrate

# 3. (Opcional) Visualizar banco com Drizzle Studio
pnpm db:studio
# Acesse: https://local.drizzle.studio
```

---

## 5. Sistema de Embeddings

### 5.1 O que são Embeddings?

**Embeddings** são representações vetoriais de texto que capturam significado semântico.

```
Exemplo:

Texto 1: "música alegre e animada"
Embedding 1: [0.8, 0.3, -0.5, 0.2, ...]

Texto 2: "canção feliz e energética"  
Embedding 2: [0.75, 0.28, -0.48, 0.19, ...]
                ↑ Muito similar!

Texto 3: "filme de terror assustador"
Embedding 3: [-0.6, -0.9, 0.7, -0.4, ...]
                ↑ Muito diferente!
```

**Como funciona:**
1. Modelo treinado em bilhões de textos
2. Aprende relações semânticas entre palavras
3. Projeta texto em espaço vetorial de alta dimensão
4. Textos similares ficam próximos no espaço vetorial

### 5.2 Gemini Embeddings vs OpenAI

| Característica | Gemini (text-embedding-004) | OpenAI (text-embedding-ada-002) |
|----------------|----------------------------|----------------------------------|
| **Dimensões** | 768 | 1536 |
| **Custo** | Grátis até 1500 req/dia | $0.10 / 1M tokens |
| **Performance** | Excelente | Excelente |
| **Rate Limit** | 1500 req/min | 3000 req/min |
| **Idiomas** | Multilíngue | Multilíngue |

### 5.3 Implementar Sistema de Embeddings

Criar `lib/ai/embedding.ts`:

```typescript
import { google } from '@ai-sdk/google';
import { embed, embedMany } from 'ai';
import { cosineDistance, desc, gt, sql } from 'drizzle-orm';
import { db } from '../db';
import { embeddings } from '../db/schema/embeddings';
import { resources } from '../db/schema/resources';

// Modelo de embeddings do Gemini
const embeddingModel = google.textEmbeddingModel('text-embedding-004');

/**
 * Gera embedding para um único texto
 * @param value - Texto para gerar embedding
 * @returns Array de números representando o embedding (768 dimensões)
 */
export const generateEmbedding = async (value: string): Promise<number[]> => {
  // Limpar texto (remover quebras de linha e espaços extras)
  const input = value.replaceAll('\\n', ' ').trim();

  // Validar entrada
  if (!input || input.length === 0) {
    throw new Error('Cannot generate embedding for empty input');
  }

  try {
    const { embedding } = await embed({
      model: embeddingModel,
      value: input,
    });

    return embedding;
  } catch (error) {
    console.error('Failed to generate embedding:', error);
    throw error;
  }
};

/**
 * Gera embeddings para múltiplos textos (mais eficiente que gerar um por vez)
 * @param values - Array de textos
 * @returns Array de embeddings
 */
export const generateEmbeddings = async (
  values: string[]
): Promise<number[][]> => {
  // Limpar todos os textos
  const inputs = values.map((v) => v.replaceAll('\\n', ' ').trim());

  // Validar entradas
  if (inputs.some((input) => !input || input.length === 0)) {
    throw new Error('Cannot generate embeddings for empty inputs');
  }

  try {
    const { embeddings } = await embedMany({
      model: embeddingModel,
      values: inputs,
    });

    return embeddings;
  } catch (error) {
    console.error('Failed to generate embeddings:', error);
    throw error;
  }
};

/**
 * Interface para resultados de busca
 */
interface SimilarityResult {
  content: string;
  similarity: number;
  song_id: string;
}

interface SearchResults {
  results: SimilarityResult[];
  idealResponse: SimilarityResult | null;
  highestSimilarity: number;
}

/**
 * Busca conteúdo relevante usando similaridade de embeddings
 * @param userQuery - Pergunta do usuário
 * @param limit - Número máximo de resultados (padrão: 4)
 * @param threshold - Threshold mínimo de similaridade (padrão: 0.5)
 * @returns Resultados ordenados por similaridade
 */
export const findRelevantContent = async ({
  userQuery,
  limit = 4,
  threshold = 0.5,
}: {
  userQuery: string;
  limit?: number;
  threshold?: number;
}): Promise<SearchResults> => {
  try {
    // 1. Gerar embedding da pergunta do usuário
    const userQueryEmbedding = await generateEmbedding(userQuery);

    // 2. Calcular similaridade usando distância de cosseno
    // Fórmula: similarity = 1 - cosine_distance
    // Resultado: 0 (totalmente diferente) a 1 (idêntico)
    const similarity = sql<number>`1 - (${cosineDistance(
      embeddings.embedding,
      userQueryEmbedding
    )})`;

    // 3. Buscar documentos similares
    const similarResults = await db
      .select({
        content: resources.content,
        similarity,
        song_id: resources.song_id,
      })
      .from(embeddings)
      .innerJoin(resources, sql`${embeddings.resource_id} = ${resources.id}`)
      .where(gt(similarity, threshold))
      .orderBy(desc(similarity))
      .limit(limit);

    // 4. Processar resultados
    if (similarResults.length === 0) {
      console.log('❌ No results above threshold');
      return {
        results: [],
        idealResponse: null,
        highestSimilarity: 0,
      };
    }

    // 5. O primeiro resultado é o mais relevante (maior similaridade)
    const idealResponse = {
      content: similarResults[0].content,
      similarity: similarResults[0].similarity,
      song_id: similarResults[0].song_id,
    };

    console.log(`✅ Found ${similarResults.length} similar results`);
    console.log(`📊 Highest similarity: ${idealResponse.similarity.toFixed(3)}`);

    return {
      results: similarResults,
      idealResponse,
      highestSimilarity: idealResponse.similarity,
    };
  } catch (error) {
    console.error('❌ Error finding relevant content:', error);
    return {
      results: [],
      idealResponse: null,
      highestSimilarity: 0,
    };
  }
};
```

**Conceitos-chave explicados:**

1. **Cosine Distance:**
   ```
   Distância = 1 - (A · B) / (||A|| × ||B||)
   
   Onde:
   - A · B = produto escalar dos vetores
   - ||A||, ||B|| = normas (magnitudes) dos vetores
   
   Resultado:
   - 0 = vetores idênticos (mesma direção)
   - 1 = vetores opostos
   ```

2. **Similarity Score:**
   ```
   Similarity = 1 - Cosine Distance
   
   Interpretação:
   - 0.9 - 1.0 = Extremamente similar (quase duplicata)
   - 0.7 - 0.9 = Muito similar (mesmo tópico)
   - 0.5 - 0.7 = Similar (relacionado)
   - < 0.5    = Pouco relacionado (ignorar)
   ```

3. **Threshold (0.5):**
   - Filtra resultados muito diferentes
   - Evita respostas irrelevantes
   - Pode ser ajustado conforme necessidade

4. **Batch Processing:**
   - `generateEmbeddings()` processa múltiplos textos de uma vez
   - Mais eficiente que chamar `generateEmbedding()` em loop
   - Reduz chamadas à API

---

## 6. API de Ingestão de Dados

### 6.1 Por que precisamos de ingestão?

O chatbot só pode responder sobre informações que estão no banco de dados. A **API de ingestão** permite:

1. ✅ Adicionar novas músicas ao sistema
2. ✅ Atualizar informações existentes
3. ✅ Processar dados em lote (bulk insert)
4. ✅ Gerar embeddings automaticamente

**Fluxo de ingestão:**

```
Dados de entrada (JSON) 
  ↓
Validação (Zod)
  ↓
Salvar na tabela 'resources'
  ↓
Gerar embedding do conteúdo
  ↓
Salvar na tabela 'embeddings'
  ↓
Retornar confirmação
```

### 6.2 Criar Server Actions

Criar `lib/actions/resources.ts`:

```typescript
'use server';

import { db } from '@/lib/db';
import {
  resources,
  insertResourceSchema,
  type NewResourceParams,
} from '@/lib/db/schema/resources';
import { embeddings as embeddingsTable } from '@/lib/db/schema/embeddings';
import { generateEmbedding } from '@/lib/ai/embedding';

/**
 * Cria um novo recurso (música) e gera seu embedding
 * @param input - Dados da música (content, song_id)
 * @returns Mensagem de sucesso ou erro
 */
export const createResource = async (input: NewResourceParams) => {
  try {
    // 1. Validar input com Zod
    const { content, song_id } = insertResourceSchema.parse(input);

    // 2. Inserir na tabela resources
    const [resource] = await db
      .insert(resources)
      .values({ content, song_id })
      .returning();

    console.log(`✅ Resource created: ${resource.id}`);

    // 3. Gerar embedding do conteúdo
    console.log('⏳ Generating embedding...');
    const embedding = await generateEmbedding(content);

    // 4. Salvar embedding na tabela embeddings
    await db.insert(embeddingsTable).values({
      resource_id: resource.id,
      embedding: embedding,
    });

    console.log(`✅ Embedding created for resource ${resource.id}`);

    return 'Resource successfully created and embedded.';
  } catch (error) {
    console.error('❌ Error creating resource:', error);
    return error instanceof Error && error.message.length > 0
      ? error.message
      : 'Error, please try again.';
  }
};

/**
 * Deleta todos os recursos do banco (usar com cuidado!)
 * @returns Mensagem de confirmação
 */
export const truncateResources = async () => {
  try {
    await db.delete(resources);
    console.log('✅ Resources table truncated');
    return 'Resources table successfully truncated.';
  } catch (error) {
    console.error('❌ Error truncating resources:', error);
    return error instanceof Error && error.message.length > 0
      ? error.message
      : 'Error truncating resources table, please try again.';
  }
};
```

**Por que Server Actions?**
- ✅ Executam no servidor (acesso seguro ao banco)
- ✅ TypeScript end-to-end
- ✅ Não precisam de API routes separadas
- ✅ Marcador `'use server'` indica execução no servidor

### 6.3 Criar API Route de Populate

Criar `app/api/populate/route.ts`:

```typescript
import { createResource, truncateResources } from '@/lib/actions/resources';
import { insertResourceSchema } from '@/lib/db/schema/resources';
import { z } from 'zod';

// Schema para validar requisição com múltiplos recursos
const populateSchema = z.object({
  resources: z
    .array(insertResourceSchema)
    .min(1, 'At least one resource is required'),
});

/**
 * POST /api/populate
 * Adiciona múltiplas músicas ao banco de dados
 */
export async function POST(req: Request) {
  try {
    // 1. Parse do body
    const body = await req.json();

    // 2. Validar com Zod
    const { resources } = populateSchema.parse(body);

    // 3. Processar cada recurso
    const results = [];

    for (const resource of resources) {
      try {
        const result = await createResource(resource);
        results.push({
          content: resource.content.substring(0, 50) + '...', // Mostrar apenas início
          status: 'success',
          message: result,
        });
      } catch (error) {
        results.push({
          content: resource.content.substring(0, 50) + '...',
          status: 'error',
          message: error instanceof Error ? error.message : 'Unknown error',
        });
      }
    }

    // 4. Calcular estatísticas
    const successCount = results.filter((r) => r.status === 'success').length;
    const errorCount = results.filter((r) => r.status === 'error').length;

    // 5. Retornar resposta
    return Response.json({
      message: `Processed ${resources.length} resources: ${successCount} successful, ${errorCount} failed`,
      results,
      summary: {
        total: resources.length,
        successful: successCount,
        failed: errorCount,
      },
    });
  } catch (error) {
    console.error('Populate endpoint error:', error);

    // Tratamento de erros de validação Zod
    if (error instanceof z.ZodError) {
      return Response.json(
        {
          error: 'Validation error',
          details: error.errors,
        },
        { status: 400 }
      );
    }

    // Outros erros
    return Response.json(
      {
        error: 'Internal server error',
        message: error instanceof Error ? error.message : 'Unknown error',
      },
      { status: 500 }
    );
  }
}

/**
 * DELETE /api/populate
 * Remove todas as músicas do banco de dados
 */
export async function DELETE() {
  try {
    const result = await truncateResources();

    return Response.json({
      message: result,
      status: 'success',
    });
  } catch (error) {
    console.error('Delete resources error:', error);

    return Response.json(
      {
        error: 'Failed to truncate resources table',
        message: error instanceof Error ? error.message : 'Unknown error',
      },
      { status: 500 }
    );
  }
}
```

### 6.4 Criar Dados de Exemplo

Criar `scripts/populate-sample-data.ts`:

```typescript
import * as dotenv from 'dotenv';

dotenv.config({ path: '.env.local' });

const sampleSongs = [
  {
    content: `A música "Bohemian Rhapsody" é interpretada por Queen e foi lançada em 1975 no álbum "A Night at the Opera". 
              Esta obra-prima do rock progressivo tem duração de 5:55 minutos e foi composta por Freddie Mercury. 
              A música é conhecida por sua estrutura incomum, combinando elementos de balada, ópera e hard rock. 
              Considerada uma das maiores músicas de todos os tempos, alcançou o topo das paradas em diversos países 
              e o videoclipe é considerado um dos primeiros music videos da história.`,
    song_id: 'queen-bohemian-rhapsody',
  },
  {
    content: `"Imagine" de John Lennon foi lançada em 1971 no álbum de mesmo nome. 
              Esta canção icônica de 3:01 minutos transmite uma mensagem de paz e união mundial. 
              Com melodia simples ao piano e letra profunda, Imagine tornou-se um hino pacifista. 
              A música alcançou o 3º lugar nas paradas americanas e continua sendo uma das 
              composições mais influentes de Lennon após os Beatles.`,
    song_id: 'john-lennon-imagine',
  },
  {
    content: `"Hotel California" é uma música da banda Eagles, lançada em 1977 no álbum de mesmo nome. 
              Com 6:30 minutos de duração, a música é famosa por seu solo de guitarra duplo icônico 
              executado por Don Felder e Joe Walsh. A letra misteriosa fala sobre um hotel misterioso 
              na Califórnia e pode ser interpretada como uma crítica ao estilo de vida hedonista 
              de Los Angeles. Ganhou o Grammy de Gravação do Ano em 1978.`,
    song_id: 'eagles-hotel-california',
  },
  {
    content: `"Billie Jean" de Michael Jackson foi lançada em 1983 como parte do álbum "Thriller". 
              A música tem 4:54 minutos e foi escrita e composta pelo próprio Michael Jackson. 
              Conhecida por sua linha de baixo marcante e o característico "hiccup" vocal de Jackson, 
              Billie Jean permaneceu 7 semanas no topo da Billboard Hot 100. 
              O videoclipe foi um dos primeiros de um artista negro a receber rotação pesada na MTV.`,
    song_id: 'michael-jackson-billie-jean',
  },
  {
    content: `"Smells Like Teen Spirit" do Nirvana foi lançada em 1991 no álbum "Nevermind". 
              Com 5:01 minutos de duração, esta música de grunge marcou uma geração e é considerada 
              o hino do movimento grunge. Composta por Kurt Cobain, a música combina guitarras 
              distorcidas com uma melodia cativante. Apesar de Cobain não gostar da atenção 
              que a música recebeu, ela catapultou o Nirvana para o estrelato mundial.`,
    song_id: 'nirvana-smells-like-teen-spirit',
  },
  {
    content: `"Like a Prayer" de Madonna foi lançada em 1989 como single principal do álbum de mesmo nome. 
              A música de 5:43 minutos mistura pop com elementos gospel, apresentando um coro de igreja. 
              Produzida por Madonna e Patrick Leonard, a canção gerou polêmica devido ao seu videoclipe 
              que misturava temas religiosos com sexualidade. Mesmo assim, tornou-se um dos maiores 
              sucessos da carreira de Madonna, alcançando o topo das paradas em diversos países.`,
    song_id: 'madonna-like-a-prayer',
  },
];

async function populateDatabase() {
  try {
    console.log('🎵 Starting to populate database with sample songs...\n');

    const response = await fetch('http://localhost:3000/api/populate', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        resources: sampleSongs,
      }),
    });

    const result = await response.json();

    console.log('\n📊 Results:');
    console.log(JSON.stringify(result, null, 2));

    if (result.summary) {
      console.log('\n✅ Summary:');
      console.log(`   Total: ${result.summary.total}`);
      console.log(`   Successful: ${result.summary.successful}`);
      console.log(`   Failed: ${result.summary.failed}`);
    }
  } catch (error) {
    console.error('❌ Error populating database:', error);
    process.exit(1);
  }
}

populateDatabase();
```

**Como usar:**

```bash
# 1. Certifique-se que o servidor Next.js está rodando
pnpm dev

# 2. Em outro terminal, execute o script
pnpm populate
```

### 6.5 Testar a API com cURL

```bash
# Adicionar uma música
curl -X POST http://localhost:3000/api/populate \
  -H "Content-Type: application/json" \
  -d '{
    "resources": [{
      "content": "Teste de música pop animada com batida eletrônica",
      "song_id": "test-song-001"
    }]
  }'

# Limpar todas as músicas (cuidado!)
curl -X DELETE http://localhost:3000/api/populate
```

---

## 7. API de Chat com RAG

### 7.1 Fluxo Completo do Chat

```
┌─────────────────────────────────────────────────────────┐
│  1. Usuário digita: "Me fale sobre músicas do Queen"    │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  2. Frontend envia para: POST /api/chat                 │
│     Body: { messages: [...] }                           │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  3. Backend gera embedding da pergunta                  │
│     [0.234, -0.567, 0.891, ...]                         │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  4. Busca no PostgreSQL (similaridade > 0.5)            │
│     Retorna: Top 4 músicas mais similares               │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  5. Monta prompt com contexto:                          │
│     System: "Você é um assistente musical..."           │
│     Context: [Informações das 4 músicas encontradas]    │
│     User: "Me fale sobre músicas do Queen"              │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  6. Envia para Gemini (streaming)                       │
│     Modelo: gemini-1.5-flash                            │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  7. Resposta em streaming para o frontend               │
│     "Tenho informações sobre Bohemian Rhapsody..."      │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Implementar API de Chat

Criar `app/api/chat/route.ts`:

```typescript
import { findRelevantContent } from '@/lib/ai/embedding';
import { google } from '@ai-sdk/google';
import { streamText, tool } from 'ai';
import { z } from 'zod';

// Permitir streaming por até 30 segundos
export const maxDuration = 30;

// Modelo Gemini para geração de texto
const model = google('gemini-1.5-flash-latest');

export async function POST(req: Request) {
  try {
    // 1. Parse do body
    const { messages } = await req.json();

    // 2. Extrair última mensagem do usuário
    const latestMessage = messages[messages.length - 1];
    const userQuery = latestMessage?.content || '';

    console.log(`\n🔍 User query: "${userQuery}"\n`);

    // 3. Buscar conteúdo relevante (RAG)
    const { results, highestSimilarity } = await findRelevantContent({
      userQuery,
      limit: 4,
      threshold: 0.5,
    });

    console.log(`📊 Found ${results.length} relevant songs`);
    console.log(`📈 Highest similarity: ${highestSimilarity.toFixed(3)}\n`);

    // 4. Preparar contexto para o LLM
    let context = '';
    if (results.length > 0) {
      context = results
        .map((r, i) => `[${i + 1}] ${r.content}`)
        .join('\n\n');
    }

    // 5. Tool para buscar informações (usado pelo Gemini quando necessário)
    const getInformation = tool({
      description:
        'Busca informações sobre músicas na base de conhecimento para responder perguntas.',
      parameters: z.object({
        question: z.string().describe('A pergunta do usuário'),
      }),
      execute: async ({ question }) => {
        const searchResults = await findRelevantContent({
          userQuery: question,
          limit: 4,
        });
        return searchResults.results.map((r) => r.content).join('\n\n');
      },
    });

    // 6. Gerar resposta com streaming
    const response = streamText({
      model,
      messages,
      system: `Você é um assistente musical especializado que ajuda usuários a descobrir informações sobre músicas.

REGRAS IMPORTANTES:
1. Responda APENAS com base nas informações fornecidas no contexto
2. Se não houver informações relevantes, diga: "Desculpe, não tenho informações sobre isso na minha base de dados."
3. Seja preciso e cite detalhes específicos (artista, ano, álbum, duração)
4. Não invente ou "aluci ne" informações
5. Use um tom amigável e conversacional
6. Se o usuário fizer perguntas fora do escopo de música, redirecione gentilmente

${context ? `\nCONTEXTO DISPONÍVEL:\n${context}` : '\nNENHUM CONTEXTO RELEVANTE ENCONTRADO.'}`,

      // Tools disponíveis para o Gemini
      tools: {
        getInformation,
      },

      // Configurações de geração
      temperature: 0.7, // Criatividade moderada
      maxTokens: 500, // Respostas concisas
    });

    // 7. Retornar stream
    return response.toDataStreamResponse();
  } catch (error) {
    console.error('❌ Chat API Error:', error);

    return Response.json(
      {
        error: 'Internal server error',
        message: error instanceof Error ? error.message : 'Unknown error',
      },
      { status: 500 }
    );
  }
}
```

**Conceitos importantes:**

1. **Streaming Response:**
   ```typescript
   streamText({ ... }).toDataStreamResponse()
   ```
   - Resposta é enviada em chunks (pedaços)
   - Usuário vê texto aparecer em tempo real
   - Melhor experiência de usuário

2. **Tools (Function Calling):**
   - Gemini pode decidir usar a tool `getInformation`
   - Útil para buscas iterativas ou refinamento
   - Aumenta autonomia do assistente

3. **Temperature (0.7):**
   - 0.0 = Determinístico (sempre mesma resposta)
   - 1.0 = Muito criativo (pode ser impreciso)
   - 0.7 = Equilíbrio entre precisão e naturalidade

4. **System Prompt:**
   - Define comportamento do assistente
   - Estabelece regras e limitações
   - Injeta contexto relevante

### 7.3 Adicionar Tratamento de Erros

Criar `lib/api/errors/error-handler.ts`:

```typescript
export type ErrorType =
  | 'API Error'
  | 'Network Error'
  | 'Validation Error'
  | 'Not Found'
  | 'Internal Error';

export type ErrorCode =
  | 'GEMINI_API_ERROR'
  | 'EMBEDDING_ERROR'
  | 'DATABASE_ERROR'
  | 'VALIDATION_ERROR'
  | 'NETWORK_ERROR'
  | 'UNKNOWN_ERROR';

export interface ErrorInfo {
  errorType: ErrorType;
  errorCode: ErrorCode;
  message: string;
}

/**
 * Classifica erros e retorna informações estruturadas
 */
export function classifyError(error: any): ErrorInfo {
  // Erro de API do Gemini
  if (error.status === 429) {
    return {
      errorType: 'API Error',
      errorCode: 'GEMINI_API_ERROR',
      message: 'Rate limit excedido. Tente novamente em alguns segundos.',
    };
  }

  if (error.status === 401 || error.status === 403) {
    return {
      errorType: 'API Error',
      errorCode: 'GEMINI_API_ERROR',
      message: 'Erro de autenticação com Gemini API. Verifique sua API key.',
    };
  }

  // Erro de rede
  if (error.name === 'FetchError' || error.code === 'ENOTFOUND') {
    return {
      errorType: 'Network Error',
      errorCode: 'NETWORK_ERROR',
      message: 'Erro de conexão. Verifique sua internet.',
    };
  }

  // Erro de validação (Zod)
  if (error.name === 'ZodError') {
    return {
      errorType: 'Validation Error',
      errorCode: 'VALIDATION_ERROR',
      message: 'Dados inválidos fornecidos.',
    };
  }

  // Erro de banco de dados
  if (error.code?.startsWith('23')) {
    return {
      errorType: 'Internal Error',
      errorCode: 'DATABASE_ERROR',
      message: 'Erro ao acessar banco de dados.',
    };
  }

  // Erro genérico
  return {
    errorType: 'Internal Error',
    errorCode: 'UNKNOWN_ERROR',
    message: error.message || 'Erro desconhecido.',
  };
}
```

Atualizar `app/api/chat/route.ts` para usar tratamento de erros:

```typescript
import { classifyError } from '@/lib/api/errors/error-handler';

export async function POST(req: Request) {
  try {
    // ... código existente
  } catch (error: any) {
    console.error('❌ Chat API Error:', error);

    const { errorType, errorCode, message } = classifyError(error);

    return Response.json(
      {
        error: errorType,
        code: errorCode,
        message: message,
      },
      { status: error.status || 500 }
    );
  }
}
```

---

## 8. Interface do Usuário

### 8.1 Componente de Chat

Criar `app/page.tsx`:

```typescript
'use client';

import { useChat } from 'ai/react';
import { Send } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Avatar, AvatarFallback } from '@/components/ui/avatar';

export default function ChatPage() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } =
    useChat({
      api: '/api/chat',
    });

  return (
    <div className="flex flex-col h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900">
      {/* Header */}
      <header className="bg-black/30 backdrop-blur-sm border-b border-white/10 p-4">
        <div className="max-w-4xl mx-auto">
          <h1 className="text-2xl font-bold text-white flex items-center gap-2">
            🎵 Music Bot
            <span className="text-sm font-normal text-purple-300">
              Powered by Gemini
            </span>
          </h1>
          <p className="text-sm text-gray-300 mt-1">
            Pergunte sobre suas músicas favoritas!
          </p>
        </div>
      </header>

      {/* Messages */}
      <div className="flex-1 overflow-y-auto p-4">
        <div className="max-w-4xl mx-auto space-y-4">
          {messages.length === 0 && (
            <div className="text-center py-12">
              <div className="text-6xl mb-4">🎸</div>
              <h2 className="text-2xl font-bold text-white mb-2">
                Bem-vindo ao Music Bot!
              </h2>
              <p className="text-gray-300">
                Faça perguntas sobre músicas e artistas
              </p>
              <div className="mt-6 grid grid-cols-1 md:grid-cols-2 gap-3 max-w-2xl mx-auto">
                {[
                  'Me fale sobre Bohemian Rhapsody',
                  'Quais músicas do Queen você conhece?',
                  'O que você sabe sobre John Lennon?',
                  'Músicas com solo de guitarra marcante',
                ].map((suggestion) => (
                  <button
                    key={suggestion}
                    onClick={() => {
                      handleInputChange({
                        target: { value: suggestion },
                      } as any);
                    }}
                    className="p-3 bg-white/5 hover:bg-white/10 border border-white/10 rounded-lg text-left text-sm text-gray-200 transition-colors"
                  >
                    {suggestion}
                  </button>
                ))}
              </div>
            </div>
          )}

          {messages.map((message) => (
            <div
              key={message.id}
              className={`flex gap-3 ${
                message.role === 'user' ? 'justify-end' : 'justify-start'
              }`}
            >
              {message.role === 'assistant' && (
                <Avatar className="w-8 h-8 bg-purple-600">
                  <AvatarFallback className="text-white">🎵</AvatarFallback>
                </Avatar>
              )}

              <div
                className={`max-w-[80%] rounded-2xl px-4 py-3 ${
                  message.role === 'user'
                    ? 'bg-purple-600 text-white'
                    : 'bg-white/10 text-gray-100 backdrop-blur-sm'
                }`}
              >
                <p className="whitespace-pre-wrap">{message.content}</p>
              </div>

              {message.role === 'user' && (
                <Avatar className="w-8 h-8 bg-slate-600">
                  <AvatarFallback className="text-white">👤</AvatarFallback>
                </Avatar>
              )}
            </div>
          ))}

          {isLoading && (
            <div className="flex gap-3">
              <Avatar className="w-8 h-8 bg-purple-600">
                <AvatarFallback className="text-white">🎵</AvatarFallback>
              </Avatar>
              <div className="bg-white/10 backdrop-blur-sm rounded-2xl px-4 py-3">
                <div className="flex gap-1">
                  <div className="w-2 h-2 bg-purple-400 rounded-full animate-bounce" />
                  <div className="w-2 h-2 bg-purple-400 rounded-full animate-bounce [animation-delay:0.2s]" />
                  <div className="w-2 h-2 bg-purple-400 rounded-full animate-bounce [animation-delay:0.4s]" />
                </div>
              </div>
            </div>
          )}
        </div>
      </div>

      {/* Input */}
      <div className="border-t border-white/10 bg-black/30 backdrop-blur-sm p-4">
        <form
          onSubmit={handleSubmit}
          className="max-w-4xl mx-auto flex gap-2"
        >
          <Input
            value={input}
            onChange={handleInputChange}
            placeholder="Pergunte sobre músicas..."
            className="flex-1 bg-white/10 border-white/20 text-white placeholder:text-gray-400 focus:border-purple-500"
            disabled={isLoading}
          />
          <Button
            type="submit"
            disabled={isLoading || !input.trim()}
            className="bg-purple-600 hover:bg-purple-700"
          >
            <Send className="w-4 h-4" />
          </Button>
        </form>
      </div>
    </div>
  );
}
```

**Recursos da UI:**

1. **useChat Hook (AI SDK):**
   ```typescript
   const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat()
   ```
   - Gerencia estado do chat automaticamente
   - Conecta com `/api/chat`
   - Streaming em tempo real

2. **Sugestões iniciais:**
   - Ajuda usuário a começar
   - Demonstra capacidades do bot

3. **Loading state:**
   - Animação de "typing..."
   - Feedback visual

4. **Design moderno:**
   - Gradiente de fundo
   - Glassmorphism (backdrop-blur)
   - Responsivo

### 8.2 Estilos Globais

Atualizar `app/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 271 91% 65%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 271 91% 65%;
    --radius: 0.5rem;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}

/* Animação customizada para loading */
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

.animate-bounce {
  animation: bounce 0.6s infinite;
}
```

### 8.3 Layout Principal

Atualizar `app/layout.tsx`:

```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Music Bot - Seu Assistente Musical',
  description: 'Chatbot inteligente para descobrir informações sobre músicas',
  keywords: ['música', 'chatbot', 'AI', 'Gemini'],
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="pt-BR" className="dark">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

---

## 9. Integração com Google Gemini

### 9.1 Por que Gemini?

**Vantagens do Gemini:**

| Aspecto | Gemini | OpenAI GPT |
|---------|--------|-----------|
| **Custo** | Grátis (1500 req/dia) | Pago desde req 1 |
| **Performance** | Excelente | Excelente |
| **Context Window** | 1M tokens (1.5 Pro) | 128k tokens |
| **Multilíngue** | Otimizado | Muito bom |
| **Streaming** | Suportado | Suportado |
| **Function Calling** | Suportado | Suportado |

### 9.2 Modelos Disponíveis

```typescript
// Geração de texto
google('gemini-1.5-flash-latest')  // Rápido, econômico, bom para chat
google('gemini-1.5-pro-latest')    // Mais poderoso, contexto de 1M tokens
google('gemini-2.0-flash-exp')     // Experimental, mais recente

// Embeddings
google.textEmbeddingModel('text-embedding-004')  // 768 dimensões
```

**Quando usar cada modelo:**

- **gemini-1.5-flash**: Chat geral, respostas rápidas (RECOMENDADO para este projeto)
- **gemini-1.5-pro**: Análises complexas, contexto muito grande
- **gemini-2.0-flash-exp**: Testes de funcionalidades futuras

### 9.3 Configuração Avançada

Criar `lib/ai/gemini-config.ts`:

```typescript
import { google } from '@ai-sdk/google';

/**
 * Configurações para diferentes modelos Gemini
 */
export const GEMINI_MODELS = {
  chat: google('gemini-1.5-flash-latest', {
    // Configurações de segurança (opcional)
    safetySettings: [
      {
        category: 'HARM_CATEGORY_HATE_SPEECH',
        threshold: 'BLOCK_MEDIUM_AND_ABOVE',
      },
      {
        category: 'HARM_CATEGORY_SEXUALLY_EXPLICIT',
        threshold: 'BLOCK_MEDIUM_AND_ABOVE',
      },
    ],
  }),
  embedding: google.textEmbeddingModel('text-embedding-004'),
} as const;

/**
 * Parâmetros de geração padrão
 */
export const DEFAULT_GENERATION_CONFIG = {
  temperature: 0.7, // Criatividade balanceada
  topP: 0.95, // Nucleus sampling
  topK: 40, // Top-k sampling
  maxTokens: 500, // Limite de tokens na resposta
} as const;
```

### 9.4 Monitoramento de Uso

Criar `lib/ai/usage-tracker.ts`:

```typescript
/**
 * Rastreia uso da API do Gemini
 * (Útil para monitorar o limite de 1500 req/dia)
 */

interface UsageStats {
  requests: number;
  tokens: number;
  errors: number;
  lastReset: Date;
}

class UsageTracker {
  private stats: UsageStats = {
    requests: 0,
    tokens: 0,
    errors: 0,
    lastReset: new Date(),
  };

  trackRequest(tokens: number = 0) {
    // Reset diário
    if (this.shouldReset()) {
      this.reset();
    }

    this.stats.requests++;
    this.stats.tokens += tokens;

    this.logStatus();
  }

  trackError() {
    this.stats.errors++;
  }

  private shouldReset(): boolean {
    const now = new Date();
    const hoursSinceReset =
      (now.getTime() - this.stats.lastReset.getTime()) / (1000 * 60 * 60);
    return hoursSinceReset >= 24;
  }

  private reset() {
    this.stats = {
      requests: 0,
      tokens: 0,
      errors: 0,
      lastReset: new Date(),
    };
    console.log('🔄 Usage stats reset');
  }

  private logStatus() {
    const limit = 1500; // Limite diário do Gemini (free tier)
    const percentage = ((this.stats.requests / limit) * 100).toFixed(1);

    console.log(`📊 API Usage: ${this.stats.requests}/${limit} (${percentage}%)`);

    if (this.stats.requests > limit * 0.9) {
      console.warn('⚠️  Approaching daily limit!');
    }
  }

  getStats(): UsageStats {
    return { ...this.stats };
  }
}

export const usageTracker = new UsageTracker();
```

Usar no `app/api/chat/route.ts`:

```typescript
import { usageTracker } from '@/lib/ai/usage-tracker';

export async function POST(req: Request) {
  try {
    // Trackear requisição
    usageTracker.trackRequest();

    // ... resto do código
  } catch (error) {
    usageTracker.trackError();
    // ... tratamento de erro
  }
}
```

### 9.5 Rate Limiting

Criar `lib/api/rate-limit.ts`:

```typescript
/**
 * Rate limiter simples para evitar abuse
 */

interface RateLimitEntry {
  count: number;
  resetTime: number;
}

class RateLimiter {
  private requests = new Map<string, RateLimitEntry>();
  private readonly maxRequests: number;
  private readonly windowMs: number;

  constructor(maxRequests: number = 10, windowMs: number = 60000) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }

  check(identifier: string): { allowed: boolean; retryAfter?: number } {
    const now = Date.now();
    const entry = this.requests.get(identifier);

    // Primeira requisição ou janela expirou
    if (!entry || now > entry.resetTime) {
      this.requests.set(identifier, {
        count: 1,
        resetTime: now + this.windowMs,
      });
      return { allowed: true };
    }

    // Dentro da janela
    if (entry.count < this.maxRequests) {
      entry.count++;
      return { allowed: true };
    }

    // Limite excedido
    return {
      allowed: false,
      retryAfter: Math.ceil((entry.resetTime - now) / 1000),
    };
  }

  cleanup() {
    const now = Date.now();
    for (const [key, entry] of this.requests.entries()) {
      if (now > entry.resetTime) {
        this.requests.delete(key);
      }
    }
  }
}

// 10 requisições por minuto
export const rateLimiter = new RateLimiter(10, 60000);

// Limpar periodicamente (a cada 5 minutos)
setInterval(() => rateLimiter.cleanup(), 5 * 60 * 1000);
```

Usar no `app/api/chat/route.ts`:

```typescript
import { rateLimiter } from '@/lib/api/rate-limit';

export async function POST(req: Request) {
  // Identificar cliente (IP ou user ID)
  const identifier = req.headers.get('x-forwarded-for') || 'anonymous';

  // Verificar rate limit
  const { allowed, retryAfter } = rateLimiter.check(identifier);

  if (!allowed) {
    return Response.json(
      {
        error: 'Too many requests',
        message: `Please try again in ${retryAfter} seconds`,
      },
      {
        status: 429,
        headers: {
          'Retry-After': String(retryAfter),
        },
      }
    );
  }

  // ... resto do código
}
```

---

## 10. Deploy e Próximos Passos

### 10.1 Preparar para Produção

**Checklist de produção:**

- [ ] Variáveis de ambiente configuradas
- [ ] Rate limiting implementado
- [ ] Logs estruturados
- [ ] Tratamento de erros completo
- [ ] Testes básicos
- [ ] README.md atualizado

### 10.2 Deploy no Vercel

```bash
# 1. Instalar Vercel CLI
pnpm add -g vercel

# 2. Login
vercel login

# 3. Deploy
vercel

# Seguir instruções:
# - Link to existing project? No
# - Project name? music-bot
# - Directory? ./
```

**Configurar variáveis de ambiente no Vercel:**

1. Acesse: https://vercel.com/dashboard
2. Selecione seu projeto
3. Settings → Environment Variables
4. Adicionar:
   - `DATABASE_URL`
   - `GOOGLE_GENERATIVE_AI_API_KEY`

**Configurar PostgreSQL para produção:**

Opções:
1. **Vercel Postgres** (recomendado)
   - Integração nativa
   - Já inclui pgvector
2. **Supabase** (grátis até 500MB)
3. **Neon** (serverless Postgres)

### 10.3 GitHub

```bash
# Inicializar repositório
git init
git add .
git commit -m "Initial commit: Music Bot with RAG"

# Criar repositório no GitHub
# https://github.com/carloswimmer

# Push
git remote add origin https://github.com/carloswimmer/music-bot.git
git branch -M main
git push -u origin main
```

Criar `.gitignore`:

```
# Dependencies
node_modules/
.pnpm-debug.log

# Next.js
.next/
out/
dist/

# Environment
.env
.env.local
.env*.local

# Database
*.db
*.sqlite

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
```

### 10.4 README.md do Projeto

Criar `README.md`:

```markdown
# 🎵 Music Bot

Chatbot inteligente para descobrir informações sobre músicas, construído com **RAG (Retrieval-Augmented Generation)** e **Google Gemini**.

## ✨ Funcionalidades

- 💬 Chat em tempo real com streaming
- 🔍 Busca semântica usando embeddings
- 🎸 Base de conhecimento de músicas clássicas
- 🤖 Powered by Google Gemini
- 🚀 Deploy fácil no Vercel

## 🛠️ Stack Tecnológica

- **Frontend:** Next.js 14 + React + TailwindCSS
- **Backend:** Next.js API Routes
- **Database:** PostgreSQL + pgvector
- **ORM:** Drizzle ORM
- **LLM:** Google Gemini (gemini-1.5-flash)
- **Embeddings:** text-embedding-004
- **AI SDK:** Vercel AI SDK

## 🚀 Como Executar

### Pré-requisitos

- Node.js 18+
- pnpm
- Docker (para PostgreSQL)
- Google Gemini API Key

### Setup

\`\`\`bash
# 1. Clone o repositório
git clone https://github.com/carloswimmer/music-bot.git
cd music-bot

# 2. Instale dependências
pnpm install

# 3. Configure variáveis de ambiente
cp .env.example .env.local
# Edite .env.local e adicione sua GOOGLE_GENERATIVE_AI_API_KEY

# 4. Inicie o PostgreSQL
docker-compose up -d

# 5. Execute migrações
pnpm db:generate
pnpm db:migrate

# 6. Popule o banco com dados de exemplo
pnpm dev  # Em outro terminal:
pnpm populate

# 7. Acesse http://localhost:3000
\`\`\`

## 📚 Conceitos Aprendidos

- ✅ RAG (Retrieval-Augmented Generation)
- ✅ Vector Embeddings e Similarity Search
- ✅ PostgreSQL com pgvector
- ✅ Drizzle ORM
- ✅ Google Gemini API
- ✅ Next.js 14 App Router
- ✅ Vercel AI SDK
- ✅ Streaming de respostas
- ✅ TypeScript avançado

## 🎯 Próximos Passos

- [ ] Adicionar autenticação
- [ ] Integração com YouTube Music API
- [ ] Sistema de favoritos
- [ ] Histórico de conversas
- [ ] Testes automatizados
- [ ] CI/CD com GitHub Actions

## 📄 Licença

MIT - Carlos Wimmer
\`\`\`

### 10.5 Próximos Passos de Aprendizado

Após completar este projeto, você pode:

1. **Melhorar a base de conhecimento:**
   - Integrar com YouTube Music API
   - Web scraping de letras de músicas
   - Adicionar informações de álbuns completos

2. **Adicionar funcionalidades:**
   - Autenticação com NextAuth.js
   - Histórico de conversas persistente
   - Sistema de recomendação de músicas
   - Análise de sentimento das letras

3. **Otimizações:**
   - Cache de embeddings (Redis)
   - Compressão de contexto (LangChain)
   - Reranking de resultados
   - Ajuste fino de thresholds

4. **Testes:**
   - Unit tests (Vitest)
   - Integration tests (Playwright)
   - E2E tests
   - Performance benchmarks

5. **DevOps:**
   - CI/CD com GitHub Actions
   - Monitoramento (Sentry)
   - Analytics (PostHog)
   - Feature flags

---

## 📋 Resumo de Comandos

### Durante o Desenvolvimento

```bash
# Iniciar servidor de desenvolvimento
pnpm dev

# Gerar migrations
pnpm db:generate

# Aplicar migrations
pnpm db:migrate

# Visualizar banco de dados
pnpm db:studio

# Popular banco com dados
pnpm populate

# Limpar banco
curl -X DELETE http://localhost:3000/api/populate

# Build de produção
pnpm build

# Iniciar produção
pnpm start
```

### Docker

```bash
# Iniciar PostgreSQL
docker-compose up -d

# Ver logs
docker-compose logs -f

# Parar
docker-compose down

# Resetar tudo (CUIDADO: apaga dados)
docker-compose down -v
```

---

## 🎓 Conceitos-Chave para Revisão

### 1. RAG (Retrieval-Augmented Generation)
- Combina busca de informações + geração de texto
- Evita "alucinações" do LLM
- Base: dados verificados

### 2. Embeddings
- Representações vetoriais de texto
- Captura significado semântico
- Permite busca por similaridade

### 3. Cosine Similarity
- Mede ângulo entre vetores
- 0 = muito diferente, 1 = idêntico
- Threshold: 0.5 (filtro)

### 4. pgvector
- Extensão PostgreSQL para vetores
- Índice HNSW para busca rápida
- Operadores de distância integrados

### 5. Streaming
- Resposta em tempo real
- Melhor UX
- Menor latência percebida

### 6. Server Actions
- Funções que rodam no servidor
- Type-safe
- Marcador `'use server'`

---

## 🙏 Créditos

Projeto de estudos baseado no chatbot-guest-faq, adaptado para aprendizado de conceitos de RAG, embeddings e LLMs.

**Autor:** Carlos Wimmer  
**GitHub:** https://github.com/carloswimmer  
**Data:** Outubro 2025

---

## 📞 Suporte

Se tiver dúvidas durante o desenvolvimento:

1. Revisar este documento
2. Consultar documentação oficial:
   - [Next.js](https://nextjs.org/docs)
   - [Vercel AI SDK](https://sdk.vercel.ai/docs)
   - [Drizzle ORM](https://orm.drizzle.team/docs)
   - [Google Gemini](https://ai.google.dev/docs)
3. GitHub Issues do projeto

---

## ✅ Checklist de Progresso

Marque conforme completa cada etapa:

### Setup (Seção 3)
- [ ] 3.1 - Projeto Next.js criado
- [ ] 3.2 - Dependências instaladas
- [ ] 3.3 - shadcn-ui configurado
- [ ] 3.4 - Variáveis de ambiente configuradas
- [ ] 3.5 - Gemini API Key obtida

### Database (Seção 4)
- [ ] 4.1 - PostgreSQL rodando no Docker
- [ ] 4.2 - Drizzle configurado
- [ ] 4.3 - Schemas criados (resources, embeddings)
- [ ] 4.4 - Conexão com DB testada
- [ ] 4.5 - Migrações executadas com sucesso
- [ ] 4.6 - Drizzle Studio acessível

### Embeddings (Seção 5)
- [ ] 5.1 - Sistema de embeddings implementado
- [ ] 5.2 - Função de busca por similaridade funcionando
- [ ] 5.3 - Testes de embeddings bem-sucedidos

### API de Ingestão (Seção 6)
- [ ] 6.1 - Server Actions criadas
- [ ] 6.2 - API `/api/populate` funcionando
- [ ] 6.3 - Script de população executado
- [ ] 6.4 - Dados de exemplo no banco

### API de Chat (Seção 7)
- [ ] 7.1 - API `/api/chat` implementada
- [ ] 7.2 - RAG funcionando corretamente
- [ ] 7.3 - Tratamento de erros implementado
- [ ] 7.4 - Streaming de respostas funcionando

### Interface (Seção 8)
- [ ] 8.1 - Componente de chat criado
- [ ] 8.2 - UI responsiva e bonita
- [ ] 8.3 - Sugestões de perguntas funcionando
- [ ] 8.4 - Estados de loading implementados

### Gemini (Seção 9)
- [ ] 9.1 - Integração com Gemini funcionando
- [ ] 9.2 - Models configurados corretamente
- [ ] 9.3 - Usage tracker implementado
- [ ] 9.4 - Rate limiting funcionando

### Deploy (Seção 10)
- [ ] 10.1 - Projeto pronto para produção
- [ ] 10.2 - Deploy no Vercel realizado
- [ ] 10.3 - Repositório GitHub criado
- [ ] 10.4 - README.md completo

---

**Boa sorte com seus estudos! 🚀🎵**

