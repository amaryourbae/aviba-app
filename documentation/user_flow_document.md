# AVIBA - Alur Pengguna (User Flow) Detail

## 1. Ikhtisar Alur Pengguna

AVIBA memiliki dua tipe pengguna utama dengan alur yang berbeda:
- **Pengguna Publik** (Public Users) - Menjelajahi konten tanpa login
- **Pengguna Admin** (Admin Users) - Mengelola konten dengan autentikasi

## 2. Alur Pengguna Publik

### 2.1 Landing Page Flow

```
[User Landing] → [Hero Section] → [Browse Options] → [Navigation Choice]
     ↓
[Quick Search] → [Search Results] → [Archive Detail] → [Related Content]
     ↓
[Categories] → [Category Filter] → [Filtered Archives] → [Archive Detail]
     ↓
[Eras Timeline] → [Era Selection] → [Era Archives] → [Archive Detail]
     ↓
[Featured Content] → [Archive Detail] → [Media Gallery] → [AI Assistant]
```

**Halaman: `/` (Landing Page)**

**Komponen:**
- Hero section dengan pencarian cepat
- Era timeline interaktif
- Kategori grid dengan ikon
- Featured archives carousel
- Statistics cards (total archives, media types)
- AI Assistant preview

**Interaksi:**
1. **Search Bar**: Real-time search dengan suggestions
2. **Era Timeline**: Visual timeline untuk navigasi era
3. **Category Cards**: Klik untuk filter berdasarkan kategori
4. **Featured Content**: Swipeable carousel untuk konten unggulan

### 2.2 Browse Archives Flow

```
[Archive Listing] → [Filter Options] → [Apply Filters] → [Sorted Results]
     ↓                    ↓              ↓               ↓
[Search Input]    → [Era Filter] → [Category Filter] → [Media Type Filter]
     ↓                    ↓              ↓               ↓
[Instant Search]  → [Tag Filter] → [Date Range]    → [Sort Options]
     ↓                    ↓              ↓               ↓
[Results Update]  → [Pagination] → [Load More]    → [View Details]
```

**Halaman: `/archive` (Archive Listing)**

**Fitur Pencarian & Filter:**
- **Search**: Teks lengkap dengan highlight hasil
- **Era Filter**: Checkbox untuk multi-select era
- **Category Filter**: Visual category selector
- **Media Type**: Filter berdasarkan jenis media
- **Date Range**: Slider rentang waktu
- **Tags**: Tag cloud dengan click-to-filter
- **Sort Options**: Terpopuler, Terbaru, Abjad, Paling Banyak Dilihat

**Component Breakdown:**
```typescript
// components/archive/ArchiveBrowser.tsx
interface ArchiveBrowserProps {
  initialFilters?: ArchiveFilters;
  searchQuery?: string;
}

interface ArchiveFilters {
  eras: string[];
  categories: string[];
  mediaTypes: MediaType[];
  tags: string[];
  dateRange?: {
    start: Date;
    end: Date;
  };
  sortBy: 'popular' | 'newest' | 'alphabetical' | 'views';
}
```

### 2.3 Archive Detail Flow

```
[Archive Detail] → [Content Sections] → [Media Interactions] → [Related Actions]
       ↓                  ↓                   ↓                   ↓
[Header Info]    → [Main Content]   → [Media Gallery]   → [Share/Like]
       ↓                  ↓                   ↓                   ↓
[Meta Info]     → [Location Map]   → [360° Viewer]    → [AI Chat]
       ↓                  ↓                   ↓                   ↓
[Navigation]    → [Audio Player]  → [Video Player]   → [Related Archives]
       ↓                  ↓                   ↓                   ↓
[Breadcrumbs]   → [Image Gallery] → [Document View]  → [External Links]
```

**Halaman: `/archive/[slug]` (Archive Detail)**

**Struktur Halaman:**
1. **Header Section**
   - Breadcrumb navigation
   - Archive title dengan era badge
   - Meta information (date, location, category)
   - Action buttons (Like, Share, Bookmark)

