# AVIBA Database Schema - Spesifikasi Lengkap

## 1. Ikhtisar Database

Database AVIBA menggunakan PostgreSQL dengan Drizzle ORM sebagai layer abstraksi. Desain database mengikuti prinsip normalization yang baik dengan performa optimal untuk query kompleks.

## 2. Struktur Database Lengkap

### 2.1 Tabel `archives` - Tabel Utama Arsip

```sql
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
    location_address TEXT,
    date_created DATE,
    date_end DATE, -- untuk peristiwa yang berjangka waktu
    view_count INTEGER DEFAULT 0,
    like_count INTEGER DEFAULT 0,
    is_published BOOLEAN DEFAULT FALSE,
    is_featured BOOLEAN DEFAULT FALSE,
    metadata JSONB, -- untuk fleksibilitas tambahan
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    created_by UUID REFERENCES users(id) ON DELETE SET NULL,
    updated_by UUID REFERENCES users(id) ON DELETE SET NULL
);

-- Indexes untuk performa
CREATE INDEX idx_archives_era_id ON archives(era_id);
CREATE INDEX idx_archives_category_id ON archives(category_id);
CREATE INDEX idx_archives_is_published ON archives(is_published);
CREATE INDEX idx_archives_view_count ON archives(view_count DESC);
CREATE INDEX idx_archives_date_created ON archives(date_created);
CREATE INDEX idx_archives_location ON archives USING GIST(point(location_lng, location_lat)) WHERE location_lat IS NOT NULL;

-- Full-text search index
CREATE INDEX idx_archives_search ON archives USING GIN(
    to_tsvector('indonesian', title || ' ' || COALESCE(description, '') || ' ' || COALESCE(content, ''))
);
```

### 2.2 Tabel `media` - Aset Media

```sql
CREATE TABLE media (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    archive_id UUID NOT NULL REFERENCES archives(id) ON DELETE CASCADE,
    type media_type NOT NULL,
    title VARCHAR(255),
    description TEXT,
    url VARCHAR(500) NOT NULL,
    alt_text TEXT,
    file_size BIGINT,
    mime_type VARCHAR(100),
    width INTEGER,
    height INTEGER,
    duration INTEGER, -- dalam detik untuk audio/video
    thumbnail_url VARCHAR(500), -- untuk video/gambar besar
    is_featured BOOLEAN DEFAULT FALSE,
    sort_order INTEGER DEFAULT 0,
    metadata JSONB, -- EXIF data, video info, dll
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enum untuk media_type
CREATE TYPE media_type AS ENUM (
    'image', 'video', 'audio', 'document', '360_image', '360_video', 'map', 'other'
);

-- Indexes
CREATE INDEX idx_media_archive_id ON media(archive_id);
CREATE INDEX idx_media_type ON media(type);
CREATE INDEX idx_media_is_featured ON media(is_featured);
CREATE INDEX idx_media_sort_order ON media(archive_id, sort_order);
```

### 2.3 Tabel `eras` - Era Sejarah

```sql
CREATE TABLE eras (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    period_start DATE NOT NULL,
    period_end DATE, -- NULL untuk era yang masih berlanjut
    color VARCHAR(7) DEFAULT '#6B7280', -- hex color
    cover_image_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT 0,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_eras_is_active ON eras(is_active);
CREATE INDEX idx_eras_sort_order ON eras(sort_order);
CREATE INDEX idx_eras_period ON eras(period_start, period_end);
```

### 2.4 Tabel `categories` - Kategori Konten

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    icon VARCHAR(50), -- nama icon dari library
    color VARCHAR(7) DEFAULT '#6B7280',
    parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_categories_parent_id ON categories(parent_id);
CREATE INDEX idx_categories_is_active ON categories(is_active);
CREATE INDEX idx_categories_sort_order ON categories(sort_order);
```

### 2.5 Tabel `tags` - Tag untuk Kategorisasi Fleksibel

```sql
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7),
    usage_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_tags_usage_count ON tags(usage_count DESC);
CREATE INDEX idx_tags_name ON tags(name);
```

### 2.6 Tabel Hubung `archive_tags` (Many-to-Many)

```sql
CREATE TABLE archive_tags (
    archive_id UUID NOT NULL REFERENCES archives(id) ON DELETE CASCADE,
    tag_id UUID NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    PRIMARY KEY (archive_id, tag_id)
);

