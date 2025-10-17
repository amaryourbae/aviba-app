# AVIBA - Spesifikasi Teknis Aplikasi Web Arsip Digital Virtual Banyuwangi

## 1. Ikhtisar Proyek

AVIBA (Arsip Digital Virtual Banyuwangi) adalah aplikasi web berbasis arsip digital yang menampilkan sejarah Banyuwangi dari era kerajaan, masa kolonial, hingga saat ini. Aplikasi ini mengintegrasikan berbagai format konten multimedia dengan asisten AI cerdas untuk pengalaman eksplorasi sejarah yang interaktif.

## 2. Teknologi Stack

### Frontend
- **Framework**: Next.js 15 (App Router)
- **UI Components**: shadcn/ui
- **Styling**: Tailwind CSS
- **State Management**: React Hooks & Context API
- **Form Handling**: React Hook Form
- **Notifications**: SweetAlert 2
- **File Upload**: react-dropzone

### Backend
- **Runtime**: Node.js (Next.js API Routes)
- **Database**: PostgreSQL
- **ORM**: Drizzle ORM
- **Authentication**: Better Auth
- **File Storage**: Cloud Storage (AWS S3 atau alternatif)
- **AI Integration**: Google Gemini 2.5 Flash API

### DevOps & Infrastructure
- **Deployment**: Vercel (Recommended) atau Docker
- **Database Hosting**: Supabase, Railway, atau AWS RDS
- **CDN**: CloudFront atau Vercel Edge Network
- **Monitoring**: Built-in Analytics + Custom Dashboard

## 3. Fitur Inti

### 3.1 Manajemen Konten Arsip
- **CRUD Operations**: Create, Read, Update, Delete untuk arsip sejarah
- **Kategorisasi**: Era historis (Kerajaan, Kolonial, Modern, Kontemporer)
- **Tagging**: Tags tematik untuk konten yang terkait
- **Metadata**: Informasi lengkap setiap arsip (tanggal, sumber, deskripsi)

### 3.2 Manajemen Media
- **Multi-format Support**:
  - Foto (JPG, PNG, WebP)
  - Video (MP4, WebM)
  - Gambar 360° (equirectangular format)
  - Audio (MP3, WAV, OGG)
  - Dokumen (PDF, TXT)
  - Lokasi Bersejarah (Maps Integration)
- **File Processing**: Otomatisasi thumbnail, kompresi, dan optimasi
- **Cloud Storage**: Penyimpanan terdistribusi dengan CDN

### 3.3 Portal Publik
- **Browse & Search**: Pencarian teks lengkap dengan filter
- **Detail View**: Tampilan mendetail dengan gallery media
- **360° Viewer**: Viewer interaktif untuk gambar panoramik
- **Audio/Video Player**: Integrated multimedia player
- **Location Maps**: Peta interaktif untuk lokasi bersejarah

### 3.4 Admin Dashboard
- **Analytics Overview**: Statistik penggunaan dan konten populer
- **Content Management**: Interface CRUD untuk semua konten
- **Media Management**: Upload, organize, dan manage file
- **User Management**: Manajemen pengguna dan permissions

### 3.5 AI Chat Assistant
- **Gemini 2.5 Integration**: Chat AI untuk penjelasan sejarah
- **Context-Aware**: Responses berdasarkan database arsip
- **Recommendations**: Rekomendasi konten terkait
- **Streaming**: Real-time chat dengan streaming responses

## 4. Alur Pengguna (User Flow)

### 4.1 Public User Flow
1. **Landing Page** → Browse archives atau search
2. **Archive Listing** → Filter berdasarkan era, kategori, media
3. **Archive Detail** → View konten lengkap, gallery, lokasi
4. **AI Assistant** → Chat untuk penjelasan lebih lanjut
5. **Related Content** → Navigasi ke arsip terkait

### 4.2 Admin User Flow
1. **Login** → Authentication via Better Auth
2. **Dashboard** → View analytics dan quick stats
3. **Content Management** → CRUD operations untuk arsip
4. **Media Upload** → Upload dan organize media files
5. **Analytics** → Monitor popularitas dan engagement

## 5. Skema Database (Drizzle ORM)

### 5.1 Core Tables

