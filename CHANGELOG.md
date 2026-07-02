# Changelog

Newest release at the top.

## 2.5.0 — 2026-06-24

- **New: cast and trailer in the NFO.** Scraped movies and shows now include the cast (`<actor>` blocks — name, role, billing order, and each actor's photo from TMDB) and a trailer link (`<trailer>` pointing at the best official YouTube trailer, in the form Kodi plays). This is what makes the **Cast** list and **Trailer** button appear in the Kodi remote apps, matching what a tinyMediaManager NFO provides. Cast is capped at the top 30 by billing. Editing an item preserves its cast and trailer. (This goes a little beyond the original "no trailers/cast" scope — added on request.)
  - Cast photos are referenced by their TMDB URL (no extra image files are downloaded), and trailers/cast are written for movies and shows but not individual episodes.
  - **Action needed for existing items:** as with the recent NFO fixes, **Force re-scrape** (or **Re-scrape all** for a source) to add cast/trailer to items scraped by an earlier version, then refresh in Kodi.

## 2.4.2 — 2026-06-24

- **Fixed: posters now show in Kodi for scraped items.** The app was only dropping `poster.jpg`/`fanart.jpg` files next to the video and writing no artwork references inside the NFO. Kodi (and the mobile apps that read its library) takes movie/show poster and fanart from the NFO's `<thumb>`/`<fanart>` tags — the same way tinyMediaManager writes them — so kmm-scraped items had metadata but no poster, while tmm-imported items showed posters fine. The app now writes `<thumb aspect="poster">`, `<fanart>`, and per-season poster tags into the NFO (pointing at the TMDB image, exactly like tmm), in addition to the local `.jpg` files. Editing an item preserves its existing artwork tags.
  - **Action needed for already-scraped items:** as with 2.4.1, items written by an earlier version need their NFO rewritten — **Force re-scrape** them (or **Re-scrape all** for the source in Settings), then refresh in Kodi. tinyMediaManager-imported items were never affected.

## 2.4.1 — 2026-06-24

- **Fixed: NFOs written by the app were invalid XML, so Kodi couldn't read them.** The `default` attribute on the `<rating>` and `<uniqueid>` tags was being written bare (`default`) instead of `default="true"`. A bare, value-less attribute is not valid XML, and Kodi's strict parser rejects the whole file — which is why freshly-scraped or edited items showed no plot and no poster in Kodi even though the app displayed them correctly. (The app's own reader is lenient, so it never noticed.) All NFOs the app writes from now on are valid.
  - **Action needed for already-scraped items:** items written by an earlier version still have the bad NFO on disk. To fix them, **Force re-scrape** the affected items (right-click → Force re-scrape), or use **Re-scrape all** for a source in Settings, which rewrites every item that has a stored TMDB id. Then refresh those items in Kodi (or remove and re-add the source) so it re-reads the corrected files. Items imported from tinyMediaManager were never affected.

## 2.4.0 — 2026-06-24

- **Scanning now removes items deleted from your sources.** Previously a scan only ever *added* — if you deleted a movie folder or a show folder from disk, its entry lingered in the library forever (and could show as a duplicate). Now **Scan movie sources** drops movies whose folder is gone, and the TV scan drops shows whose folder is gone (along with their episodes). The scan summary reports how many were removed.
- **Safe by design.** A source is only reconciled if it read cleanly this scan, and an item is only dropped once its folder is confirmed absent on disk — so an offline or unreachable network share never removes anything. This only ever deletes **library entries**, never your actual files. (Episode-level cleanup within a single show already shipped in 2.3.0; this extends the same idea to whole movies/shows on a full source scan.)

## 2.3.3 — 2026-06-24

- **Stopped attempting the dead imdb.com fallback.** Repeated testing confirmed imdb.com always serves an AWS WAF anti-bot page to the app, so the IMDb-direct text scrape could never succeed — it only produced a confusing error and a leftover dump file in your Temp folder. Now, when **Use IMDb ID** is used for a title TMDB has no record of, the app stops immediately with a plain message — *"…isn't on TMDB. imdb.com can't be read (bot-protected)… use Edit fields… to enter the metadata by hand"* — and writes nothing. No fetch attempt, no Temp dump. The IMDb ID match still works fully whenever TMDB knows the ID (including a documentary/special TMDB files as a movie); only the doomed last-resort branch was removed.

## 2.3.2 — 2026-06-24

Documentation.

- **README brought up to date with v2.** The "Known limitations" section no longer lists things that now exist: multi-episode files are recognised, file renaming is supported, and IMDb-ID matching works through TMDB. It now accurately describes IMDb as an ID lookup resolved via TMDB (with the imdb.com-direct path blocked by AWS WAF), notes that editing is movie/show level only, and documents stacked multi-episode NFOs and the rename action. Added an FAQ entry for the "IMDb lookup failed (AWS WAF)" message. No app changes.

## 2.3.1 — 2026-06-24

- **Honest IMDb-ID help text.** Confirmed again on a second title (tt1260997): imdb.com serves an AWS WAF anti-bot challenge to the app, so the direct imdb.com text scrape can't work — it's only ever reachable for titles TMDB has no record of at all, and for those it's blocked. The IMDb ID field's hint now says this plainly and points to **Edit fields…** as the way to enter metadata by hand for a title no provider has. The IMDb ID match itself still works whenever TMDB knows the ID (including a documentary/special whose ID TMDB lists as a movie).

## 2.3.0 — 2026-06-24

Final slice of SPEC-v2: **rename & cleanup (F2)**. This completes v2.

- **New: rename files to the naming convention.** Right-click a movie → *Rename to "Title (Year)"* renames the folder, the video file, and its matching sidecars (`.nfo`, basename-matched `-poster.jpg`/`-fanart.jpg`). Right-click a show → *Rename episode files* renames every episode's video + sidecars to `Show - SxxExx - Episode Title` (a multi-episode file becomes `Show - S01E02-E03 - Title`). The video itself is renamed too, so Kodi's "match NFO/artwork to video by identical name" keeps working.
- **Safe by construction.** These are pure renames — nothing is ever deleted. If a target name already exists the operation stops (movie) or skips that file (show) and leaves everything exactly as it was; it never overwrites or merges. The library database is updated in step, so no re-scan is needed afterwards. Illegal filename characters are stripped, and generic folder artwork (`poster.jpg`/`fanart.jpg`/`banner.jpg`) is left alone.
- **Cleanup: stale entries are now removed.** Re-scanning a show now prunes episode entries whose file has genuinely been moved or deleted on disk — but only when the show folder reads cleanly (so an unreachable network share can never trigger a mass removal) and the file is confirmed gone. This clears the leftover entries the 2.2.1 fix could only skip.

## 2.2.4 — 2026-06-14

Setup/build diagnostics (no app changes).

- **setup.ps1 now captures the real error when `npm install` fails.** Previously the PowerShell transcript didn't record the npm command's own output, so a failure logged only "npm install failed" with no reason. Each external step (npm install, Electron download, the better-sqlite3 prebuilt fetch, the source-build fallback) now tees its full output to a dedicated log (e.g. `npm-install.log`) and prints the tail on failure.
- **Added a preflight check.** Setup now verifies Node and npm are on PATH before doing anything and prints their versions; if they're missing it says so directly (with the Node LTS install command) instead of failing cryptically. It also confirms it's running in the folder that contains `package.json`.
- This addresses a setup that failed in ~1 second on a fresh machine with no usable error — the most likely cause (Node not installed) is now reported explicitly, and any other cause is captured in the step logs.

## 2.2.3 — 2026-06-14

IMDb-ID matching: handle the real-world cases.

- **Confirmed: imdb.com blocks the direct text scrape.** The page IMDb returns to the app is an Amazon AWS WAF anti-bot challenge (it needs a real browser to run a JavaScript verification), not the title page. A plain fetch can't get past it, so the IMDb-direct fallback isn't usable in practice. The app now recognises that page and says so plainly instead of reporting a format change.
- **But IMDb-ID matching still works when TMDB has the title — even cross-type.** When you enter an IMDb ID on a **show** and TMDB only has that title as a **movie** (common for documentaries/specials filed under TV, e.g. *Empire of Dreams*), the app now uses that TMDB movie record to fill the show's metadata **and artwork**, instead of falling through to the blocked IMDb path. Its episodes are left as-is (there's no TMDB series to match them to). The status bar notes when a movie record was used.
- Net effect: IMDb-ID matching resolves through TMDB whenever TMDB knows the title in any form; the imdb.com fallback remains only for titles TMDB genuinely lacks, where it will report the bot block.

## 2.2.2 — 2026-06-14

Fixes.

- **Review Queue now scrolls.** With a large queue the list ran off the bottom of the window with no scrollbar; it's now independently scrollable like the movie and TV lists.
- **IMDb-ID fallback hardened.** The direct imdb.com scrape now sends fuller browser-style headers (IMDb serves a stripped page to obvious bots), and falls back to the page's Open Graph / title tags when the embedded JSON isn't present. When it still can't extract anything it now saves the exact page it received to a temp file and says whether it looked like a bot/verification page or a genuine format change — so the failure can be diagnosed and the parser updated. Note: a title that TMDB only lists as a *movie* (e.g. a documentary filed under TV) has no TV record for `/find` to return, so it falls through to this imdb.com path by design.

## 2.2.1 — 2026-06-14

Bug fix.

- **Force re-scrape no longer aborts when an episode's file has moved or been deleted on disk.** Previously, if a video file was moved or its folder removed outside the app, the leftover database entry pointed at a path that no longer existed, and writing its NFO failed with an ENOENT error that stopped the whole show's re-scrape. Now that single episode is skipped and counted, the rest of the show is written normally, and the status bar reports how many were skipped (e.g. "3 episodes skipped — file moved/missing on disk"). Stale entries are **not** auto-deleted — a temporarily unreachable network share looks the same as a deleted file, so removing entries on a write error would be unsafe. (Tidying up stale entries is planned for the rename & cleanup feature.)

## 2.2.0 — 2026-06-13

Slice 3 of SPEC-v2: **IMDb ID fallback (F4)**.

- **New: match by IMDb ID.** In **Search & re-match…** there's now an **IMDb ID** field — type a `tt#######` and click **Use IMDb ID** to match a movie or show that the title search can't find.
- **Resolution order:** it first asks TMDB whether it knows that IMDb ID. If so, the item is scraped the normal way — **full metadata and artwork** (and, for shows, episode NFOs). Only if TMDB has no record does it fall back to reading the page on imdb.com.
- **IMDb fallback is text-only:** title, year, plot, rating and genres. **No artwork** is taken from IMDb, and shows pulled this way get the show NFO only — their **episodes stay New** until TMDB later matches them. The status bar tells you which source was used.
- This is the one narrow exception to "TMDB is the only scraper." No new dependency — it uses the built-in fetcher and reads the page's embedded JSON. IMDb occasionally changes that JSON; if a lookup starts returning blanks, the parser needs a small update. Personal/local, low-volume use only.
- Internal: the `shows` table gained an `imdb_id` column (added automatically on first launch); TMDB show matches now record it too.

## 2.1.0 — 2026-06-13

Slice 2 of SPEC-v2: **manual field editing (F3)**.

- **New: edit metadata by hand.** The details pane now has an **Edit fields...** button for movies and shows. You can change title, year, plot, genres, rating and TMDB ID directly — movies also expose premiered date and IMDb ID; shows expose status (Continuing / Ended). Saving rewrites the NFO from your values straight away and marks the item done. Artwork is left untouched (this is metadata-only editing).
- No per-episode editing (out of scope) — editing is at the movie/show level. Re-match still exists; manual editing is just a second way to fix metadata without going back to TMDB.
- Movie NFOs written this way still never contain a `<set>` element.

## 2.0.0 — 2026-06-13

v2 begins. Slice 1 of SPEC-v2: **multi-episode files (F1)**.

- **New: multi-episode files.** A single video containing several episodes (e.g. `S01E01E02`, or a `S01E01-E03` range) is now recognised. Supported forms: consecutive `SxxExxExx` and dashed `SxxExx-Exx` / `SxxExx-xx` (ranges are expanded, so `S01E01-E03` covers E01, E02 and E03). The episode number is read from the **video filename or its containing folder** — whichever gives a clean multi-episode result (covers cases like a `Heroes.Reborn.S01E01E02` folder holding a compact-named file).
- Each episode becomes its own entry in the library, but they all point at the same physical file.
- **One NFO per file, stacked.** A multi-episode file gets a single `.nfo` next to it containing one `<episodedetails>` block per episode — the convention Kodi uses for file-stacking — instead of several separate NFO files.
- **Existing multi-episode files that already have an NFO** import as episode 1 only (the import path is unchanged). To pull in the remaining episodes, run **Force re-scrape** on that show: it now re-walks the show folder to add the missing episodes, then rewrites the stacked NFO covering all of them.
- DB: the one-row-per-file restriction on episodes was removed so multiple episodes can share a file. Existing databases are migrated automatically on first launch (your data is preserved).


## 1.1.1 — 2026-06-13

- **FIX: details pane rendered below the list instead of on the right.** The 1.1.0 layout rework accidentally dropped `display:flex` from the `.split` container, so the list and the details pane stacked vertically. Restored the flex row and pinned the details pane width. Affects both the Movies and TV Shows tabs.

## 1.1.0 — 2026-06-13

- **New: right-click "Re-scan this show"** — re-walks a single show's folder for new episode files, instead of re-scanning the whole TV source. Much faster for adding a few new episodes to one show.
- **New: season offset in re-match** — handles shows whose file season numbers differ from TMDB's (e.g. Money Heist files labelled S04/S05 that are TMDB's 3-season cut, or Disenchantment's parts). In Search & re-match for a show, set "Season offset" (added to your local season to find the TMDB season; e.g. files S04 that are TMDB S01 -> offset -3). Stays within TMDB; no extra provider.
- Header banner enlarged (~2x) for a more prominent wordmark; app layout reworked so content fills the window with a single bottom status bar and no stray scrollbars.
- README rewritten into a full user guide, including step-by-step **how to get a TMDB API token**, the season-offset workflow, folder layout, what gets written, pointing Kodi at the library, limitations, and troubleshooting.

### Note on IMDb scraping (requested, not added)
IMDb has no free API and its episode pages can't be scraped reliably (and doing so breaks IMDb's terms), so a `tt########` provider isn't a sound addition and remains out of scope. The shows this was wanted for (Money Heist, Disenchantment, etc.) are all on TMDB — the real issue was season numbering, which the new **season offset** solves within the existing TMDB-only design.