-- Indexes
CREATE INDEX idx_archive_tags_tag_id ON archive_tags(tag_id);
```

### 2.7 Tabel `analytics` - Tracking Penggunaan

```sql
CREATE TABLE analytics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    archive_id UUID REFERENCES archives(id) ON DELETE CASCADE,
    event_type analytics_event NOT NULL DEFAULT 'view',
    view_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    user_agent TEXT,
    ip_address INET,
    referrer TEXT,
    session_id VARCHAR(255),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    duration_seconds INTEGER, -- waktu yang dihabiskan di halaman
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enum untuk event types
CREATE TYPE analytics_event AS ENUM (
    'view', 'like', 'share', 'download', 'chat_interaction', 'search'
);

-- Indexes untuk performa analytics
CREATE INDEX idx_analytics_archive_id ON analytics(archive_id);
CREATE INDEX idx_analytics_view_date ON analytics(view_date DESC);
CREATE INDEX idx_analytics_event_type ON analytics(event_type);
CREATE INDEX idx_analytics_user_id ON analytics(user_id);
CREATE INDEX idx_analytics_session_id ON analytics(session_id);
```

### 2.8 Tabel `users` - Manajemen Pengguna

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    role user_role NOT NULL DEFAULT 'user',
    avatar_url VARCHAR(500),
    bio TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    email_verified BOOLEAN DEFAULT FALSE,
    last_login TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enum untuk user roles
CREATE TYPE user_role AS ENUM (
    'user', 'admin', 'editor', 'moderator'
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_is_active ON users(is_active);
```

### 2.9 Tabel `search_logs` - Log Pencarian

```sql
CREATE TABLE search_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    query VARCHAR(255) NOT NULL,
    results_count INTEGER,
    filters JSONB, -- filter yang digunakan
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_search_logs_query ON search_logs(query);
CREATE INDEX idx_search_logs_created_at ON search_logs(created_at DESC);
```

### 2.10 Tabel `chat_conversations` - Riwayat Chat AI

```sql
CREATE TABLE chat_conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id VARCHAR(255) NOT NULL,
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    message JSONB NOT NULL, -- {role: 'user'/'assistant', content: string}
    context_archive_ids UUID[], -- arsip yang direferensikan
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_chat_conversations_session_id ON chat_conversations(session_id);
CREATE INDEX idx_chat_conversations_user_id ON chat_conversations(user_id);
CREATE INDEX idx_chat_conversations_created_at ON chat_conversations(created_at DESC);
```

## 3. Drizzle ORM Schema Implementation

### 3.1 Schema Types Definition

