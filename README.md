# GAARO — User App (GitHub Pages)

This folder is **ready to publish on GitHub Pages**. `index.html` is the full GAARO user
app (a single self-contained file). It is a **copy of `../GAARO-USER-APP.html`** — if you
change the app, re-copy it:

```bash
cp ../GAARO-USER-APP.html index.html
```

(The guard always serves the fresh `GAARO-USER-APP.html` itself at `http://localhost:8787/`;
this `docs/index.html` is only for the GitHub-hosted copy.)

---

## Publish it

1. Push this repo to GitHub.
2. Repo **Settings → Pages**.
3. **Source:** *Deploy from a branch* → branch `main`, folder **`/docs`** → Save.
4. After a minute your app is live at `https://<you>.github.io/<repo>/`.

---

## How the hosted app reaches your guard — read this

The app is just a screen. The **guard** (the SCC: camera + recognition + the API) runs on
your machine at home. The hosted page has to talk to it, and a browser imposes two rules:

1. **HTTPS only.** A page served over `https://` (GitHub Pages always is) may not call a
   plain `http://` address. So the app must reach the guard over **HTTPS** →
   **start the guard with `GAARO_HTTPS=on`** and use its **`:8788`** address, not `:8787`.

2. **Accept the certificate once.** The guard signs its own (self-signed) certificate. The
   browser won't trust a self-signed cert for background calls until you've accepted it. So,
   one time per browser: open the guard's HTTPS address **directly**
   (e.g. `https://192.168.1.5:8788/`) → *Advanced → Proceed*. After that, the hosted app's
   calls to that address work.

Cross-origin calls are already allowed by the guard (it sends permissive CORS headers).

Cross-origin calls from the hosted page must be **opted in** on the guard (it is closed by
default — the guard isn't shaped around any host). Set the page's origin:
`GAARO_CORS_ORIGINS="https://<you>.github.io"`.

### Steps to test from your laptop browser

1. Start the guard with HTTPS **and your hosted origin allowed**:
   ```powershell
   $env:GAARO_FACE_RECOGNITION="on"; $env:GAARO_PROFILE="opencv"
   $env:GAARO_FACE_MODEL="antelopev2"; $env:GAARO_HTTPS="on"
   $env:GAARO_CORS_ORIGINS="https://<you>.github.io"
   python run_scc.py
   ```
2. In the laptop browser, open `https://localhost:8788/` once → **Advanced → Proceed**
   (accept the cert).
3. Open your GitHub Pages URL (`https://<you>.github.io/<repo>/`).
4. The app will say **"Connect to your guard"** → enter **`https://localhost:8788`** (or
   `https://<guard-ip>:8788` from another device) → Connect → sign in.

---

## The honest limitation (important)

This makes the hosted app work **on your own network**, from a browser that has accepted the
guard's certificate. It does **not** give you access from *outside* your home (different
network / mobile data), because the guard still lives on your private LAN and the self-signed
cert + LAN IP only work locally.

**True remote access is the POC-2 cloud plane** — a cloud relay/tunnel so the app (here on
GitHub Pages) talks to a public endpoint that bridges to the guard at home, plus push
notifications and a TURN-backed WebRTC path for live video. This `docs/` layout is the
groundwork: when that bridge exists, you just point the app at the cloud endpoint instead of
the LAN IP, and nothing else changes.
