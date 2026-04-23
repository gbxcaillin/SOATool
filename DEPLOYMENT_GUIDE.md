# SOA Agent — Deployment Guide
### GitHub → Render → Live URL with Login
**GBX Professional Services · Internal Use Only**

---

## What You'll Have When Done

A live URL like `https://soa-agent.onrender.com` (or your own domain like `tools.gbxps.com`) that:
- Shows a branded GBX login page
- Only lets in users with a valid username + password
- Runs the full SOA completion tool
- Is accessible from any browser, anywhere

Total time: ~30 minutes. No coding required.

---

## What You Need Before Starting

- Your GitHub account (you said you have one)
- A free Render account — sign up at render.com using your GitHub login
- The 4 files from this folder: `app.py`, `index.html`, `requirements.txt`, `README.md`

---

## PART 1 — Put the Files on GitHub

### Step 1 — Create a new repository

1. Go to **github.com** and sign in
2. Click the **+** button top right → **New repository**
3. Name it: `soa-agent`
4. Set to **Private** (important — this is internal tooling)
5. Leave everything else as default
6. Click **Create repository**

---

### Step 2 — Upload the files

1. On the new repository page, click **uploading an existing file**
2. Drag and drop all 4 files into the upload area:
   - `app.py`
   - `index.html`
   - `requirements.txt`
   - `README.md`
3. Scroll down, click **Commit changes**

Your repository should now show all 4 files.

---

## PART 2 — Deploy on Render

### Step 3 — Create a Render account

1. Go to **render.com**
2. Click **Get Started** → **Sign up with GitHub**
3. Authorise Render to access your GitHub

---

### Step 4 — Create a new Web Service

1. In the Render dashboard, click **New +** → **Web Service**
2. Click **Connect a repository**
3. Find and select your `soa-agent` repository
4. Click **Connect**

---

### Step 5 — Configure the service

Fill in the settings exactly as follows:

| Setting | Value |
|---|---|
| **Name** | `soa-agent` (or anything you like) |
| **Region** | Singapore (closest to Melbourne) |
| **Branch** | `main` |
| **Runtime** | `Python 3` |
| **Build Command** | `pip install -r requirements.txt` |
| **Start Command** | `gunicorn app:app` |
| **Instance Type** | Free (to start — upgrade later if needed) |

Click **Create Web Service** — Render will now build and deploy. This takes about 2–3 minutes the first time.

---

### Step 6 — Set your environment variables (passwords live here)

This is where you set usernames and passwords. They are stored securely in Render — never in your code.

1. In your Render service dashboard, click **Environment** in the left sidebar
2. Click **Add Environment Variable** for each of the following:

---

**Variable 1 — Secret Key**

| Key | Value |
|---|---|
| `SECRET_KEY` | Any long random string, e.g. `gbxps-soa-agent-2024-xK9mP3qR7vL2` |

This encrypts the login sessions. Make it random, keep it secret.

---

**Variable 2 — Users**

| Key | Value |
|---|---|
| `USERS` | `username1:hash1,username2:hash2` |

Passwords are stored as hashes (scrambled — never plain text).

**To generate a hash for a password:**

Open Terminal (Mac) or Command Prompt (Windows) and run:
```
python3 -c "import hashlib; print(hashlib.sha256('YourPasswordHere'.encode()).hexdigest())"
```

Replace `YourPasswordHere` with the actual password. Copy the long string it outputs — that's the hash.

**Example — setting up two users:**

If you want:
- Username: `thomas` / Password: `Brightday2024!`
- Username: `rose` / Password: `Adviser2024!`

Run the python command twice (once for each password) to get two hashes, then set USERS to:
```
thomas:HASH_FOR_THOMAS,rose:HASH_FOR_ROSE
```

Paste the actual hashes in place of HASH_FOR_THOMAS etc.

---

3. Click **Save Changes**
4. Render will automatically redeploy with the new settings (takes ~1 minute)

---

### Step 7 — Test the live URL

1. In your Render dashboard, find your service URL at the top — it looks like:
   `https://soa-agent-xxxx.onrender.com`
2. Open it in a browser
3. You should see the GBX login page
4. Log in with one of your usernames and passwords
5. The SOA tool should appear

If it works — you're live. ✓

---

## PART 3 — Custom Domain (Optional but Recommended)

If you want `tools.gbxps.com` instead of the Render URL:

### Step 8 — Add custom domain in Render

1. In Render, go to your service → **Settings** → **Custom Domains**
2. Click **Add Custom Domain**
3. Type: `tools.gbxps.com`
4. Render will show you a CNAME record to add

### Step 9 — Add the DNS record

1. Log into wherever gbxps.com DNS is managed (likely your domain registrar — GoDaddy, Namecheap, etc.)
2. Go to DNS settings
3. Add a new **CNAME record**:
   - **Name/Host:** `tools`
   - **Value/Points to:** the value Render gave you (looks like `soa-agent-xxxx.onrender.com`)
   - **TTL:** 3600 (or default)
4. Save

DNS propagation takes 5–30 minutes. After that, `tools.gbxps.com` will load your login page.

Render also provides a free SSL certificate automatically — so it'll be `https://tools.gbxps.com`.

---

## PART 4 — Link From Squarespace (Optional)

You don't need to embed anything. Just add a button or link on your Squarespace site:

1. In Squarespace, edit the page you want the link on
2. Add a **Button block** or **Text block**
3. Link it to `https://tools.gbxps.com` (or your Render URL)
4. Label it something like "SOA Agent — Staff Login"

You can put this on a password-protected Squarespace page if you want a double layer, but since the tool itself has its own login, it's not necessary.

---

## PART 5 — Adding or Changing Users

You never touch the code to change passwords. Just update the environment variable in Render.

1. Go to Render → your service → **Environment**
2. Find the `USERS` variable and click edit
3. Add, remove, or change username:hash pairs
4. Click **Save** — Render redeploys automatically

To generate a new hash: run the python command above with the new password.

---

## PART 6 — Keeping It Updated

When you make changes to the code (e.g. adding new field mappings):

1. Update the file(s) on GitHub — go to the file, click the pencil edit icon, make changes, commit
2. Render automatically detects the change and redeploys within ~2 minutes

No manual deployment steps needed.

---

## Costs

| Service | Cost |
|---|---|
| GitHub private repo | Free |
| Render Free tier | Free (spins down after 15 min inactivity — slow first load) |
| Render Starter tier | ~$7 USD/month (always-on, recommended for daily use) |
| Custom domain SSL | Free (included with Render) |

For an internal tool used by a small team daily, the Starter tier ($7/mo) is worth it to avoid the cold-start delay on the free tier.

---

## Troubleshooting

**Login page appears but says invalid credentials**
→ Double-check the hash was generated from the exact password (case-sensitive)
→ Make sure there are no spaces around the `:` in the USERS variable

**Render build fails**
→ Check the build logs in Render dashboard
→ Most common cause: a typo in requirements.txt

**Page loads but tool doesn't work**
→ Open browser developer tools (F12) → Console tab
→ Look for red error messages and share them

**Forgot to set SECRET_KEY**
→ Sessions won't persist — users get logged out instantly
→ Add the environment variable and redeploy

---

*Guide version 1.0 — GBX Professional Services*
