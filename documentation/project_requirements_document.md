# Project Requirements Document (PRD)

## 1. Project Overview

The **aviba-app (codeguide-starter-fullstack)** is a pre-configured full-stack web application starter template built on Next.js 15. It offers out-of-the-box user authentication, a protected dashboard, modern UI components, dark-mode theming, and a containerized PostgreSQL setup. By bundling these core building blocks, it dramatically reduces the time and effort required to kick off new web projects that need secure user accounts, a data-driven interface, and a production-ready environment.

This template exists to solve the common problem of spending weeks on boilerplate: setting up auth, database connections, theming, and deployment scaffolding. Its key objectives are:

- Provide a secure, type-safe authentication system backed by Better Auth and Drizzle ORM.
- Showcase a protected dashboard layout that developers can extend with real data.
- Deliver a polished, accessible UI built with shadcn/ui components and Tailwind CSS.
- Enable dark mode out of the box, respecting system preferences and user toggles.
- Offer Docker support for PostgreSQL and best practices (ESLint, Prettier) for code quality and deployment readiness (e.g., Vercel).

Success criteria include:

- A developer can clone the repo, run `docker-compose up` and `npm run dev`, and have a working auth + dashboard in under five minutes.
- Authentication flows (sign-up, sign-in, protected routes) work correctly, with type safety enforced by TypeScript and Drizzle.
- UI components render consistently in light and dark modes, passing basic accessibility checks.
- The project passes linting and formatting rules without manual fixes.

## 2. In-Scope vs. Out-of-Scope

### In-Scope (First Version)

- Email/password user registration and login via Better Auth.
- Protected `/dashboard` route only accessible to authenticated users.
- Static dashboard content pulled from a local `data.json` file.
- UI built with shadcn/ui components, styled by Tailwind CSS v4.
- Dark mode support via `next-themes`, with auto-detect and manual toggle.
- Type-safe database schema for users and sessions using Drizzle ORM.
- Local database provisioning via Docker Compose with PostgreSQL.
- Code quality enforcement using ESLint and Prettier configurations.
- Deployment-ready configuration for platforms like Vercel.

### Out-of-Scope (Planned for Future Phases)

- Dynamic, database-driven dashboard data queries and mutations.
- Password reset and email verification flows.
- Social/OAuth logins (e.g., Google, GitHub).
- Comprehensive logging, metrics, and monitoring integration.
- Automated testing (unit, integration, end-to-end).
- CI/CD pipeline setup (e.g., GitHub Actions).
- Advanced caching or performance optimizations beyond Next.js defaults.
- Internationalization (i18n) support.

## 3. User Flow

When a new visitor arrives, they land on the public homepage. From there, they see clear navigation links for **Sign Up** and **Sign In**. Clicking **Sign Up** brings up a form where they enter email and password. On successful registration, the user is automatically logged in and redirected to the **Dashboard**.

Authenticated users land on a layout that includes a **Site Header** (with logo and theme toggle) and a left **Sidebar** for navigation. The main content area shows dashboard cards, charts, and tables sourced from `data.json`. Users can switch between light and dark themes via the toggle in the header, and they can sign out using a button in the header or sidebar. If an unauthenticated user tries to access `/dashboard`, they are redirected to **Sign In**, preserving the intended destination.

## 4. Core Features

- **Authentication Module**: Email/password sign-up, sign-in, session management with Better Auth and Next.js API routes.
- **Protected Routes**: Middleware or route guards to restrict access to authenticated users only.
- **Dashboard Layout**: Pre-built dashboard page with static JSON data, illustrating cards (`section-cards.tsx`), interactive charts (`chart-area-interactive.tsx`), and data tables (`data-table.tsx`).
- **UI Component Library**: shadcn/ui components (`button.tsx`, `input.tsx`, `dialog.tsx`, etc.) customized via Tailwind CSS.
- **Theming & Dark Mode**: Automatic system theme detection plus manual toggle, powered by `next-themes` and CSS variables.
- **Type-Safe Database**: Drizzle ORM schemas for `users` and `sessions`, with a centralized `db/index.ts` connection.
- **Dockerized Database**: `docker-compose.yml` to spin up PostgreSQL locally.
- **Code Quality Tools**: ESLint for linting, Prettier for formatting, TypeScript for type checking.
- **Deployment Config**: Ready-for-use Next.js settings optimized for hosting platforms (e.g., Vercel).

## 5. Tech Stack & Tools

- **Frontend**: Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS v4, shadcn/ui components, next-themes.
- **Backend**: Next.js API Routes, Better Auth (self-hosted), Drizzle ORM, PostgreSQL.
- **DevOps & Containers**: Docker & Docker Compose for local PostgreSQL, Vercel for production deploy.
- **Code Quality**: ESLint, Prettier, TypeScript compiler.
- **Potential IDE Integrations**: VS Code with TypeScript and Tailwind CSS IntelliSense, ESLint, Prettier plugins.

_No AI models are integrated in this version._

## 6. Non-Functional Requirements

- **Performance**: Initial page load time under 1 second on 3G; subsequent navigations via client-side transitions.
- **Scalability**: Architecture supports horizontal scaling of Next.js instances; PostgreSQL can be hosted on managed service.
- **Security**: All API endpoints accessible only over HTTPS; password storage via bcrypt (managed by Better Auth); CSRF protection via Next.js defaults; CORS disabled by default.
- **Accessibility**: Aim for WCAG 2.1 AA compliance; use semantic HTML, focus management, and ARIA roles in shadcn/ui components.
- **Usability**: Responsive design for mobile/desktop; intuitive sidebar navigation and theme toggle.
- **Reliability**: 99.9% uptime target in production; automated health checks for database connection.

## 7. Constraints & Assumptions

- Requires Node.js v18+ and Docker installed for local development.
- PostgreSQL version locked to the image in `docker-compose.yml` (e.g., 15.x).
- Better Auth API must be available and configured via environment variables in `.env`.
- Assumes developers are familiar with Next.js App Router conventions and TypeScript.
- No legacy browser support beyond evergreen browsers.

## 8. Known Issues & Potential Pitfalls

- **Static Data Source**: Dashboard uses a local JSON file. Forgetting to replace this with live queries could lead to stale displays. _Mitigation_: Create placeholder API routes and Drizzle queries early.
- **API Route Growth**: As features grow, the `/app/api` folder may become cluttered. _Mitigation_: Plan subdirectories (e.g., `/api/users`, `/api/dashboard`).
- **Schema Migrations**: Drizzle ORM requires manual migrations; forgetting migrations can break schema. _Mitigation_: Adopt a migration tool or script early (e.g., Drizzle Migrate).
- **Docker Networking**: Misconfigured Docker host/port can block local dev. _Mitigation_: Document common fixes in `README.md`.
- **Dark Mode Flicker**: If `next-themes` is not initialized properly, users may see a flash of the wrong theme. _Mitigation_: Follow `next-themes` SSR best practices and include `ThemeProvider` in root layout.
- **Authentication Edge Cases**: Better Auth’s default error messages may be too generic. _Mitigation_: Catch and customize error responses in API routes.

---

This PRD provides the definitive guide for all subsequent deliverables (Tech Stack Document, Frontend Guidelines, Backend Structure, App Flow, etc.). Every detail here is unambiguous, ensuring the AI and engineers can generate implementation specifics without needing further clarification.