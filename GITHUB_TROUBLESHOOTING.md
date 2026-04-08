# Khalorna Xpress — GitHub Setup & Troubleshooting Guide

---

## IMPORTANT: About Conversation / Transcript Files

The transcripts from Claude sessions are stored **inside Claude.ai only**.
They are **NOT** automatically uploaded to GitHub and cannot be accessed
from outside Claude. They exist solely so Claude can maintain context
within long sessions.

**What DOES go to GitHub:**
- `fleetflow.html` — the app itself (you upload this manually once)
- `data/kx_data.json` — app data (trips, users, vehicles) pushed automatically by the app
- `.github/workflows/deploy.yml` — auto-deployment workflow

---

## PART 1 — Why the Upload Is Failing

### Common Causes & Fixes

---

### Cause 1 — Wrong Token Format / Permissions

**Symptom:** Red toast says "Invalid token" or Settings test shows
`Token is invalid or expired`

**Fix:**
1. Go to https://github.com/settings/tokens
2. If your token starts with `ghp_` it is a Classic token — this is correct
3. If it starts with `github_pat_` it is a Fine-grained token — see note below
4. Click your token → confirm **repo** scope is ticked (full control of private repos)
5. If expired, click **Regenerate token**, copy the new value
6. In the app: Settings → paste new token → Save & Test

**Fine-grained tokens note:**  
Fine-grained tokens need these permissions on your repo:
- Contents: **Read and write**
- Metadata: **Read-only**

---

### Cause 2 — Wrong Repo Name Format

**Symptom:** Test shows `Repo "..." not found`

**Fix:**  
The repo field must be `owner/repo`, for example:
```
khalorna-xpress/khalorna-xpress
```
NOT just `khalorna-xpress` and NOT the full URL.

---

### Cause 3 — data/kx_data.json File Does Not Exist

**Symptom:** First-ever push fails with a 404 or encoding error

**Fix — create the file manually in GitHub first:**
1. Open your repo on GitHub
2. Click **Add file → Create new file**
3. In the filename field type: `data/kx_data.json`
   (the `/` creates the folder automatically)
4. Paste this content exactly:
```json
{
  "v": 6,
  "nid": 200,
  "_note": "Managed by Khalorna Xpress app",
  "invites": [], "expenses": [], "notifications": [],
  "recurring": [], "destinations": [], "users": [],
  "vehicles": [], "contracts": [], "trips": []
}
```
5. Click **Commit new file**
6. Go back to the app Settings → click **⬇ Pull Now**

---

### Cause 4 — Repository Is Private Without Correct Plan

**Symptom:** GitHub Pages not working OR 403 error in sync

**Fix:**  
GitHub Pages is free only on **public** repos (unless you have GitHub Pro/Teams).

To make your repo public:
1. Open repo → **Settings** → scroll to **Danger Zone**
2. Click **Change repository visibility** → Public

---

### Cause 5 — GitHub Pages Not Enabled

**Symptom:** App URL returns 404

**Fix:**
1. Repo → **Settings** → **Pages** (left sidebar)
2. Source: choose **GitHub Actions**
3. Save
4. Go to **Actions** tab → run the workflow manually if it hasn't started

---

### Cause 6 — Data File Too Large (1MB GitHub API limit)

**Symptom:** Sync fails silently after many trips, or toast shows
`Data too large for GitHub sync`

**Fix:**
- The app automatically strips POD photos before pushing — but if you
  have hundreds of trips with long notes, the file can still grow large
- Use **Export Backup** (Settings → Data Management) to save a local copy
- Then clear old completed trips from the app to reduce database size
- The 1MB limit is a GitHub API restriction, not the app's

---

### Cause 7 — Browser CORS or Security Policy Blocking GitHub API

**Symptom:** Sync never fires, no toast appears, console shows
`blocked by CORS policy` or `net::ERR_BLOCKED_BY_CLIENT`

**Fix:**
- Disable ad blockers / privacy extensions for the app's GitHub URL
- Try a different browser (Chrome or Safari work best)
- Corporate networks may block `api.github.com` — use mobile data to test

---

## PART 2 — Step-by-Step: Upload fleetflow.html to GitHub

### Method A — GitHub Web Interface (no software needed)

1. Go to https://github.com and log in
2. Open your repo (e.g. `github.com/khalorna-xpress/khalorna-xpress`)
3. Click **Add file → Upload files**
4. Drag `fleetflow.html` into the upload box
5. Scroll down — write commit message: `Update app`
6. Click **Commit changes**
7. GitHub Actions will deploy it automatically (check the **Actions** tab)