## 1.0.4 — 2026-06-13

- **FIX: installer build still failed (real cause found).** electron-builder 26 rejected the config at `configuration.win`. The culprit was `publisherName`: in v26 it is no longer a valid `win` key (nor a top-level one) — it moved under `win.signtoolOptions.publisherName`. Placed it there. Verified by running electron-builder 26 directly: config now validates and packaging proceeds (previously it aborted before packaging).
- **build.ps1 now captures electron-builder's real output.** `Start-Transcript` does not record a child process's stderr, which is why the last two failures logged nothing useful past "Running electron-builder". The build step now redirects stderr→stdout and tees to `electron-builder.log`, and prints the last 40 lines on failure, so any future error is actually visible.

## 1.0.3 — 2026-06-13

- **FIX: installer build failed.** `publisherName` was set at the top level of the electron-builder `build` config, which is an invalid key there and aborted the build during schema validation. Moved it to `build.win.publisherName` where it belongs. build.bat now completes.
- Banner replaced with a clean **text-only wordmark** (blue accent bar + "SCAN · SCRAPE · ORGANISE" tagline). The previous vector film-strip art couldn't match the photographic look and the wordmark sat over busy artwork hurting readability; text-only is fully legible at any size and the title is never obscured.

## 1.0.2 — 2026-06-13

