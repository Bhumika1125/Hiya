# Hiya — Setup Guide (No coding knowledge needed)

This folder is a real Android app project called **Hiya**. It already has all the
screens you asked for built: login/signup, the 4-step onboarding (period info,
medical conditions, food/location/allergies, body stats/workout style), and a
dashboard with Period Tracker, Diet Plan, Workout, Ayurvedic Remedies, and an
AI Assistant chat — all in your lilac theme.

You don't need to write any code. You just need to follow the steps below,
which are mostly clicking buttons in free tools.

---

## What you need to install first (all free)

1. **Android Studio** — this is the program that turns this project into an
   actual app. Download it from: https://developer.android.com/studio
   Install it like any normal program (Next, Next, Finish).

That's the only program you need on your computer. Everything else below
happens inside Android Studio or in your web browser.

---

## Step 1 — Open the project

1. Open Android Studio.
2. Click **"Open"** (not "New Project").
3. Select this `Hiya` folder (the one this README is in).
4. Wait a few minutes the first time — Android Studio downloads some pieces
   automatically. You'll see a progress bar at the bottom. Just let it finish.

---

## Step 2 — Turn on Google Sign-In (Firebase) — 5 minutes

Hiya's login is now a single **"Continue with Google"** button — the person
taps it, picks their Google account from the list Android shows them, and
they're signed in. No password, no OTP, and it's completely free with no
usage limits (unlike phone-number texting, which costs money past a small
free quota). You've already created your Firebase project and sent me its
`google-services.json` — I've placed it in `app/google-services.json`, and
you've since sent an updated one that already has Google Sign-In enabled.

I've also already generated the app's signing key (`app/hiya-debug.keystore`)
and wired it into the project, so the app's SHA-1 fingerprint is now fixed
and permanent — it'll be the exact same value no matter whose computer
builds the app, so you'll never need to go hunting for it in Android Studio.
The fingerprint is:

```
EA:BA:15:1B:F1:AA:C7:4D:C9:D3:8A:74:25:31:2D:72:87:A7:56:DA
```

One click left, only doable by you since it's inside your Firebase account:

1. Go to https://console.firebase.google.com and open your **"hiya-94693"**
   project.
2. **Project settings** (gear icon, top left) → scroll to **"Your apps"** →
   find the Hiya Android app → click **"Add fingerprint"** → paste the SHA-1
   above → Save.
3. Also turn on **Build → Firestore Database → Create database** if you
   haven't already (any region close to India, start in test mode). This is
   where each user's onboarding answers get saved.

That's it — tapping "Continue with Google" will now work. First-time users
go straight into the 4-step onboarding; returning users go straight to
their dashboard.

---

## Step 3 — Turn on the AI Assistant (Gemini) — 2 minutes

**This step is no longer optional** — every suggestion in the app (diet,
workout, Ayurveda, and the chat) is now generated live by Google's
**Gemini API**, which has a free usage tier. Without a key, those screens
will show a "something went wrong" message instead of a real plan.

1. Go to https://aistudio.google.com/apikey and sign in with any Google account.
2. Click **"Create API key"**. Copy the long string of letters/numbers it gives you.
3. In this project, open the file:
   `app/src/main/java/com/hiya/app/data/remote/GeminiAIService.kt`
4. Find this line near the top:
   ```
   private val apiKey = "PASTE_YOUR_GEMINI_API_KEY_HERE"
   ```
5. Replace the text between the quotes with your copied key, so it looks like:
   ```
   private val apiKey = "AIzaSy...your actual key..."
   ```
6. Save the file (Ctrl+S / Cmd+S), rebuild, and every suggestion — diet,
   workout, Ayurveda, and the "Ask Hiya AI" chat — will now be a real,
   personalized answer instead of a placeholder.

**Before you publish this app publicly** (Play Store or sharing widely):
anyone who downloads the app can technically extract this key from the file
and use it under your name. For testing on your own phone this is perfectly
fine. When you're ready to publish, message me and I'll move the key into a
small free backend (a Firebase Cloud Function) so it's never inside the app
itself — a quick change that won't disturb anything else.

---

## Step 3.5 — Granting people "Premium" access

**Important to understand:** the Gemini API key you just added is the same
for *every* copy of the app — it lives in the app itself, not tied to any
one person. So the key's location doesn't control who gets AI access; a
separate setting per person does that. Every AI-powered screen (Diet,
Workout, Ayurveda, the chat) checks one thing before showing real
suggestions: whether that person's profile has `isPremium` set to `true` in
your Firestore database. Everyone starts as `false` (free) by default.

Since there's no in-app payment system yet, you grant premium manually,
person by person, from the Firebase console:

1. Go to https://console.firebase.google.com → your **hiya-94693** project
   → **Build → Firestore Database → Data**.
2. Click the **"users"** collection, then find the person you want to
   upgrade (their document is named after their Google account's internal
   ID — easiest way to find the right one is to match the `name` or `email`
   field shown in the document preview).
