# Grocery Buddy — Setup Guide

This turns your app into something everyone in the family can install on their phone and
use together, with the list/pantry/recipes synced live between everyone.

There are two steps: (1) set up free shared storage, (2) put the app online.

---

## Step 1 — Shared storage (Supabase, free)

1. Go to https://supabase.com and sign up (free tier is plenty for this).
2. Click **New Project**. Pick any name/password/region, wait ~2 minutes for it to spin up.
3. Once it's ready, go to the **SQL Editor** (left sidebar) and run this:

   ```sql
   create table household_data (
     key text primary key,
     value jsonb,
     updated_at timestamptz default now()
   );

   alter table household_data enable row level security;

   create policy "Allow anonymous read" on household_data
     for select using (true);

   create policy "Allow anonymous insert" on household_data
     for insert with check (true);

   create policy "Allow anonymous update" on household_data
     for update using (true);

   alter publication supabase_realtime add table household_data;
   ```

   This creates the table that stores your grocery list, pantry, categories, and
   recipes, and turns on live sync so everyone sees updates instantly. (It's wide
   open on purpose for now, since there's no login yet — fine for family testing,
   worth locking down before a public launch.)

4. Go to **Project Settings > API**. Copy the **Project URL** and the **anon public** key.
5. Open `config.js` in this folder and paste them in:

   ```js
   window.SUPABASE_URL = "https://your-project.supabase.co";
   window.SUPABASE_ANON_KEY = "your-anon-key-here";
   ```

---

## Step 2 — Put it online

Easiest option, no account needed:

1. Go to https://app.netlify.com/drop
2. Drag this whole folder (`index.html`, `config.js`, `manifest.json`, `sw.js`,
   the icon files) onto the page.
3. Netlify gives you a live URL in seconds (something like `random-name.netlify.app`).
4. Text that link to your family.

(Once you're happy with things, you can claim a Netlify account to get a permanent
URL and easy re-deploys — same drag-and-drop, just tied to your account.)

---

## Step 3 — Install on a phone

**iPhone (Safari):** open the link → tap the Share icon → **Add to Home Screen**.

**Android (Chrome):** open the link → tap the **⋮** menu → **Add to Home screen** /
**Install app**.

It'll show up with its own icon and open full-screen, like a normal app.

---

## Later — individual accounts

When you're ready to move past "everyone shares one list," Supabase has built-in
auth (email/password, magic links, or Google/Apple sign-in) that plugs into the
same database. That mainly means: add a login screen, and tag each row of data
with a `user_id` instead of one shared row per list. Happy to build that when you're there.

---

## Later — the app stores

Once this is solid, the fastest way onto the Apple App Store and Google Play
**without a rewrite** is a tool called [Capacitor](https://capacitorjs.com/). It
wraps this same web app into real iOS/Android app files you submit for review.
Broad strokes:

1. `npm install @capacitor/core @capacitor/cli`
2. `npx cap init`, `npx cap add ios`, `npx cap add android`
3. Point Capacitor at this folder as the web assets, then open the generated
   projects in Xcode / Android Studio to build and submit.
4. You'll need an Apple Developer account ($99/year) and a Google Play
   developer account ($25 one-time).

I can walk through this step by step when you're ready for it.