2. **Main Content**
   - Deskripsi lengkap dengan rich text formatting
   - Gallery media dengan viewer yang sesuai
   - Peta lokasi interaktif (jika ada)
   - Timeline konteks historis

3. **Media Gallery**
   - Grid view untuk semua media
   - Modal viewer untuk detail media
   - 360° image viewer untuk panoramik
   - Audio/video player dengan controls

4. **Sidebar/Bottom Section**
   - AI Assistant integration
   - Related archives
   - Tags dan metadata
   - Share options

### 2.4 AI Chat Assistant Flow

```
[Chat Interface] → [User Input] → [Processing] → [AI Response] → [Follow-up]
       ↓               ↓             ↓             ↓              ↓
[Chat History]  → [Text Input]  → [Context]    → [Streaming]   → [Suggestions]
       ↓               ↓             ↓             ↓              ↓
[Typing Indic.] → [Voice Input] → [Archive Ref] → [Media Links] → [Actions]
       ↓               ↓             ↓             ↓              ↓
[Quick Actions] → [File Upload] → [History]    → [Save Chat]   → [New Topic]
```

**Component: `components/chat/ChatAssistant.tsx`**

**Fitur Chat:**
- **Real-time Streaming**: Gemini 2.5 Flash responses
- **Context Awareness**: Referensi arsip dari database
- **Media Integration**: Sertakan media dalam responses
- **History**: Persistent chat history per session
- **Voice Input**: Speech-to-text untuk mobile
- **Quick Actions**: Predefined questions dan commands

### 2.5 Era-Specific Browsing Flow

```
[Era Landing] → [Era Overview] → [Featured Archives] → [Browse All]
      ↓               ↓                  ↓                    ↓
[Timeline]    → [Description] → [Hero Carousel] → [Archive Grid]
      ↓               ↓                  ↓                    ↓
[Key Events]  → [Gallery]     → [Statistics]    → [Filter Options]
      ↓               ↓                  ↓                    ↓
[Navigation]  → [Map View]    → [Related Eras]  → [Search Bar]
```

**Halaman: `/era/[slug]` (Era Detail)**

**Konten Era-Specific:**
- Timeline visualization dengan key events
- Statistik era (jumlah arsip, media types)
- Featured archives dari era tersebut
- Peta dengan lokasi-lokasi penting era
- Gallery media representative

## 3. Alur Pengguna Admin

### 3.1 Authentication Flow

```
[Login Page] → [Email Input] → [Password] → [Validation] → [Dashboard]
       ↓            ↓            ↓           ↓             ↓
[Form]    → [Validation] → [Show/Hide] → [Error/Succes] → [Redirect]
       ↓            ↓            ↓           ↓             ↓
[Remember] → [Forgot Pass] → [OAuth Opt.] → [2FA Check] → [Session]
```

**Halaman: `/login`**

**Security Features:**
- Email/password validation
- Rate limiting untuk login attempts
- Session management dengan secure cookies
- Multi-factor authentication (opsional)
- OAuth integration (Google, GitHub)

### 3.2 Dashboard Overview Flow

```
[Dashboard] → [Analytics Cards] → [Quick Actions] → [Recent Activity]
     ↓               ↓                    ↓               ↓
[Stats Grid] → [Charts & Graphs] → [Add Content] → [Activity Log]
     ↓               ↓                    ↓               ↓
[KPI Cards] → [Trend Analysis] → [Quick Links] → [Notifications]
     ↓               ↓                    ↓               ↓
[Date Range] → [Export Data]    → [Help/Docs]   → [User Profile]
```

**Halaman: `/dashboard`**

**Dashboard Components:**
1. **Statistics Cards**
   - Total archives
   - Total media files
   - Page views (harian/mingguan/bulanan)
   - Popular content

2. **Analytics Charts**
   - Visitor trends
   - Content performance
   - Search analytics
   - Geographic distribution

3. **Quick Actions**
   - Add new archive
   - Upload media
   - Manage users
   - View reports

4. **Recent Activity**
   - Recent uploads
   - User activity
   - System notifications
   - Content changes