```typescript
// Archives Table
archives: {
  id: string (uuid, primary key)
  title: string (varchar, 255)
  description: text
  slug: string (varchar, 255, unique)
  era_id: string (foreign key → eras.id)
  category_id: string (foreign key → categories.id)
  content: text (rich text content)
  location_lat: decimal (optional)
  location_lng: decimal (optional)
  location_name: string (optional)
  date_created: date
  view_count: integer (default: 0)
  is_published: boolean (default: false)
  created_at: timestamp
  updated_at: timestamp
}

// Media Assets Table
media: {
  id: string (uuid, primary key)
  archive_id: string (foreign key → archives.id)
  type: enum ('image', 'video', 'audio', 'document', '360')
  title: string (varchar, 255)
  url: string (varchar, 500)
  alt_text: text (optional)
  file_size: bigint
  mime_type: string (varchar, 100)
  width: integer (optional, for images/videos)
  height: integer (optional, for images/videos)
  duration: integer (optional, for audio/videos in seconds)
  is_featured: boolean (default: false)
  sort_order: integer (default: 0)
  created_at: timestamp
}

// Historical Eras Table
eras: {
  id: string (uuid, primary key)
  name: string (varchar, 100)
  slug: string (varchar, 100, unique)
  description: text
  period_start: date
  period_end: date (optional, for ongoing eras)
  color: string (varchar, 7) // hex color for UI
  sort_order: integer (default: 0)
  is_active: boolean (default: true)
  created_at: timestamp
}

// Categories Table
categories: {
  id: string (uuid, primary key)
  name: string (varchar, 100)
  slug: string (varchar, 100, unique)
  description: text
  icon: string (varchar, 50) // icon name
  color: string (varchar, 7) // hex color for UI
  sort_order: integer (default: 0)
  is_active: boolean (default: true)
  created_at: timestamp
}

// Analytics Table
analytics: {
  id: string (uuid, primary key)
  archive_id: string (foreign key → archives.id)
  view_date: timestamp
  user_agent: text (optional)
  ip_address: string (varchar, 45) // supports IPv6
  referrer: text (optional)
  session_id: string (varchar, 255, optional)
  created_at: timestamp
}

// Tags Table
tags: {
  id: string (uuid, primary key)
  name: string (varchar, 50, unique)
  slug: string (varchar, 50, unique)
  color: string (varchar, 7, optional)
  created_at: timestamp
}

// Archive-Tag Relationship (Many-to-Many)
archive_tags: {
  archive_id: string (foreign key → archives.id)
  tag_id: string (foreign key → tags.id)
  created_at: timestamp
}
```

### 5.2 Indexes & Performance
- **Primary Keys**: UUID dengan auto-generation
- **Search Indexes**: Full-text search pada `archives.title` dan `archives.description`
- **Foreign Key Indexes**: Optimize JOIN operations
- **Analytics Indexes**: Time-based queries untuk reporting
- **Slug Indexes**: Unique indexes untuk SEO-friendly URLs

## 6. API Routes Structure

### 6.1 Public API Routes
```
GET  /api/archives          - List archives dengan pagination
GET  /api/archives/[slug]   - Detail archive
GET  /api/eras              - List historical eras
GET  /api/categories        - List categories
GET  /api/search?q=keyword  - Search functionality
POST /api/analytics         - Track page views
GET  /api/chat              - AI Chat streaming endpoint
```

### 6.2 Admin API Routes
```
GET    /api/admin/archives        - Admin archive list
POST   /api/admin/archives        - Create new archive
PUT    /api/admin/archives/[id]   - Update archive
DELETE /api/admin/archives/[id]   - Delete archive
POST   /api/admin/upload          - File upload
GET    /api/admin/analytics       - Analytics data
```

## 7. Frontend Architecture

### 7.1 Page Structure
```
app/
├── (auth)/
│   ├── login/
│   └── register/
├── dashboard/
│   ├── page.tsx                 - Admin dashboard
│   ├── archives/
│   │   ├── page.tsx            - Archive list (CRUD)
│   │   ├── new/page.tsx        - Create archive
│   │   └── [id]/edit/page.tsx  - Edit archive
│   ├── media/
│   └── analytics/
├── archive/
│   ├── page.tsx                - Public archive listing
│   ├── [slug]/page.tsx         - Archive detail view
│   └── era/[slug]/page.tsx     - Era-specific browsing
├── api/                        - API routes
└── globals.css
```

