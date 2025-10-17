# AVIBA - Backend Architecture Document

## 1. Ikhtisar Arsitektur Backend

Backend AVIBA dibangun menggunakan Next.js 15 dengan App Router, menyediakan API modern yang scalable dan performan. Arsitektur ini mengikuti best practices untuk security, performance, dan maintainability.

### 1.1 Core Technologies
- **Runtime**: Node.js 18+ dengan Next.js 15
- **API Routes**: Next.js App Router API routes
- **Database**: PostgreSQL dengan connection pooling
- **ORM**: Drizzle ORM dengan TypeScript
- **Authentication**: Better Auth dengan session management
- **File Storage**: AWS S3 atau Cloudinary
- **Caching**: Redis (opsional untuk production)
- **AI Integration**: Google Gemini 2.5 Flash API

### 1.2 Framework and Patterns
- Next.js App Router handles routing, server-side rendering (SSR), dan API endpoints dalam struktur terpadu.
- Server Components dan Client Components memisahkan concerns rendering UI dan data fetching.
- Struktur direktori modular (`/app`, `/api`, `/db`, `/lib`) mengorganisir kode berdasarkan fitur.

### 1.3 Key Libraries
- Better Auth: mengelola sign-up, sign-in, session creation, dan validation.
- Drizzle ORM: menyediakan type-safe database interactions.
- TypeScript: menegakkan types di seluruh backend code, mengurangi runtime errors.

### 1.4 Scalability and Performance
- Stateless API routes membuat horizontal scaling straightforward (auto-scaling di Vercel).
- Connection pooling di Drizzle memastikan penggunaan efisien database connections.
- API routes masa depan dapat diperkenalkan di bawah `/app/api` tanpa mengganggu code yang ada.

### 1.5 Maintainability
- Clearly separated concerns (authentication, database, utilities) memungkinkan tim bekerja paralel.
- Type-safe schemas dan shared types antara frontend dan backend meminimalkan mismatches.
- Docker configuration untuk database mengisolasi environment differences.

## 2. Project Structure Backend

```
backend/
├── app/
│   ├── api/                    # API Routes
│   │   ├── auth/              # Authentication endpoints
│   │   │   ├── login/route.ts
│   │   │   ├── logout/route.ts
│   │   │   ├── register/route.ts
│   │   │   └── session/route.ts
│   │   ├── archives/          # Archive management
│   │   │   ├── route.ts       # GET /api/archives
│   │   │   ├── [id]/route.ts  # GET/PUT/DELETE /api/archives/[id]
│   │   │   └── search/route.ts # POST /api/archives/search
│   │   ├── media/             # Media management
│   │   │   ├── route.ts       # GET/POST /api/media
│   │   │   ├── upload/route.ts # POST /api/media/upload
│   │   │   └── [id]/route.ts  # GET/DELETE /api/media/[id]
│   │   ├── eras/              # Historical eras
│   │   │   └── route.ts       # GET /api/eras
│   │   ├── categories/        # Content categories
│   │   │   └── route.ts       # GET /api/categories
│   │   ├── analytics/         # Analytics & tracking
│   │   │   ├── route.ts       # GET /api/analytics
│   │   │   ├── track/route.ts # POST /api/analytics/track
│   │   │   └── export/route.ts # GET /api/analytics/export
│   │   ├── chat/              # AI Chat Assistant
│   │   │   ├── route.ts       # POST /api/chat (streaming)
│   │   │   └── history/route.ts # GET/POST /api/chat/history
│   │   ├── search/            # Search functionality
│   │   │   ├── route.ts       # GET /api/search
│   │   │   └── suggestions/route.ts # GET /api/search/suggestions
│   │   └── admin/             # Admin-only endpoints
│   │       ├── archives/      # Admin archive management
│   │       ├── media/         # Admin media management
│   │       ├── users/         # User management
│   │       └── system/        # System configuration
│   └── dashboard/             # Server-side pages
│       ├── page.tsx           # Admin dashboard
│       ├── archives/          # Archive management pages
│       ├── media/             # Media management pages
│       └── analytics/         # Analytics pages
├── lib/
│   ├── db/
│   │   ├── index.ts           # Database connection
│   │   ├── schema.ts          # Drizzle schema definitions
│   │   ├── migrations/        # Database migrations
│   │   └── seed.ts            # Database seeding
│   ├── auth/
│   │   ├── config.ts          # Better Auth configuration
│   │   ├── middleware.ts      # Auth middleware
│   │   └── utils.ts           # Auth helper functions
│   ├── storage/
│   │   ├── s3.ts              # AWS S3 integration
│   │   ├── cloudinary.ts      # Cloudinary integration
│   │   └── local.ts           # Local file storage
│   ├── ai/
│   │   ├── gemini.ts          # Gemini AI integration
│   │   ├── prompts.ts         # AI prompt templates
│   │   └── context.ts         # Context building for AI
│   ├── utils/
│   │   ├── validation.ts      # Input validation schemas
│   │   ├── pagination.ts      # Pagination utilities
│   │   ├── cache.ts           # Caching utilities
│   │   ├── logger.ts          # Logging configuration
│   │   └── errors.ts          # Error handling utilities
│   ├── middleware/
│   │   ├── auth.ts            # Authentication middleware
│   │   ├── cors.ts            # CORS configuration
│   │   ├── rate-limit.ts      # Rate limiting
│   │   └── analytics.ts       # Analytics tracking
│   └── types/
│       ├── api.ts             # API type definitions
│       ├── database.ts        # Database type definitions
│       └── auth.ts            # Auth type definitions
├── actions/
│   ├── archives.ts            # Archive CRUD actions
│   ├── media.ts               # Media management actions
│   ├── analytics.ts           # Analytics actions
│   ├── search.ts              # Search actions
│   └── admin.ts               # Admin-specific actions
├── config/
│   ├── database.ts            # Database configuration
│   ├── storage.ts             # Storage configuration
│   ├── ai.ts                  # AI service configuration
│   └── analytics.ts           # Analytics configuration
├── middleware.ts              # Global middleware
├── env.ts                     # Environment validation
└── error.ts                   # Global error handler
```