```typescript
// lib/db/schema.ts
import { 
  pgTable, 
  uuid, 
  varchar, 
  text, 
  integer, 
  boolean, 
  timestamp, 
  decimal, 
  jsonb,
  index,
  primaryKey
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// Enums
export const userRoleEnum = pgEnum('user_role', ['user', 'admin', 'editor', 'moderator']);
export const mediaTypeEnum = pgEnum('media_type', ['image', 'video', 'audio', 'document', '360_image', '360_video', 'map', 'other']);
export const analyticsEventEnum = pgEnum('analytics_event', ['view', 'like', 'share', 'download', 'chat_interaction', 'search']);

// Users Table
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 255 }),
  role: userRoleEnum('role').notNull().default('user'),
  avatarUrl: varchar('avatar_url', { length: 500 }),
  bio: text('bio'),
  isActive: boolean('is_active').notNull().default(true),
  emailVerified: boolean('email_verified').notNull().default(false),
  lastLogin: timestamp('last_login', { withTimezone: true }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  emailIdx: index('idx_users_email').on(table.email),
  roleIdx: index('idx_users_role').on(table.role),
  isActiveIdx: index('idx_users_is_active').on(table.isActive)
}));

// Eras Table
export const eras = pgTable('eras', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 100 }).notNull(),
  slug: varchar('slug', { length: 100 }).notNull().unique(),
  description: text('description'),
  periodStart: timestamp('period_start', { mode: 'date' }).notNull(),
  periodEnd: timestamp('period_end', { mode: 'date' }),
  color: varchar('color', { length: 7 }).default('#6B7280'),
  coverImageUrl: varchar('cover_image_url', { length: 500 }),
  isActive: boolean('is_active').notNull().default(true),
  sortOrder: integer('sort_order').notNull().default(0),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  isActiveIdx: index('idx_eras_is_active').on(table.isActive),
  sortOrderIdx: index('idx_eras_sort_order').on(table.sortOrder),
  periodIdx: index('idx_eras_period').on(table.periodStart, table.periodEnd)
}));

// Categories Table
export const categories = pgTable('categories', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 100 }).notNull(),
  slug: varchar('slug', { length: 100 }).notNull().unique(),
  description: text('description'),
  icon: varchar('icon', { length: 50 }),
  color: varchar('color', { length: 7 }).default('#6B7280'),
  parentId: uuid('parent_id').references(() => categories.id, { onDelete: 'set null' }),
  isActive: boolean('is_active').notNull().default(true),
  sortOrder: integer('sort_order').notNull().default(0),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  parentIdIdx: index('idx_categories_parent_id').on(table.parentId),
  isActiveIdx: index('idx_categories_is_active').on(table.isActive),
  sortOrderIdx: index('idx_categories_sort_order').on(table.sortOrder)
}));

// Archives Table
export const archives = pgTable('archives', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: varchar('title', { length: 255 }).notNull(),
  slug: varchar('slug', { length: 255 }).notNull().unique(),
  description: text('description'),
  content: text('content'),
  eraId: uuid('era_id').notNull().references(() => eras.id, { onDelete: 'restrict' }),
  categoryId: uuid('category_id').notNull().references(() => categories.id, { onDelete: 'restrict' }),
  locationLat: decimal('location_lat', { precision: 10, scale: 8 }),
  locationLng: decimal('location_lng', { precision: 11, scale: 8 }),
  locationName: varchar('location_name', { length: 255 }),
  locationAddress: text('location_address'),
  dateCreated: timestamp('date_created', { mode: 'date' }),
  dateEnd: timestamp('date_end', { mode: 'date' }),
  viewCount: integer('view_count').notNull().default(0),
  likeCount: integer('like_count').notNull().default(0),
  isPublished: boolean('is_published').notNull().default(false),
  isFeatured: boolean('is_featured').notNull().default(false),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
  createdBy: uuid('created_by').references(() => users.id, { onDelete: 'set null' }),
  updatedBy: uuid('updated_by').references(() => users.id, { onDelete: 'set null' })
}, (table) => ({
  eraIdIdx: index('idx_archives_era_id').on(table.eraId),
  categoryIdIdx: index('idx_archives_category_id').on(table.categoryId),
  isPublishedIdx: index('idx_archives_is_published').on(table.isPublished),
  viewCountIdx: index('idx_archives_view_count').on(table.viewCount),
  dateCreatedIdx: index('idx_archives_date_created').on(table.dateCreated)
}));

// Media Table
export const media = pgTable('media', {
  id: uuid('id').primaryKey().defaultRandom(),
  archiveId: uuid('archive_id').notNull().references(() => archives.id, { onDelete: 'cascade' }),
  type: mediaTypeEnum('type').notNull(),
  title: varchar('title', { length: 255 }),
  description: text('description'),
  url: varchar('url', { length: 500 }).notNull(),
  altText: text('alt_text'),
  fileSize: integer('file_size'), // in bytes
  mimeType: varchar('mime_type', { length: 100 }),
  width: integer('width'),
  height: integer('height'),
  duration: integer('duration'), // in seconds
  thumbnailUrl: varchar('thumbnail_url', { length: 500 }),
  isFeatured: boolean('is_featured').notNull().default(false),
  sortOrder: integer('sort_order').notNull().default(0),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  archiveIdIdx: index('idx_media_archive_id').on(table.archiveId),
  typeIdx: index('idx_media_type').on(table.type),
  isFeaturedIdx: index('idx_media_is_featured').on(table.isFeatured),
  sortOrderIdx: index('idx_media_sort_order').on(table.archiveId, table.sortOrder)
}));

// Tags Table
export const tags = pgTable('tags', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 50 }).notNull().unique(),
  slug: varchar('slug', { length: 50 }).notNull().unique(),
  description: text('description'),
  color: varchar('color', { length: 7 }),
  usageCount: integer('usage_count').notNull().default(0),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  usageCountIdx: index('idx_tags_usage_count').on(table.usageCount),
  nameIdx: index('idx_tags_name').on(table.name)
}));

// Archive Tags Junction Table
export const archiveTags = pgTable('archive_tags', {
  archiveId: uuid('archive_id').notNull().references(() => archives.id, { onDelete: 'cascade' }),
  tagId: uuid('tag_id').notNull().references(() => tags.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  pk: primaryKey({ columns: [table.archiveId, table.tagId] }),
  tagIdIdx: index('idx_archive_tags_tag_id').on(table.tagId)
}));

// Analytics Table
export const analytics = pgTable('analytics', {
  id: uuid('id').primaryKey().defaultRandom(),
  archiveId: uuid('archive_id').references(() => archives.id, { onDelete: 'cascade' }),
  eventType: analyticsEventEnum('event_type').notNull().default('view'),
  viewDate: timestamp('view_date', { withTimezone: true }).notNull().defaultNow(),
  userAgent: text('user_agent'),
  ipAddress: varchar('ip_address', { length: 45 }),
  referrer: text('referrer'),
  sessionId: varchar('session_id', { length: 255 }),
  userId: uuid('user_id').references(() => users.id, { onDelete: 'set null' }),
  durationSeconds: integer('duration_seconds'),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow()
}, (table) => ({
  archiveIdIdx: index('idx_analytics_archive_id').on(table.archiveId),
  viewDateIdx: index('idx_analytics_view_date').on(table.viewDate),
  eventTypeIdx: index('idx_analytics_event_type').on(table.eventType),
  userIdIdx: index('idx_analytics_user_id').on(table.userId),
  sessionIdIdx: index('idx_analytics_session_id').on(table.sessionId)
}));

// Relations
export const archivesRelations = relations(archives, ({ one, many }) => ({
  era: one(eras, {
    fields: [archives.eraId],
    references: [eras.id]
  }),
  category: one(categories, {
    fields: [archives.categoryId],
    references: [categories.id]
  }),
  media: many(media),
  tags: many(tags, { relationName: 'archiveTags' }),
  analytics: many(analytics),
  createdBy: one(users, {
    fields: [archives.createdBy],
    references: [users.id]
  }),
  updatedBy: one(users, {
    fields: [archives.updatedBy],
    references: [users.id]
  })
}));

export const mediaRelations = relations(media, ({ one }) => ({
  archive: one(archives, {
    fields: [media.archiveId],
    references: [archives.id]
  })
}));

export const tagsRelations = relations(tags, ({ many }) => ({
  archives: many(archives, { relationName: 'archiveTags' })
}));
```

