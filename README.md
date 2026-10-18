# Vera Gayrimenkul Website

Corporate website and listings platform for a real estate agency, built with Next.js 14, Prisma and PostgreSQL, with a protected admin panel.

## Overview

The site presents Vera Gayrimenkul and Vera Grup: corporate pages, services (valuation, consulting, market analysis), references, team and a searchable property catalogue. Visitors can submit property and career applications. An admin panel behind JWT cookie auth lets staff manage listings, team members, applications and internal notes per listing.

The whole app (pages and API routes) lives in one Next.js project under `frontend/` and is deployed as a standalone Docker image on CapRover.

## Features

### Public site (Turkish UI)
- Home, corporate pages (`/kurumsal/...`: Vera Gayrimenkul, Vera Grup, service policy, KVKK notice)
- Services pages: valuation, consulting, analysis
- Listings page (`/ilanlar`) with text search and filters, plus listing detail pages (`/ilanlar/[id]`)
- Team, references, career and contact pages
- Property application form ("list your property")
- SEO: metadata component, JSON-LD structured data, `sitemap.ts`, `robots.ts`, web manifest
- Optional Google Analytics and Google Tag Manager

### Admin panel (`/admin`)
- Login with email + bcrypt-hashed password from environment variables; JWT stored in an HTTP-only cookie (signed/verified with `jose`)
- Route protection via Next.js middleware
- Listings CRUD with type-specific fields (apartment, land, field, commercial), statuses (active, passive, sold, rented) and ordering
- Admin notes per listing with type and priority
- Team member management
- Listing applications and career applications views
- Image uploads to the local `public/uploads` folder or, optionally, Cloudinary

## Tech Stack

| Area | Technologies |
|---|---|
| Framework | Next.js 14 (App Router, standalone output), React, TypeScript |
| Styling / UI | Tailwind CSS, Radix UI dialog, Headless UI, Heroicons, Lucide, class-variance-authority |
| Data | PostgreSQL, Prisma ORM |
| Auth | bcryptjs, jose (JWT) |
| Deployment | Docker (multi-stage), CapRover, optional PM2 + Nginx |

## Data Storage Notes

- **Listings** and **admin notes** use Prisma/PostgreSQL. The listings API falls back to a JSON file in `data/` if the database is unavailable.
- **Team members** are stored in `data/team.json`.
- **Listing and career applications** are currently kept in memory in their API routes, so they reset on restart.

## Project Structure

```
.
├── frontend/
│   ├── prisma/
│   │   ├── schema.prisma        # User, Listing, AdminNote, TeamMember, ...
│   │   └── migrations/
│   ├── src/
│   │   ├── app/                 # Pages (App Router) and API routes (app/api/*)
│   │   ├── components/          # Layout, SEO, analytics, UI primitives
│   │   └── middleware.ts        # /admin route protection
│   ├── ecosystem.config.js      # PM2 config
│   ├── nginx-config.conf        # Reverse proxy example
│   └── deploy.sh, deploy-safe.sh, migrate-and-start.sh
├── Dockerfile.caprover
├── captain-definition
└── deploy-caprover.sh
```

## Getting Started

### Prerequisites
- Node.js 18+
- PostgreSQL

### Run locally

```bash
cd frontend
npm install          # also runs prisma generate
npm run db:push      # create tables from schema.prisma
npm run dev          # http://localhost:3000
```

### Environment variables

Create `frontend/.env` (names only, set your own values):

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Secret for admin JWTs |
| `ADMIN_EMAIL` | Admin login email |
| `ADMIN_PASSWORD_HASH` | bcrypt hash of the admin password |
| `USE_CLOUD_STORAGE` | `true` to upload images to Cloudinary |
| `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` | Cloudinary credentials (when cloud storage is on) |
| `NEXT_PUBLIC_GA_ID`, `NEXT_PUBLIC_GTM_ID` | Optional analytics IDs |

### Scripts

| Script | Description |
|---|---|
| `npm run dev` | Development server |
| `npm run build` | `prisma generate` + `next build` |
| `npm start` | `prisma db push` then start the standalone server |
| `npm run lint` | Next.js lint |
| `npm run db:migrate` | `prisma migrate deploy` |
| `npm run db:push` | Sync schema to the database |

## Deployment

### CapRover

`captain-definition` points to `Dockerfile.caprover`, which builds a Next.js standalone image on Node 18 Alpine, runs as a non-root user and exposes port `3000`. The `/app/public/uploads` folder is mounted as a persistent volume so uploaded images survive redeploys. Set the environment variables above in the CapRover app settings.

You can also build and push the image yourself:

```bash
docker buildx build --platform linux/amd64 -f Dockerfile.caprover -t <registry>/vera-gayrimenkul:latest --push .
```

then deploy it in CapRover with "Deploy via ImageName". Alternatively, `deploy-caprover.sh` packs the repo into a tarball for `caprover deploy`.

### VPS with PM2

`frontend/deploy.sh` and `ecosystem.config.js` run the app with PM2 behind Nginx (`nginx-config.conf`).

---

Built by [Mertcan Eren](https://github.com/mertcaneren0)
