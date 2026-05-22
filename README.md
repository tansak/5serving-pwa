# 5serving PWA — Claude Code Deployment Guide

## What You Need Before Starting
- [ ] **Node.js 18+** installed → check with `node -v`
- [ ] **Git** installed → check with `git --version`
- [ ] **GitHub account** → github.com
- [ ] **Vercel account** → vercel.com (free, sign in with GitHub)
- [ ] **Supabase account** → supabase.com (free)
- [ ] **Anthropic API key** → console.anthropic.com

---

## STEP 1 — Install Claude Code

Open your terminal and run:

```bash
npm install -g @anthropic-ai/claude-code
```

Verify it works:
```bash
claude --version
```

---

## STEP 2 — Set Up the Project Folder

Copy the `5serving-deploy` folder from Claude (the one that contains this guide) to your computer. Then open terminal and navigate into it:

```bash
cd path/to/5serving-deploy
```

---

## STEP 3 — Set Up Supabase (Database)

1. Go to **supabase.com** → New Project → name it `5serving`
2. Once created, go to **SQL Editor** → click **New Query**
3. Paste and run this SQL:

```sql
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

alter table users enable row level security;
alter table daily_checks enable row level security;
alter table health_entries enable row level security;

create policy "allow_all" on users for all using (true) with check (true);
create policy "allow_all" on daily_checks for all using (true) with check (true);
create policy "allow_all" on health_entries for all using (true) with check (true);
```

4. Go to **Project Settings → API**
5. Copy:
   - **Project URL** (looks like `https://xxxx.supabase.co`)
   - **anon / public key** (the long string under "Project API keys")

---

## STEP 4 — Open Claude Code in Your Project

In your terminal (inside the `5serving-deploy` folder):

```bash
claude
```

This opens Claude Code. It will read the `CLAUDE.md` file and understand the project automatically.

---

## STEP 5 — Install Vercel Plugin for Claude Code

Inside Claude Code, type exactly:

```
/install-github-app
```

Then when prompted, also run:

```
npx plugins add vercel/vercel-plugin
```

This gives Claude Code the ability to deploy to Vercel directly.

---

## STEP 6 — Tell Claude Code to Prepare for Deployment

Type this prompt inside Claude Code:

```
I need to deploy this 5serving PWA to Vercel. Please:
1. Initialize a git repository in this folder
2. Create a GitHub repository called "5serving-pwa"
3. Push all files to GitHub
4. Set up Vercel deployment connected to the GitHub repo
5. The project uses a Vercel Edge Function at /api/chat and static files in /public
6. The root of the project serves public/index.html as index
```

Claude Code will run all the git and Vercel CLI commands for you.

---

## STEP 7 — Add Environment Variables in Vercel

After Claude Code deploys, you'll get a Vercel URL (like `5serving-pwa.vercel.app`).

Now add the secret keys:

1. Go to **vercel.com** → your project → **Settings → Environment Variables**
2. Add these three variables:

| Name | Value |
|------|-------|
| `ANTHROPIC_API_KEY` | Your Anthropic API key from console.anthropic.com |
| `NEXT_PUBLIC_SUPABASE_URL` | Your Supabase Project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Your Supabase anon/public key |

3. Click **Save** for each one
4. Go back to Claude Code and type:

```
Redeploy the project so the new environment variables take effect.
```

Or in Claude Code type: `/deploy`

---

## STEP 8 — Update index.html with Supabase Values

In Claude Code, type:

```
In public/index.html, replace 'YOUR_SUPABASE_URL' with the actual Supabase URL 
and 'YOUR_SUPABASE_ANON_KEY' with the actual anon key from our Supabase project. 
Then commit and push the changes.
```

Claude Code will edit the file and push automatically.

---

## STEP 9 — Add a Custom Domain (Optional)

If you have `5serving.com` registered:

1. Vercel dashboard → your project → **Settings → Domains**
2. Add `5serving.com` and `www.5serving.com`
3. Vercel will show you DNS records to add in your domain registrar

Or ask Claude Code:
```
Help me connect the custom domain 5serving.com to this Vercel project
```

---

## STEP 10 — Test PWA Install on Android

1. Open Chrome on your Android phone
2. Visit your Vercel URL
3. Tap the **three dots menu** → **Add to Home Screen**
4. Name it "5serving" → tap **Add**
5. The app icon appears on your home screen like a native app ✅

For iPhone:
1. Open Safari (must be Safari, not Chrome)
2. Visit your URL
3. Tap **Share button** (box with arrow) → **Add to Home Screen**

---

## Admin Access

Once deployed, tap the ⚙️ icon in the app header.
Password: `admin@5serving`

You'll see all registered users, their phases, join dates, and latest health check-in scores.

---

## Making Updates Later

Whenever you want to update the app, open Claude Code in the project folder:

```bash
cd 5serving-deploy
claude
```

Describe what you want to change, and end with:
```
After making changes, commit and deploy to production.
```

Or just type `/deploy` after Claude Code makes the changes.

---

## Useful Claude Code Commands

| Command | What it does |
|---------|-------------|
| `/deploy` | Deploy current code to Vercel production |
| `/vercel-logs` | See recent deployment logs |
| `/vercel-setup` | Reconnect to Vercel if needed |
| `git log --oneline` | See deployment history |

---

## Support

If anything goes wrong, open Claude Code and describe the error. Claude Code can read deployment logs, fix issues, and redeploy — all from the terminal.