### 3.3 Content Management Flow

```
[Archive List] → [View Options] → [Filter/Search] → [Select Action]
       ↓              ↓               ↓                 ↓
[Table View] → [Card View] → [Search Bar] → [Bulk Actions]
       ↓              ↓               ↓                 ↓
[Sorting]    → [Pagination] → [Advanced Filter] → [Individual Actions]
       ↓              ↓               ↓                 ↓
[Status]     → [Quick Edit] → [Export Data]    → [Create/Edit/Delete]
```

**Halaman: `/dashboard/archives`**

**Features:**
- **Data Table** dengan sorting, filtering, pagination
- **Bulk Operations** untuk multiple archives
- **Quick Edit** untuk fast updates
- **Status Management** (published/draft)
- **Preview Mode** untuk melihat tampilan publik

### 3.4 Create/Edit Archive Flow

```
[Archive Form] → [Basic Info] → [Content Editor] → [Media Management]
       ↓              ↓               ↓                    ↓
[Title/Slug] → [Description] → [Rich Text] → [Upload Media]
       ↓              ↓               ↓                    ↓
[Era/Category] → [Date/Location] → [Formatting] → [Media Gallery]
       ↓              ↓               ↓                    ↓
[Metadata] → [Tags Selection] → [Preview] → [Save/Publish]
       ↓              ↓               ↓                    ↓
[SEO Settings] → [Status Options] → [Validation] → [Redirect]
```

**Halaman: `/dashboard/archives/new` dan `/dashboard/archives/[id]/edit`**

**Form Sections:**
1. **Basic Information**
   - Title dengan auto-generate slug
   - Description dan content (rich text editor)
   - Era dan category selection
   - Date dan location information

2. **Media Management**
   - File upload dengan drag-and-drop
   - Media gallery dengan reordering
   - Metadata editing untuk setiap media
   - Featured media selection

3. **SEO & Metadata**
   - Meta description
   - Tags selection dengan autocomplete
   - Featured content options
   - Publishing status

### 3.5 Media Upload Flow

```
[Upload Page] → [File Selection] → [Processing] → [Metadata] → [Save]
       ↓              ↓                ↓             ↓           ↓
[Drag & Drop] → [File Validation] → [Compression] → [Auto-tag] → [Gallery]
       ↓              ↓                ↓             ↓           ↓
[Multiple Files] → [Progress Bar] → [Thumbnails] → [Manual Edit] → [Integration]
       ↓              ↓                ↓             ↓           ↓
[Type Detection] → [Error Handling] → [Cloud Storage] → [Preview] → [Success]
```

**Component: `components/admin/MediaUpload.tsx`**

**Upload Features:**
- **Drag & Drop Interface** dengan react-dropzone
- **Multiple File Support** dengan progress indicators
- **File Validation** (type, size, format)
- **Auto-compression** untuk images
- **Thumbnail Generation** untuk videos
- **Cloud Storage Integration** (AWS S3)
- **Bulk Operations** untuk multiple files

### 3.6 Analytics & Reporting Flow

```
[Analytics Dashboard] → [Time Period] → [Metrics Selection] → [Export]
          ↓                    ↓                 ↓               ↓
[Overview Cards] → [Date Range Picker] → [Chart Types] → [PDF/Excel]
          ↓                    ↓                 ↓               ↓
[Trend Charts] → [Comparison Mode] → [Drill Down] → [Email Report]
          ↓                    ↓                 ↓               ↓
[Top Content] → [User Demographics] → [Custom Reports] → [API Access]
```

**Halaman: `/dashboard/analytics`**

**Analytics Features:**
- **Real-time Statistics** visitor data
- **Content Performance** tracking
- **User Behavior Analysis**
- **Search Query Insights**
- **Geographic Distribution** maps
- **Custom Date Range** comparisons
- **Export Capabilities** untuk reporting

## 4. Error Handling & Edge Cases

### 4.1 Common Error Scenarios

