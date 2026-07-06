# SermonCraft — accounts & saved library setup

This adds free sign-up (email + password) so visitors can save everything they generate and come back to it from any device — the "My Library" tab. It uses **Supabase** (free), which handles passwords, security, and reset emails so you never store a password yourself.

**Until you do this setup, the site works exactly as before** — the accounts features simply stay hidden. Takes about 15 minutes.

---

## Step 1 — Create a free Supabase project

1. Go to https://supabase.com and sign up (free).
2. Click **New project**. Name it `sermoncraft`, set a strong database password (save it somewhere — you rarely need it again), and pick a **region close to your users** (e.g. Singapore is a good pick if many users are in the Philippines / Southeast Asia).
3. Wait a minute or two while it sets up.

## Step 2 — Copy your two keys into the website

1. In your Supabase project, go to **Project Settings → API**.
2. Copy the **Project URL** and the **anon public** key.
3. Open `index.html` and near the top of the script find:
   ```
   const SUPABASE_URL = "";
   const SUPABASE_ANON_KEY = "";
   ```
   Paste your values between the quotes.

**Important — don't be alarmed that this key sits in the website code.** The anon key is *designed* to be public; the database rules you'll add in Step 3 are what keep each user's data private. This is completely different from your **Anthropic key**, which must stay secret on the server (in Vercel's environment variables) — never paste that one into `index.html`.

## Step 3 — Create the library table (copy-paste)

1. In Supabase, open **SQL Editor**, click **New query**, paste ALL of this, and click **Run**:

```sql
create table public.generations (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null default auth.uid() references auth.users(id) on delete cascade,
  tool text not null,
  title text,
  language text,
  content text not null,
  created_at timestamptz not null default now()
);

alter table public.generations enable row level security;

create policy "read own"   on public.generations for select using (auth.uid() = user_id);
create policy "insert own" on public.generations for insert with check (auth.uid() = user_id);
create policy "update own" on public.generations for update using (auth.uid() = user_id);
create policy "delete own" on public.generations for delete using (auth.uid() = user_id);

create table public.notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null default auth.uid() references auth.users(id) on delete cascade,
  title text,
  content text not null default '',
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

alter table public.notes enable row level security;

create policy "notes read own"   on public.notes for select using (auth.uid() = user_id);
create policy "notes insert own" on public.notes for insert with check (auth.uid() = user_id);
create policy "notes update own" on public.notes for update using (auth.uid() = user_id);
create policy "notes delete own" on public.notes for delete using (auth.uid() = user_id);
```

(If you already ran an earlier version of this SQL that only had the `generations` table, just run the `notes` half — everything from `create table public.notes` down.)

Those four "policies" are the security: each signed-in user can only ever see, save, edit, or delete **their own** rows. This is why the public anon key is safe.

## Step 4 — Decide on email confirmation (optional)

In **Authentication** settings, look for the **Confirm email** option for the Email provider:
- **On** (default): new users must click a link in their email before first sign-in. More secure, slight friction.
- **Off**: sign-up works instantly. Nice for a frictionless free tool; you can turn it on later.

## Step 5 — Publish and test

1. Re-upload the edited `index.html` to your GitHub repository (Vercel redeploys automatically).
2. On the live site: click **Sign in → Create an account**, sign up, generate a sermon outline, then open the **My Library** tab — it should be there, and the result panel should say "✓ Saved to your library."
3. Sign out, sign back in — your library should still be there. That's the whole feature working.

---

## Your newsletter opt-in list

At sign-up, users see an **unchecked** box: "Email me occasional SermonCraft news and ministry resources (optional)." Only those who check it are opted in — that's a genuine, legally clean list.

**Where the list lives:** on each user's account record in Supabase. To see or export it any time, open **SQL Editor** and run:

```sql
select email, created_at
from auth.users
where raw_user_meta_data->>'newsletter_opt_in' = 'true';
```

**When you're ready to actually send a newsletter (someday):**
- Supabase only sends *account* emails (confirmations, password resets) — it is not a newsletter tool.
- Sign up for a newsletter service (Buttondown, MailerLite, and Mailchimp all have free tiers), export the list with the SQL above, and import it there.
- Two rules that keep you on the right side of US email law (CAN-SPAM) and basic trust: only ever import people who checked the box, and every email must include a working unsubscribe link (the services above add this automatically).

---

## Good things to know

- **What users get:** everything they generate while signed in is saved automatically; they can reopen, copy, print, edit ("Save changes"), and delete items; "Forgot password" works via an emailed reset link, automatically.
- **Free tier:** generous for this project's scale. One quirk: Supabase may **pause a free project after ~a week of no activity** — if the site's accounts stop working after a quiet spell, open your Supabase dashboard and click restore (one click).
- **Emails:** Supabase's built-in email service (confirmation + password resets) is fine to start but has low sending limits. If sign-ups grow, connect your own SMTP provider in Supabase's auth settings.
- **Privacy — please do this:** you now store users' email addresses and their saved content (which can include pastoral notes about their congregations). Add a simple line to the site footer, e.g.: *"Your email and saved items are stored only so you can access your library. We never sell or share your data."* And treat it as true.
- **Anti-abuse option for later:** with accounts in place, you can require sign-in to generate (or give visitors a few free tries first). That's the strongest protection for your monthly API cap — ask and it can be wired in.
