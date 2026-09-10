# Deploying OpenGa to Streamlit Community Cloud

Streamlit Cloud deploys from a GitHub repository. It does not upload folders, so the
first half of this is getting the code onto GitHub; the second half takes about two
minutes.

Free tier gives you unlimited public apps and **one private app**. If the thesis is not
submitted yet, use the private repo route in step 2 and spend that one private slot here.

---

## 1. Decide what the repository contains

The repository root must be **`OpenGa_app_V1/`**, not `openga_project/`. Streamlit needs
`requirements.txt` and `.streamlit/` at the top level of the repo, and the app's imports
assume the repo root is the folder containing `openga_v1/`.

Copy the folder somewhere outside iCloud first. iCloud Drive and git do not get along —
`.git` gets partially evicted to the cloud and you end up with corrupt objects at the
worst possible moment.

```bash
cp -R ~/Library/Mobile\ Documents/com~apple~CloudDocs/Thesis/WP\'s/Latest\ Models/openga_project/OpenGa_app_V1 ~/OpenGa_app_V1
cd ~/OpenGa_app_V1
```

You should see this at the top level:

```
README.md  DEPLOY.md  requirements.txt  run.sh  selftest.py
.streamlit/config.toml
app/streamlit_app.py  app/charts.py
openga_v1/*.py
```

Confirm it still works before you push anything:

```bash
python3 selftest.py          # expect internal 14 of 14, regression 10 of 10
streamlit run app/streamlit_app.py
```

---

## 2. Put it on GitHub

```bash
cd ~/OpenGa_app_V1
git init -b main
git add .
git commit -m "OpenGa techno-economic and carbon model, Streamlit interface"
```

Then create the remote. With the GitHub CLI (`brew install gh` if you do not have it):

```bash
gh auth login
gh repo create openga-app --private --source=. --remote=origin --push
```

Use `--public` instead of `--private` if you are happy for it to be visible. Without the
CLI: create an empty repository at github.com/new, do **not** add a README or licence,
then

```bash
git remote add origin https://github.com/<your-username>/openga-app.git
git push -u origin main
```

---

## 3. Deploy

1. Go to **share.streamlit.io** and sign in with GitHub.
2. If the repo is private, authorise Streamlit for private repositories when prompted.
   It asks separately from the public-repo permission and is easy to skip past.
3. **Create app** → **Deploy a public app from GitHub** (this is also the route for
   private repos once authorised).
4. Fill in:
   - **Repository** `<your-username>/openga-app`
   - **Branch** `main`
   - **Main file path** `app/streamlit_app.py`
   - **App URL** whatever subdomain you want
5. Open **Advanced settings** and set **Python version to 3.11**. This matters — see
   the pinning note below.
6. **Deploy**. First build takes two to four minutes while it installs pandas.

The app is then at `https://<your-subdomain>.streamlit.app`. Every push to `main`
redeploys it automatically.

---

## 4. The one thing that will break it

`requirements.txt` is pinned to the exact versions the app was built and tested against:

```
streamlit==1.63.0
pandas==3.0.2
altair==6.2.2
```

**pandas 3.0 requires Python 3.11 or newer.** If you leave Cloud on an older default the
build fails with a wheel error that does not obviously say so. Set 3.11 in advanced
settings and it is fine.

Pinned versions are deliberate. Streamlit Cloud rebuilds from `requirements.txt` on every
push, so an unpinned `streamlit>=1.32` means an upstream release can change your app
without you touching it. That is a bad thing to discover the morning of a presentation.
If you later want the newest versions, relax the pins and redeploy well before you need
the app to work.

The app already handles the one API change that is in flight: Streamlit is renaming
`use_container_width` to `width="stretch"`, and `_width_kwargs()` in `streamlit_app.py`
detects which spelling the installed version takes rather than assuming.

---

## 5. Other things worth knowing

**Sleeping.** Free apps sleep after about a week without traffic and take roughly thirty
seconds to wake. Open it the day before you present so the first person to click does not
watch a spinner.

**Resources.** The free tier gives about 1 GB of memory. This app uses a fraction of it —
a single model run is 0.3 ms and nothing is cached to disk — so you will not hit the
limit. A five-thousand-trial Monte Carlo is still under five seconds.

**Nothing secret.** There are no API keys, no database and no secrets file, so there is
nothing to configure under Settings → Secrets. If you make the repo public, everything in
it is public, including every input value and source note. That is fine for a thesis
model but check it is what you want before flipping the repo to public.

**Custom subdomain.** You can change it later under Settings → General without
redeploying.

---

## If you would rather not use GitHub

**Local, shown on the projector** — simplest and has no failure modes you do not control:

```bash
cd ~/OpenGa_app_V1 && ./run.sh
```

**Local, but other people on the same wifi can open it on their laptops:**

```bash
streamlit run app/streamlit_app.py --server.address 0.0.0.0
```

Streamlit prints a Network URL like `http://192.168.1.42:8501`. Anyone on that network
can open it. This will not work through university guest wifi with client isolation, and
it will not work if your firewall blocks the port — test it on the actual network before
relying on it.

**Hugging Face Spaces** is the closest alternative to Streamlit Cloud: create a Space with
the Streamlit SDK, push the same files, rename `app/streamlit_app.py` to `app.py` at the
repo root or set `app_file` in the Space's `README.md` front matter. Free CPU tier, and
private Spaces are unlimited on the free plan.

---

## A note for the presentation itself

If someone asks whether they can play with it, the deployed link is the answer. Two
things to point them at, because both are honest limits rather than features:

- Every input on the **Inputs** tab carries its evidence status. Most are PLACEHOLDER or
  ESTIMATE. That is not a gap in the app, it is the state of the evidence.
- The **Checks & sources** tab shows what has actually been verified — arithmetic,
  closure and reproduction of the workbook — and states plainly that no input is
  validated by any of it.
