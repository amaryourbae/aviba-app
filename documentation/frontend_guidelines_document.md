# AVIBA - Frontend Guidelines & Best Practices

## 1. Ikhtisar Frontend Architecture

Frontend AVIBA dibangun menggunakan Next.js 15 dengan App Router, TypeScript, dan Tailwind CSS untuk menciptakan pengalaman pengguna yang modern, responsif, dan performan dalam menjelajahi arsip digital Banyuwangi.

### 1.1 Technology Stack
- **Framework**: Next.js 15 dengan App Router
- **Language**: TypeScript (Strict Mode)
- **Styling**: Tailwind CSS dengan custom design system
- **UI Components**: shadcn/ui yang customizable
- **State Management**: React Hooks & Context API
- **Form Handling**: React Hook Form dengan Zod validation
- **Animations**: Framer Motion untuk micro-interactions
- **Icons**: Lucide React icons
- **Notifications**: SweetAlert 2 & React Hot Toast
- **File Upload**: react-dropzone

### 1.2 Design Philosophy
- **Mobile-First**: Responsive design yang optimal di semua devices
- **Accessibility**: WCAG 2.1 AA compliance
- **Performance**: Optimal loading dengan lazy loading dan caching
- **User Experience**: Intuitive navigation dan engaging interactions
- **Cultural Context**: Design yang mencerminkan budaya Banyuwangi

### 1.3 Framework and Libraries
- **Next.js 15 (App Router)**: Handles routing, server-side rendering (SSR), dan API routes
- **React 19**: Powers interactive user interfaces dengan components
- **TypeScript**: Adds type safety di seluruh codebase
- **Tailwind CSS v4**: Utility-first CSS framework untuk fast, consistent styling
- **shadcn/ui**: Collection of accessible, customizable UI components
- **next-themes**: Manages light/dark theme switching

### 1.4 Architecture Benefits
- **Scalability**: File-based routing dan App Router memudahkan penambahan pages
- **Maintainability**: TypeScript enforces consistent data shapes
- **Performance**: Server components dan SSR mengurangi bundle sizes
- **Component-Driven**: Modular dan reusable components

## 2. Project Structure Frontend

