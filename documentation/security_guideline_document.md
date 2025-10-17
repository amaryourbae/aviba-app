# Security Guidelines for aviba-app (codeguide-starter-fullstack)

This document outlines actionable security best practices tailored to the aviba-app starter template. It aligns with core security principles—Security by Design, Least Privilege, Defense in Depth, and Secure Defaults—to help you build a robust, production-ready application.

---

## 1. Authentication & Access Control

**Principles:** Strong authentication, secure session handling, and role-based access control (RBAC).

- **Password Policy & Storage**
  - Enforce a minimum length (≥ 12 chars), complexity rules (upper/lower, digits, symbols) on the client and server side.
  - Use Argon2 or bcrypt with a unique salt per user. Do **not** use MD5, SHA1, or plaintext.
  - Enforce password rotation or expiration for privileged accounts when applicable.

- **Session Management**
  - Store session identifiers in `HttpOnly`, `Secure`, `SameSite=Strict` cookies.
  - Implement idle (e.g., 30 min) and absolute (e.g., 12 hr) timeouts.
  - On logout, invalidate the session server-side (Drizzle sessions table) and clear the cookie.
  - Protect against session fixation by issuing a fresh session ID upon login.

- **API Route Protection**
  - All API routes under `/app/api/*` must verify a valid session before executing logic.
  - Centralize auth checks in middleware (Next.js 15 App Router middleware) to avoid omissions.

- **Role-Based Access Control (future enhancement)**
  - Define user roles (e.g., `user`, `admin`) in the `users` schema.
  - Authorize each protected route against the user’s role.

- **Multi-Factor Authentication (MFA)**
  - Plan for optional TOTP or SMS/Email OTP flows for sensitive operations.

---

## 2. Input Handling & Processing

**Principles:** Validate everything, sanitize all user inputs, and encode outputs to prevent injection.

- **Server-Side Validation**
  - Use a schema validation library (e.g., Zod) in API handlers to enforce required fields, types, lengths, and patterns.
  - Do **not** rely solely on client-side checks.

- **SQL/ORM Safety**
  - Use Drizzle ORM’s parameterized queries—avoid string concatenation for SQL.

- **XSS & Output Encoding**
  - For any user-supplied content rendered in React, use built-in JSX escaping.
  - If you must dangerously set inner HTML, sanitize with a vetted library (e.g., DOMPurify).
  - Implement a strict Content Security Policy (CSP) header (see Section 5).

- **Redirect Validation**
  - When redirecting after sign-in or sign-out, only allow paths within a predefined allow-list to prevent open-redirects.

- **File Uploads** (if introduced)
  - Validate file type, size, and content.
  - Store outside the webroot or in an S3 bucket with restrictive ACLs.
  - Scan for malware.

---

## 3. Data Protection & Privacy

**Principles:** Encrypt in transit & at rest; safeguard secrets; prevent information leakage.

- **Transport Encryption**
  - Enforce HTTPS (TLS 1.2+) in production. Redirect HTTP to HTTPS.
  - Use strong cipher suites and disable weak protocols (SSLv3, TLS 1.0/1.1).

- **Sensitive-Data Encryption**
  - If storing PII (emails, names), consider database-level encryption or field-level encryption.

- **Secret Management**
  - Store credentials (DB URL, JWT secrets, API keys) in environment variables managed by a vault (e.g., AWS Secrets Manager, HashiCorp Vault).
  - Do **not** commit `.env` files to source control. Provide an example (`.env.example`) only.

- **Error Handling & Logging**
  - In production, return generic error messages (`"An error occurred"`) to clients. Do **not** expose stack traces or internal paths.
  - Use structured logging (e.g., Pino, Winston) and centrally mask PII in logs.

---

## 4. API & Service Security

**Principles:** Authenticate & authorize every request; limit abuse; minimize exposed data.

- **Rate Limiting & Throttling**
  - Protect authentication endpoints (`/api/auth/*`) with rate limits (e.g., 5 attempts/minute/IP).
  - Implement a general rate-limiting strategy for other critical endpoints.