- **FIX: header banner rendered wrong + caused a second scrollbar.** The banner SVG was stretching to the full window width (and overflowing vertically), which pushed the page past 100vh. Reworked the layout: `body` is now a fixed-height flex column with internal scrolling only, and the banner has a fixed intrinsic size.
- New banner artwork: a realistic gold **film strip** (perspective, sprocket holes, sheen) in place of the flat blue cells, matching a classic film-reel look.
- **TV Shows list now has column headers** (Title / Year / Seasons / Eps / Status / New / Art), aligned to the tree rows — parity with the Movies table.
- Branding: installer Publisher now reads **Kodi Media Manager** (was "Simon") via `author` + `nsis.publisherName`.

### Known limitation
- **Money Heist (and similarly re-cut shows):** TMDB's main listing for *La Casa de Papel* (id 71446) only exposes **3 seasons** (the original Spanish broadcast); the familiar 5-season Netflix structure exists only as a separate TMDB "episode group", which kmm v1 doesn't query. Files labelled S04/S05 therefore can't match and stay **New**. This is correct behaviour (no wrong data written), not a scrape failure. Workarounds: rename those folders to TMDB's 3-season layout, or leave them unscraped. Episode-group support is a v2 candidate.

## 1.0.1 — 2026-06-13

- Branding: installer, desktop/Start-menu shortcut, app window and taskbar now read **Kodi Media Manager** (electron-builder `productName` + `shortcutName`, window `title`/`icon`). `appId` updated to `com.simon.kodimediamanager` — note this changes the install identity, so this version installs alongside any existing "kmm" install; uninstall the old one from Apps & Features.
- New application icon (film-cell + play, "Icon C"): multi-resolution `build/icon.ico` with 16/24/32/48/64/128/256 px frames so it renders crisply in the taskbar, file lists, Start menu and title bar. Runtime window icon shipped in the app bundle.
- New in-app header banner (film contact-sheet + wordmark, "Banner 2") above the tab bar, as inline SVG (crisp at any DPI).

