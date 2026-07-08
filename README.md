# Milesta

Turn a goal into a plan you'll actually follow. Milesta is the client for a goal-planning app where you build (or AI-generate) a roadmap, break it into milestones, check things off as you go, and optionally borrow — "fork" — a roadmap someone else already proved out.

This is the Next.js frontend. It talks to the [Milesta API](../timeline-app-backend-main) for everything except AI copy generation for ad-hoc segments, which is handled by a local route in this app.

## What you can do with it

- **Plan a goal two ways** — a fixed-cadence **Roadmap** (daily/weekly/monthly, set duration) or a free-form **Chronicle** of milestones with no schedule attached
- **Generate a plan instead of building one** — describe the goal, domain, skill level, and audience, and get a full set of milestones with goals and reference links back
- **Track progress** — check off goals inside a segment, mark a segment complete, or schedule it to a specific date
- **Browse and fork public roadmaps** on the Explore page — copy someone else's plan into your account and make it yours
- **Manage your account** — update your name, upload and switch between avatar photos
- **Pick a theme** — Emerald, Indigo, or Slate, from Settings

## Pages

| Route | What lives there |
|---|---|
| `/` | Public landing page |
| `/login` | Sign in / sign up |
| `/dashboard` | Your roadmaps and chronicles |
| `/dashboard/timeline/[id]` | A single plan's milestone board |
| `/explore` | Public roadmaps from the community, forkable |
| `/settings` | Theme and preferences |
| `/user/account` | Profile, avatar, account details |

## Tech stack

| Layer | Choice |
|---|---|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS v4 |
| Animation | Motion (Framer Motion successor) |
| Icons | Lucide React |
| State | React Context (`TimelineContext`) + `localStorage`-backed session |
| AI | Google Gemini (`@google/genai`), called from a local API route |

## Getting started

### Prerequisites

- Node.js 18+
- The [Milesta API](../timeline-app-backend-main) running somewhere reachable (defaults to `http://localhost:8000`)

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment

Copy `.env.example` to `.env.local`:

```env
# Where the Milesta API is running
NEXT_PUBLIC_API_URL="http://localhost:8000"

# Optional — enables AI segment generation from a text prompt.
# Without it, generation silently falls back to a local template generator, so the app stays fully usable.
GEMINI_API_KEY=""

# The public URL of this app itself (used for self-referential links)
APP_URL="http://localhost:3000"

# Disable Webpack HMR if it's causing issues in your environment
DISABLE_HMR="false"
```

### 3. Run it

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). If the API isn't reachable, most read paths quietly fall back to a local mock dataset (`services/mockData.ts`) so the UI still renders — but writes need a live backend.

## Project layout

```
app/
├── page.tsx                          # Landing page
├── login/page.tsx                    # Auth gateway
├── (app)/                            # Authenticated shell (sidebar + header)
│   ├── dashboard/page.tsx
│   ├── dashboard/timeline/[id]/page.tsx
│   ├── explore/page.tsx
│   ├── settings/page.tsx
│   └── user/account/page.tsx
└── api/segments/generate/route.ts    # Gemini-backed AI generation, with local fallback

components/     # One component per page/section (LandingPage, DashboardTab, ExploreTab, ...)
hooks/          # TimelineContext — the single source of truth for session + data state
services/       # timelineService.ts (backend client) + mockData.ts (offline fallback)
types/          # Shared TypeScript types for timelines and segments
```

## How auth works here

The client stores a JWT in `localStorage` and attaches it to every request. If the API responds `401`, `fetchWithAuth` (in `services/timelineService.ts`) transparently calls `/api/auth/refresh` and retries the original request once — concurrent requests during a refresh are queued and replayed rather than each triggering their own refresh call.

## Production build

```bash
npm run build
```

This compiles in Next's `standalone` output mode (`next.config.ts`), producing a `.next/standalone` folder with only what's needed to run the server — no full `node_modules` required in the deploy image.

## Docker

```bash
docker build -t milesta-client .
docker run -p 3000:3000 milesta-client
```

## Scripts

| Command | Purpose |
|---|---|
| `npm run dev` | Start the dev server |
| `npm run build` | Production build |
| `npm start` | Run the production build |
| `npm run lint` | Lint the codebase |
| `npm run clean` | Remove the `.next` build output |
