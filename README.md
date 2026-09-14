# Servio — Service Enquiry & Booking System

A landing page and enquiry pipeline for **Servio**, a local-services booking product. Visitors browse services, submit an enquiry through a modal form, receive an email confirmation, and admins review submissions from a protected dashboard.

**Repository:** https://github.com/haseeb-012/service_enquiry-_and_booking_system

---

## Overview

Servio is a Next.js application with two surfaces:

- **Public marketing site** — a full landing page (hero, services, how it works, benefits, testimonials, FAQ, service finder, CTA) with an enquiry modal that lets visitors request a service and see an in-modal confirmation once it's submitted.
- **Admin dashboard** — a JWT-protected area where an admin logs in and reviews incoming enquiries in a table.

Submissions are stored in MongoDB via Mongoose, and a confirmation email is sent to the submitter through Gmail SMTP (Nodemailer). Route protection for `/admin/*` is handled by `proxy.ts`.

---

## Features

### Marketing site (`/`)
- Hero, services, how it works, benefits, testimonials, FAQ, service finder, and CTA sections
- Enquiry modal (`EnquiryDialog`) built with React Hook Form + Zod, with service presets passed in from the hero / service finder
- In-modal "Enquiry sent" confirmation state after a successful submission — no separate thank-you page
- Smooth scrolling (Lenis) and motion (Framer Motion) throughout
- Shared `BrandMark` / `BrandLink` components for consistent branding in the navbar and footer
- Privacy Policy (`/privacy`) and Terms & Conditions (`/terms`) pages, both rendered through a shared `LegalPage` component

### Public enquiry flow
- Fields: full name, email, phone, service type, preferred date, message
- Client-side validation with loading and success states
- Best-effort confirmation email to the submitter (the enquiry still saves even if SMTP fails or credentials are missing)

### Admin dashboard (`/admin/login` → `/admin`)
- Email / password login backed by `ADMIN_EMAIL` / `ADMIN_PASSWORD`
- JWT issued on login and stored in an HTTP-only cookie (1-hour expiry)
- Submissions table (newest first) with name, email, phone, service, preferred date, message, submitted date, and status
- Toggle a submission between **Pending** and **Reviewed**
- Summary stats (total / pending / reviewed counts)
- Logout clears the session cookie
- Unauthenticated visitors to `/admin/*` are redirected to `/admin/login` by `proxy.ts`

> The submissions API also supports `DELETE` (`/api/admin/submissions/[id]`), but the current admin UI does not yet expose a delete button — only the reviewed/pending toggle is wired up.

---

## Tech stack

| Area | Stack |
|------|--------|
| Framework | Next.js 16 (App Router), React 19, TypeScript |
| UI | Tailwind CSS 4, shadcn/ui (Base UI style, `components.json`), Lucide icons, Sonner toasts |
| Motion | Framer Motion, Lenis (smooth scroll) |
| Forms | React Hook Form, Zod, `@hookform/resolvers` |
| Data | MongoDB (Mongoose ODM) |
| Email | Nodemailer over Gmail SMTP |
| Auth | `jsonwebtoken` (JWT in an HTTP-only cookie) |
| Package manager | pnpm |

---

## Project structure

```
service_enquiry-_and_booking_system/
├── app/
│   ├── page.tsx                       # Landing page
│   ├── layout.tsx                     # Root layout, fonts, metadata, toaster
│   ├── globals.css                    # Design tokens (teal brand palette) + utilities
│   ├── privacy/page.tsx               # Privacy Policy (uses LegalPage)
│   ├── terms/page.tsx                 # Terms & Conditions (uses LegalPage)
│   ├── components/                    # Landing sections, enquiry dialog, shared UI
│   │   ├── Hero.tsx, Services.tsx, HowItWorks.tsx, Benefits.tsx,
│   │   │   Testimonials.tsx, FAQ.tsx, ServiceFinder.tsx, CTA.tsx
│   │   ├── EnquiryDialog.tsx           # Enquiry form + in-modal confirmation
│   │   ├── DialogController.tsx        # Shared open/close state for the enquiry dialog
│   │   ├── BrandMark.tsx               # BrandMark / BrandLink branding components
│   │   ├── LegalPage.tsx               # Shared layout for Privacy / Terms pages
│   │   ├── Navbar.tsx, Footer.tsx, Section.tsx
│   │   └── LenisProvider.tsx           # Smooth scroll + scroll progress bar
│   ├── admin/
│   │   ├── login/page.tsx              # Admin login form
│   │   └── page.tsx                    # Admin dashboard (submissions table)
│   └── api/
│       ├── submit/route.ts             # POST — create submission + send email
│       ├── health/route.ts             # GET — DB connectivity check
│       └── admin/
│           ├── login/route.ts          # POST — verify credentials, set JWT cookie
│           ├── logout/route.ts         # POST — clear JWT cookie
│           └── submissions/
│               ├── route.ts            # GET — list submissions (admin only)
│               └── [id]/route.ts       # PATCH — toggle reviewed, DELETE — remove submission
├── components/ui/                      # shadcn/ui primitives (button, dialog, input, select, ...)
├── lib/
│   ├── auth.ts                         # JWT verification helper for API routes
│   ├── db/index.ts                     # MongoDB/Mongoose connection
│   ├── email.ts                        # Nodemailer transporter
│   ├── motion.ts                       # Shared Framer Motion easing / variants
│   └── utils.ts                        # cn() class-name helper
├── models/submission.ts                # Mongoose Submission schema
├── types/global.d.ts
├── proxy.ts                            # Redirects unauthenticated /admin/* requests to login
├── components.json                     # shadcn/ui configuration
└── .env.example                        # Template for required environment variables
```

