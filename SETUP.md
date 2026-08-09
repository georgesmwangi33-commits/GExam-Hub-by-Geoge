# Exam Room — Setup Guide

Two pages:
- **index.html** — public page. Visitors enter the visitor password, then browse/download exams by subject.
- **admin.html** — your page only. Real login (email + password), upload exams, delete exams.

Both pull live data from a free Supabase project. Follow these steps in order.

---

## 1. Create your Supabase project

1. Go to https://supabase.com → sign up (no card needed) → **New project**.
2. Pick any name/region, set a database password (save it somewhere), wait ~2 min for it to spin up.

## 2. Create the database table

In your project, open **SQL Editor** → **New query**, paste this, and run it:

```sql
create table exams (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  subject text not null check (subject in ('geo','sport','ict')),
  duration text,
  file_path text not null,
  file_url text not null,
  created_at timestamptz default now()
);

alter table exams enable row level security;

-- anyone can read the exam list (needed for the public page)
create policy "public read" on exams
  for select using (true);

-- only signed-in users (you) can add or remove exams
create policy "admin write" on exams
  for insert with check (auth.role() = 'authenticated');

create policy "admin delete" on exams
  for delete using (auth.role() = 'authenticated');
```

## 3. Create the storage bucket

1. Go to **Storage** in the sidebar → **New bucket**.
2. Name it exactly `exams`, toggle **Public bucket** ON → Create.
3. Go to **Storage → Policies** for the `exams` bucket and add a policy allowing uploads/deletes only for authenticated users (the bucket UI walks you through this — choose "authenticated users" for INSERT and DELETE, and leave SELECT public since the bucket is public).

## 4. Create your admin login

1. Go to **Authentication → Users → Add user**.
2. Enter your own email and a password. This is what you'll use to log into `admin.html`.
3. (Authentication → Providers → make sure "Email" is enabled, and turn off "Confirm email" under Settings if you want to skip the verification email step.)

## 5. Get your API keys

Go to **Settings → API**. Copy:
- **Project URL**
- **anon public** key

Paste both into the top of **index.html** and **admin.html**, replacing `YOUR_SUPABASE_URL` and `YOUR_SUPABASE_ANON_KEY`.

## 6. Put the site on GitHub

1. Create a free GitHub account if you don't have one, and a new repository (e.g. `exam-room`).
2. Upload `index.html`, `admin.html`, and this file into it (GitHub's web upload works fine — no command line needed).

## 7. Deploy for free with Netlify (this makes it publicly visible and Google-indexable)

1. Go to https://netlify.com → sign up with your GitHub account.
2. **Add new site → Import an existing project** → pick your `exam-room` repo.
3. Leave build settings blank (it's a static site) → **Deploy**.
4. Netlify gives you a live URL like `exam-room-123.netlify.app`. You can rename it for free in **Site settings → Change site name**, or attach your own domain later.

That URL is public and crawlable — once it's live and linked from anywhere (or you submit it in Google Search Console), Google can index it.

## 8. Using it day to day

- Visit `yoursite.netlify.app/admin.html`, sign in, upload an exam (title, subject, duration, file). It appears instantly on the public page.
- Visitors go to `yoursite.netlify.app`, enter the visitor password, and see/download whatever you've uploaded — no more editing HTML by hand.
- Any time you push changes to the GitHub repo, Netlify redeploys automatically.

## Notes on security

- The **visitor password** on `index.html` is a soft gate (visible in page source) — good for keeping casual visitors out, not for protecting sensitive content from determined students.
- The **admin login** on `admin.html` is real authentication via Supabase — only accounts you create in the Supabase dashboard can upload or delete.
- The storage bucket is public, meaning anyone with a direct file link can open it — but links are only ever shown through the site's own listing.