## 1.0.0 — 2026-06-13

First complete release. NSIS installer builds (build.bat) and installs cleanly. All five build slices done and live-validated on the real library (~274 movies, 307 shows / 18k+ episodes): movie + TV scan/scrape, review queue, re-match, loose-file foldering, multi-select and per-source re-scrape.

- **FIX: EPERM on force re-scrape / re-match over existing scene NFOs.** Writing an NFO or artwork file failed with `EPERM: operation not permitted` when the existing target (a scene-release `.nfo`, or tmm-era artwork) carried the Windows read-only or hidden attribute — a plain truncating open is refused. All NFO and artwork writes now go through a helper that, on EPERM/EACCES, clears the attributes and removes the blocking file before writing. Affected House of Lies, Person of Interest, Comic Book Men, Brooklyn Nine-Nine and any other show/movie with read-only release NFOs next to the media.

## 0.7.0 — 2026-06-13 (slice 5)

- **Loose-file foldering (movies).** Videos sitting directly in a movie source root (no containing folder) are detected on scan; the new "Loose files" button opens a dialog listing each one with a proposed folder name derived from the filename (title + year, scene-junk stripped, Windows-illegal characters removed). Edit the name if needed, then "Create & move" (or "Create & move all"). The file is moved into the new folder; closing the dialog re-scans so the item appears as a normal New movie. Collisions (folder already exists) and missing files are reported per-row and leave the file untouched.
- Final README pass; v1 is now feature-complete across all five build slices.