---

## Environment variables

Copy `.env.example` to `.env.local` and fill in real values:

```bash
cp .env.example .env.local
```

| Variable | Purpose |
|----------|---------|
| `MONGODB_URI` | MongoDB connection string used by `lib/db/index.ts` to store submissions |
| `EMAIL_USER` | Gmail address used as the SMTP sender for enquiry confirmation emails |
| `EMAIL_PASS` | Gmail [App Password](https://support.google.com/accounts/answer/185833) for `EMAIL_USER` |
| `ADMIN_EMAIL` | Email required to log in to `/admin/login` |
| `ADMIN_PASSWORD` | Password required to log in to `/admin/login` |
| `JWT_SECRET` | Secret used to sign/verify the admin session JWT |

Notes:
- `MONGODB_URI` is required for the app to start `/api/submit`, `/api/health`, and all admin routes — the app throws if it's missing when the admin routes are loaded.
- The confirmation email is best-effort: if `EMAIL_USER` / `EMAIL_PASS` are not set (or SMTP fails), the submission still saves and the request still succeeds.
- Never commit real secrets — only `.env.example` (with placeholder values) is tracked in git.

---

## Getting started

### Prerequisites
- Node.js (a recent LTS version) and pnpm
- A MongoDB connection string (e.g. from MongoDB Atlas)
- A Gmail account with an App Password (optional — enables confirmation emails)

### Install and run

```bash
git clone https://github.com/haseeb-012/service_enquiry-_and_booking_system.git
cd service_enquiry-_and_booking_system
pnpm install
cp .env.example .env.local   # then fill in real values
pnpm dev
```

| Surface | URL |
|---------|-----|
| Landing page + enquiry form | http://localhost:3000 |
| Admin login | http://localhost:3000/admin/login |
| Admin dashboard | http://localhost:3000/admin |

### Production build

```bash
pnpm build
pnpm start
```

---

## Available scripts

Defined in `package.json`:

| Script | Command | Description |
|--------|---------|-------------|
| `pnpm dev` | `next dev` | Start the development server |
| `pnpm build` | `next build` | Build the app for production |
| `pnpm start` | `next start` | Run the production build |
| `pnpm lint` | `eslint` | Lint the codebase |

---

## API routes

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/submit` | Public | Create a submission and send a confirmation email |
| `GET` | `/api/health` | Public | Report MongoDB connectivity status |
| `POST` | `/api/admin/login` | Public | Verify admin credentials and set the JWT cookie |
| `POST` | `/api/admin/logout` | Public | Clear the JWT cookie |
| `GET` | `/api/admin/submissions` | Admin (JWT cookie) | List all submissions |
| `PATCH` | `/api/admin/submissions/[id]` | Admin (JWT cookie) | Toggle a submission's reviewed status |
| `DELETE` | `/api/admin/submissions/[id]` | Admin (JWT cookie) | Delete a submission |

Admin routes read the `token` cookie and verify it with `getAdminFromRequest` in `lib/auth.ts`; requests without a valid JWT get a `401`.

---

## How to test locally

1. Run `pnpm dev` and open http://localhost:3000.
2. Open the enquiry dialog from the hero, services, or service finder section and submit it with valid data.
3. Confirm the in-modal "Enquiry sent" state appears, and (if email env vars are set) check the submitter's inbox.
4. Sign in at `/admin/login` using `ADMIN_EMAIL` / `ADMIN_PASSWORD`.
5. Confirm the new submission appears in the dashboard table; toggle it to Reviewed.
6. Log out and confirm you're redirected back to `/admin/login`, and that visiting `/admin` directly while logged out also redirects there.

---

## Deployment

This is a standard Next.js App Router project, so it deploys well to any Next.js-compatible host (e.g. Vercel). There is no deployment configuration committed to this repository — to deploy:

1. Push the repository to GitHub (already done).
2. Import the project into your hosting provider.
3. Set the environment variables listed above in the provider's project settings.
4. Deploy — the build command is `pnpm build`, the start command is `pnpm start`.

---

## Known limitations

- Single admin account, configured via environment variables — no multi-user support or password reset flow
- Admin session expires after 1 hour (JWT `expiresIn: '1h'`); there is no refresh mechanism
- No email notification to the admin when a new enquiry is submitted (only the submitter gets an email)
- The admin dashboard has no pagination, search, or filtering — all submissions load at once
- The `DELETE /api/admin/submissions/[id]` endpoint exists but isn't wired into the admin dashboard UI yet
- No rate limiting or CAPTCHA on the public `/api/submit` endpoint
- Servio branding and content are for demo / portfolio purposes

---

## Author

**Haseeb Sajjad** — [@haseeb-012](https://github.com/haseeb-012)
