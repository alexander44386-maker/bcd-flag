# Flag Atlas 🌍

A self-contained, single-file web game for learning world flags, capitals, and fun facts — built as a Duolingo-style path with daily lessons and quizzes.

No build step, no backend, no dependencies other than a Google Fonts stylesheet. Progress is saved locally in the player's browser via `localStorage`.

## Run it locally

Just open `index.html` in any browser — double-click it, or:

```bash
# Python
python3 -m http.server 8000
# then visit http://localhost:8000

# or Node
npx serve .
```

## Deploy to GitHub

```bash
cd flag-atlas
git init
git add .
git commit -m "Initial commit: Flag Atlas"
git branch -M main
git remote add origin https://github.com/<your-username>/<your-repo>.git
git push -u origin main
```

(Create the empty repo on GitHub first at https://github.com/new, without a README, then run the commands above from inside this folder.)

## Deploy to Render

This repo includes a `render.yaml`, so Render will auto-detect it as a **Static Site**:

1. Push the repo to GitHub (above).
2. Go to https://dashboard.render.com → **New** → **Static Site**.
3. Connect your GitHub account and select this repository.
4. Render will read `render.yaml` automatically. If asked to confirm settings manually instead:
   - **Build Command:** *(leave blank)*
   - **Publish Directory:** `.`
5. Click **Create Static Site**. Render will give you a live URL like `https://flag-atlas.onrender.com` within a minute or two.

That's it — every time you push to `main`, Render redeploys automatically.

## Set up cross-device cloud save (Firebase) — do this once

By default the game saves to `localStorage` only, which is per-browser. To make **Save** sync across your phone and computer, you need a free Firebase Realtime Database and a one-line edit to `index.html`. Takes about 5 minutes, no credit card required.

1. Go to https://console.firebase.google.com → **Add project** → give it any name (e.g. `flag-atlas`) → finish the wizard (you can skip Google Analytics).
2. In the left sidebar, go to **Build → Realtime Database** → **Create Database**.
   - Pick any region.
   - When asked about security rules, choose **Start in test mode** (this allows public read/write — fine here since there are no accounts; see the security note below).
3. After it's created, copy the database URL shown at the top of the Realtime Database page. It looks like:
   ```
   https://flag-atlas-xxxxx-default-rtdb.firebaseio.com
   ```
4. Open `index.html` in this project, find this line near the top of the `<script>` block (search for `FIREBASE_DB_URL`):
   ```js
   const FIREBASE_DB_URL = "https://YOUR-PROJECT-ID-default-rtdb.firebaseio.com";
   ```
   Replace it with the URL you copied in step 3.
5. Save the file, commit, and push:
   ```bash
   git add index.html
   git commit -m "Connect Firebase cloud save"
   git push
   ```
   Render redeploys automatically.

That's it — clicking **Save** in the game now writes to Firebase, and any device that opens your Render URL will load the same shared progress.

### Security note
Test mode makes the database **publicly readable and writable by anyone who has the URL** — there's no login, by design (you chose "one shared save, no accounts"). That's appropriate for a personal/family project. If you ever want to lock it down (e.g. only your domain can write to it), go to **Realtime Database → Rules** in Firebase and tighten the rules — ask me and I can write them for you.

### Test mode rules expire after 30 days
Firebase's "test mode" rules automatically revert to fully locked-down (no access at all) 30 days after creation — this is a safety default, not something Render or this app controls. Before then, go to **Realtime Database → Rules** and set:
```json
{
  "rules": {
    "save": {
      ".read": true,
      ".write": true
    }
  }
}
```
This scopes public access to just the `save` key (instead of the whole database) and never expires. Click **Publish**.

## Notes

- Progress is stored in Firebase Realtime Database (shared across devices). If the cloud save is unreachable (offline, or before you've set `FIREBASE_DB_URL`), the game falls back to a local copy in `localStorage` automatically.
- The only other external network call is for Google Fonts; the rest of the game runs entirely client-side.