```typescript
// Error boundary implementations
const ErrorHandling = {
  // Network errors
  NetworkError: {
    message: 'Koneksi internet terputus',
    action: 'Coba lagi',
    retry: true
  },
  
  // Not found errors
  NotFoundError: {
    message: 'Arsip tidak ditemukan',
    action: 'Kembali ke beranda',
    redirect: '/archive'
  },
  
  // Permission errors
  PermissionError: {
    message: 'Anda tidak memiliki akses',
    action: 'Login atau hubungi admin',
    login: true
  },
  
  // Validation errors
  ValidationError: {
    message: 'Data tidak valid',
    action: 'Periksa kembali input',
    focus: true
  }
};
```

### 4.2 Loading States

```typescript
// Loading skeleton components
const LoadingStates = {
  ArchiveList: 'archive-list-skeleton',
  ArchiveDetail: 'archive-detail-skeleton',
  MediaGallery: 'media-gallery-skeleton',
  ChatInterface: 'chat-interface-skeleton',
  Dashboard: 'dashboard-skeleton'
};
```

## 5. Mobile Responsiveness

### 5.1 Mobile-Specific Flows

```typescript
// Mobile navigation patterns
const MobileNavigation = {
  // Bottom tab bar for main navigation
  BottomTabs: ['Home', 'Browse', 'Search', 'Chat', 'Profile'],
  
  // Hamburger menu for secondary navigation
  SideMenu: ['Categories', 'Eras', 'About', 'Contact'],
  
  // Swipe gestures for gallery
  SwipeGestures: {
    Gallery: 'horizontal-swipe',
    Chat: 'vertical-scroll',
    Maps: 'pinch-zoom'
  }
};
```

### 5.2 Touch Interactions

- **Pull to Refresh** untuk archive lists
- **Infinite Scroll** untuk pagination
- **Touch Gestures** untuk media gallery
- **Voice Search** untuk mobile input
- **Haptic Feedback** untuk interactions

## 6. Accessibility Features

### 6.1 WCAG Compliance

```typescript
// Accessibility implementations
const AccessibilityFeatures = {
  // Keyboard navigation
  KeyboardNavigation: {
    TabIndex: 'logical-order',
    SkipLinks: 'content-navigation',
    FocusManagement: 'visible-focus'
  },
  
  // Screen reader support
  ScreenReader: {
    AriaLabels: 'descriptive-labels',
    LiveRegions: 'dynamic-content',
    SemanticHTML: 'proper-structure'
  },
  
  // Visual accessibility
  VisualAccessibility: {
    HighContrast: 'theme-support',
    FontSizing: 'scalable-text',
    ColorBlindness: 'friendly-palette'
  }
};
```

### 6.2 Multi-language Support

```typescript
// i18n implementation
const Internationalization = {
  defaultLocale: 'id',
  supportedLocales: ['id', 'en'],
  fallbackLocale: 'id',
  
  // Content translation
  contentTranslation: {
    archives: 'full-translation',
    categories: 'label-translation',
    ui: 'interface-translation'
  }
};
```

## 7. Performance Optimization

### 7.1 Loading Strategy

```typescript
// Progressive loading
const LoadingStrategy = {
  // Above the fold content
  CriticalContent: 'server-render',
  
  // Image loading
  Images: {
    LazyLoading: 'intersection-observer',
    Progressive: 'blur-up-effect',
    Responsive: 'srcset-sizes'
  },
  
  // Code splitting
  Components: 'dynamic-imports',
  
  // Data fetching
  DataFetching: {
    InitialLoad: 'ssr',
    Subsequent: 'client-side',
    Caching: 'swr-pattern'
  }
};
```

### 7.2 Caching Strategy

```typescript
// Multi-layer caching
const CachingStrategy = {
  // Browser cache
  BrowserCache: 'static-assets',
  
  // CDN cache
  CDNCache: 'media-files',
  
  // Application cache
  ApplicationCache: {
    Archives: 'react-query',
    Categories: 'local-storage',
    UserPreferences: 'persistent-storage'
  }
};
```

User flow ini dirancang untuk memberikan pengalaman yang intuitif, responsif, dan menyenangkan bagi semua tipe pengguna AVIBA.