## 4. Database Migration Strategy

### 4.1 Migration Commands

```bash
# Generate migration from schema changes
npm run db:generate

# Run migrations
npm run db:migrate

# Drop and recreate (development only)
npm run db:push

# Seed initial data
npm run db:seed
```

### 4.2 Initial Seed Data

```typescript
// lib/db/seed.ts
import { db } from './index';
import { eras, categories, tags } from './schema';

const seedData = async () => {
  // Seed Historical Eras
  await db.insert(eras).values([
    {
      name: 'Era Kerajaan Blambangan',
      slug: 'era-kerajaan-blambangan',
      description: 'Periode kejayaan Kerajaan Blambangan sebelum kolonialisasi',
      periodStart: new Date('1200-01-01'),
      periodEnd: new Date('1743-12-31'),
      color: '#8B4513',
      sortOrder: 1
    },
    {
      name: 'Masa Kolonial Belanda',
      slug: 'masa-kolonial-belanda',
      description: 'Periode penjajahan Belanda di Banyuwangi',
      periodStart: new Date('1743-01-01'),
      periodEnd: new Date('1942-12-31'),
      color: '#4169E1',
      sortOrder: 2
    },
    {
      name: 'Era Kemerdekaan',
      slug: 'era-kemerdekaan',
      description: 'Periode pasca-kemerdekaan Indonesia',
      periodStart: new Date('1945-01-01'),
      periodEnd: new Date('2000-12-31'),
      color: '#228B22',
      sortOrder: 3
    },
    {
      name: 'Era Modern',
      slug: 'era-modern',
      description: 'Banyuwangi di era modern dan kontemporer',
      periodStart: new Date('2000-01-01'),
      periodEnd: null, // ongoing
      color: '#FF6347',
      sortOrder: 4
    }
  ]);

  // Seed Categories
  await db.insert(categories).values([
    {
      name: 'Sejarah',
      slug: 'sejarah',
      description: 'Dokumen dan artefak bersejarah',
      icon: 'Book',
      color: '#8B4513',
      sortOrder: 1
    },
    {
      name: 'Budaya',
      slug: 'budaya',
      description: 'Warisan budaya dan tradisi',
      icon: 'Music',
      color: '#FF6347',
      sortOrder: 2
    },
    {
      name: 'Tokoh',
      slug: 'tokoh',
      description: 'Biografi tokoh penting Banyuwangi',
      icon: 'Users',
      color: '#4169E1',
      sortOrder: 3
    },
    {
      name: 'Arsitektur',
      slug: 'arsitektur',
      description: 'Bangunan bersejarah dan arsitektur tradisional',
      icon: 'Building',
      color: '#228B22',
      sortOrder: 4
    },
    {
      name: 'Lokasi',
      slug: 'lokasi',
      description: 'Tempat-tempat bersejarah',
      icon: 'MapPin',
      color: '#9370DB',
      sortOrder: 5
    }
  ]);

  // Seed Tags
  await db.insert(tags).values([
    { name: 'blambangan', slug: 'blambangan', color: '#8B4513' },
    { name: 'kerajaan', slug: 'kerajaan', color: '#FFD700' },
    { name: 'kolonial', slug: 'kolonial', color: '#4169E1' },
    { name: 'kemerdekaan', slug: 'kemerdekaan', color: '#228B22' },
    { name: 'tradisional', slug: 'tradisional', color: '#FF6347' },
    { name: 'pesisir', slug: 'pesisir', color: '#00CED1' },
    { name: 'pertanian', slug: 'pertanian', color: '#228B22' },
    { name: 'kesenian', slug: 'kesenian', color: '#9370DB' }
  ]);
};
```

