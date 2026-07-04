# Dropbox Folder Tracker

A single-page web app that lists the subfolders inside Dropbox **shared links**,
sorted newest-first, and highlights what's changed since the last time you looked.
Built for syncing 3D-print files with a collaborator.

- **No install** for the person using it — it's just a web page.
- **No server / backend** — everything runs in the browser.
- **No secrets in the code** — each user signs in with their *own* free Dropbox
  account via OAuth (PKCE). The app only ever reads.
- Remembers your links and "last seen" state in the browser (`localStorage`).

---

## One-time setup (you, the owner — ~3 minutes)

### 1. Create a Dropbox app
1. Go to <https://www.dropbox.com/developers/apps> → **Create app**.
2. Choose **Scoped access**.
3. Access type: **Full Dropbox** (needed so it can read arbitrary shared links).
4. Name it anything (e.g. `folder-tracker`).

### 2. Set permissions (scopes)
On the app's **Permissions** tab, enable:
- `account_info.read`
- `files.metadata.read`
- `sharing.read`
- `files.content.read`  *(only needed for the in-app Download buttons)*

Click **Submit**. (If you add scopes after anyone has connected, they'll need to
reconnect.)

### 3. Add the redirect URI
On the **Settings** tab, under **OAuth 2 → Redirect URIs**, add the exact URL
where you'll host the page, e.g.:

```
https://YOURNAME.github.io/dropbox-folder-tracker/
```

⚠️ It must match exactly, including the trailing slash. For local testing you can
also add `http://localhost:8000/`.

### 4. Fill in the app key
Copy the **App key** from the Settings tab and paste it into `index.html`:

```js
const CONFIG = {
  APP_KEY: "PASTE_YOUR_DROPBOX_APP_KEY",   // <-- put your app key here
  ...
};
```

The app key is *public* — it's safe to commit and host. There is **no app secret**
in this app (PKCE doesn't need one).

### 5. Host it
Any static host works. Easiest is **GitHub Pages**:
1. Put `index.html` in a repo.
2. Repo **Settings → Pages → Deploy from branch → main / root**.
3. Wait a minute, then open the Pages URL (must match the redirect URI from step 3).

Free alternatives: Netlify (drag-and-drop the folder), Cloudflare Pages.

---

## Using it (your collaborator)

1. Open the URL you send him.
2. Click **Connect Dropbox** once → log into his own Dropbox → **Allow**.
   (A free Dropbox account is fine. He does **not** need to own the files.)
3. Paste the shared link(s) you gave him → **Add link**. Optionally nickname each.
4. Click **Check for updates**.

He'll see every subfolder across all links, **newest first**, with a **NEW** badge
on anything modified since his last visit. Expand a folder to see its files
(newest first) with Download buttons. **Mark all as seen** clears the highlights
so next time only newer changes stand out.

His links and history are stored only in his browser on that computer.

---

## Notes & limits

- **Rate limits:** The app fetches each shared link's whole tree in one recursive,
  paginated call and limits itself to `CONFIG.CONCURRENCY` (default 3) links at
  once. If Dropbox returns HTTP 429, it reads the `Retry-After` header and backs
  off automatically. For a handful of links you'll never notice.
- **Folder dates:** Dropbox doesn't give folders a meaningful "modified" time, so a
  folder's date here = the newest file inside it (`server_modified`, the reliable
  server-side timestamp).
- **Downloads** pull the file through the browser, so very large files use browser
  memory. Fine for typical STL/3MF/gcode; for multi-GB files, download from Dropbox
  directly.
- **Shared link format:** Works with modern `dropbox.com/scl/fo/...` folder links
  and older `/sh/` links, as long as they're "anyone with the link can view".
