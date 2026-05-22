# 5serving PWA — Project Memory

## What This Is
A Progressive Web App for the **5serving 6-Month Healthy Eating Challenge** by Upskill Global Technologies / 5serving.com (Bengaluru, India).

Users track 5 daily nutrition "windows", log health improvements, and chat with an AI coach. Satish (admin) can see all registered users and their progress.

## Core Concept: The 5 Windows
| # | Name | Time | Nutrient |
|---|------|------|---------|
| W1 | Morning Boost | 6:00 AM | Vitamin C + Iron |
| W2 | Green Power | 10:30 AM | Iron + Folate + Calcium |
| W3 | Rainbow Salad | 1:00 PM | Calcium + Mg + Vit A |
| W4 | Afternoon Fruit | 4:00 PM | Potassium + Antioxidants |
| W5 | Dinner Greens | 7:30 PM | Zinc + B-Vits + Gut |

## Tech Stack
- **Frontend**: Vanilla HTML + React 18 (CDN) + Babel Standalone — single `public/index.html`
- **AI Coach**: Anthropic Claude (`claude-sonnet-4-20250514`) via `/api/chat` Edge Function
- **Database**: Supabase (Postgres) — users, daily_checks, health_entries tables
- **Auth**: Simple UUID-based user identity (no login required; UUID stored in localStorage)
- **Deployment**: Vercel (static public/ + Edge Functions in api/)
- **PWA**: manifest.json + sw.js in public/

## Project Structure
```
5serving-deploy/
├── public/
│   ├── index.html       ← main app (React, Babel standalone, Supabase client)
│   ├── manifest.json    ← PWA manifest
│   ├── sw.js            ← service worker (offline + install)
│   └── icon-512.png     ← app icon (generate if missing)
├── api/
│   └── chat.js          ← Vercel Edge Function (proxies Anthropic API)
├── vercel.json          ← routing config
├── CLAUDE.md            ← this file
└── README.md
```

## Environment Variables (set in Vercel dashboard)
| Variable | Description |
|----------|-------------|
| `ANTHROPIC_API_KEY` | Anthropic API key for AI coach |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon/public key |

## Supabase Schema
```sql
-- Run these in Supabase SQL editor

create table users (
  id text primary key,
  name text not null,
  phone text not null,
  email text not null,
  phase integer default 1,
  joined_at timestamptz default now()
);

create table daily_checks (
  user_id text references users(id) on delete cascade,
  date text not null,
  checks jsonb default '{}',
  primary key (user_id, date)
);

create table health_entries (
  id uuid default gen_random_uuid() primary key,
  user_id text references users(id) on delete cascade,
  date text not null,
  energy integer,
  sleep integer,
  skin integer,
  mood integer,
  notes text,
  created_at timestamptz default now()
);

-- Enable Row Level Security (allow all for now; tighten later)
alter table users enable row level security;
alter table daily_checks enable row level security;
alter table health_entries enable row level security;

create policy "allow_all_users" on users for all using (true) with check (true);
create policy "allow_all_checks" on daily_checks for all using (true) with check (true);
create policy "allow_all_health" on health_entries for all using (true) with check (true);
```

## Admin Access
- Tap ⚙️ icon in app header
- Password: `admin@5serving`

## Key Design Decisions
- No user login/password — identity is a UUID in localStorage (simple onboarding)
- AI Coach prompts include user name, current phase, and streak for personalisation
- Phase progression: Foundation (W1+W2) → Building (W1+W2+W4) → Full 5 → Seasonal → Lifestyle → Champion
- Health entries store per user in Supabase; visible to admin

## Deployment Notes
- `vercel.json` rewrites `/api/*` to Edge Functions
- `public/` is served as static files
- After deploying, set ANTHROPIC_API_KEY + Supabase env vars in Vercel dashboard
- Test PWA install by opening the Vercel URL in Chrome on Android → "Add to Home Screen"