```
frontend/
├── app/
│   ├── (auth)/                     # Authentication routes
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (public)/                   # Public routes
│   │   ├── page.tsx                # Landing page
│   │   ├── archive/
│   │   │   ├── page.tsx            # Archive listing
│   │   │   └── [slug]/
│   │   │       └── page.tsx        # Archive detail
│   │   ├── era/
│   │   │   └── [slug]/
│   │   │       └── page.tsx        # Era detail
│   │   └── search/
│   │       └── page.tsx            # Search results
│   ├── dashboard/                  # Admin dashboard
│   │   ├── page.tsx                # Dashboard overview
│   │   ├── archives/
│   │   │   ├── page.tsx            # Archive management
│   │   │   ├── new/
│   │   │   │   └── page.tsx        # Create archive
│   │   │   └── [id]/
│   │   │       └── edit/
│   │   │           └── page.tsx    # Edit archive
│   │   ├── media/
│   │   │   └── page.tsx            # Media management
│   │   ├── analytics/
│   │   │   └── page.tsx            # Analytics dashboard
│   │   └── settings/
│   │       └── page.tsx            # Settings page
│   ├── api/                        # API routes (shared with backend)
│   ├── globals.css                 # Global styles
│   ├── layout.tsx                  # Root layout
│   ├── error.tsx                   # Error boundary
│   ├── not-found.tsx               # 404 page
│   └── loading.tsx                 # Loading states
├── components/
│   ├── ui/                         # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── input.tsx
│   │   ├── dialog.tsx
│   │   ├── dropdown-menu.tsx
│   │   └── ... (other base components)
│   ├── layout/
│   │   ├── Header.tsx              # Main navigation
│   │   ├── Footer.tsx              # Footer component
│   │   ├── Sidebar.tsx             # Admin sidebar
│   │   ├── Navigation.tsx          # Navigation logic
│   │   └── MobileMenu.tsx          # Mobile navigation
│   ├── shared/
│   │   ├── SearchBar.tsx           # Search functionality
│   │   ├── LoadingSkeleton.tsx     # Loading states
│   │   ├── ErrorBoundary.tsx       # Error handling
│   │   ├── ThemeToggle.tsx         # Dark/light mode
│   │   └── BackToTop.tsx           # Scroll to top
│   ├── archive/
│   │   ├── ArchiveCard.tsx         # Archive preview card
│   │   ├── ArchiveGrid.tsx         # Grid layout
│   │   ├── ArchiveList.tsx         # List layout
│   │   ├── ArchiveDetail.tsx       # Detail view
│   │   ├── MediaGallery.tsx        # Media gallery viewer
│   │   ├── MediaPlayer.tsx         # Media player component
│   │   ├── LocationMap.tsx         # Interactive map
│   │   ├── EraTimeline.tsx         # Historical timeline
│   │   └── CategoryFilter.tsx      # Category filtering
│   ├── admin/
│   │   ├── DashboardCards.tsx      # Dashboard overview cards
│   │   ├── ArchiveForm.tsx         # Archive creation/edit form
│   │   ├── MediaUpload.tsx         # File upload component
│   │   ├── DataTable.tsx           # Data table with pagination
│   │   ├── AnalyticsChart.tsx      # Analytics visualization
│   │   ├── BulkActions.tsx         # Bulk operations
│   │   └── RichTextEditor.tsx      # WYSIWYG editor
│   ├── chat/
│   │   ├── ChatInterface.tsx       # AI chat interface
│   │   ├── ChatMessage.tsx         # Individual message
│   │   ├── ChatInput.tsx           # Chat input component
│   │   ├── ChatHistory.tsx         # Chat history sidebar
│   │   └── TypingIndicator.tsx     # Typing animation
│   └── forms/
│       ├── FormField.tsx           # Reusable form field
│       ├── FileUpload.tsx          # File upload with preview
│       ├── DatePicker.tsx          # Date selection
│       ├── TagSelector.tsx         # Tag selection
│       └── LocationPicker.tsx      # Location selection
├── lib/
│   ├── hooks/
│   │   ├── useArchives.ts          # Archive data hook
│   │   ├── useAuth.ts              # Authentication hook
│   │   ├── useLocalStorage.ts      # Local storage hook
│   │   ├── useDebounce.ts          # Debounce utility
│   │   ├── useInfiniteScroll.ts    # Infinite scroll
│   │   └── useAnalytics.ts         # Analytics tracking
│   ├── utils/
│   │   ├── api.ts                  # API utilities
│   │   ├── format.ts               # Text formatting
│   │   ├── date.ts                 # Date utilities
│   │   ├── validation.ts           # Form validation schemas
│   │   ├── constants.ts            # App constants
│   │   └── helpers.ts              # General helpers
│   ├── store/
│   │   ├── authStore.ts            # Auth state management
│   │   ├── themeStore.ts           # Theme preferences
│   │   └── searchStore.ts          # Search state
│   └── styles/
│       ├── globals.css             # Global styles
│       └── components.css          # Component-specific styles
├── types/
│   ├── api.ts                      # API response types
│   ├── archive.ts                  # Archive data types
│   ├── auth.ts                     # Authentication types
│   ├── ui.ts                       # UI component types
│   └── index.ts                    # Type exports
├── public/
│   ├── images/                     # Static images
│   ├── icons/                      # App icons
│   └── fonts/                      # Custom fonts
└── styles/
    ├── globals.css                 # Global styles
    ├── components.css              # Component styles
    └── themes.css                  # Theme definitions
```

## 3. Design Principles

### 3.1 Key Principles
- **Usability**: Simple, predictable interfaces dengan common patterns
- **Accessibility**: Built on Radix dan shadcn/ui dengan ARIA attributes
- **Responsiveness**: Mobile-first design untuk semua screen sizes
- **Consistency**: Shared design system (Tailwind + shadcn/ui)
- **Cultural Context**: Design yang mencerminkan budaya Banyuwangi

### 3.2 Applying These Principles
- Forms menggunakan clear labels dan validation feedback
- Color contrast meets accessibility standards di light dan dark modes
- Layout menggunakan flexible grids dan utility classes
- Interactive elements memiliki hover, focus, dan active states
- Typography yang readable dengan proper hierarchy
- Loading states dan error handling yang user-friendly

## 3. Styling and Theming

**Styling Approach**
- **Utility-First (Tailwind CSS v4)**: Write classes directly in JSX (e.g., `className="p-4 bg-primary text-white"`), reducing context switching between HTML and CSS.
- **No Preprocessor Needed**: Tailwind’s built-in features cover most needs. If custom logic is required, you can extend Tailwind’s config.