## 3. Database Management

### 3.1 Database Configuration
- **Database Type**: SQL (PostgreSQL). Dipilih karena reliability, maturity, dan strong ecosystem.
- **ORM**: Drizzle ORM handles schema definition, type-safe queries, dan migrations.
- **Connection**: Connection pooling untuk optimal performance.

### 3.2 Data Flow
- API routes memanggil Drizzle queries untuk membaca/menulis data.
- Session data dipersist di sessions table, user credentials di users table.
- Dalam development, PostgreSQL berjalan di dalam Docker container untuk consistency.

### 3.3 Best Practices
- Gunakan parameterized queries untuk mencegah SQL injection (handled oleh Drizzle).
- Simpan credentials dan connection strings di environment variables.
- Index fields yang sering difilter (contoh: `session_token`, `archives.slug`).

## 4. Database Schema

### 4.1 Core AVIBA Schema (SQL)

```sql
-- Archives table - Main content storage
CREATE TABLE archives (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    content TEXT,
    era_id UUID NOT NULL REFERENCES eras(id) ON DELETE RESTRICT,
    category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
    location_lat DECIMAL(10, 8),
    location_lng DECIMAL(11, 8),
    location_name VARCHAR(255),
    date_created DATE,
    view_count INTEGER DEFAULT 0,
    like_count INTEGER DEFAULT 0,
    is_published BOOLEAN DEFAULT FALSE,
    is_featured BOOLEAN DEFAULT FALSE,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL
);

-- Media assets table
CREATE TABLE media (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    archive_id UUID NOT NULL REFERENCES archives(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL, -- 'image', 'video', 'audio', 'document', '360'
    title VARCHAR(255),
    url VARCHAR(500) NOT NULL,
    alt_text TEXT,
    file_size BIGINT,
    mime_type VARCHAR(100),
    width INTEGER,
    height INTEGER,
    duration INTEGER, -- seconds for audio/video
    thumbnail_url VARCHAR(500),
    is_featured BOOLEAN DEFAULT FALSE,
    sort_order INTEGER DEFAULT 0,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Historical eras table
CREATE TABLE eras (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    period_start DATE NOT NULL,
    period_end DATE,
    color VARCHAR(7) DEFAULT '#6B7280',
    cover_image_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT 0,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Categories table
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    icon VARCHAR(50),
    color VARCHAR(7) DEFAULT '#6B7280',
    parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Analytics table for tracking
CREATE TABLE analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    archive_id UUID REFERENCES archives(id) ON DELETE CASCADE,
    event_type VARCHAR(20) NOT NULL DEFAULT 'view',
    view_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    user_agent TEXT,
    ip_address INET,
    referrer TEXT,
    session_id VARCHAR(255),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    duration_seconds INTEGER,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Tags table
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7),
    usage_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Archive tags junction table
CREATE TABLE archive_tags (
    archive_id UUID NOT NULL REFERENCES archives(id) ON DELETE CASCADE,
    tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (archive_id, tag_id)
);

-- Authentication tables (Better Auth)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    role VARCHAR(20) NOT NULL DEFAULT 'user',
    avatar_url VARCHAR(500),
    bio TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    last_login TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) UNIQUE NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Chat conversations for AI assistant
CREATE TABLE chat_conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id VARCHAR(255) NOT NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    message JSONB NOT NULL,
    context_archive_ids UUID[],
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 4.2 Indexes for Performance

```sql
-- Performance indexes
CREATE INDEX idx_archives_slug ON archives(slug);
CREATE INDEX idx_archives_era_id ON archives(era_id);
CREATE INDEX idx_archives_category_id ON archives(category_id);
CREATE INDEX idx_archives_is_published ON archives(is_published);
CREATE INDEX idx_archives_view_count ON archives(view_count DESC);
CREATE INDEX idx_archives_date_created ON archives(date_created);