## 5. Performance Optimization

### 5.1 Query Optimization

```typescript
// Efficient queries with proper indexing
export const getArchivesWithPagination = async (
  page: number = 1,
  limit: number = 12,
  filters?: {
    eraId?: string;
    categoryId?: string;
    search?: string;
  }
) => {
  const offset = (page - 1) * limit;
  
  let query = db
    .select({
      id: archives.id,
      title: archives.title,
      slug: archives.slug,
      description: archives.description,
      dateCreated: archives.dateCreated,
      viewCount: archives.viewCount,
      isFeatured: archives.isFeatured,
      era: {
        id: eras.id,
        name: eras.name,
        color: eras.color
      },
      category: {
        id: categories.id,
        name: categories.name,
        icon: categories.icon
      },
      featuredMedia: {
        url: media.url,
        type: media.type
      }
    })
    .from(archives)
    .leftJoin(eras, eq(archives.eraId, eras.id))
    .leftJoin(categories, eq(archives.categoryId, categories.id))
    .leftJoin(media, and(
      eq(media.archiveId, archives.id),
      eq(media.isFeatured, true)
    ))
    .where(eq(archives.isPublished, true))
    .orderBy(desc(archives.isFeatured), desc(archives.viewCount))
    .limit(limit)
    .offset(offset);

  if (filters?.eraId) {
    query = query.where(eq(archives.eraId, filters.eraId));
  }
  
  if (filters?.categoryId) {
    query = query.where(eq(archives.categoryId, filters.categoryId));
  }
  
  if (filters?.search) {
    query = query.where(
      or(
        ilike(archives.title, `%${filters.search}%`),
        ilike(archives.description, `%${filters.search}%`)
      )
    );
  }

  return await query;
};
```

### 5.2 Caching Strategy

```typescript
// lib/cache.ts
import { cache } from 'react';

// Cache frequently accessed data
export const getCachedEras = cache(async () => {
  return await db.select().from(eras).where(eq(eras.isActive, true)).orderBy(eras.sortOrder);
});

export const getCachedCategories = cache(async () => {
  return await db.select().from(categories).where(eq(categories.isActive, true)).orderBy(categories.sortOrder);
});

export const getCachedPopularTags = cache(async (limit: number = 20) => {
  return await db
    .select()
    .from(tags)
    .orderBy(desc(tags.usageCount))
    .limit(limit);
});
```

## 6. Backup & Recovery

### 6.1 Backup Strategy

```bash
# Daily backups
pg_dump -h localhost -U postgres -d aviba_db > backup_$(date +%Y%m%d).sql

# Compressed backup
pg_dump -h localhost -U postgres -d aviba_db | gzip > backup_$(date +%Y%m%d).sql.gz

# Restore
psql -h localhost -U postgres -d aviba_db < backup_20231201.sql
```

### 6.2 Data Retention

```sql
-- Archive old analytics data (keep 2 years)
DELETE FROM analytics 
WHERE created_at < NOW() - INTERVAL '2 years';

-- Archive old search logs (keep 1 year)
DELETE FROM search_logs 
WHERE created_at < NOW() - INTERVAL '1 year';
```

## 7. Monitoring & Maintenance

### 7.1 Database Health Checks

```sql
-- Table sizes
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Index usage
SELECT 
  indexrelname,
  idx_tup_read,
  idx_tup_fetch,
  idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

Database schema ini dirancang untuk skalabilitas, performa, dan fleksibilitas dalam mengelola arsip digital Banyuwangi dengan berbagai jenis konten multimedia.