### Method B — Replace Existing File (when you have already uploaded before)

1. In your repo, click on `fleetflow.html`
2. Click the pencil ✏️ edit icon (top right of file view)
3. Click the trash 🗑️ icon to delete, then re-upload — OR
4. Use Method C below (command line) for reliable large-file replacement

### Method C — Git Command Line

```bash
# First time setup
git clone https://github.com/YOUR_USERNAME/khalorna-xpress.git
cd khalorna-xpress

# Copy updated file into the folder
cp /path/to/fleetflow.html .

# Commit and push
git add fleetflow.html
git commit -m "Update Khalorna Xpress app"
git push origin main
```

---

## PART 3 — Create All Required Files

You need exactly these files in your GitHub repo:

```
khalorna-xpress/          ← repo root
├── fleetflow.html         ← the app (upload manually)
├── data/
│   └── kx_data.json       ← app data (create manually, updated by app)
└── .github/
    └── workflows/
        └── deploy.yml     ← auto-deploy workflow (copy from files provided)
```

### Create data/kx_data.json

In your repo: **Add file → Create new file** → type `data/kx_data.json`

Paste this content:
```json
{
  "v": 6,
  "nid": 200,
  "_note": "Managed by Khalorna Xpress app. Do not edit manually.",
  "invites": [], "expenses": [], "notifications": [],
  "recurring": [], "destinations": [], "users": [],
  "vehicles": [], "contracts": [], "trips": []
}
```

### Create .github/workflows/deploy.yml

In your repo: **Add file → Create new file** → type `.github/workflows/deploy.yml`

Paste the contents of the `deploy.yml` file provided in this package.

---

## PART 4 — Configure Data Sync in the App

1. Open your live app: `https://YOUR_USERNAME.github.io/khalorna-xpress/fleetflow.html`
2. Log in as **admin / admin123**
3. Tap **⚙️ Settings** (last tab in the bottom nav)
4. Scroll to **GitHub Data Sync**
5. Fill in:

| Field | Value |
|---|---|
| Personal Access Token | `ghp_xxxxxxxxxxxx` (from github.com/settings/tokens) |
| Repo | `khalorna-xpress/khalorna-xpress` |
| Branch | `main` |
| File Path | `data/kx_data.json` |

6. Tap **💾 Save & Test** — you should see a green message with your GitHub username
7. If red, the message tells you exactly what is wrong

---

## PART 5 — Cross-Device Sync Explained

| What syncs automatically | How |
|---|---|
| Trip data, users, vehicles, payroll | `sDB()` pushes to GitHub 2.5s after every change |
| Repo / branch / file path config | Stored in `db.settings` — travels with the data |
| Personal Access Token | Stays in browser only (security) — enter once per device |
| App HTML file | Must be uploaded manually when you get a new version |

**On a new device or new browser:**
1. Open the app URL
2. Enter token in Settings (one time per device)
3. Tap **⬇ Pull Now** — imports all data from GitHub
4. From then on, changes sync automatically

---

## PART 6 — Diagnosing Sync Issues Live

Open the browser **Developer Tools** (F12 or Cmd+Opt+I):

1. Click the **Console** tab
2. Trigger a data change (mark a trip paid, change a setting)
3. Watch for messages:
   - `KX gDB error` — database is corrupted, clear `kx_v6` from Application → Local Storage
   - `GH push error` — network or token problem; details shown
   - `GH pull error` — same
4. Click the **Network** tab → filter for `api.github.com`
5. Click the failing request → check **Response** for the exact GitHub error message

---

## PART 7 — Emergency Recovery

### App data lost / corrupted
1. Open the app
2. Go to Settings → **⬇ Pull Now** (restores from GitHub)

### GitHub data also lost
1. Open your repo on GitHub
2. Click **History** on `data/kx_data.json`
3. Find the last good version → click **<>** (browse at this point)
4. Copy the file content → go to current version → Edit → paste → Commit

### Nuclear reset (start fresh)
1. In browser: F12 → Application → Local Storage → delete `kx_v6`
2. Reload the page — seed data re-initialises
3. Then tap **⬇ Pull Now** if you want to restore from GitHub

---

*Khalorna Xpress Ltd — Haulage Operator Management System*
*Support: 876-559-3028*
