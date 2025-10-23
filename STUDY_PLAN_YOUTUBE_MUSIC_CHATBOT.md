# 🎵 Study Plan: YouTube Music Chatbot with RAG

**Project:** Music Bot - YouTube Music Information Chatbot  
**LLM:** Google Gemini  
**Author:** Carlos Wimmer  
**GitHub:** https://github.com/carloswimmer/chatbot-youtube-music  
**Date:** October 2025

---

## 📚 Table of Contents

1. [Introduction and Fundamental Concepts](#1-introduction-and-fundamental-concepts)
2. [System Architecture](#2-system-architecture)
3. [Initial Project Setup](#3-initial-project-setup)
4. [Database with PostgreSQL and pgvector](#4-database-with-postgresql-and-pgvector)
5. [Embeddings System](#5-embeddings-system)
6. [Data Ingestion API](#6-data-ingestion-api)
7. [Chat API with RAG](#7-chat-api-with-rag)
8. [User Interface](#8-user-interface)
9. [Google Gemini Integration](#9-google-gemini-integration)
10. [Deploy and Next Steps](#10-deploy-and-next-steps)

---

## 1. Introduction and Fundamental Concepts

### 1.1 What will you learn?

This project will teach you how to build a **RAG chatbot (Retrieval-Augmented Generation)** from scratch. You will master:

- ✅ **RAG (Retrieval-Augmented Generation)**: How to make an LLM respond using only specific information
- ✅ **Vector Embeddings**: How to transform text into numerical vectors for semantic search
- ✅ **Similarity Search**: How to find relevant information using cosine distance
- ✅ **Next.js 14 App Router**: Modern React framework with Server Actions
- ✅ **Drizzle ORM**: TypeScript-first ORM for PostgreSQL
- ✅ **Google Gemini API**: Google's LLM for text generation
- ✅ **AI SDK (Vercel)**: Library for working with LLMs in a standardized way
- ✅ **Docker**: Application and database containerization

### 1.2 What is RAG?

**RAG (Retrieval-Augmented Generation)** is a technique that combines:

1. **Retrieval**: Search for relevant information from a knowledge base
2. **Generation**: Use an LLM to generate responses based on retrieved information

**Why use RAG?**

- ❌ **Without RAG**: The LLM can "hallucinate" (make up) information
- ✅ **With RAG**: The LLM responds only based on verified data

**Practical example:**

```
User: "What is Taylor Swift's most recent album?"

Without RAG:
→ LLM might invent or use outdated training data

With RAG:
→ System searches database → Finds "The Tortured Poets Department (2024)"
→ LLM uses this information to respond → Accurate and up-to-date answer
```

### 1.3 How does vector search work?

**Vector Embeddings** are numerical representations of text:

```
Text: "upbeat pop song"
→ Embedding: [0.234, -0.567, 0.891, ... ] (1536 dimensions)

Text: "cheerful pop tune"
→ Embedding: [0.221, -0.543, 0.876, ... ] (very similar!)
```

**Semantic Search:**

- Calculates **cosine distance** between vectors
- The smaller the distance, the more similar the meaning
- Typical threshold: 0.5 (50% similarity)

### 1.4 Complete System Flow

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: DATA INGESTION (Runs once)                        │
└─────────────────────────────────────────────────────────────┘

    1. Music data (JSON/TXT)
          ↓
    2. Process into text chunks
          ↓
    3. Generate embeddings with Gemini
          ↓
    4. Save to PostgreSQL (pgvector)


┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: CHAT (Runs for each question)                     │
└─────────────────────────────────────────────────────────────┘

    1. User asks question
          ↓
    2. Generate question embedding
          ↓
    3. Search for similar embeddings (cosine distance)
          ↓
    4. Return top 4 most relevant songs/information
          ↓
    5. Gemini generates response using context
          ↓
    6. User receives accurate response
```

---

## 2. System Architecture

### 2.1 Technology Stack

| Layer               | Technology              | Why use it?                               |
| ------------------- | ----------------------- | ----------------------------------------- |
| **Frontend**        | Next.js 14 + React      | Modern full-stack framework, SEO-friendly |
| **Styling**         | TailwindCSS + shadcn-ui | Fast and consistent design system         |
| **Backend API**     | Next.js API Routes      | APIs integrated in the same project       |
| **Database**        | PostgreSQL 16           | Robust relational database                |
| **Vector Search**   | pgvector                | Extension for vector search in Postgres   |
| **ORM**             | Drizzle ORM             | TypeScript-first, type-safe, performant   |
| **LLM**             | Google Gemini           | Free/accessible Google API                |
| **Embeddings**      | text-embedding-004      | Gemini embedding model                    |
| **AI SDK**          | Vercel AI SDK           | Standardized abstraction for LLMs         |
| **Validation**      | Zod                     | TypeScript schema validation              |
| **Package Manager** | pnpm                    | Faster and more efficient than npm        |

### 2.2 Folder Structure

```
chatbot-youtube-music/
├── app/                          # Next.js 14 App Router
│   ├── api/
│   │   ├── chat/
│   │   │   └── route.ts         # RAG chat API
│   │   └── populate/
│   │       └── route.ts         # Data ingestion API
│   ├── layout.tsx               # Main layout
│   ├── page.tsx                 # Chat interface
│   └── globals.css              # Global styles
├── components/
│   └── ui/                      # shadcn-ui components
├── lib/
│   ├── ai/
│   │   └── embedding.ts         # Embeddings and search logic
│   ├── db/
│   │   ├── index.ts            # Database connection
│   │   ├── migrate.ts          # Migration script
│   │   └── schema/
│   │       ├── resources.ts    # Resources schema (songs)
│   │       └── embeddings.ts   # Embeddings schema
│   └── actions/
│       └── resources.ts         # Server Actions for CRUD
├── scripts/
│   └── populate-sample-data.ts  # Script to populate database
├── docker-compose.yml           # Docker for Postgres
├── drizzle.config.ts           # Drizzle configuration
├── package.json
├── tsconfig.json
└── .env                        # Environment variables
```

### 2.3 Database Schema

```sql
-- Resources table (song information)
CREATE TABLE resources (
  id VARCHAR(191) PRIMARY KEY,
  content TEXT NOT NULL,              -- Full text about the song
  song_id VARCHAR(191) NOT NULL,      -- Song ID on YouTube Music
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Embeddings table (vectors for search)
CREATE TABLE embeddings (
  id VARCHAR(191) PRIMARY KEY,
  resource_id VARCHAR(191) REFERENCES resources(id) ON DELETE CASCADE,
  embedding VECTOR(768) NOT NULL,     -- 768-dimension vector (Gemini)
);

-- HNSW index for fast vector search
CREATE INDEX embedding_index ON embeddings
  USING hnsw (embedding vector_cosine_ops);
```

**Why two tables?**

- `resources`: Stores original data (easy to read/edit)
- `embeddings`: Stores vectors (optimized for search)
- Separation allows managing data and vectors independently

---

## 3. Initial Project Setup

### 3.1 Prerequisites

```bash
# Check installed versions
node --version    # >= 18.17.0
pnpm --version    # >= 8.0.0
docker --version  # >= 20.10.0
```

### 3.2 Create Next.js Project

```bash
# 1. Create project directory
mkdir music-bot
cd music-bot

# 2. Initialize Next.js with TypeScript
pnpm create next-app@latest . --typescript --tailwind --app --src-dir=false

# Choose options:
# ✔ Would you like to use TypeScript? → Yes
# ✔ Would you like to use ESLint? → Yes
# ✔ Would you like to use Tailwind CSS? → Yes
# ✔ Would you like to use `src/` directory? → No
# ✔ Would you like to use App Router? → Yes
# ✔ Would you like to customize the default import alias? → No
```

### 3.3 Install Dependencies

```bash
# AI and LLM
pnpm add ai @ai-sdk/google

# Database and ORM
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit

# Validation
pnpm add zod drizzle-zod

# UI Components
pnpm add @radix-ui/react-slot @radix-ui/react-avatar
pnpm add class-variance-authority clsx tailwind-merge lucide-react

# Utilities
pnpm add nanoid

# Dev tools
pnpm add -D tsx dotenv @types/pg
```

**Why each library?**

| Library          | Purpose                              |
| ---------------- | ------------------------------------ |
| `ai`             | Vercel AI SDK - abstraction for LLMs |
| `@ai-sdk/google` | Gemini provider for AI SDK           |
| `drizzle-orm`    | TypeScript-first ORM                 |
| `postgres`       | PostgreSQL driver for Node.js        |
| `zod`            | Schema validation                    |
| `nanoid`         | Unique ID generator                  |
| `lucide-react`   | SVG icons                            |

### 3.4 Configure shadcn-ui

```bash
# Install shadcn-ui CLI
pnpm dlx shadcn@latest init

# Settings:
# ✔ Which style would you like to use? → Default
# ✔ Which color would you like to use as base color? → Slate
# ✔ Would you like to use CSS variables for colors? → Yes

# Add necessary components
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add avatar
```

### 3.5 Configure Environment Variables

Create `.env.local` file:

```bash
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/musicbot

# Google Gemini API
GOOGLE_GENERATIVE_AI_API_KEY=your-key-here

# Next.js
NODE_ENV=development
```

**How to get Gemini API Key:**

1. Access: https://makersuite.google.com/app/apikey
2. Click "Create API Key"
3. Copy the generated key
4. Paste into `.env.local`

### 3.6 Configure TypeScript

Update `tsconfig.json`:

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

## 4. Database with PostgreSQL and pgvector

### 4.1 Why PostgreSQL + pgvector?

**PostgreSQL:**

- ✅ Open-source and mature (30+ years)
- ✅ Supports JSON, arrays, custom types
- ✅ ACID compliant (reliable transactions)
- ✅ Excellent performance

**pgvector:**

- ✅ Extension that adds vector support
- ✅ HNSW (Hierarchical Navigable Small World) index for fast search
- ✅ Optimized similarity operations
- ✅ Native integration with PostgreSQL

**Alternatives considered:**

- Pinecone: Paid, vendor lock-in
- Weaviate: Complex to get started
- ChromaDB: Less mature
- ✅ **pgvector**: Simple, open-source, integrated

### 4.2 Docker Setup for PostgreSQL

Create `docker-compose.yml`:

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
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U postgres']
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

Create `init-db.sql`:

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;
```

**Start the database:**

```bash
docker-compose up -d

# Check if it's running
docker-compose ps
docker-compose logs postgres
```

### 4.3 Configure Drizzle ORM

Create `drizzle.config.ts`:

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

**Why Drizzle?**

- ✅ TypeScript-first (complete type safety)
- ✅ SQL-like syntax (easy migration from raw SQL)
- ✅ Automatic migrations
- ✅ Zero runtime overhead
- ✅ Native pgvector support

### 4.4 Create Database Schemas

Create `lib/db/schema/resources.ts`:

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
  created_at: timestamp('created_at')
    .notNull()
    .default(sql`now()`),
  updated_at: timestamp('updated_at')
    .notNull()
    .default(sql`now()`),
});

// Validation schema for insertion
export const insertResourceSchema = createSelectSchema(resources).omit({
  id: true,
  created_at: true,
  updated_at: true,
});

export type NewResourceParams = z.infer<typeof insertResourceSchema>;
```

**Code explanation:**

- `pgTable`: Defines table in Drizzle
- `nanoid()`: Generates unique short IDs (better than UUID)
- `createSelectSchema`: Automatically generates Zod validation
- `omit`: Removes auto-generated fields from insertion schema

Create `lib/db/schema/embeddings.ts`:

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
    resource_id: varchar('resource_id', { length: 191 }).references(
      () => resources.id,
      { onDelete: 'cascade' }
    ),
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

**Important details:**

- `vector('embedding', { dimensions: 768 })`:
  - Gemini text-embedding-004 generates 768-dimension vectors
  - OpenAI uses 1536 dimensions (hence the difference)
- `references(..., { onDelete: 'cascade' })`:
  - If you delete a song, it also deletes the embedding
- `hnsw`: Index algorithm for fast vector search
- `vector_cosine_ops`: Operator to calculate cosine distance

### 4.5 Create Database Connection

Create `lib/db/index.ts`:

```typescript
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema/resources';
import * as embeddingsSchema from './schema/embeddings';

if (!process.env.DATABASE_URL) {
  throw new Error('DATABASE_URL environment variable is not set');
}

// Connection for queries
const connectionString = process.env.DATABASE_URL;
const client = postgres(connectionString, { prepare: false });

export const db = drizzle(client, {
  schema: { ...schema, ...embeddingsSchema },
});
```

**Why `prepare: false`?**

- Avoids issues with prepared statements in serverless
- Necessary for Vercel/Next.js

### 4.6 Create Migration Script

Create `lib/db/migrate.ts`:

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

### 4.7 Add Scripts to package.json

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

### 4.8 Run Migrations

```bash
# 1. Generate migration files
pnpm db:generate

# 2. Apply migrations to database
pnpm db:migrate

# 3. (Optional) View database with Drizzle Studio
pnpm db:studio
# Access: https://local.drizzle.studio
```

---

## 5. Embeddings System

### 5.1 What are Embeddings?

**Embeddings** are vector representations of text that capture semantic meaning.

```
Example:

Text 1: "upbeat happy song"
Embedding 1: [0.8, 0.3, -0.5, 0.2, ...]

Text 2: "cheerful energetic tune"
Embedding 2: [0.75, 0.28, -0.48, 0.19, ...]
                ↑ Very similar!

Text 3: "scary horror movie"
Embedding 3: [-0.6, -0.9, 0.7, -0.4, ...]
                ↑ Very different!
```

**How it works:**

1. Model trained on billions of texts
2. Learns semantic relationships between words
3. Projects text into high-dimensional vector space
4. Similar texts are close in vector space

### 5.2 Gemini Embeddings vs OpenAI

| Feature         | Gemini (text-embedding-004) | OpenAI (text-embedding-ada-002) |
| --------------- | --------------------------- | ------------------------------- |
| **Dimensions**  | 768                         | 1536                            |
| **Cost**        | Free up to 1500 req/day     | $0.10 / 1M tokens               |
| **Performance** | Excellent                   | Excellent                       |
| **Rate Limit**  | 1500 req/min                | 3000 req/min                    |
| **Languages**   | Multilingual                | Multilingual                    |

### 5.3 Implement Embeddings System

Create `lib/ai/embedding.ts`:

```typescript
import { google } from '@ai-sdk/google';
import { embed, embedMany } from 'ai';
import { cosineDistance, desc, gt, sql } from 'drizzle-orm';
import { db } from '../db';
import { embeddings } from '../db/schema/embeddings';
import { resources } from '../db/schema/resources';

// Gemini embedding model
const embeddingModel = google.textEmbeddingModel('text-embedding-004');

/**
 * Generates embedding for a single text
 * @param value - Text to generate embedding for
 * @returns Array of numbers representing the embedding (768 dimensions)
 */
export const generateEmbedding = async (value: string): Promise<number[]> => {
  // Clean text (remove line breaks and extra spaces)
  const input = value.replaceAll('\\n', ' ').trim();

  // Validate input
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
 * Generates embeddings for multiple texts (more efficient than one by one)
 * @param values - Array of texts
 * @returns Array of embeddings
 */
export const generateEmbeddings = async (
  values: string[]
): Promise<number[][]> => {
  // Clean all texts
  const inputs = values.map((v) => v.replaceAll('\\n', ' ').trim());

  // Validate inputs
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
 * Interface for search results
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
 * Searches for relevant content using embedding similarity
 * @param userQuery - User's question
 * @param limit - Maximum number of results (default: 4)
 * @param threshold - Minimum similarity threshold (default: 0.5)
 * @returns Results ordered by similarity
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
    // 1. Generate embedding for user's question
    const userQueryEmbedding = await generateEmbedding(userQuery);

    // 2. Calculate similarity using cosine distance
    // Formula: similarity = 1 - cosine_distance
    // Result: 0 (completely different) to 1 (identical)
    const similarity = sql<number>`1 - (${cosineDistance(
      embeddings.embedding,
      userQueryEmbedding
    )})`;

    // 3. Search for similar documents
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

    // 4. Process results
    if (similarResults.length === 0) {
      console.log('❌ No results above threshold');
      return {
        results: [],
        idealResponse: null,
        highestSimilarity: 0,
      };
    }

    // 5. First result is most relevant (highest similarity)
    const idealResponse = {
      content: similarResults[0].content,
      similarity: similarResults[0].similarity,
      song_id: similarResults[0].song_id,
    };

    console.log(`✅ Found ${similarResults.length} similar results`);
    console.log(
      `📊 Highest similarity: ${idealResponse.similarity.toFixed(3)}`
    );

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

**Key concepts explained:**

1. **Cosine Distance:**

   ```
   Distance = 1 - (A · B) / (||A|| × ||B||)

   Where:
   - A · B = dot product of vectors
   - ||A||, ||B|| = norms (magnitudes) of vectors

   Result:
   - 0 = identical vectors (same direction)
   - 1 = opposite vectors
   ```

2. **Similarity Score:**

   ```
   Similarity = 1 - Cosine Distance

   Interpretation:
   - 0.9 - 1.0 = Extremely similar (almost duplicate)
   - 0.7 - 0.9 = Very similar (same topic)
   - 0.5 - 0.7 = Similar (related)
   - < 0.5    = Not very related (ignore)
   ```

3. **Threshold (0.5):**
   - Filters very different results
   - Avoids irrelevant responses
   - Can be adjusted as needed

4. **Batch Processing:**
   - `generateEmbeddings()` processes multiple texts at once
   - More efficient than calling `generateEmbedding()` in a loop
   - Reduces API calls

---

## 6. Data Ingestion API

### 6.1 Why do we need ingestion?

The chatbot can only respond about information in the database. The **ingestion API** allows:

1. ✅ Add new songs to the system
2. ✅ Update existing information
3. ✅ Process data in batches (bulk insert)
4. ✅ Automatically generate embeddings

**Ingestion flow:**

```
Input data (JSON)
  ↓
Validation (Zod)
  ↓
Save to 'resources' table
  ↓
Generate content embedding
  ↓
Save to 'embeddings' table
  ↓
Return confirmation
```

### 6.2 Create Server Actions

Create `lib/actions/resources.ts`:

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
 * Creates a new resource (song) and generates its embedding
 * @param input - Song data (content, song_id)
 * @returns Success or error message
 */
export const createResource = async (input: NewResourceParams) => {
  try {
    // 1. Validate input with Zod
    const { content, song_id } = insertResourceSchema.parse(input);

    // 2. Insert into resources table
    const [resource] = await db
      .insert(resources)
      .values({ content, song_id })
      .returning();

    console.log(`✅ Resource created: ${resource.id}`);

    // 3. Generate content embedding
    console.log('⏳ Generating embedding...');
    const embedding = await generateEmbedding(content);

    // 4. Save embedding to embeddings table
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
 * Deletes all resources from database (use with caution!)
 * @returns Confirmation message
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

**Why Server Actions?**

- ✅ Execute on server (secure database access)
- ✅ TypeScript end-to-end
- ✅ Don't need separate API routes
- ✅ `'use server'` marker indicates server execution

### 6.3 Create Populate API Route

Create `app/api/populate/route.ts`:

```typescript
import { createResource, truncateResources } from '@/lib/actions/resources';
import { insertResourceSchema } from '@/lib/db/schema/resources';
import { z } from 'zod';

// Schema to validate request with multiple resources
const populateSchema = z.object({
  resources: z
    .array(insertResourceSchema)
    .min(1, 'At least one resource is required'),
});

/**
 * POST /api/populate
 * Adds multiple songs to the database
 */
export async function POST(req: Request) {
  try {
    // 1. Parse body
    const body = await req.json();

    // 2. Validate with Zod
    const { resources } = populateSchema.parse(body);

    // 3. Process each resource
    const results = [];

    for (const resource of resources) {
      try {
        const result = await createResource(resource);
        results.push({
          content: resource.content.substring(0, 50) + '...', // Show only beginning
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

    // 4. Calculate statistics
    const successCount = results.filter((r) => r.status === 'success').length;
    const errorCount = results.filter((r) => r.status === 'error').length;

    // 5. Return response
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

    // Zod validation error handling
    if (error instanceof z.ZodError) {
      return Response.json(
        {
          error: 'Validation error',
          details: error.errors,
        },
        { status: 400 }
      );
    }

    // Other errors
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
 * Removes all songs from the database
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

### 6.4 Create Sample Data

Create `scripts/populate-sample-data.ts`:

```typescript
import * as dotenv from 'dotenv';

dotenv.config({ path: '.env.local' });

const sampleSongs = [
  {
    content: `The song "Bohemian Rhapsody" is performed by Queen and was released in 1975 on the album "A Night at the Opera". 
              This progressive rock masterpiece has a duration of 5:55 minutes and was composed by Freddie Mercury. 
              The song is known for its unusual structure, combining elements of ballad, opera and hard rock. 
              Considered one of the greatest songs of all time, it reached the top of the charts in several countries 
              and the music video is considered one of the first music videos in history.`,
    song_id: 'queen-bohemian-rhapsody',
  },
  {
    content: `"Imagine" by John Lennon was released in 1971 on the album of the same name. 
              This iconic 3:01 minute song conveys a message of peace and global unity. 
              With a simple piano melody and profound lyrics, Imagine became a pacifist anthem. 
              The song reached 3rd place on the American charts and continues to be one of 
              Lennon's most influential compositions after the Beatles.`,
    song_id: 'john-lennon-imagine',
  },
  {
    content: `"Hotel California" is a song by the band Eagles, released in 1977 on the album of the same name. 
              With 6:30 minutes duration, the song is famous for its iconic dual guitar solo 
              performed by Don Felder and Joe Walsh. The mysterious lyrics talk about a mysterious hotel 
              in California and can be interpreted as a critique of the hedonistic lifestyle 
              of Los Angeles. It won the Grammy for Record of the Year in 1978.`,
    song_id: 'eagles-hotel-california',
  },
  {
    content: `"Billie Jean" by Michael Jackson was released in 1983 as part of the "Thriller" album. 
              The song is 4:54 minutes long and was written and composed by Michael Jackson himself. 
              Known for its striking bass line and Jackson's characteristic vocal "hiccup", 
              Billie Jean remained 7 weeks at the top of the Billboard Hot 100. 
              The music video was one of the first by a black artist to receive heavy rotation on MTV.`,
    song_id: 'michael-jackson-billie-jean',
  },
  {
    content: `"Smells Like Teen Spirit" by Nirvana was released in 1991 on the album "Nevermind". 
              With 5:01 minutes duration, this grunge song marked a generation and is considered 
              the anthem of the grunge movement. Composed by Kurt Cobain, the song combines distorted guitars 
              with a catchy melody. Despite Cobain not liking the attention 
              the song received, it catapulted Nirvana to worldwide stardom.`,
    song_id: 'nirvana-smells-like-teen-spirit',
  },
  {
    content: `"Like a Prayer" by Madonna was released in 1989 as the lead single from the album of the same name. 
              The 5:43 minute song mixes pop with gospel elements, featuring a church choir. 
              Produced by Madonna and Patrick Leonard, the song generated controversy due to its music video 
              that mixed religious themes with sexuality. Even so, it became one of the biggest 
              hits of Madonna's career, reaching the top of the charts in several countries.`,
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

**How to use:**

```bash
# 1. Make sure Next.js server is running
pnpm dev

# 2. In another terminal, run the script
pnpm populate
```

### 6.5 Test API with cURL

```bash
# Add a song
curl -X POST http://localhost:3000/api/populate \
  -H "Content-Type: application/json" \
  -d '{
    "resources": [{
      "content": "Test upbeat pop song with electronic beat",
      "song_id": "test-song-001"
    }]
  }'

# Clear all songs (careful!)
curl -X DELETE http://localhost:3000/api/populate
```

---

## 7. Chat API with RAG

### 7.1 Complete Chat Flow

```
┌─────────────────────────────────────────────────────────┐
│  1. User types: "Tell me about Queen songs"             │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  2. Frontend sends to: POST /api/chat                   │
│     Body: { messages: [...] }                           │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  3. Backend generates question embedding                │
│     [0.234, -0.567, 0.891, ...]                         │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  4. Search PostgreSQL (similarity > 0.5)                │
│     Returns: Top 4 most similar songs                   │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  5. Build prompt with context:                          │
│     System: "You are a music assistant..."              │
│     Context: [Information from 4 songs found]           │
│     User: "Tell me about Queen songs"                   │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  6. Send to Gemini (streaming)                          │
│     Model: gemini-1.5-flash                             │
└─────────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────────┐
│  7. Stream response to frontend                         │
│     "I have information about Bohemian Rhapsody..."     │
└─────────────────────────────────────────────────────────┘
```

### 7.2 Implement Chat API

Create `app/api/chat/route.ts`:

```typescript
import { findRelevantContent } from '@/lib/ai/embedding';
import { google } from '@ai-sdk/google';
import { streamText, tool } from 'ai';
import { z } from 'zod';

// Allow streaming for up to 30 seconds
export const maxDuration = 30;

// Gemini model for text generation
const model = google('gemini-1.5-flash-latest');

export async function POST(req: Request) {
  try {
    // 1. Parse body
    const { messages } = await req.json();

    // 2. Extract latest user message
    const latestMessage = messages[messages.length - 1];
    const userQuery = latestMessage?.content || '';

    console.log(`\n🔍 User query: "${userQuery}"\n`);

    // 3. Search for relevant content (RAG)
    const { results, highestSimilarity } = await findRelevantContent({
      userQuery,
      limit: 4,
      threshold: 0.5,
    });

    console.log(`📊 Found ${results.length} relevant songs`);
    console.log(`📈 Highest similarity: ${highestSimilarity.toFixed(3)}\n`);

    // 4. Prepare context for LLM
    let context = '';
    if (results.length > 0) {
      context = results.map((r, i) => `[${i + 1}] ${r.content}`).join('\n\n');
    }

    // 5. Tool to search information (used by Gemini when needed)
    const getInformation = tool({
      description:
        'Search for music information in the knowledge base to answer questions.',
      parameters: z.object({
        question: z.string().describe("The user's question"),
      }),
      execute: async ({ question }) => {
        const searchResults = await findRelevantContent({
          userQuery: question,
          limit: 4,
        });
        return searchResults.results.map((r) => r.content).join('\n\n');
      },
    });

    // 6. Generate response with streaming
    const response = streamText({
      model,
      messages,
      system: `You are a specialized music assistant that helps users discover information about songs.

IMPORTANT RULES:
1. Answer ONLY based on information provided in the context
2. If there is no relevant information, say: "Sorry, I don't have information about that in my database."
3. Be precise and cite specific details (artist, year, album, duration)
4. Do not invent or "hallucinate" information
5. Use a friendly and conversational tone
6. If the user asks questions outside the music scope, gently redirect

${context ? `\nAVAILABLE CONTEXT:\n${context}` : '\nNO RELEVANT CONTEXT FOUND.'}`,

      // Tools available to Gemini
      tools: {
        getInformation,
      },

      // Generation settings
      temperature: 0.7, // Moderate creativity
      maxTokens: 500, // Concise responses
    });

    // 7. Return stream
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

**Important concepts:**

1. **Streaming Response:**

   ```typescript
   streamText({ ... }).toDataStreamResponse()
   ```

   - Response is sent in chunks
   - User sees text appear in real-time
   - Better user experience

2. **Tools (Function Calling):**
   - Gemini can decide to use the `getInformation` tool
   - Useful for iterative searches or refinement
   - Increases assistant autonomy

3. **Temperature (0.7):**
   - 0.0 = Deterministic (always same response)
   - 1.0 = Very creative (may be imprecise)
   - 0.7 = Balance between precision and naturalness

4. **System Prompt:**
   - Defines assistant behavior
   - Establishes rules and limitations
   - Injects relevant context

### 7.3 Add Error Handling

Create `lib/api/errors/error-handler.ts`:

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
 * Classifies errors and returns structured information
 */
export function classifyError(error: any): ErrorInfo {
  // Gemini API error
  if (error.status === 429) {
    return {
      errorType: 'API Error',
      errorCode: 'GEMINI_API_ERROR',
      message: 'Rate limit exceeded. Try again in a few seconds.',
    };
  }

  if (error.status === 401 || error.status === 403) {
    return {
      errorType: 'API Error',
      errorCode: 'GEMINI_API_ERROR',
      message: 'Authentication error with Gemini API. Check your API key.',
    };
  }

  // Network error
  if (error.name === 'FetchError' || error.code === 'ENOTFOUND') {
    return {
      errorType: 'Network Error',
      errorCode: 'NETWORK_ERROR',
      message: 'Connection error. Check your internet.',
    };
  }

  // Validation error (Zod)
  if (error.name === 'ZodError') {
    return {
      errorType: 'Validation Error',
      errorCode: 'VALIDATION_ERROR',
      message: 'Invalid data provided.',
    };
  }

  // Database error
  if (error.code?.startsWith('23')) {
    return {
      errorType: 'Internal Error',
      errorCode: 'DATABASE_ERROR',
      message: 'Error accessing database.',
    };
  }

  // Generic error
  return {
    errorType: 'Internal Error',
    errorCode: 'UNKNOWN_ERROR',
    message: error.message || 'Unknown error.',
  };
}
```

Update `app/api/chat/route.ts` to use error handling:

```typescript
import { classifyError } from '@/lib/api/errors/error-handler';

export async function POST(req: Request) {
  try {
    // ... existing code
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

## 8. User Interface

### 8.1 Chat Component

Create `app/page.tsx`:

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
            Ask about your favorite songs!
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
                Welcome to Music Bot!
              </h2>
              <p className="text-gray-300">
                Ask questions about songs and artists
              </p>
              <div className="mt-6 grid grid-cols-1 md:grid-cols-2 gap-3 max-w-2xl mx-auto">
                {[
                  'Tell me about Bohemian Rhapsody',
                  'Which Queen songs do you know?',
                  'What do you know about John Lennon?',
                  'Songs with iconic guitar solos',
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
            placeholder="Ask about songs..."
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

**UI Features:**

1. **useChat Hook (AI SDK):**

   ```typescript
   const { messages, input, handleInputChange, handleSubmit, isLoading } =
     useChat();
   ```

   - Automatically manages chat state
   - Connects to `/api/chat`
   - Real-time streaming

2. **Initial suggestions:**
   - Helps user get started
   - Demonstrates bot capabilities

3. **Loading state:**
   - "Typing..." animation
   - Visual feedback

4. **Modern design:**
   - Background gradient
   - Glassmorphism (backdrop-blur)
   - Responsive

### 8.2 Global Styles

Update `app/globals.css`:

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

/* Custom loading animation */
@keyframes bounce {
  0%,
  100% {
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

### 8.3 Main Layout

Update `app/layout.tsx`:

```typescript
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Music Bot - Your Music Assistant',
  description: 'Intelligent chatbot to discover information about songs',
  keywords: ['music', 'chatbot', 'AI', 'Gemini'],
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className="dark">
      <body className={inter.className}>{children}</body>
    </html>
  );
}
```

---

## 9. Google Gemini Integration

### 9.1 Why Gemini?

**Gemini Advantages:**

| Aspect               | Gemini              | OpenAI GPT      |
| -------------------- | ------------------- | --------------- |
| **Cost**             | Free (1500 req/day) | Paid from req 1 |
| **Performance**      | Excellent           | Excellent       |
| **Context Window**   | 1M tokens (1.5 Pro) | 128k tokens     |
| **Multilingual**     | Optimized           | Very good       |
| **Streaming**        | Supported           | Supported       |
| **Function Calling** | Supported           | Supported       |

### 9.2 Available Models

```typescript
// Text generation
google('gemini-1.5-flash-latest'); // Fast, economical, good for chat
google('gemini-1.5-pro-latest'); // More powerful, 1M token context
google('gemini-2.0-flash-exp'); // Experimental, latest

// Embeddings
google.textEmbeddingModel('text-embedding-004'); // 768 dimensions
```

**When to use each model:**

- **gemini-1.5-flash**: General chat, fast responses (RECOMMENDED for this project)
- **gemini-1.5-pro**: Complex analysis, very large context
- **gemini-2.0-flash-exp**: Testing future features

### 9.3 Advanced Configuration

Create `lib/ai/gemini-config.ts`:

```typescript
import { google } from '@ai-sdk/google';

/**
 * Configurations for different Gemini models
 */
export const GEMINI_MODELS = {
  chat: google('gemini-1.5-flash-latest', {
    // Safety settings (optional)
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
 * Default generation parameters
 */
export const DEFAULT_GENERATION_CONFIG = {
  temperature: 0.7, // Balanced creativity
  topP: 0.95, // Nucleus sampling
  topK: 40, // Top-k sampling
  maxTokens: 500, // Token limit in response
} as const;
```

### 9.4 Usage Monitoring

Create `lib/ai/usage-tracker.ts`:

```typescript
/**
 * Tracks Gemini API usage
 * (Useful for monitoring the 1500 req/day limit)
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
    // Daily reset
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
    const limit = 1500; // Gemini daily limit (free tier)
    const percentage = ((this.stats.requests / limit) * 100).toFixed(1);

    console.log(
      `📊 API Usage: ${this.stats.requests}/${limit} (${percentage}%)`
    );

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

Use in `app/api/chat/route.ts`:

```typescript
import { usageTracker } from '@/lib/ai/usage-tracker';

export async function POST(req: Request) {
  try {
    // Track request
    usageTracker.trackRequest();

    // ... rest of code
  } catch (error) {
    usageTracker.trackError();
    // ... error handling
  }
}
```

### 9.5 Rate Limiting

Create `lib/api/rate-limit.ts`:

```typescript
/**
 * Simple rate limiter to prevent abuse
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

    // First request or window expired
    if (!entry || now > entry.resetTime) {
      this.requests.set(identifier, {
        count: 1,
        resetTime: now + this.windowMs,
      });
      return { allowed: true };
    }

    // Within window
    if (entry.count < this.maxRequests) {
      entry.count++;
      return { allowed: true };
    }

    // Limit exceeded
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

// 10 requests per minute
export const rateLimiter = new RateLimiter(10, 60000);

// Clean up periodically (every 5 minutes)
setInterval(() => rateLimiter.cleanup(), 5 * 60 * 1000);
```

Use in `app/api/chat/route.ts`:

```typescript
import { rateLimiter } from '@/lib/api/rate-limit';

export async function POST(req: Request) {
  // Identify client (IP or user ID)
  const identifier = req.headers.get('x-forwarded-for') || 'anonymous';

  // Check rate limit
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

  // ... rest of code
}
```

---

## 10. Deploy and Next Steps

### 10.1 Prepare for Production

**Production checklist:**

- [ ] Environment variables configured
- [ ] Rate limiting implemented
- [ ] Structured logs
- [ ] Complete error handling
- [ ] Basic tests
- [ ] Updated README.md

### 10.2 Deploy to Vercel

```bash
# 1. Install Vercel CLI
pnpm add -g vercel

# 2. Login
vercel login

# 3. Deploy
vercel

# Follow instructions:
# - Link to existing project? No
# - Project name? music-bot
# - Directory? ./
```

**Configure environment variables on Vercel:**

1. Access: https://vercel.com/dashboard
2. Select your project
3. Settings → Environment Variables
4. Add:
   - `DATABASE_URL`
   - `GOOGLE_GENERATIVE_AI_API_KEY`

**Configure PostgreSQL for production:**

Options:

1. **Vercel Postgres** (recommended)
   - Native integration
   - Already includes pgvector
2. **Supabase** (free up to 500MB)
3. **Neon** (serverless Postgres)

### 10.3 GitHub

```bash
# Initialize repository
git init
git add .
git commit -m "Initial commit: Music Bot with RAG"

# Create repository on GitHub
# https://github.com/carloswimmer

# Push
git remote add origin https://github.com/carloswimmer/chatbot-youtube-music.git
git branch -M main
git push -u origin main
```

Create `.gitignore`:

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

### 10.4 Project README.md

Create `README.md`:

````markdown
# 🎵 Music Bot

Intelligent chatbot to discover information about songs, built with **RAG (Retrieval-Augmented Generation)** and **Google Gemini**.

## ✨ Features

- 💬 Real-time chat with streaming
- 🔍 Semantic search using embeddings
- 🎸 Knowledge base of classic songs
- 🤖 Powered by Google Gemini
- 🚀 Easy deploy on Vercel

## 🛠️ Tech Stack

- **Frontend:** Next.js 14 + React + TailwindCSS
- **Backend:** Next.js API Routes
- **Database:** PostgreSQL + pgvector
- **ORM:** Drizzle ORM
- **LLM:** Google Gemini (gemini-1.5-flash)
- **Embeddings:** text-embedding-004
- **AI SDK:** Vercel AI SDK

## 🚀 How to Run

### Prerequisites

- Node.js 18+
- pnpm
- Docker (for PostgreSQL)
- Google Gemini API Key

### Setup

\`\`\`bash

# 1. Clone repository

git clone https://github.com/carloswimmer/chatbot-youtube-music.git
cd chatbot-youtube-music

# 2. Install dependencies

pnpm install

# 3. Configure environment variables

cp .env.example .env.local

# Edit .env.local and add your GOOGLE_GENERATIVE_AI_API_KEY

# 4. Start PostgreSQL

docker-compose up -d

# 5. Run migrations

pnpm db:generate
pnpm db:migrate

# 6. Populate database with sample data

pnpm dev # In another terminal:
pnpm populate

# 7. Access http://localhost:3000

\`\`\`

## 📚 Concepts Learned

- ✅ RAG (Retrieval-Augmented Generation)
- ✅ Vector Embeddings and Similarity Search
- ✅ PostgreSQL with pgvector
- ✅ Drizzle ORM
- ✅ Google Gemini API
- ✅ Next.js 14 App Router
- ✅ Vercel AI SDK
- ✅ Response streaming
- ✅ Advanced TypeScript

## 🎯 Next Steps

- [ ] Add authentication
- [ ] YouTube Music API integration
- [ ] Favorites system
- [ ] Conversation history
- [ ] Automated tests
- [ ] CI/CD with GitHub Actions

## 📄 License

MIT - Carlos Wimmer
\`\`\`

### 10.5 Next Learning Steps

After completing this project, you can:

1. **Improve knowledge base:**
   - Integrate with YouTube Music API
   - Web scraping for song lyrics
   - Add complete album information

2. **Add features:**
   - Authentication with NextAuth.js
   - Persistent conversation history
   - Music recommendation system
   - Lyrics sentiment analysis

3. **Optimizations:**
   - Embeddings cache (Redis)
   - Context compression (LangChain)
   - Results reranking
   - Fine-tune thresholds

4. **Testing:**
   - Unit tests (Vitest)
   - Integration tests (Playwright)
   - E2E tests
   - Performance benchmarks

5. **DevOps:**
   - CI/CD with GitHub Actions
   - Monitoring (Sentry)
   - Analytics (PostHog)
   - Feature flags

---

## 📋 Command Summary

### During Development

```bash
# Start development server
pnpm dev

# Generate migrations
pnpm db:generate

# Apply migrations
pnpm db:migrate

# View database
pnpm db:studio

# Populate database with data
pnpm populate

# Clear database
curl -X DELETE http://localhost:3000/api/populate

# Production build
pnpm build

# Start production
pnpm start
```
````

### Docker

```bash
# Start PostgreSQL
docker-compose up -d

# View logs
docker-compose logs -f

# Stop
docker-compose down

# Reset everything (WARNING: deletes data)
docker-compose down -v
```

---

## 🎓 Key Concepts for Review

### 1. RAG (Retrieval-Augmented Generation)

- Combines information search + text generation
- Prevents LLM "hallucinations"
- Based on verified data

### 2. Embeddings

- Vector representations of text
- Captures semantic meaning
- Enables similarity search

### 3. Cosine Similarity

- Measures angle between vectors
- 0 = very different, 1 = identical
- Threshold: 0.5 (filter)

### 4. pgvector

- PostgreSQL extension for vectors
- HNSW index for fast search
- Built-in distance operators

### 5. Streaming

- Real-time response
- Better UX
- Lower perceived latency

### 6. Server Actions

- Functions that run on server
- Type-safe
- `'use server'` marker

---

## 🙏 Credits

Study project based on chatbot-guest-faq, adapted for learning RAG, embeddings and LLMs concepts.

**Author:** Carlos Wimmer  
**GitHub:** https://github.com/carloswimmer  
**Date:** October 2025

---

## 📞 Support

If you have questions during development:

1. Review this document
2. Check official documentation:
   - [Next.js](https://nextjs.org/docs)
   - [Vercel AI SDK](https://sdk.vercel.ai/docs)
   - [Drizzle ORM](https://orm.drizzle.team/docs)
   - [Google Gemini](https://ai.google.dev/docs)
3. Project GitHub Issues

---

## ✅ Progress Checklist

Mark as you complete each step:

### Setup (Section 3)

- [ ] 3.1 - Next.js project created
- [ ] 3.2 - Dependencies installed
- [ ] 3.3 - shadcn-ui configured
- [ ] 3.4 - Environment variables configured
- [ ] 3.5 - Gemini API Key obtained

### Database (Section 4)

- [ ] 4.1 - PostgreSQL running in Docker
- [ ] 4.2 - Drizzle configured
- [ ] 4.3 - Schemas created (resources, embeddings)
- [ ] 4.4 - DB connection tested
- [ ] 4.5 - Migrations executed successfully
- [ ] 4.6 - Drizzle Studio accessible

### Embeddings (Section 5)

- [ ] 5.1 - Embeddings system implemented
- [ ] 5.2 - Similarity search function working
- [ ] 5.3 - Embeddings tests successful

### Ingestion API (Section 6)

- [ ] 6.1 - Server Actions created
- [ ] 6.2 - `/api/populate` API working
- [ ] 6.3 - Population script executed
- [ ] 6.4 - Sample data in database

### Chat API (Section 7)

- [ ] 7.1 - `/api/chat` API implemented
- [ ] 7.2 - RAG working correctly
- [ ] 7.3 - Error handling implemented
- [ ] 7.4 - Response streaming working

### Interface (Section 8)

- [ ] 8.1 - Chat component created
- [ ] 8.2 - Responsive and beautiful UI
- [ ] 8.3 - Question suggestions working
- [ ] 8.4 - Loading states implemented

### Gemini (Section 9)

- [ ] 9.1 - Gemini integration working
- [ ] 9.2 - Models configured correctly
- [ ] 9.3 - Usage tracker implemented
- [ ] 9.4 - Rate limiting working

### Deploy (Section 10)

- [ ] 10.1 - Project ready for production
- [ ] 10.2 - Deployed to Vercel
- [ ] 10.3 - GitHub repository created
- [ ] 10.4 - Complete README.md

---

**Good luck with your studies! 🚀🎵**