## 0.6.2 — 2026-06-13

- **FIX: TV tree collapse.** Shows and seasons would only expand, never collapse. Cause: `.hidden` was defined only for `.details`, `.modal`, and `#ctx-menu` — there was no global rule — so toggling `hidden` on `.tree-seasons`/`.tree-episodes` flipped the arrow but didn't actually hide anything. Added a global `.hidden { display: none }`.

## 0.6.1 — 2026-06-13

- **Compact scene episode numbering** ("24.401.hdtv-lol" = S4E01, validated against the `Season 4` folder it sits in). Only used as a fallback after SxxExx/1x01 fail, and ONLY when the leading digits equal the known folder season — so resolutions like 480p/1080p can never be misread as episodes. Cut a large share of the "Cannot parse SxxExx" errors on real libraries.
- **Scrollable scan-errors modal** replaces the native alert (6k+ error lines were unusable). Groups errors by class — non-XML scene `.nfo`s (item becomes New, fixed by "Scrape new items") vs. unreadable filenames (episode skipped) — with a "Copy all" button for the full list.
- **TV tree column alignment** fixed: year / seasons-episodes / status / status-badge / artwork-badge are now fixed-width, right-aligned slots, so the variable-width "N new ep" badge no longer shifts the P/F/B column out of line.
- Title bar / taskbar now reads **Kodi Media Manager** (was "kmm").

