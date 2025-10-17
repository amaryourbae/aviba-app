# Tech Stack for aviba-app

This document explains, in everyday language, the main technologies we chose for the aviba-app full-stack starter template. It shows how each part fits together and why it makes development fast, reliable, and easy to extend—even for non-technical readers.

## 1. Frontend Technologies
These are the tools that build what you see and interact with in your web browser:

- **Next.js 15 (App Router)**
  • A framework that handles page routing, server-side rendering, and building both pages and API endpoints.
  • Gives us a fast, SEO-friendly site that can load some parts on the server and some in the browser.
- **React 19**
  • A library for building dynamic user interfaces with components that update automatically when data changes.
- **TypeScript**
  • Adds strict type checking (like grammar rules) so we catch errors while writing code, not in production.
- **Tailwind CSS v4**
  • A utility-first styling approach where we build designs by composing small, reusable classes.
  • Speeds up styling and keeps a consistent look across the app.
- **shadcn/ui**
  • A set of ready-made, accessible UI components (buttons, forms, dialogs) built on Radix UI and styled with Tailwind.
  • Developers can copy and customize these components directly to match any design.
- **next-themes**
  • Manages light, dark, or system-preference themes automatically.
  • Integrates seamlessly into our layouts so users can switch themes with a click.

Together, these frontend choices ensure a polished, responsive, and easily customizable user interface.

## 2. Backend Technologies
These handle data storage, business logic, and user authentication behind the scenes:

- **Next.js API Routes**
  • Built-in endpoints for handling server logic (sign-up, sign-in, data fetches).
  • Keep backend code close to frontend code in a single project.
- **Better Auth**
  • A self-hosted authentication solution plugged into Next.js API routes.
  • Manages sign-up, sign-in, session cookies, and secure password storage.
- **Drizzle ORM**
  • A type-safe database toolkit for writing SQL queries in TypeScript.
  • Ensures compile-time checks against your actual database schema.
- **PostgreSQL**
  • A reliable, open-source relational database.
  • Stores user accounts, sessions, and any future application data.

These backend components work together so you can log in, stay logged in securely, and interact with data safely.

## 3. Infrastructure and Deployment
This covers how and where we run the application, and how we keep the code organized:

- **Docker for PostgreSQL**
  • Provides a local database container that developers can spin up with one command.
  • Ensures everyone works against the same environment.
- **Vercel (Deployment)**
  • A hosting platform optimized for Next.js that handles building and scaling automatically.
  • Offers built-in previews for each code change, so you can review features before they go live.
- **Version Control (Git/GitHub)**
  • Tracks every change in code, making collaboration and rollback simple.
- **ESLint & Prettier**
  • Automatic code linting and formatting rules keep the codebase consistent and easy to read.

This setup makes onboarding new developers quick, and deployments predictable and repeatable.

## 4. Third-Party Integrations
These extra tools add functionality without reinventing the wheel:

- **Radix UI (via shadcn/ui)**
  • Provides unstyled, accessible building blocks (dialogs, tooltips) that we style with Tailwind.
- **next-themes**
  • Handles theme switching for light and dark modes.
- **Better Auth**
  • While self-hosted, it’s a packaged solution that integrates authentication flows without custom boilerplate.

By reusing these mature packages, we save time and reduce potential bugs.

## 5. Security and Performance Considerations
How we keep your data safe and the app fast:

- **Authentication Security**
  • Passwords are hashed and stored securely.
  • Sessions are managed with HTTP-only cookies to prevent client-side tampering.
- **Type Safety with TypeScript and Drizzle**
  • Catch mismatches between code and database at compile time, reducing runtime errors.
- **Server-Side Rendering (SSR) and Static Rendering**
  • Next.js intelligently decides what to render on the server for SEO and what to render in the browser for interactivity.
- **CSS Variables for Theming**
  • Enables fast theme changes without heavy JavaScript.
- **Linting and Formatting**
  • ESLint flags potential security issues and code smells before deployment.

Performance optimizations like these help pages load quickly and behave reliably.

## 6. Conclusion and Overall Tech Stack Summary
We chose a modern, full-stack setup that balances developer productivity, security, and user experience:

- Frontend: Next.js, React, TypeScript, Tailwind CSS, shadcn/ui, next-themes
- Backend: Next.js API Routes, Better Auth, Drizzle ORM, PostgreSQL
- Infrastructure: Docker, Vercel, Git, ESLint, Prettier
- Integrations: Radix UI, next-themes, Better Auth

Unique Highlights:

- **Type-Safe End-to-End:** From UI components to database queries, TypeScript ensures consistency.
- **Component-Driven UI:** Ready-to-use, accessible components that speed up design work.
- **Built-In Dark Mode:** Automatic and manual theme switching using CSS variables.
- **Full-Stack in One Repo:** Frontend, backend, and database config live together for a smooth development experience.

This stack provides a solid foundation for any web application, letting teams focus on features, not setup.