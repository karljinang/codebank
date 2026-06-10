# CodeBank — Project Overview

> **One fast, searchable, AI-enhanced hub for all your dev knowledge & resources.**

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Target Users](#target-users)
3. [Feature Set](#feature-set)
4. [Data Models & Prisma Schema](#data-models--prisma-schema)
5. [Tech Stack](#tech-stack)
6. [Monetization](#monetization)
7. [UI/UX Guidelines](#uiux-guidelines)
8. [Architecture Overview](#architecture-overview)
9. [Item Types Reference](#item-types-reference)
10. [Project Constraints & Notes](#project-constraints--notes)

---

## Problem Statement

Developers scatter their essentials across too many places:

| Resource      | Where it lives               |
| ------------- | ---------------------------- |
| Code snippets | VS Code, Notion, Gists       |
| AI prompts    | Chat history                 |
| Context files | Buried in project folders    |
| Useful links  | Browser bookmarks            |
| Documentation | Random folders               |
| Commands      | `.txt` files or bash history |

This creates context-switching overhead, lost institutional knowledge, and inconsistent workflows.

**CodeBank fixes this with one unified hub.**

---

## Target Users

| User Type                      | Primary Need                                                    |
| ------------------------------ | --------------------------------------------------------------- |
| **Everyday Developer**         | Quickly grab snippets, prompts, commands, and links             |
| **AI-first Developer**         | Save and organize prompts, contexts, workflows, system messages |
| **Content Creator / Educator** | Store code blocks, explanations, and course notes               |
| **Full-stack Builder**         | Collect patterns, boilerplates, and API examples                |

---

## Feature Set

### A. Items & Item Types

Items have a `type` that determines how they are stored and displayed. Types fall into three content categories:

| Category | Types                                  |
| -------- | -------------------------------------- |
| `text`   | `snippet`, `note`, `prompt`, `command` |
| `url`    | `link`                                 |
| `file`   | `file` _(pro)_, `image` _(pro)_        |

**System types** are built-in and cannot be modified. Users will eventually be able to create custom types (Pro roadmap).

URL pattern for type views: `/items/snippets`, `/items/prompts`, etc.

Items can be created and accessed quickly from a **slide-over drawer** — no full page navigation required.

---

### B. Collections

Collections group items of any type. Items can belong to **multiple collections** simultaneously.

Examples:

- `React Patterns` → snippets, notes
- `Context Files` → files
- `Python Snippets` → snippets
- `Interview Prep` → snippets, prompts

---

### C. Search

Full-text search across:

- Item content
- Title
- Tags
- Type

---

### D. Authentication

- Email / password
- GitHub OAuth

Provider: **NextAuth v5**

---

### E. Core Features

- Favorite collections and items
- Pin items to top
- Recently used items
- Import code from a file
- Markdown editor for text-type items
- File upload for `file` / `image` types
- Export data (JSON / ZIP — Pro)
- Dark mode (default)
- Add/remove items to/from multiple collections
- View which collections an item belongs to

---

### F. AI Features (Pro only)

- Auto-tag suggestions
- Item summaries
- Explain This Code
- Prompt optimizer

Model: `gpt-5-nano` via OpenAI API

> During development, all users can access Pro and AI features freely.

---

## Data Models & Prisma Schema

> **Never run `db push` or modify the database directly. Always create and run migrations.**

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                   String       @id @default(cuid())
  name                 String?
  email                String?      @unique
  emailVerified        DateTime?
  image                String?
  isPro                Boolean      @default(false)
  stripeCustomerId     String?      @unique
  stripeSubscriptionId String?      @unique
  createdAt            DateTime     @default(now())
  updatedAt            DateTime     @updatedAt

  accounts    Account[]
  sessions    Session[]
  items       Item[]
  collections Collection[]
  itemTypes   ItemType[]
}

// NextAuth required models
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

model Item {
  id          String   @id @default(cuid())
  title       String
  contentType String   // "text" | "file" | "url"
  content     String?  // text content; null if file
  fileUrl     String?  // Cloudflare R2 URL; null if text
  fileName    String?  // original filename; null if text
  fileSize    Int?     // bytes; null if text
  url         String?  // for link types only
  description String?
  isFavorite  Boolean  @default(false)
  isPinned    Boolean  @default(false)
  language    String?  // optional, for code snippets
  lastUsedAt  DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  userId     String
  itemTypeId String

  user        User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  itemType    ItemType         @relation(fields: [itemTypeId], references: [id])
  tags        Tag[]            @relation("ItemTags")
  collections ItemCollection[]
}

model ItemType {
  id       String  @id @default(cuid())
  name     String  // "snippet" | "prompt" | "note" | "command" | "file" | "image" | "link"
  icon     String  // Lucide icon name
  color    String  // hex color
  isSystem Boolean @default(false)

  userId String? // null for system types

  user  User?  @relation(fields: [userId], references: [id], onDelete: Cascade)
  items Item[]

  @@unique([name, userId]) // system types are unique by name; user types scoped per user
}

model Collection {
  id            String   @id @default(cuid())
  name          String
  description   String?
  isFavorite    Boolean  @default(false)
  defaultTypeId String?  // preferred item type for new collections with no items yet
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  userId String

  user  User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  items ItemCollection[]
}

model ItemCollection {
  itemId       String
  collectionId String
  addedAt      DateTime @default(now())

  item       Item       @relation(fields: [itemId], references: [id], onDelete: Cascade)
  collection Collection @relation(fields: [collectionId], references: [id], onDelete: Cascade)

  @@id([itemId, collectionId])
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  items Item[] @relation("ItemTags")
}
```

---

## Tech Stack

| Layer            | Technology                                                                        |
| ---------------- | --------------------------------------------------------------------------------- |
| **Framework**    | [Next.js 16](https://nextjs.org/) + React 19                                      |
| **Language**     | TypeScript                                                                        |
| **Database**     | [Neon](https://neon.tech/) (serverless PostgreSQL)                                |
| **ORM**          | [Prisma 7](https://www.prisma.io/docs)                                            |
| **Auth**         | [NextAuth v5](https://authjs.dev/) — email/password + GitHub OAuth                |
| **File Storage** | [Cloudflare R2](https://developers.cloudflare.com/r2/)                            |
| **AI**           | [OpenAI API](https://platform.openai.com/docs) — `gpt-5-nano`                     |
| **CSS**          | [Tailwind CSS v4](https://tailwindcss.com/) + [shadcn/ui](https://ui.shadcn.com/) |
| **Payments**     | [Stripe](https://stripe.com/docs) (subscriptions)                                 |
| **Caching**      | Redis _(optional / TBD)_                                                          |
| **Rendering**    | SSR pages with dynamic components; API routes for backend                         |

> **Prisma 7 note:** Breaking changes from v6 include updated schema syntax and import paths. Always fetch the latest Prisma 7 docs before writing migrations or client code.

---

## Monetization

### Free Tier

- 50 items total
- 3 collections
- All system types except `file` and `image`
- Basic search
- No file/image uploads
- No AI features

### Pro — $8/month or $72/year

- Unlimited items
- Unlimited collections
- `file` and `image` type support
- File & image uploads (Cloudflare R2)
- Custom item types _(coming later)_
- AI auto-tagging
- AI code explanation
- AI prompt optimizer
- Export data (JSON / ZIP)
- Priority support

> **Dev mode:** All users access Pro features during development. The `isPro` flag on `User` is the gate — flip it server-side when ready to enforce limits.

---

## UI/UX Guidelines

### General

- Modern, minimal, developer-focused aesthetic
- **Dark mode by default**, light mode optional
- Clean typography, generous whitespace
- Subtle borders and shadows
- Reference: [Notion](https://notion.so), [Linear](https://linear.app), [Raycast](https://raycast.com)
- Syntax highlighting on all code blocks

### Layout

```
┌─────────────────────────────────────────────────────────┐
│  Sidebar (collapsible)      │  Main Content Area         │
│                             │                            │
│  Item Types                 │  Collections grid          │
│  ├─ Snippets                │  (color-coded cards)       │
│  ├─ Prompts                 │                            │
│  ├─ Commands                │  Items under collection    │
│  ├─ Notes                   │  (color-coded border cards)│
│  ├─ Files (pro)             │                            │
│  ├─ Images (pro)            │  [Item opens in drawer →]  │
│  └─ Links                   │                            │
│                             │                            │
│  Collections (latest)       │                            │
│  └─ ...                     │                            │
└─────────────────────────────────────────────────────────┘
```

- Sidebar collapses to a drawer on mobile
- Collection cards: background color determined by the dominant item type in that collection
- Item cards: border color matches the item type color
- Individual items open in a **slide-over drawer** for quick access and editing

### Micro-interactions

- Smooth transitions on drawer open/close, card hover
- Hover states on all cards
- Toast notifications for create / update / delete / copy actions
- Loading skeletons while data fetches

---

## Item Types Reference

| Type    | Icon         | Color   | Hex       | Content Category | Free   |
| ------- | ------------ | ------- | --------- | ---------------- | ------ |
| Snippet | `Code`       | Blue    | `#3b82f6` | `text`           | ✅     |
| Prompt  | `Sparkles`   | Purple  | `#8b5cf6` | `text`           | ✅     |
| Command | `Terminal`   | Orange  | `#f97316` | `text`           | ✅     |
| Note    | `StickyNote` | Yellow  | `#fde047` | `text`           | ✅     |
| Link    | `Link`       | Emerald | `#10b981` | `url`            | ✅     |
| File    | `File`       | Gray    | `#6b7280` | `file`           | ❌ Pro |
| Image   | `Image`      | Pink    | `#ec4899` | `file`           | ❌ Pro |

Icons are from [Lucide React](https://lucide.dev/icons/).

---

## Architecture Overview

```
Next.js App (SSR + API Routes)
│
├── /app
│   ├── (auth)          → Login, register pages (NextAuth)
│   ├── (dashboard)
│   │   ├── /items/[type]   → Type-filtered item lists
│   │   ├── /collections    → Collections grid
│   │   └── /collections/[id] → Collection detail with items
│   └── /api
│       ├── /items          → CRUD for items
│       ├── /collections    → CRUD for collections
│       ├── /upload         → Cloudflare R2 upload handler
│       ├── /ai             → OpenAI proxy (tag, explain, summarize, optimize)
│       └── /webhooks/stripe → Stripe subscription events
│
├── Neon PostgreSQL (via Prisma 7)
│
├── Cloudflare R2 (file/image storage)
│
└── OpenAI API (gpt-5-nano, Pro features)
```

---

## Project Constraints & Notes

- **No `db push` or direct schema edits** — always write and run Prisma migrations.
- **Prisma 7** has breaking changes vs v6; fetch latest docs before writing ORM code.
- **NextAuth v5** (Auth.js) API differs from v4 — don't rely on v4 patterns.
- **Tailwind v4** config syntax has changed from v3 — use the v4 docs.
- **Free tier limits are enforced server-side** — validate item/collection counts in API routes before writes.
- **Pro gate during dev:** All features accessible; `isPro` flag is the enforcement switch for production.
- **File uploads go directly to R2** — never buffer large files through the Next.js server. Use presigned URLs.
- **AI calls are server-side only** — never expose the OpenAI key to the client.