3. Click into their document, find the `isPremium` field, and change its
   value from `false` to `true`. Save.
4. That's it — the next time they open the app (or pull-to-refresh, once
   we add that), their AI features unlock. No app update needed.

**Being upfront about the limits of this approach:** since the check
happens inside the app itself rather than on a server, it's a reasonable
gate for casual use, but not something that would stop a determined person
from bypassing it if they really tried (the same is true of the API key
being extractable, as noted above). If you eventually want this to be
airtight — say, once real paying customers are involved — the fix is
moving the AI calls behind your own backend that checks `isPremium` server-side
before ever using your key. Worth doing before a real public launch; not
urgent for testing with friends and family now.

---

## Step 4 — Connect Google Health Connect (optional) — and how the syncing actually works

The Home tab shows today's steps, calories burned, and last night's sleep,
read from Google's **Health Connect** app. For this to work:
1. The person using the app needs the free "Health Connect" app installed
   from the Play Store (on newer phones it's often built in already).
2. On the Home tab, tap **"Sync now"** the first time — Android will show a
   permission screen, they tap Allow, and the numbers appear.

**How "syncing throughout the day" actually works, honestly explained:**
Health Connect doesn't push live updates the instant someone takes a step —
no app is allowed to run continuously in the background on Android, by
design, to protect battery life. So "syncing throughout the day" means:
- A background job re-reads Health Connect every **30 minutes**
  automatically, even if Hiya isn't open, and caches the latest numbers.
- Opening the app always shows that latest cached reading immediately.
- Tapping **"Sync now"** gets a reading for *right now*, on demand, whenever
  they want it fresher than the last 30-minute cycle.

This is the same approach real fitness apps use — Android simply doesn't
allow anything more "live" than this without draining the battery fast.

No setup needed from you for any of this — it works automatically once
Health Connect is installed and permission is granted once.

---

## Step 5 — Run it on your phone

1. On your Android phone: Settings → About phone → tap "Build number" 7
   times (this turns on Developer Mode). Then go to Settings → Developer
   options → turn on **USB debugging**.
2. Plug your phone into your computer with a USB cable.
3. In Android Studio, at the top you'll see a green triangle "Run" button ▶
   and a dropdown showing your phone's name. Click the green ▶ button.
4. Android Studio installs the app on your phone automatically — Hiya will
   open on your screen in a minute or two.

---

## Step 6 — Build the file to upload / share (the .apk / .aab)

For testing and sharing with friends/testers right now, every build (Run
button, or Build → Build APK) already produces a properly signed file using
the fixed key I set up in Step 2 — nothing extra to do.

**Before publishing on the Google Play Store specifically**, you'll want
your own securely-stored release key instead of the convenience one I
generated (that one's password is sitting in plain text in the project
file, which is fine for development but not for a real public release):
1. In Android Studio's top menu: **Build → Generate Signed App Bundle / APK**.
2. Choose **Android App Bundle (.aab)**.
3. Follow the on-screen wizard to create a new signing key — this time
   store the password somewhere private (a password manager, not the
   project). Keep this file safe; you'll need the exact same one for every
   future update, forever — losing it means you can never update the app again.
4. The finished file appears in `app/release/`. That's what you upload to
   the Play Store.

---

## What's already built vs. what needs your input

**Fully working out of the box (no setup needed):**
- Google Sign-In login (once Step 2 is done)
- Period tracker with real interactive flow-logging (Light/Medium/Heavy — actually saves now)
- Meal logging — add a note, a photo from your gallery, or both; recent meals show below
- Profile screen showing everything Hiya knows about you, with an "Update my answers" button
- Lilac theme throughout

**Needs Step 3 (Gemini key) to actually respond:**
- Diet suggestions — personalized using your phase, profile, AND recently logged meals
- Workout suggestions — personalized using your phase and workout style
- Ayurvedic remedies — personalized using your phase and medical conditions
- The "Ask Hiya AI" chat
(All of these now run entirely through Gemini — the old rule-based
generators are still in the project under `logic/`, unused, in case you
ever want a free offline fallback for when there's no internet or API key.)

**Needs the setup steps above:**
- Google Sign-In (Step 2) — required for login to actually work
- Gemini AI (Step 3) — required for every suggestion in the app to work
- Health Connect (Step 4) — works automatically once installed + permission granted once

**Good next additions (ask me anytime and I'll build these into the project):**
- Uploading medical certificate/scan report files during onboarding
- A custom launcher icon (right now it's a simple placeholder bloom mark) and app store screenshots
- "Use my current location" turning into an actual region (currently the
  onboarding screen has the toggle, but converting GPS coordinates to a
  region name needs one more small piece)
- Push notifications ("Your period is due in 2 days")
- Camera capture for meal photos (right now you pick from your gallery — adding
  a direct in-app camera option is a small addition if you want it)