### 7.2 Component Structure
```
components/
├── ui/                         - shadcn/ui components
├── shared/
│   ├── Header.tsx
│   ├── Footer.tsx
│   ├── Navigation.tsx
│   └── SearchBar.tsx
├── archive/
│   ├── ArchiveCard.tsx
│   ├── ArchiveGrid.tsx
│   ├── MediaGallery.tsx
│   ├── MediaPlayer.tsx
│   └── LocationMap.tsx
├── admin/
│   ├── ArchiveForm.tsx
│   ├── MediaUpload.tsx
│   ├── AnalyticsCards.tsx
│   └── DataTable.tsx
└── chat/
    ├── ChatInterface.tsx
    ├── ChatMessage.tsx
    └── ChatInput.tsx
```

## 8. Security & Authentication

### 8.1 Authentication System
- **Better Auth Integration**: Secure authentication
- **Session Management**: JWT tokens dengan refresh mechanism
- **Role-Based Access**: Admin vs Public user permissions
- **API Protection**: Middleware untuk protected routes

### 8.2 Security Measures
- **File Upload Validation**: Type checking, size limits, virus scanning
- **SQL Injection Prevention**: Parameterized queries via Drizzle ORM
- **XSS Protection**: Input sanitization dan CSP headers
- **Rate Limiting**: API rate limiting untuk abuse prevention
- **CORS Configuration**: Proper cross-origin resource sharing

## 9. Performance Optimization

### 9.1 Frontend Optimization
- **Code Splitting**: Dynamic imports untuk routes
- **Image Optimization**: Next.js Image component dengan CDN
- **Lazy Loading**: Component dan media lazy loading
- **Caching**: SWR atau React Query untuk data fetching

### 9.2 Backend Optimization
- **Database Optimization**: Indexes dan query optimization
- **Caching Strategy**: Redis untuk frequently accessed data
- **CDN Integration**: Static assets via CDN
- **API Response Caching**: Cache headers untuk API responses

## 10. Analytics & Monitoring

### 10.1 Custom Analytics
- **Page Views**: Track arsip popularity
- **User Engagement**: Time spent, media interactions
- **Search Analytics**: Popular search terms
- **Geographic Data**: User location distribution

### 10.2 Admin Dashboard Metrics
- **Content Overview**: Total archives per era/category
- **Popular Content**: Most viewed archives
- **Upload Statistics**: Media upload trends
- **User Activity**: Admin login dan activity tracking

## 11. Deployment & Infrastructure

### 11.1 Environment Configuration
```bash
# Database
DATABASE_URL=postgresql://...
NEXT_PUBLIC_APP_URL=https://your-domain.com

# Authentication
BETTER_AUTH_SECRET=your-secret-key
BETTER_AUTH_URL=https://your-domain.com

# AI Integration
GEMINI_API_KEY=your-gemini-api-key

# File Storage
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=...
AWS_S3_BUCKET=...
```

### 11.2 Deployment Strategy
- **Staging Environment**: Pre-production testing
- **Production Deployment**: Zero-downtime deployment
- **Database Migrations**: Automated migration via Drizzle
- **Backup Strategy**: Regular database backups

## 12. Development Workflow

### 12.1 Local Development
```bash
# Setup
npm install
cp .env.example .env.local

# Database
npm run db:generate
npm run db:migrate
npm run db:seed

# Development
npm run dev
```

### 12.2 Quality Assurance
- **TypeScript**: Strict type checking
- **ESLint**: Code quality enforcement
- **Prettier**: Code formatting
- **Testing**: Unit dan integration tests
- **Code Review**: Pull request process

## 13. Future Enhancements

### 13.1 Phase 2 Features
- **Virtual Tours**: 3D reconstruction dari situs bersejarah
- **AR Integration**: Augmented reality experiences
- **Mobile App**: React Native mobile application
- **Multi-language Support**: Internationalisasi
- **User Generated Content**: Community contributions

### 13.3 Technical Roadmap
- **Progressive Web App**: PWA capabilities
- **Offline Support**: Service workers untuk offline browsing
- **Advanced Search**: Semantic search dan AI-powered recommendations
- **API Versioning**: RESTful API dengan versioning
- **GraphQL Integration**: Alternative API interface

---

## Timeline Estimasi

- **Phase 1 (MVP)**: 6-8 minggu
  - Database setup dan basic CRUD
  - Public portal dengan search
  - Basic admin dashboard
  - Media upload management

- **Phase 2 (AI Integration)**: 3-4 minggu
  - Gemini AI integration
  - Chat assistant development
  - Enhanced analytics

- **Phase 3 (Advanced Features)**: 4-6 minggu
  - 360° viewer
  - Location maps
  - Advanced filtering
  - Performance optimization

Total estimasi development: **13-18 minggu** untuk complete solution.