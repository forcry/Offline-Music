# NIGHTWAVE

An offline-first music player for your own audio files. Installable PWA (Android Chrome), built with Vite + React + TypeScript.
Successor to the "Offline" prototype; nothing from that code is edited in place.

## Commands
    npm install
    npm run dev          # local dev server
    npm run typecheck
    npm test             # unit tests (Vitest + fake-indexeddb)
    npm run build        # typecheck + production build into dist/
    npm run preview      # serve dist/ locally

Install on a phone: host `dist/` on any HTTPS static host, open it in Chrome, then Menu > Install app.
Import music with the + button (Android's file picker lists Google Drive; Drive files download on pick, so use Wi-Fi).

## What changed from the prototype
| Prototype problem | Here |
|---|---|
| `icons/` missing, so the service worker never installed | `scripts/make-icons.py` generates `public/icons/` (192, 512, and a padded maskable icon). The build precaches them. |
| Cache name never changed, so installed copies never updated | `vite-plugin-pwa` (Workbox) generates a content-hashed precache manifest; every changed build yields a new `sw.js`. `autoUpdate` + `cleanupOutdatedCaches` |
| Blob and metadata in one IndexedDB record | Separate `songs` (metadata), `blobs` (audio) and `art` stores. Listing the library never reads audio. Add/delete are single transactions across stores |
| Every row stored all song IDs in `data-ids` | Rows are React components; the play list is a closure. No ID lists in the DOM |
| Also fixed | `sort()` mutating the main array; `prompt`/`confirm` replaced with in-app dialogs; nested buttons; UTF-16 BOM and ID3v2.2 handling; out-of-order async loads on fast skipping |

## Layout
- `src/db.ts` storage (idb) · `src/id3.ts` tag parser · `src/import.ts` import pipeline
- `src/queue.ts` pure queue logic · `src/player.ts` audio + Media Session · `src/art.ts` cover URL cache
- `src/services.ts` singletons · `src/library.ts` React data hook · `src/App.tsx`, `src/ui.tsx` UI
- `e2e/` real-browser check: `e2e/make-fixtures.sh`, then serve `dist/` on :8099 and run `CHROME=/path/to/chrome python3 e2e/run.py` (needs Playwright for Python)

## Known limits
- Data lives in IndexedDB per browser profile; clearing site data deletes the library. The app requests persistent storage.
- The old prototype's database (`offline-player`) is not migrated. Re-import your files.
- Lock-screen controls rely on the Media Session API and were not testable in headless Chrome.
- Only import audio you have the right to use.