CREATE INDEX idx_media_archive_id ON media(archive_id);
CREATE INDEX idx_media_type ON media(type);
CREATE INDEX idx_media_is_featured ON media(is_featured);

CREATE INDEX idx_analytics_archive_id ON analytics(archive_id);
CREATE INDEX idx_analytics_view_date ON analytics(view_date DESC);
CREATE INDEX idx_analytics_event_type ON analytics(event_type);

-- Full-text search index
CREATE INDEX idx_archives_search ON archives USING GIN(
    to_tsvector('indonesian', title || ' ' || COALESCE(description, '') || ' ' || COALESCE(content, ''))
);
```

### 4.3 Schema Notes:
- `archives.slug` unique untuk SEO-friendly URLs.
- `archives.is_published` mengontrol visibility konten.
- `archives.view_count` untuk analytics tracking.
- Foreign key constraints ensure referential integrity.
- `metadata` JSONB columns untuk fleksibilitas tambahan.
- Full-text search index untuk performa pencarian.

## 5. API Design and Endpoints

### 5.1 Core API Routes Structure

All backend logic is exposed via Next.js API routes following a RESTful style.

#### Public API Routes
- **Authentication** (`/app/api/auth/`)
  - `POST /api/auth/login` - User login
  - `POST /api/auth/logout` - User logout  
  - `POST /api/auth/register` - User registration
  - `GET /api/auth/session` - Get current session

- **Archives** (`/app/api/archives/`)
  - `GET /api/archives` - List archives with pagination
  - `GET /api/archives/[slug]` - Get archive detail by slug
  - `POST /api/archives/search` - Advanced search with filters

- **Content Discovery** (`/app/api/`)
  - `GET /api/eras` - List historical eras
  - `GET /api/categories` - List content categories
  - `GET /api/search` - Global search functionality
  - `GET /api/search/suggestions` - Search autocomplete

- **Analytics** (`/app/api/analytics/`)
  - `POST /api/analytics/track` - Track page views and interactions

#### Admin API Routes
- **Admin Archives** (`/app/api/admin/archives/`)
  - `GET /api/admin/archives` - Admin archive list with full data
  - `POST /api/admin/archives` - Create new archive
  - `PUT /api/admin/archives/[id]` - Update existing archive
  - `DELETE /api/admin/archives/[id]` - Delete archive

- **Media Management** (`/app/api/admin/media/`)
  - `POST /api/admin/media/upload` - Upload media files
  - `GET /api/admin/media/[id]` - Get media details
  - `DELETE /api/admin/media/[id]` - Delete media file

- **AI Chat** (`/app/api/chat/`)
  - `POST /api/chat` - AI chat streaming endpoint
  - `GET /api/chat/history` - Get chat history

### 5.2 Design Principles
- Gunakan HTTP verbs secara semantik: `GET`, `POST`, `PUT`, `DELETE`
- Return JSON responses dengan struktur konsisten: `{ success: boolean, data?: any, error?: string }`
- Session-based authentication untuk protected routes
- Rate limiting untuk mencegah abuse
- Proper HTTP status codes
- Input validation untuk semua endpoints
- Error handling yang comprehensive

### 5.3 Response Format Standard

```typescript
// Success Response
{
  "success": true,
  "data": {
    // actual data
  },
  "meta": {
    // pagination, timestamps, etc.
  }
}