Note on the live 0.6.0 scan (307 shows imported, 18,136 episodes, 6,571 errors): most of those errors were scene-release `.nfo` files next to episodes (reported as "Unusable episode NFO") — those episodes were still imported as New and will be filled in by "Scrape new items"; they were noise, not lost data. The genuine misses were compact-numbered filenames, now handled.

## 0.6.0 — 2026-06-13

- **FIX: TV scan no longer freezes the app.** Root cause: one `existsSync` network round trip PER EPISODE (~26k over SMB) plus duplicate `readdir`s, all synchronous in the main process — minutes of "Not Responding" with no feedback. Scanner rewritten: one `readdir` per directory (NFO + artwork presence checked against that listing), async gather phase with 8 shows scanned in parallel, live progress in the status bar, DB commit still one transaction. 1,000 shows / 20k episodes scan in <1s on local disk in container tests; network drives are bound by readdir latency only.
- Global status bar at the bottom of the window — all scan/scrape/re-match progress and results report there (was per-tab status text squeezed next to the filter box).
- Filter boxes right-aligned in the toolbar.
- **Re-scrape all** button per source (Settings): force re-scrapes every item under that source that has a stored TMDB id (movies: NFO + artwork; shows: tvshow.nfo + ALL episode NFOs + artwork). Items without a TMDB id are skipped and counted.
- Multi-select: Ctrl+click toggles (movies + shows), Shift+click selects a range (movies). Right-click with 2+ selected adds "Force re-scrape N selected" to the context menu.

## 0.5.0 — 2026-06-13 (slice 4)

- TV pipeline end-to-end: scan → flag New → scrape TMDB → tvshow.nfo + per-episode NFOs + artwork → Show → Season → Episode tree.
- tvScanner: `Source\Show\Season N\episodes` (+ `Specials` = season 0, + one optional extra nesting level per episode, largest video wins there); SxxExx/1x01 parsed from filename with nested-folder-name fallback; stragglers directly in the show root accepted if they parse; unparseable files reported, not silently dropped. Existing tvshow.nfo / episode NFOs imported as done (tmm migration); whole scan runs in one SQLite transaction (52k-episode scale). Rescan is idempotent AND picks up new episode files under known shows (weekly additions).
- tvNfo: per real-sample rules — bare `<id>` (TVDB in tmm tvshow.nfo) is never trusted; IDs from `<uniqueid type=>`/`<tmdbid>` only; `<status>` ↔ status_text; episode root `<episodedetails>`. Writes modern uniqueid/ratings blocks (tmdb + imdb + tvdb ids).
- tvScraper: `/search/tv` with `first_air_date_year`, same ±1-year auto-match rule as movies; ambiguous/no-result shows go to the review queue (`item_type='show'`). Episodes fetched once per needed season via `/tv/{id}/season/{n}`; only `status=new` episodes get NFOs (re-match/force rewrites all). Episodes absent on TMDB (e.g. local Specials) stay New and are counted — no episode-level review queue (deliberate: 52k-item queue would be a chore).
- Artwork: show poster.jpg + fanart.jpg + seasonNN-poster.jpg (`season-specials-poster.jpg` for S0, Kodi convention). **banner.jpg is never downloaded — TMDB has no banner asset type** (TVDB/fanart.tv concept); existing tmm-era banners are preserved, badged, and protected from artwork cleanup, as are season posters.
- TV tab UI: lazily-expanded Show → Season → Episode tree (title, year, S/E counts, Continuing/Ended, NEW/REVIEW/new-ep/P/F/B badges), show filter, details pane with artwork, context menu (Open folder / Force re-scrape incl. all episode NFOs / Search & re-match).
- Review queue is now type-aware (movies + shows, with a type tag per item); the TMDB match modal serves both.

## 0.4.2 — 2026-06-12

- Context menu: "Open folder" — opens the movie's source folder in Explorer (UNC paths included).

## 0.4.1 — 2026-06-12

- Token save now validates live against TMDB and reports "validated ✓" or the rejection reason, with a hint that the long eyJ… read-access token is required (not the short API key). Was previously stored blind — a missing/wrong token only surfaced as scrape failures.

