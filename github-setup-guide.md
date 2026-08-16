# Publishing AnaTrail to GitHub

This guide walks through putting the AnaTrail project into a GitHub repository, step by step — no prior Git experience assumed. At the end, you'll also have the option to publish it as a live website for free using GitHub Pages.

**Updated for the multi-file project structure** (see `SCALABILITY-GUIDE.md`): this is no longer a single HTML file — it's a small project folder with a data file and a media folder, which is exactly the kind of thing Git and GitHub are built for.

---

## Before you start

You'll need:
- A GitHub account — sign up free at [github.com](https://github.com) if you don't have one.
- Git installed on your computer — download from [git-scm.com](https://git-scm.com/downloads) (Windows/Mac/Linux).
- All of the project's files and folders, saved together in one place on your computer:

  ```
  anatrail/
    index.html                 (rename from turkey-archaeology-map.html)
    data/
      sites.json
    media/
      README.md
      catalhoyuk/
        1.jpg  2.jpg  3.jpg  video.mp4
      aktopraklik-mound/
        1.jpg  2.jpg  3.jpg  video.mp4
      <your future sites>/
    SCALABILITY-GUIDE.md
    github-setup-guide.md
  ```

To check Git is installed, open a terminal (Mac/Linux: **Terminal**; Windows: **Git Bash**, installed alongside Git) and run:

```bash
git --version
```

If it prints a version number, you're set.

---

## Step 1 — Create a new repository on GitHub

1. Go to [github.com](https://github.com) and log in.
2. Click the **+** icon in the top-right corner → **New repository**.
3. Fill in:
   - **Repository name** — e.g. `anatrail` (lowercase, no spaces; use hyphens if needed).
   - **Description** (optional) — e.g. "A field map of archaeological sites across Turkey."
   - **Public** or **Private** — choose **Public** if you want GitHub Pages to host it for free; **Private** keeps it visible only to you (Pages still works on private repos with GitHub Pro, but Public is simplest).
   - **Do NOT** check "Add a README file" — we'll keep it empty for now to avoid conflicts.
4. Click **Create repository**.

GitHub will show you a page with setup instructions — keep this open, you'll need the repository URL from it (it looks like `https://github.com/your-username/anatrail.git`).

---

## Step 2 — Set up your local project folder

Create a folder and put **everything** from the project structure above inside it — the HTML file (renamed to `index.html`), the `data/` folder, the `media/` folder, and the two guide files:

```bash
mkdir anatrail
cd anatrail
```

Now copy in all the project files via Finder/File Explorer, or `cp`/`mv` in the terminal. Two things matter here:

- **Rename `turkey-archaeology-map.html` to `index.html`.** GitHub Pages (Step 6) automatically serves a file with that exact name as your site's homepage — no reason to wait, do it now so you don't forget later.
- **Keep the folder structure intact.** `index.html` expects to find `data/sites.json` and `media/<slug>/...` at those exact relative paths, right next to it.

---

## Step 3 — Test it locally before pushing (important)

Because `index.html` now loads `data/sites.json` with `fetch()`, **double-clicking the file will not work** — browsers block `fetch()` on files opened directly from disk (`file:///...`). You'll just see "Couldn't load site data."

Test it properly with a quick local server instead. From inside the `anatrail` folder:

```bash
python3 -m http.server 8000
```

Then open **http://localhost:8000** in your browser. You should see the map load with its pins. Press `Ctrl+C` in the terminal to stop the server when you're done. (See `SCALABILITY-GUIDE.md` for why this is necessary and other ways to serve it locally, e.g. VS Code's Live Server extension.)

---

## Step 4 — Initialize Git and make your first commit

Still inside the `anatrail` folder:

```bash
git init
git add .
git commit -m "Initial commit: AnaTrail archaeological map"
```

What these do:
- `git init` — turns the folder into a Git repository (tracks changes).
- `git add .` — stages all files and folders for the next commit, including `data/`, `media/`, and both guides.
- `git commit -m "..."` — saves a snapshot with a short message describing it.

> **Note on media file sizes:** if your 200 sites' photos/videos add up to a very large amount (GitHub has a 100 MB per-file limit and gets unwieldy well before that in total repo size), consider hosting `media/` on a dedicated image/video host or cloud storage instead, and pointing `demoMedia`-style URLs at that host rather than committing hundreds of megabytes of video into Git. For a first launch or a modest media budget, committing them directly as shown above is perfectly fine.

---

## Step 5 — Connect your local folder to GitHub

Set the default branch name and link to the repository you created in Step 1 (replace the URL with your actual repository URL):

```bash
git branch -M main
git remote add origin https://github.com/your-username/anatrail.git
```

---

## Step 6 — Push your code to GitHub

```bash
git push -u origin main
```

The first time, Git may prompt you to sign in — either through a browser popup or by pasting a **Personal Access Token** in place of a password (GitHub stopped accepting plain passwords for this in 2021). If asked to create one:

1. Go to **GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)**.
2. **Generate new token**, tick the **repo** scope, set an expiration, and generate it.
3. Copy the token and paste it as your password when Git asks.

Once it finishes, refresh your repository page on GitHub — `index.html`, `data/`, and `media/` should all be there.

---

## Step 7 — Publish it as a live website with GitHub Pages

1. On your repository's GitHub page, go to **Settings** → **Pages** (left sidebar).
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Under **Branch**, select **main** and folder **/ (root)**, then **Save**.
4. Wait a minute or two, then refresh the page — GitHub will show a green banner with your live URL, typically:

   ```
   https://your-username.github.io/anatrail/
   ```

Open that URL and confirm the map loads with its pins — Pages serves everything over HTTPS, so the `fetch('data/sites.json')` call that failed with `file://` in Step 3 works correctly here without any changes.

That URL is also the one you'd submit to a tool like Ecograder to get a real website score, once you're ready to replace the "Coming soon" badge.

---

## Making future updates

This is the part you'll do repeatedly as you add sites toward 200. Whenever you edit `data/sites.json`, add a new `media/<slug>/` folder, or change anything else:

```bash
git add .
git commit -m "Describe what you changed, e.g. Add Ephesus and Hattusa"
git push
```

If you used GitHub Pages, the live site updates automatically within a minute or two of each push.

**A good habit at this scale:** before committing, validate your JSON so a typo doesn't take down the whole site:

```bash
python3 -c "import json; json.load(open('data/sites.json', encoding='utf-8')); print('valid JSON, OK to publish')"
```

---

## Quick reference — all commands in order

```bash
mkdir anatrail
cd anatrail
# copy in: index.html, data/, media/, SCALABILITY-GUIDE.md, github-setup-guide.md

python3 -m http.server 8000   # test at http://localhost:8000, then Ctrl+C

git init
git add .
git commit -m "Initial commit: AnaTrail archaeological map"
git branch -M main
git remote add origin https://github.com/your-username/anatrail.git
git push -u origin main
# then enable GitHub Pages in repo Settings → Pages
```
