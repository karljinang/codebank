# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

# CodeBank

A developer knowledge hub for snippets, commands, prompts, notes, files, images, links and custom types.

## Context Files

Read the following to get the full context of the project:

- @context/project-overview.md
- @context/coding-standards.md
- @context/ai-interation.md
- @context/current-feature.md

## Commands

```bash
npm run dev      # start dev server at localhost:3000
npm run build    # production build
npm run start    # serve production build
npm run lint     # run ESLint
```

No test runner is configured.

## Stack

- **Next.js 16.2.9** with the App Router — see `AGENTS.md` for the warning about breaking changes vs. training data
- **React 19.2.4**
- **TypeScript**
- **Tailwind CSS v4** — imported via `@import "tailwindcss"` in `globals.css` (no `tailwind.config` file; v4 uses CSS-first config)
- **Geist** fonts loaded via `next/font/google` and applied as CSS variables (`--font-geist-sans`, `--font-geist-mono`) on `<html>`

## Structure

All source lives under `src/app/` (App Router):

- `layout.tsx` — root layout; sets fonts, `<html lang="en">`, and `<body className="min-h-full flex flex-col">`
- `page.tsx` — home route (`/`)
- `globals.css` — only the Tailwind import; no custom variables or base styles
