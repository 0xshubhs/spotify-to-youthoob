# spotify-to-youthoob

Find the YouTube equivalents of the tracks in a public **Spotify playlist or album**.
Paste a Spotify URL, and the app scrapes the track list, searches YouTube for each
song, and shows the best-matching video as an embedded player.

> Note: despite the name and an internal `createYouTubePlaylist()` method, the app
> does **not** create a real playlist in your YouTube account. It returns a list of
> matched videos and embeds them in the page. See "Status / what's incomplete" below.

## How it works

There are two parts:

- **`backend/`** — an Express API. On a `POST /api/convert` it uses **Puppeteer**
  (headless Chrome) to open the Spotify *embed* page for the given playlist/album,
  reads the rendered track list, then runs a YouTube search for each track and parses
  the first result with **Cheerio**. It returns the matched videos as JSON.
- **`client/`** — a **Next.js** (App Router) + Tailwind CSS single-page UI. It takes a
  Spotify URL, calls the backend, and renders the returned videos as YouTube iframes.

Data flow: `Spotify URL → backend scrapes tracks → YouTube search per track → JSON → client embeds videos`.

### Important: no API keys required

This project does **not** use the official Spotify or YouTube Data APIs, and needs **no
OAuth, client secret, or API key**. Everything is done by scraping public web pages with
a headless browser. The only configuration is the backend URL the client points at.

Because it relies on the current HTML/structure of Spotify's embed pages and YouTube's
search results, scraping can break whenever those sites change their markup.

## Stack

- Backend: Node.js, Express, Puppeteer, Cheerio, CORS
- Frontend: Next.js 14 (App Router), React 18, Tailwind CSS
- Package manager: npm

## Requirements

- Node.js 18+ (developed/verified on Node 24)
- A Chrome/Chromium binary for Puppeteer. A normal `npm install` in `backend/`
  downloads a compatible Chromium automatically; if you skipped that, run
  `npx puppeteer browsers install chrome` inside `backend/`.

## Running locally

Two terminals.

**1. Backend** (default port `3001`):

```bash
cd backend
npm install
npm run dev      # nodemon src/index.js
```

The server launches a headless browser on startup and listens on `http://localhost:3001`.

**2. Frontend** (default port `3000`):

```bash
cd client
npm install
cp .env.example .env   # optional; only needed if the backend is not on localhost:3001
npm run dev
```

Open http://localhost:3000, paste a **public** Spotify playlist or album URL, e.g.
`https://open.spotify.com/playlist/<id>` or `https://open.spotify.com/album/<id>`,
and click **Convert**.

### Configuration

- `client/.env` — `NEXT_PUBLIC_API_URL` sets the backend base URL the client calls
  (defaults to `http://localhost:3001`). See `client/.env.example`.
- `backend` — `PORT` env var overrides the backend port (default `3001`).

## Status / what's incomplete

This is an early, abandoned prototype. What exists works end-to-end as a *search-and-embed*
tool, but note:

- **No real YouTube playlist creation.** `createYouTubePlaylist()` only collects search
  results; nothing is written to a YouTube account. Actually creating a playlist would
  require the YouTube Data API with OAuth — not implemented.
- **Scraping is fragile.** Track parsing depends on the innerText layout of Spotify embed
  pages, and video matching takes the *first* YouTube search result with no relevance
  scoring, so mismatches are possible.
- **Only one endpoint.** The backend exposes a single `POST /api/convert`; there are no
  tests (`npm test` in `backend` is a placeholder that exits 1) and no persistence.
- **Single shared browser/page.** The scraper uses one shared Puppeteer page for
  playlist/album navigation, so concurrent requests can interfere with each other.
- **Startup coupling.** The backend launches the browser during `app.listen`; if Chromium
  is missing the process crashes on boot instead of failing gracefully per-request.

None of the above is fixed here — they are documented for whoever picks this up.
