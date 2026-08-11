# Document Kiosk — Setup Guide

A static website that shows a table of contents on the first screen and
displays PDFs when tapped. Content lives entirely in this GitHub repo, so you
never need to touch the iPad to update it.

## How it works

- `manifest.json` lists the documents (title, description, PDF filename).
  The homepage builds its table of contents from this file.
- `pdfs/` holds the actual PDF files.
- The page polls `manifest.json` every 5 minutes. If `version` has changed,
  it refreshes the TOC automatically — so a content update on GitHub reaches
  the iPad within a few minutes, with zero interaction on the device.
- PDFs render using Safari's native PDF viewer inside an iframe (no external
  library needed, works fully offline-tolerant/cached once loaded).
- After 3 minutes of inactivity while viewing a PDF, it automatically returns
  to the table of contents (good for a kiosk that gets walked away from).

## 1. Publish with GitHub Pages

1. Push this folder to a new GitHub repo (public, or private if you're on a
   plan that supports Pages for private repos).
2. In the repo: **Settings → Pages → Source → Deploy from branch**, pick
   `main` and `/ (root)`.
3. GitHub gives you a URL like:
   `https://<your-username>.github.io/<repo-name>/`
4. That URL is what you'll lock the iPad to in Intune.

## 2. Updating content (no iPad involved)

To replace or add a PDF:

1. Upload the new PDF into `pdfs/` (via GitHub web UI: **Add file → Upload
   files**, or `git push` from your machine).
2. Edit `manifest.json`:
   - Update the `file` path if the filename changed.
   - Update `title` / `description` if needed.
   - **Bump the `version` string** (e.g. to today's date + a counter). This
     is what tells the iPad "something changed" — if you forget this step,
     the iPad won't auto-refresh until its next full reload.
3. Commit. GitHub Pages typically republishes within 30–90 seconds.
4. The iPad picks up the change on its next 5-minute poll — no manual reload.

To go from 2 to 3 or 4 documents (or back down), just add/remove entries in
the `documents` array in `manifest.json` and bump `version`.

## 3. Locking the iPad down with Intune

1. **Supervise the device.** Kiosk (Single App) mode in Intune requires the
   iPad to be supervised — typically via Apple Configurator or Automated
   Device Enrollment (ADE/DEP).
2. In **Intune admin center**: Devices → Configuration profiles → Create →
   Platform: iOS/iPadOS → Profile type: **Templates → Kiosk**.
3. Choose **Single app mode**, select **Safari** as the app, and set the
   **Home URL** to your GitHub Pages URL from step 1.
4. Recommended Safari kiosk restrictions (all available in the same
   profile): disable the address bar / prevent navigation away from the URL,
   disable auto-fill, disable "Prevent website access" toggles that would
   block it, disable the ability to open new tabs.
5. Also worth setting: disable Auto-Lock (Settings → Display in a device
   restriction profile) so the screen doesn't sleep, and consider a
   restriction profile that disables the Home/side-button gestures you don't
   want available.
6. Assign the profile to the device/group and it pushes over the air — no
   physical access needed for future kiosk-config changes either.

## Customizing

- Colors, spacing, fonts: edit the `<style>` block at the top of
  `index.html`.
- Poll frequency: change `CHECK_INTERVAL_MS` in the `<script>` block.
- Idle-return-to-TOC delay: change `IDLE_RESET_MS` (set to a very large
  number to disable it).

## Notes / things to double check

- If your PDFs are large, first load per document may take a moment on
  cellular/weak Wi-Fi — this is a plain iframe load with no progress bar
  built in. Let me know if you'd like a loading spinner added.
- This assumes the iPad has network access to GitHub Pages at all times. If
  the kiosk needs to work offline, that's a different architecture (bundling
  PDFs locally + an MDM-pushed content update mechanism) — happy to sketch
  that out if it turns out to matter.
