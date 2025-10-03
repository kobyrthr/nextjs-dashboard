# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 15 dashboard application built with the App Router, featuring invoice and customer management with authentication. The project follows the Next.js App Router Course structure and uses PostgreSQL for data persistence.

## Commands

**Development:**
```bash
npm run dev          # Start development server with Turbopack
npm run build        # Build production application
npm start            # Start production server
```

**Package Manager:** This project uses `pnpm` (see pnpm config in package.json)

## Architecture

### Database Layer
- **Direct PostgreSQL access** via `postgres` library (not an ORM)
- SQL client initialized in multiple files: `app/lib/data.ts`, `app/lib/actions.ts`, `app/auth.ts`
- Connection requires `POSTGRES_URL` environment variable with SSL
- Database schema includes: `users`, `customers`, `invoices`, `revenue` tables

### Authentication System
- **NextAuth.js v5** (beta) with JWT session strategy
- Two providers configured:
  - Credentials provider with bcrypt password hashing
  - Google OAuth provider
- Auth configuration split across:
  - `app/auth.config.ts` - Base config with authorization callbacks
  - `app/auth.ts` - Provider configuration and user fetching logic
  - `app/middleware.ts` - Route protection middleware
- Protected routes: All `/dashboard/*` routes require authentication
- Environment variables needed: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`

### Server Actions Pattern
All data mutations use Next.js Server Actions (marked with `'use server'`):
- Located in `app/lib/actions.ts`
- Zod validation for form data
- Pattern: validate → mutate → revalidate → redirect
- Actions: `createInvoice`, `updateInvoice`, `deleteInvoice`, `authenticate`

### Data Fetching
- All data fetching functions in `app/lib/data.ts`
- Uses raw SQL queries with the `postgres` library
- Implements pagination (6 items per page) for invoices
- Search/filter functionality uses PostgreSQL `ILIKE` for case-insensitive matching

### Type System
- Manual TypeScript types in `app/lib/definitions.ts` (no ORM-generated types)
- Key types: `User`, `Customer`, `Invoice`, `Revenue`, `InvoiceForm`, `InvoicesTable`
- Status enum: invoices can be `'pending' | 'paid'`

### Route Structure
```
app/
├── dashboard/
│   ├── (overview)/     # Dashboard home with cards and charts
│   ├── invoices/       # Invoice list, create, edit pages
│   └── customers/      # Customer list page
├── login/              # Login page
├── api/auth/           # NextAuth API routes
├── lib/                # Shared utilities, actions, data fetching
└── ui/                 # Reusable UI components
```

### Styling
- Tailwind CSS with `@tailwindcss/forms` plugin
- No component library - custom UI components in `app/ui/`

## Important Conventions

**Path Aliases:**
- Use `@/` for imports from project root (configured in tsconfig.json)

**Amount Handling:**
- Amounts stored in cents (integer) in database
- Convert to dollars for display: `amount / 100`
- Convert to cents for storage: `amount * 100`

**Error Handling:**
- Server actions return error objects: `{ message: string }`
- Special error boundary: `app/dashboard/error.tsx`

**Revalidation:**
- Always call `revalidatePath()` after mutations before redirecting
- Common paths: `/dashboard/invoices`

**Form Actions:**
- Use `formData.get()` to extract form fields
- Validate with Zod schemas before processing