## 0.4.0 — 2026-06-12

- Right-click context menu on movie rows: **Force re-scrape** (rewrites NFO from the stored TMDB id, deletes non-standard artwork variants like movieset-poster.jpg, downloads fresh standard poster.jpg + fanart.jpg — with confirmation) and **Search & re-match…**. Items without a stored TMDB id fall through to the search dialog automatically.
- Re-match (details pane, review queue manual search, context menu) now also normalizes artwork to standard filenames.
- Scan fix: unusable .nfo files (scene-release text files, corrupt XML) no longer block the movie — it's inserted as New and scrapeable; warning still reported. Fixes the 3 permanently-invisible movies (Aircraft Carrier, Bee-Kept, Nitro Snowboards).

## 0.3.2 — 2026-06-12

- Artwork detection now wildcard suffix match: any `*poster.jpg` / `*fanart.jpg` (case-insensitive) counts — covers tmm naming like `movieset-poster.jpg`. Details pane loads the actual found file.
- UNC path support for artwork thumbnails (`\\server\share\...` sources).
- Rescan refreshes artwork badges on already-known items (fixes stale flags from earlier scans).
- Scan errors now shown in a dialog, not just console; F12 toggles devtools (menu bar removal had disabled it).

## 0.3.1 — 2026-06-12

- setup.bat/build.bat now always pause (output no longer vanishes on double-click success) and setup.ps1/build.ps1 write full transcripts to setup.log/build.log.

## 0.3.0 — 2026-06-12 (slice 3)

- Review queue UI: items with ambiguous/no TMDB match show guessed title/year, folder, and candidate buttons — one click applies the match (NFO + artwork written). "Search manually" for items with no candidates.
- "Search & re-match" on any movie in the details pane: manual TMDB search dialog, picking a result overwrites NFO + artwork (fixes wrong auto-matches).
- Install fix: setup.bat/setup.ps1 — installs without Python/VS Build Tools by fetching better-sqlite3's prebuilt Electron-ABI binary (root cause of npm install failure: no Node-24 prebuild exists + broken Python 3.8 leftovers killed the source-build fallback). Compile path retained as fallback with clear remediation.
- build.bat/build.ps1: standalone NSIS installer via electron-builder (new devDependency; npmRebuild disabled so packaging never needs the toolchain either).

## 0.2.0 — 2026-06-12 (slice 2)

- Movie pipeline end-to-end: scan → flag New → scrape TMDB → NFO + artwork → table.
- nameParser: title/year from scene-style folder names (bracketed years, junk-token stripping, leading-number titles like "2001..."); SxxExx + 1x01 episode parsing ready for slice 4.
- movieScanner: largest video file wins (recursive within movie folder), skips ~uTorrentPartFile*/sample.*/non-whitelist, existing NFO imported as done (tmm migration), loose root files counted, idempotent rescan.
- movieNfo: writes Kodi NFO with modern uniqueid/ratings blocks — never writes <set>. Tolerant parser: tmm legacy fields (tmdbid/id/rating), duplicate genre tags, trailing URL lines, embedded streamdetails.
- TMDB client: Bearer token, 250ms throttle, year-weighted search with no-year retry, 401 aborts run early.
- Match rule: folder year + top result within ±1 year → auto-match; else status review (queue UI = slice 3).
- Artwork: poster.jpg + fanart.jpg, skips existing files unless re-match.
- Movies UI: table with Title/Year/badges (NEW, REVIEW, NFO, P, F), text filter, details pane with artwork thumbs, scrape progress in status line.

## 0.1.0 — 2026-06-12 (slice 1)

- Electron shell: window, tab navigation (Movies / TV Shows / Review Queue / Settings).
- SQLite database (better-sqlite3) with full v1 schema: sources, movies, shows, episodes, review_queue. WAL mode, FK cascade on source removal.
- Sources UI: add/remove multiple movie and TV source folders (native folder picker).
- TMDB token entry in Settings; stored in `%APPDATA%\kmm\config.json` (never in repo). Token never displayed back.
- Scan/scrape buttons present but disabled (slices 2–4).
