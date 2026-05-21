# Setup Guide — Login, Cloud Sync & Hosting

Your dashboard (`backend-learning-dashboard.html`) now supports real accounts: email + password login, "forgot password" reset emails, and progress that syncs across all your devices. This is powered by **Supabase** (free, no credit card needed). It handles the secure server side, so your dashboard can still be a single static file you host on GitHub Pages.

You only edit **one small block** in the HTML file. The whole setup takes about 10–15 minutes.

> Note: until you complete Part A and B, the dashboard still works — it just runs in "local only" mode (progress saved in that one browser, no login). You'll see a yellow notice. After setup, login and sync turn on automatically.

---

## Part A — Create your Supabase project (the backend)

1. Go to **https://supabase.com** and click **Start your project**. Sign in with GitHub or email.
2. Click **New project**.
   - **Name:** anything, e.g. `backend-pathway`
   - **Database Password:** generate a strong one and save it somewhere (you won't need it day-to-day, but keep it).
   - **Region:** pick the one closest to you.
3. Click **Create new project** and wait ~2 minutes while it provisions.

## Part B — Create the table that stores your progress

1. In your project's left sidebar, open **SQL Editor**.
2. Click **New query**, paste the SQL below, and click **Run**.

```sql
-- Table to hold each user's progress as JSON
create table if not exists public.progress (
  user_id uuid primary key references auth.users(id) on delete cascade,
  data jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

-- Turn on Row Level Security so users can only touch their OWN row
alter table public.progress enable row level security;

-- Allow a logged-in user to read their own progress
create policy "read own progress"
  on public.progress for select
  using (auth.uid() = user_id);

-- Allow a logged-in user to insert their own progress
create policy "insert own progress"
  on public.progress for insert
  with check (auth.uid() = user_id);

-- Allow a logged-in user to update their own progress
create policy "update own progress"
  on public.progress for update
  using (auth.uid() = user_id);
```

You should see "Success. No rows returned." That's correct — you just created the table and its security rules.

> **Why Row Level Security matters:** even though everyone shares one database, these policies guarantee each person can only ever read or write their own progress row. This is real, server-enforced security — the kind you'll learn to build in Level 5 of the pathway.

## Part C — Get your two keys and paste them in

1. In Supabase, go to **Project Settings** (the gear icon) → **API**.
2. Copy two values:
   - **Project URL** — looks like `https://abcd1234xyz.supabase.co`
   - **anon public** key — a long string starting with `eyJ...`
3. Open `backend-learning-dashboard.html` in any text editor (VS Code, Notepad, etc.).
4. Near the top of the `<script>` section you'll see this block. Replace the two placeholder strings:

```js
const SUPABASE_CONFIG = {
  url:  "YOUR_SUPABASE_URL_HERE",      // <- paste your Project URL
  anon: "YOUR_SUPABASE_ANON_KEY_HERE"  // <- paste your anon public key
};
```

5. Save the file.

> **Is it safe to put the anon key in public code?** Yes. The "anon public" key is *designed* to be exposed in browsers — it's how Supabase client apps work. Your data stays protected by the Row Level Security policies you created in Part B, not by hiding the key. (Never paste the **service_role** key, though — that one is secret. You won't need it here.)

### Optional: confirmation emails
By default Supabase sends a confirmation email on sign-up. For a personal tool you can turn this off so login is instant:
- **Authentication** → **Providers** → **Email** → toggle **Confirm email** off.
Leave it **on** if you want the extra verification step.

---

## Part D — Test it locally

1. Double-click `backend-learning-dashboard.html` to open it in your browser.
2. You should see a **login screen** (no more yellow warning).
3. Click **Create an account**, enter your email + a password (6+ characters), and sign up.
4. Tick a checklist item or pass a quiz, then open the same page in a different browser (or your phone after hosting) and log in — your progress should appear. The dot in the header shows sync status (green = synced).

---

## Part E — Host it on GitHub Pages

1. Create a new repository on GitHub, e.g. `backend-pathway` (Public).
2. Upload `backend-learning-dashboard.html`. **Rename it to `index.html`** so it loads at the root URL. (You can drag-and-drop it into the repo on github.com, or use git.)
3. In the repo, go to **Settings** → **Pages**.
4. Under **Build and deployment** → **Source**, choose **Deploy from a branch**.
5. Pick branch **main** and folder **/ (root)**, then **Save**.
6. Wait ~1 minute. Your site goes live at:
   `https://YOUR-USERNAME.github.io/backend-pathway/`

### One more step for password reset on the live site
So reset emails link back to your live site (not your laptop):
- In Supabase: **Authentication** → **URL Configuration**.
- Set **Site URL** to your GitHub Pages URL (e.g. `https://YOUR-USERNAME.github.io/backend-pathway/`).
- Add the same URL under **Redirect URLs**.

That's it. Visit your live URL on any device, log in, and you'll always continue exactly where you stopped.

---

## Quick troubleshooting

- **Login screen says "Supabase isn't configured":** the keys weren't pasted correctly, or still contain `YOUR_`. Re-check Part C.
- **"Invalid login credentials":** wrong email/password, or you haven't confirmed your email yet (check inbox/spam, or disable confirmation in the optional step).
- **Progress not syncing across devices:** make sure you ran the SQL in Part B (the `progress` table + policies must exist) and that you're logged into the *same account* on both devices. The header dot turns red if a sync fails.
- **Reset email link goes to localhost:** complete the "URL Configuration" step in Part E.
- **I just want it to work offline with no login:** leave the keys as placeholders. The dashboard runs in browser-only mode and saves locally.

## How to share it with friends
The dashboard allows open sign-up, so anyone who visits your live URL can create their own account and track their own progress privately (RLS keeps everyone's data separate). If you'd rather lock it to just yourself later, you can disable sign-ups in Supabase under **Authentication → Providers → Email → Allow new users to sign up**.