// Error Response  
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": {
    // additional error details
  }
}
```

## 6. Hosting Solutions & Infrastructure

### 6.1 Production Hosting

#### Frontend & Backend
- **Next.js on Vercel (Recommended)**
  - Automatic deployments dari Git
  - Built-in edge network untuk fast SSR response times
  - Zero-config scaling dan SSL certificates
  - Serverless functions untuk API routes
  - Built-in analytics dan monitoring

#### Database Hosting
- **Development**: PostgreSQL di Docker (via `docker-compose.yml`)
- **Production Options**:
  - Vercel Postgres (Recommended untuk kemudahan)
  - AWS RDS PostgreSQL
  - Supabase PostgreSQL
  - Railway PostgreSQL
  - DigitalOcean Managed Database

#### File Storage
- **AWS S3** (Recommended)
  - Scalable object storage
  - CDN integration dengan CloudFront
  - Lifecycle policies untuk cost optimization
- **Alternatives**: Cloudinary, ImageKit, Vercel Blob

#### Additional Services
- **Redis** (Optional untuk caching):
  - Upstash Redis (Serverless)
  - AWS ElastiCache
- **AI Service**: Google Gemini API
- **Email Service**: Resend, SendGrid (untuk notifications)

### 6.2 Infrastructure Components

#### Load Balancer
- Disediakan oleh Vercel's edge network yang mendistribusikan requests across serverless function instances

#### Caching Strategy
- **Next.js ISR** (Incremental Static Regeneration) untuk public data endpoints
- **Browser caching** untuk static assets via CDN
- **Redis caching** untuk frequently accessed data
- **Database query caching** dengan connection pooling

#### Content Delivery Network (CDN)
- Vercel's global CDN serves static assets (images, scripts) dekat dengan end users
- CloudFront untuk S3 assets jika menggunakan AWS

#### Containerization
- Docker Compose untuk local PostgreSQL memastikan consistent development environments
- Docker opsional untuk deployment ke platform lain

### 6.3 Benefits
- **Reliability**: Vercel dan managed DB providers menawarkan high SLAs
- **Scalability**: Serverless functions scale dengan traffic; databases dapat scale vertically atau horizontally  
- **Cost-effectiveness**: Pay untuk what you use, minimal maintenance overhead
- **Performance**: Global CDN dan edge computing untuk low latency
- **Security**: Managed security updates dan DDoS protection

## 7. Security Measures

### 7.1 Authentication & Authorization
- **Better Auth** mengelola secure password hashing dan session tokens
- Protected routes check session validity sebelum returning data
- Role-based access control (Admin, Editor, User)
- Session management dengan secure cookies

### 7.2 Data Encryption & Protection
- **TLS/HTTPS** enforced di semua network traffic
- Database credentials secured di environment variables, tidak di code
- Sensitive data encryption di database level
- File encryption untuk sensitive media storage

### 7.3 Input Validation & Sanitization
- **Drizzle ORM** prevents SQL injection
- **Zod schemas** untuk API input validation
- XSS prevention dengan proper output encoding
- File upload validation dan scanning

### 7.4 Additional Security Hardening
- **HTTP Security Headers** (CSP, HSTS, XSS protection)
- **Rate limiting** untuk mencegah brute-force attacks
- **CORS configuration** yang proper
- Regular security updates dan dependency audits
- Environment-specific security configurations
- API key management untuk external services

### 7.5 Monitoring & Logging
- Security event logging
- Failed login attempt tracking
- Suspicious activity detection
- Regular security scans and penetration testing

## 8. Monitoring and Maintenance

### 8.1 Monitoring Tools
- **Vercel Analytics** untuk performance metrics (latency, throughput)
- **Vercel Speed Insights** untuk Core Web Vitals
- **Sentry** untuk error tracking dan performance monitoring
- **Custom logging** dengan structured JSON logs
- **Database monitoring** untuk query performance

### 8.2 Alerts and Dashboards
- Setup alerts untuk error rates, slow responses, dan database connection issues
- Use dashboards untuk track core web vitals dan API health
- Real-time monitoring untuk critical system components
- Automated alerting untuk security events

### 8.3 Maintenance Practices
- **Automated CI/CD pipeline** (GitHub Actions) untuk linting, tests, dan deployments
- **Periodic dependency audits** dan security scans (Dependabot)
- **Database migrations** managed melalui Drizzle's CLI
- **Regular backups** dan disaster recovery testing
- **Performance optimization** dan database query analysis
- **Log rotation** dan archival strategies

### 8.4 Development Workflow
- **Feature branches** dengan Pull Request reviews
- **Automated testing** (unit, integration, e2e)
- **Staging environment** untuk pre-production testing
- **Database seeding** untuk development consistency
- **Environment parity** antara development, staging, dan production

## 9. Performance Optimization

### 9.1 Backend Performance
- **Connection pooling** untuk database connections
- **Query optimization** dengan proper indexing
- **Caching strategies** (Redis, Next.js cache)
- **Lazy loading** untuk data fetching
- **Compression** untuk API responses

### 9.2 API Performance
- **Pagination** untuk large datasets
- **Rate limiting** untuk resource management
- **Request validation** untuk early error detection
- **Response compression** (gzip/brotli)
- **CDN caching** untuk static responses

## 10. Scalability Considerations

### 10.1 Database Scalability
- **Read replicas** untuk high-read workloads
- **Connection pooling** untuk efficient resource usage
- **Query optimization** dengan proper indexing
- **Data archiving** untuk historical data

### 10.2 Application Scalability
- **Serverless functions** untuk automatic scaling
- **Horizontal scaling** dengan load balancing
- **Microservices architecture** untuk future growth
- **Event-driven architecture** untuk async processing

## 11. Conclusion and Overall Backend Summary

Backend AVIBA menawarkan arsitektur yang komprehensif dan modern untuk aplikasi arsip digital Banyuwangi. Keunggulan utamanya meliputi:

### 11.1 Core Strengths
- **Unified Next.js environment** di mana frontend dan backend hidup berdampingan
- **Type-safe database interactions** melalui Drizzle ORM dan PostgreSQL
- **Secure authentication** flows powered oleh Better Auth
- **Scalable hosting** di Vercel dengan managed services
- **AI integration** dengan Google Gemini 2.5 Flash
- **Comprehensive media management** dengan cloud storage

### 11.2 Technical Excellence
- **Modern stack** dengan Next.js 15, TypeScript, dan PostgreSQL
- **Security-first approach** dengan multiple layers of protection
- **Performance optimization** dengan caching dan CDN
- **Scalable architecture** yang siap untuk growth
- **Comprehensive monitoring** dan error handling

### 11.3 Developer Experience
- **Clear separation of concerns** untuk parallel development
- **Type safety** di seluruh stack
- **Comprehensive documentation** dan best practices
- **Automated workflows** untuk CI/CD
- **Development consistency** dengan Docker dan standardization

Setup ini memungkinkan tim untuk fokus pada building features daripada wrestling dengan infrastructure. Dengan clear boundaries antara components, robust security measures, dan scalable hosting model, backend siap untuk support real-world applications dan grow sesuai kebutuhan.

Backend AVIBA dirancang untuk menjadi fondasi yang kuat untuk aplikasi arsip digital yang menampilkan sejarah Banyuwangi dengan performa tinggi, security yang ketat, dan user experience yang excellent.