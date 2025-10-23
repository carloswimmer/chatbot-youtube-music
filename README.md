# 🎵 Music Bot - RAG-Powered Music Chatbot

> An intelligent chatbot that uses **RAG (Retrieval-Augmented Generation)** and **Google Gemini** to provide accurate information about YouTube Music songs.

[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue)](https://www.typescriptlang.org/)
[![Google Gemini](https://img.shields.io/badge/Google-Gemini-orange)](https://ai.google.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📖 About the Project

This project is a **conversational chatbot** that combines semantic search with natural language generation to answer questions about music. Unlike traditional chatbots that may "hallucinate" information, this one uses **RAG** to ensure responses are based only on verified data.

### 🎯 Goals

A study project to learn and apply advanced concepts of:
- 🧠 RAG (Retrieval-Augmented Generation)
- 🔢 Vector Embeddings and Semantic Search
- 🤖 LLM Integration (Google Gemini)
- 📊 Vector Databases (pgvector)

---

## ✨ Features

- 💬 **Real-time chat** with streaming responses
- 🔍 **Semantic search** using vector embeddings
- 🎸 **Knowledge base** of classic and popular songs
- 🤖 **Powered by Google Gemini** (1.5 Flash)
- 🎨 **Modern and responsive** interface
- ⚡ **Fast and accurate** responses
- 🚫 **No hallucinations** - only database information

---

## 🛠️ Tech Stack

### Frontend
- **Next.js 14** - React framework with App Router
- **React** - UI library
- **TailwindCSS** - Utility-first CSS
- **shadcn/ui** - Modern UI components
- **Lucide React** - Icons

### Backend
- **Next.js API Routes** - REST endpoints
- **Vercel AI SDK** - LLM abstraction
- **Google Gemini** - LLM for text generation
- **Drizzle ORM** - TypeScript-first ORM

### Database
- **PostgreSQL 16** - Relational database
- **pgvector** - Vector search extension
- **Docker** - Containerization

### AI & Embeddings
- **Google Gemini API** - gemini-1.5-flash-latest
- **text-embedding-004** - Embedding model (768 dimensions)
- **Cosine Distance** - Similarity metric

### Validation & Utils
- **Zod** - TypeScript schema validation
- **nanoid** - Unique ID generation

---

## 🏗️ Architecture

### RAG Flow (Retrieval-Augmented Generation)

```
┌─────────────────────────────────────────────────────┐
│  1. User asks a question about a song               │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  2. System generates question embedding (768D)      │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  3. Search PostgreSQL for similarity                │
│     (Cosine Distance < 0.5)                         │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  4. Returns top 4 most relevant songs               │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  5. Gemini generates response using context         │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  6. Stream response to user                         │
└─────────────────────────────────────────────────────┘
```

### Database Structure

```sql
-- Resources table (song information)
resources
  ├── id: varchar(191) PK
  ├── content: text (complete song description)
  ├── song_id: varchar(191) (unique identifier)
  ├── created_at: timestamp
  └── updated_at: timestamp

-- Embeddings table (vectors for semantic search)
embeddings
  ├── id: varchar(191) PK
  ├── resource_id: varchar(191) FK → resources.id
  └── embedding: vector(768) (768-dimension vector)
```

---

## 🚀 Getting Started

### Prerequisites

Make sure you have installed:
- **Node.js** 18.17.0 or higher
- **pnpm** 8.0.0 or higher
- **Docker** 20.10.0 or higher
- **Google Gemini API Key** ([Get it here](https://makersuite.google.com/app/apikey))

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/carloswimmer/chatbot-youtube-music.git
cd chatbot-youtube-music

# 2. Install dependencies
pnpm install

# 3. Configure environment variables
cp .env.example .env.local

# 4. Edit .env.local and add your credentials:
# DATABASE_URL=postgresql://postgres:password@localhost:5432/musicbot
# GOOGLE_GENERATIVE_AI_API_KEY=your-key-here

# 5. Start PostgreSQL with Docker
docker-compose up -d

# 6. Run database migrations
pnpm db:generate
pnpm db:migrate

# 7. Start the development server
pnpm dev

# 8. In another terminal, populate the database with sample data
pnpm populate

# 9. Access http://localhost:3000
```

### Available Commands

```bash
# Development
pnpm dev              # Start development server
pnpm build            # Production build
pnpm start            # Start production server
pnpm lint             # Run linter

# Database
pnpm db:generate      # Generate migration files
pnpm db:migrate       # Apply migrations to database
pnpm db:push          # Push schema directly (dev only)
pnpm db:studio        # Open Drizzle Studio

# Data
pnpm populate         # Populate database with sample data
```

### Docker

```bash
# Start PostgreSQL
docker-compose up -d

# View logs
docker-compose logs -f postgres

# Stop services
docker-compose down

# Reset everything (WARNING: deletes all data)
docker-compose down -v
```

---

## 📚 Applied Concepts

### 1. RAG (Retrieval-Augmented Generation)

Technique that combines:
- **Retrieval**: Search for relevant information in a knowledge base
- **Generation**: Generate responses using an LLM

**Advantages:**
- ✅ Responses based on verified data
- ✅ Reduces LLM hallucinations
- ✅ Always up-to-date context
- ✅ Source traceability

### 2. Vector Embeddings

Numerical representations of text that capture semantic meaning:

```javascript
"upbeat happy song" → [0.234, -0.567, 0.891, ...]
"cheerful energetic tune" → [0.221, -0.543, 0.876, ...]
// Very close vectors = similar meanings
```

### 3. Similarity Search

Similarity search using **Cosine Distance**:
- Distance of 0 = identical texts
- Distance of 1 = completely different texts
- Threshold: 0.5 (filters out less relevant results)

### 4. pgvector

PostgreSQL extension that adds:
- `vector` data type
- Distance operators (cosine, euclidean, inner product)
- HNSW index for fast vector search

---

## 🎓 What I Learned

This project allowed me to learn and practice:

- ✅ How RAG works in practice
- ✅ Vector embeddings generation and usage
- ✅ Google Gemini API integration
- ✅ Semantic search with pgvector
- ✅ Next.js 14 App Router and Server Actions
- ✅ Drizzle ORM with PostgreSQL
- ✅ LLM response streaming
- ✅ Zod validation
- ✅ Robust error handling
- ✅ Rate limiting
- ✅ Docker for development

---

## 🔮 Next Steps

Planned features for future iterations:

### Short Term
- [ ] YouTube Music API integration for real data
- [ ] User authentication system
- [ ] Persistent conversation history
- [ ] Light/dark theme

### Medium Term
- [ ] Favorite songs system
- [ ] Export conversations to PDF
- [ ] Advanced search with filters
- [ ] Personalized recommendations

### Long Term
- [ ] Lyrics sentiment analysis
- [ ] Spotify integration
- [ ] Collaborative playlist system
- [ ] Automated tests (E2E, Unit, Integration)
- [ ] CI/CD with GitHub Actions
- [ ] Monitoring and analytics

---

## 🗂️ Project Structure

```
chatbot-youtube-music/
├── app/                          # Next.js App Router
│   ├── api/
│   │   ├── chat/
│   │   │   └── route.ts         # RAG chat API
│   │   └── populate/
│   │       └── route.ts         # Data ingestion API
│   ├── layout.tsx               # Main layout
│   ├── page.tsx                 # Chat interface
│   └── globals.css              # Global styles
├── components/
│   └── ui/                      # shadcn/ui components
├── lib/
│   ├── ai/
│   │   └── embedding.ts         # Embeddings system
│   ├── db/
│   │   ├── index.ts            # Database connection
│   │   ├── migrate.ts          # Migration script
│   │   └── schema/
│   │       ├── resources.ts    # Resources schema
│   │       └── embeddings.ts   # Embeddings schema
│   └── actions/
│       └── resources.ts         # Server Actions
├── scripts/
│   └── populate-sample-data.ts  # Sample data
├── docker-compose.yml           # Docker config
├── drizzle.config.ts           # Drizzle config
├── package.json
├── tsconfig.json
├── .env.local                  # Environment variables
└── README.md
```

---

## 🔑 Environment Variables

Create a `.env.local` file in the project root:

```bash
# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/musicbot

# Google Gemini API
GOOGLE_GENERATIVE_AI_API_KEY=your-key-here

# Next.js
NODE_ENV=development
```

---

## 🤝 Contributing

This is a personal study project, but suggestions and feedback are always welcome!

1. Fork the project
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 Author

**Carlos Wimmer**

- GitHub: [@carloswimmer](https://github.com/carloswimmer)
- LinkedIn: [Carlos Wimmer](https://www.linkedin.com/in/carloswimmer)
- Email: your-email@example.com

---

## 🙏 Acknowledgments

- **Google Gemini** for the free and powerful API
- **Vercel** for the amazing AI SDK
- **Drizzle Team** for the TypeScript-first ORM
- **shadcn** for the beautiful UI components
- **chatbot-guest-faq** project as inspiration

---

## 📚 References

- [Next.js Documentation](https://nextjs.org/docs)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)
- [Google Gemini API](https://ai.google.dev/docs)
- [Drizzle ORM](https://orm.drizzle.team/docs)
- [pgvector](https://github.com/pgvector/pgvector)
- [RAG Paper](https://arxiv.org/abs/2005.11401)

---

<div align="center">
  <p>Built with ❤️ and ☕ by Carlos Wimmer</p>
  <p>October 2025</p>
</div>

