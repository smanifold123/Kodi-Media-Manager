# Kodi Media Manager

A simplified [tinyMediaManager](https://www.tinymediamanager.org/)-style tool for [Kodi](https://kodi.tv/). It scans your movie and TV folders, looks up each title on [The Movie Database (TMDB)](https://www.themoviedb.org/), and writes Kodi-compatible **NFO files** and **artwork** (posters, fanart, season posters) right next to your media.

Kodi then reads that local metadata with its **"Local information only"** scraper. The app never talks to Kodi directly — it just prepares the files on disk, so your library is portable and works offline.

Windows desktop app, built on Electron.

---

## Contents
- [What it does](#what-it-does)
- [Getting a TMDB API token](#getting-a-tmdb-api-token)
- [First run](#first-run)
- [How scanning works](#how-scanning-works)
- [How scraping works](#how-scraping-works)
- [Fixing wrong or missing matches](#fixing-wrong-or-missing-matches)
- [Re-cut shows and the season offset](#re-cut-shows-and-the-season-offset)
- [Loose movie files](#loose-movie-files)
- [Folder layout it expects](#folder-layout-it-expects)
- [What gets written](#what-gets-written)
- [Pointing Kodi at your library](#pointing-kodi-at-your-library)
- [Known limitations](#known-limitations)
- [Building from source](#building-from-source)
- [FAQ / troubleshooting](#faq--troubleshooting)

---

## What it does

- **Scan** your movie and TV source folders to find what you have.
- **Import** any existing NFO files (e.g. from a previous tinyMediaManager library) without overwriting them — they're marked done and left alone.
- **Scrape** new items from TMDB, auto-matching by title and year, and write Kodi NFOs + artwork.
- Send anything ambiguous to a **Review Queue** so you can pick the right match by hand.
- Let you **re-match** or **re-scrape** any item, **re-scan** a single show for new episodes, **edit metadata by hand**, **rename** files to the naming convention, and tidy **loose** movie files into folders.

It does **not** transcode anything, read media details (codecs/resolution), or modify Kodi — it only writes metadata files. Renaming your media is supported but never automatic: it only happens when you explicitly trigger it on an item, and it never deletes anything.

---

## Getting a TMDB API token

The app needs a free TMDB **API Read Access Token** to look up titles. This is a one-time setup.

1. **Create a TMDB account** at <https://www.themoviedb.org/signup> (free).
2. Verify your email and sign in.
3. Go to **Settings -> API**, directly at <https://www.themoviedb.org/settings/api>.
4. Request an API key if you don't have one. Choose **"Developer"**, accept the terms, and fill in the short form. For "Type of Use" you can pick Personal; the application fields can be basic (name it anything, e.g. "Kodi Media Manager", URL can be `http://localhost`). Approval is instant.
5. On the API settings page you'll now see two credentials:
   - **API Key (v3 auth)** — a short 32-character string.
   - **API Read Access Token (v4 auth)** — a long token starting with `eyJ...`.
6. **Copy the API Read Access Token** (the long `eyJ...` one). That's what this app uses.

> **Which one?** Use the **Read Access Token** (the long `eyJ...` value), not the short v3 API key.

7. In the app, open **Settings**, paste the token into the TMDB token field, and save. The app validates it against TMDB immediately and shows a check mark if it works.

**Your token is stored locally** in a config file on your machine and is never displayed back, shared, or committed anywhere. Treat it like a password. If you ever need to revoke it, you can regenerate it from the same TMDB API settings page.

---

## First run

1. Install and launch **Kodi Media Manager**.
2. **Settings ->** paste your TMDB Read Access Token, save (look for the check mark).
3. **Settings ->** add your **movie** source folder(s) and **TV** source folder(s). You can add several of each; local drives and network/UNC paths both work.
4. Go to the **Movies** or **TV Shows** tab and click **Scan sources**.
5. Click **Scrape new items** to fetch metadata for everything new.
6. Resolve anything in the **Review Queue**.

---

## How scanning works

Scanning is **manual** — click *Scan sources* on the Movies or TV tab when you want it. It finds your media and records it; it does not contact TMDB.

- **Existing NFO = done.** If an item already has an NFO (e.g. migrated from tinyMediaManager), it's parsed, imported, marked done, and never re-scraped or overwritten unless you explicitly re-match it.
- **No NFO = New.** Items without metadata are flagged **New** (an orange badge) and wait for scraping.
- Scanning is **idempotent** and incremental — re-scanning only picks up what's changed. For TV, a re-scan also finds **new episode files** added to shows you already have (handy for weekly releases).
- **Deletions are reconciled.** If you delete a movie folder or a show folder from a source, the next scan of that source removes it from the library (a removed show takes its episodes with it). This only happens for a source that read cleanly — an offline or unreachable network share never removes anything — and it only ever deletes **library entries**, never your files. The scan summary reports how many were removed.
- Junk is skipped: partial downloads, `sample.*` files, and anything outside the video extension whitelist (`.mkv .mp4 .avi .m4v .mov .wmv .ts`).
- Any files it can't make sense of are listed in a **scan-errors** dialog, grouped so you can tell real problems (unreadable filenames) from harmless noise (scene-release `.nfo` text files, which just become New and get fixed on the next scrape).

---

## How scraping works

Scraping is a separate step (**Scrape new items**) so it never runs unexpectedly.

- Each New item's folder/filename is parsed into a **title + year** guess and searched on TMDB.
- A confident match (title found, year within a year) is **taken automatically** and its NFO + artwork written.
- Anything ambiguous or not found goes to the **Review Queue** instead of guessing wrong.
- For TV, episodes are matched by the `SxxExx` (or `1x01`, or compact `24.401`-style) numbers in the filename.
- **Multi-episode files** — a single video holding more than one episode (`S01E01E02`, or a range like `S01E01-E03`, read from the filename *or* its folder) is recognised: each episode is fetched from TMDB and written into **one stacked `.nfo`** next to the video (the Kodi file-stacking convention).

---

## Fixing wrong or missing matches

Right-click any movie or show:

- **Open folder** — jump to it in Explorer.
- **Re-scan this show** *(TV)* — re-walk just that show's folder for new episode files, without re-scanning the whole source.
- **Force re-scrape** — rewrite the NFO and standard artwork from the stored TMDB ID (for TV, rewrites every episode NFO too).
- **Search & re-match...** — manually search TMDB and pick the correct result. This is how you fix a wrong auto-match. Selecting a result rewrites the NFO and artwork.
- **Rename to convention** — bring files in line with the scrape naming convention. Movies become `Title (Year)` (folder + video + sidecars); a show's episodes become `Show - SxxExx - Episode Title` (multi-episode files use a range like `S01E02-E03`). The video file is renamed too, so Kodi keeps matching its NFO/artwork. Pure renames — nothing is ever deleted, and if a target name already exists it's skipped and left as-is.

The details pane also has an **Edit fields...** button (movies and shows). It lets you change the metadata directly — title, year, plot, genres, rating, IDs, and (movies) premiered date or (shows) status. Saving rewrites the NFO immediately and marks the item done; artwork is left as-is. This is metadata-only editing — there's no per-episode editing.

**Match by IMDb ID.** The Search & re-match dialog has an **IMDb ID** field. Enter a `tt#######` and click **Use IMDb ID** to match an item the title search can't find. It asks TMDB whether it knows that IMDb ID and, if so, scrapes it normally with **full metadata and artwork** — this also covers a show whose ID TMDB lists as a *movie* (common for documentaries/specials filed under TV), where the movie record fills the show's metadata and artwork. If TMDB has no record of the title at all, the ID can't be resolved: imdb.com fronts its pages with an anti-bot challenge (AWS WAF) that can't be read, so the app stops with a clear message rather than attempting a doomed fetch. For a title no provider has, use **Edit fields...** to enter the metadata by hand.

You can also **Ctrl+click** (and **Shift+click** for movies) to select several items and force re-scrape them together, or use **Re-scrape all** on a source in Settings.

---

## Re-cut shows and the season offset

Some shows are split or renumbered differently on TMDB than in your files. The classic case is **Money Heist**: TMDB lists it as **3 seasons** (the original Spanish broadcast), but many file sets label the parts as **S04/S05**. Disenchantment is similar (3 seasons = 5 parts). When the numbers don't line up, those episodes can't match and stay New.

To fix this, use **Search & re-match...** on the show and set the **Season offset** field:

- The offset is added to your local season number to find the TMDB season.
- Example: your files are **S04** but they're TMDB **S01** -> set offset **-3** (4 + -3 = 1).
- Leave it at **0** when your seasons already match TMDB (the normal case).

The episode *number within the season* still has to match. If a show is renumbered in a way a single offset can't express, those episodes will remain New.

---

## Loose movie files

If a video sits **directly in a movie source root** with no folder of its own, scanning flags it. The **Loose files** button opens a dialog that proposes a folder name from the filename (title + year, scene tags stripped), lets you edit it, and then creates the folder and moves the file in. After that it scans as a normal movie. Collisions and missing files are reported and never overwrite anything.

---

## Folder layout it expects

**Movies** — one folder per movie; the largest video in the folder is taken as the movie:
```
Movies\
  The Matrix (1999)\
    The.Matrix.1999.1080p.mkv
```

**TV** — `Source\Show\Season N\episodes`. A `Specials` folder is treated as season 0, and episodes may sit one extra folder deeper:
```
TV\
  Breaking Bad\
    Season 1\
      Breaking.Bad.S01E01.mkv
    Specials\
      Breaking.Bad.S00E01.mkv
```

---

## What gets written

**Movies** (next to the video):
- `<moviename>.nfo`
- `poster.jpg`, `fanart.jpg`

**TV** (in the show folder):
- `tvshow.nfo`
- `poster.jpg`, `fanart.jpg`
- `seasonNN-poster.jpg` per season (`season-specials-poster.jpg` for specials)
- one `.nfo` per episode **file** (a multi-episode file gets a single stacked NFO containing one block per episode)

> **Note on banners:** Kodi/TVDB-style `banner.jpg` is **not** downloaded — TMDB has no banner artwork (it's a TheTVDB/fanart.tv concept). Existing banners you already have are kept and shown with a **B** badge; the app just never creates or deletes them.

Movies never get a `<set>`/collection tag written — collection grouping is intentionally not used.

The NFOs also carry the poster and fanart as `<thumb aspect="poster">`/`<fanart>` tags (TV adds per-season poster tags), pointing at the TMDB image — the same way tinyMediaManager does. This is what makes the poster show in Kodi and the mobile apps; the local `.jpg` files are written too, for portability and other tools.

---

## Pointing Kodi at your library

1. In Kodi, add your media sources as usual.
2. Set the content scraper for those sources to **"Local information only"** (Movies and TV Shows both have this option).
3. Refresh the library. Kodi reads the metadata and artwork from the NFO files the app wrote. The poster/fanart tags point at the TMDB image, so Kodi caches the artwork over the internet on first read (the same as a tinyMediaManager library); the local `poster.jpg`/`fanart.jpg` files are there as a fallback.

---

## Known limitations

- **TMDB is the scraper.** IMDb is supported only as an **ID lookup through TMDB** (enter a `tt#######` and TMDB resolves it, including titles TMDB files as movies). imdb.com is never read directly — its pages are bot-protected (AWS WAF) and there's no free IMDb API wired in — so a title TMDB has no record of must be filled in with **Edit fields...**. TheTVDB and other providers aren't used.
- **Episode groups** (e.g. the 5-season Netflix cut of Money Heist that only exists as a TMDB "episode group") are not queried. Use the season-offset re-match instead.
- **Editing is movie/show level only** — there's no per-episode metadata editing.
- No codec/quality detection, no subtitles/trailers/cast images/episode thumbnails, no scheduled/automatic scanning, and no movie sets/collections.

Renaming files to the naming convention and recognising multi-episode files (e.g. `S01E01E02`) **are** supported — see *Fixing wrong or missing matches* and *How scraping works* above.

---

## Building from source

Requirements: Windows, Node.js + npm.

```bat
setup.bat   :: installs deps, fetches the Electron-ABI prebuilt better-sqlite3 binary
npm start   :: run the app
build.bat   :: produce a standalone NSIS installer in dist\
```

> Use `setup.bat`, not a plain `npm install` — the SQLite native module needs the Electron-specific prebuilt binary, which `setup.bat` fetches. The installer build (`build.bat`) writes full output to `electron-builder.log` if anything goes wrong.

The app is not code-signed, so Windows SmartScreen may warn the first time you run the installer.

---

## FAQ / troubleshooting

**"Token invalid" when I save it.** Make sure you copied the **API Read Access Token** (the long `eyJ...` value), not the short v3 API key. Re-copy from <https://www.themoviedb.org/settings/api>.

**A show won't scrape / all episodes stay New.** Usually a season-number mismatch — see [Re-cut shows and the season offset](#re-cut-shows-and-the-season-offset). Otherwise use **Search & re-match...** to confirm it's pointed at the right TMDB entry.

**Wrong match on a common title** (e.g. "Power", "Proof"). Right-click -> **Search & re-match...** and pick the correct result; the NFO and artwork are rewritten.

**Lots of scan errors.** Open the scan-errors dialog. "Non-XML NFO" lines are harmless — those are scene-release `.nfo` text files; the episode is still imported as New and gets proper metadata when you scrape. "Cannot parse" lines are filenames with no readable episode number — those are the ones worth renaming.

**My artwork shows a B badge but no poster.** That B is an existing banner the app preserved. Posters/fanart are the P/F badges. The app writes `poster.jpg`/`fanart.jpg`; if those are missing, scrape or re-scrape the item.

**Kodi shows no poster (or no plot) for a scraped item, but the app shows it fine.** Two fixes landed here: 2.4.1 corrected a malformed NFO attribute that made Kodi reject the file (no plot), and 2.4.2 added the `<thumb>`/`<fanart>` art tags Kodi needs to display the poster. Update to 2.4.2+, **Force re-scrape** the affected items (or **Re-scrape all** for the source in Settings) to rewrite the NFOs, then refresh them in Kodi. tinyMediaManager-imported items were never affected.