**Theming**
- **CSS Variables**: Define colors and spacing in `:root` and toggle values with a `data-theme` attribute on the `<html>` element.
- **next-themes**: Wrap the app in a `ThemeProvider` that reads system settings or user choice and updates classes and CSS variables accordingly.

**Overall Style**
- **Modern Flat Design**: Clean edges, minimal shadows, clear typography. Focus on content and readability.
- **Color Palette** (light / dark):
  - Primary: #6366F1 / #4F46E5
  - Secondary: #10B981 / #059669
  - Accent: #F59E0B / #D97706
  - Background: #F9FAFB / #111827
  - Surface (cards, panels): #FFFFFF / #1F2937
  - Text: #111827 / #F9FAFB

**Font**
- **Inter**: A modern, highly readable sans-serif font with system fallbacks (`-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`).

## 4. Component Structure

**Organization**
- `components/` holds all reusable pieces.
  - `components/ui/`: Core shadcn/ui components (buttons, inputs, dialogs).
  - Custom components like `AuthButtons`, `AppSidebar`, `SiteHeader`, `ThemeToggle`, and dashboard widgets (`ChartAreaInteractive`, `DataTable`, `SectionCards`).

**Why Component-Based**
- **Reusability**: Build once, use everywhere. A single `Button` component can appear in forms, dialogs, and cards.
- **Consistency**: Changes to a component (e.g., default padding) propagate across the entire app.
- **Maintainability**: Smaller files, focused responsibilities, and clear import paths make code easy to navigate.

## 5. State Management

**Current Approach**
- **Local State**: React’s `useState` and `useReducer` for form fields, toggles, and UI interactions.
- **Context API**: Used by `next-themes` to share theme state across the tree.

**Authentication State**
- Handled by Better Auth via Next.js API routes and cookies. Client-side, you can check `user` sessions through a custom hook (`useAuthSession`).

**Future Options**
- For complex server state, introduce **React Query** or **SWR**.
- For global client state, consider **Zustand** or **Redux Toolkit** if needs grow.

## 6. Routing and Navigation

**Routing**
- **Next.js App Router**: Each folder in `app/` represents a URL segment. Files like `page.tsx`, `layout.tsx`, and `loading.tsx` define what users see and how data is fetched.
- **API Routes**: Under `app/api/`, segmented endpoints handle authentication and future data operations.

**Navigation**
- **SiteHeader** and **AppSidebar** components use Next.js `<Link>` and hooks like `useRouter` to navigate.
- Protected routes (e.g., `/dashboard`) use server-side checks. Unauthenticated users are redirected to `/sign-in`.

## 7. Performance Optimization

- **Server Components & SSR**: Reduce client bundle size by rendering static or server-only components on the server.
- **CSS Purge**: Tailwind automatically removes unused styles in production.
- **Dynamic Imports**: Lazy-load heavy components (charts, maps) with `next/dynamic`.
- **Image Optimization**: Use `next/image` for automatic resizing, lazy loading, and modern formats.
- **Cache-Control Headers**: Configure fetch and API responses for efficient client caching.

## 8. Testing and Quality Assurance

**Linting & Formatting**
- **ESLint**: Enforces code style, catches common errors.
- **Prettier**: Formats code automatically on save or commit.

**Unit Tests**
- **React Testing Library** + **Jest** or **Vitest**: Test individual components and utility functions (e.g., `lib/utils.ts`).

**Integration Tests**
- **Supertest** or **Next.js Testing Library**: Test API routes (`/api/auth`), ensuring authentication flows work end to end.

**End-to-End (E2E)**
- **Cypress** or **Playwright**: Simulate real user journeys—sign-up, sign-in, navigate to dashboard, toggle theme.

**Continuous Integration**
- Run lint, tests, and build checks on every pull request using GitHub Actions or your CI of choice.

## 9. Conclusion and Overall Frontend Summary

The aviba-app frontend combines Next.js’s modern App Router, React, TypeScript, and Tailwind CSS with an accessible component library (shadcn/ui) to deliver a scalable, maintainable, and high-performance experience. User authentication is handled cleanly via Better Auth and Next.js API routes, while theming and dark mode give the UI a polished, user-friendly look. This setup accelerates development by providing a solid foundation that can be extended with more complex data fetching, advanced state management, and thorough testing—all without losing the consistency and clarity this guidelines document strives to maintain.