- **CORS Configuration**
  - In `next.config.js`, restrict `origin` to known domains (e.g., your frontend URL).
  - Do **not** use a wildcard (`*`) in production.

- **HTTP Methods & Status Codes**
  - Use appropriate verbs: `POST` for create/auth, `GET` for read, `PUT/PATCH` for updates, `DELETE` for removals.
  - Return proper status codes (e.g., `401 Unauthorized`, `403 Forbidden`, `422 Unprocessable Entity`).

- **API Versioning**
  - Prefix future API routes with `v1/`, `v2/`, etc., to manage breaking changes safely.

---

## 5. Web Application Security Hygiene

**Principles:** Harden client behavior and leverage security headers.

- **Security Headers** (can be set in `next.config.js` or middleware)
  - `Content-Security-Policy`: Define trusted sources for scripts, styles, images, and fonts. Example:
    ```
    Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline';
    ```
  - `Strict-Transport-Security`: `max-age=31536000; includeSubDomains; preload`
  - `X-Frame-Options`: `DENY`
  - `X-Content-Type-Options`: `nosniff`
  - `Referrer-Policy`: `strict-origin-when-cross-origin`

- **CSRF Protection**
  - For state-changing requests, implement anti-CSRF tokens. Next.js’s built-in CSRF support or a library like `csurf` can be used.

- **Secure Cookies**
  - Already covered under Session Management: set `Secure`, `HttpOnly`, and `SameSite=Strict`.

- **Third-Party Scripts**
  - Use Subresource Integrity (SRI) when loading from a CDN.

- **Client-Side Storage**
  - Avoid storing tokens in `localStorage` or `sessionStorage`. Rely on cookies and HTTP-only storage.

---

## 6. Infrastructure & Configuration Management

**Principles:** Harden environments, minimize exposed attack surface, and maintain up-to-date components.

- **Docker & Database**
  - In `docker-compose.yml`, run PostgreSQL under a non-root user with a strong password.
  - Expose only the necessary port (5432) on localhost during development.

- **Server Hardening**
  - Disable default accounts and unused services on production hosts.
  - Apply OS-level patches regularly.

- **TLS/SSL**
  - Use up-to-date certificates from a trusted CA (e.g., Let’s Encrypt).

- **Debug Settings**
  - Ensure `NODE_ENV=production` in production; disable verbose logging and stack traces.

---

## 7. Dependency & Supply Chain Management

**Principles:** Vet, lock, and monitor dependencies to prevent known vulnerabilities.

- **Secure Dependencies**
  - Review each package for maintenance status and reported CVEs before adding.

- **Lockfiles**
  - Commit `package-lock.json` or `pnpm-lock.yaml` for deterministic installs.

- **Automated Scanning**
  - Integrate Dependabot or similar SCA tools in your CI pipeline to surface outdated or vulnerable packages.

- **Minimal Footprint**
  - Remove unused dependencies; consider lighter alternatives where possible.

---

## 8. DevOps & CI/CD Security

**Principles:** Automate security checks and enforce policies before deployment.

- **Continuous Integration**
  - Run linting, type checks, unit tests, and security scans on every pull request.
  - Prevent merging on failing checks.

- **Secrets in CI**
  - Store environment variables in the CI platform’s secret store; never expose them in logs.

- **Continuous Deployment**
  - On successful builds, deploy automatically to staging. Gate production deploys behind manual approval or additional tests.

- **Infrastructure as Code**
  - If you provision cloud resources (e.g., Vercel, AWS), manage them through Terraform or similar to track changes and enforce least privilege.

---

## 9. Ongoing Security Practices & Monitoring

- **Logging & Monitoring**
  - Centralize application logs and monitor for anomalies (e.g., spikes in login failures).
  - Integrate alerting for critical events.

- **Penetration Testing & Audits**
  - Schedule regular security assessments, vulnerability scans, and code reviews.

- **Incident Response Plan**
  - Define steps for breach detection, containment, eradication, and recovery.

---

By following these guidelines, the aviba-app starter template will be fortified against common web application threats and ready for secure production use. Regularly revisit and update these controls as your application evolves and new threats emerge.