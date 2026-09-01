# TED Talks — unofficial video RSS feed

Generates a video podcast RSS feed from https://www.ted.com/talks?sort=newest
(TED's own feed at ted.com/feeds/talks.rss is audio-only, so this scrapes each
talk page for TED's own hosted .mp4 URLs and packages them into a proper
video-podcast RSS file that Downcast — or any podcast app — can subscribe to).

## How it works

- `ted_rss_scraper.py` fetches the newest-talks listing, then each talk's
  page, and pulls the direct mp4 URL + title/description/thumbnail/duration
  out of the page's embedded data.
- It writes the result to `docs/feed.xml`.
- A GitHub Actions workflow (`.github/workflows/update-feed.yml`) runs this
  automatically every 6 hours and commits the updated file.
- GitHub Pages serves the `docs/` folder, so `docs/feed.xml` gets a stable
  public URL you can paste straight into Downcast.

No server to run, no computer that has to stay on — GitHub does the
scraping and hosting for you, for free, on a schedule.

## One-time setup (~5 minutes)

1. **Create a new GitHub repo** (public or private both work; if private,
   Downcast will need the feed URL with a GitHub Pages token — public is
   simpler). Name it anything, e.g. `ted-video-feed`.
2. **Upload these files** to the repo, keeping the folder structure:
   - `ted_rss_scraper.py`
   - `.github/workflows/update-feed.yml`
   - `docs/` (can be empty initially — the Action will populate it)
3. **Enable GitHub Pages**: repo → Settings → Pages → "Build and deployment"
   → Source: *Deploy from a branch* → Branch: `main`, folder: `/docs` → Save.
4. **Run the workflow once manually**: repo → Actions tab → "Update TED video
   RSS feed" → Run workflow. Wait ~1–2 minutes for it to finish (it needs to
   scrape ~40 talk pages).
5. Your feed will now be live at:
   ```
   https://<your-github-username>.github.io/<repo-name>/feed.xml
   ```
   Open that URL in a browser first to confirm it loads and shows `<item>`
   entries with `<enclosure url="...mp4" ...>` tags.

## Add it to Downcast on iPhone

1. Open Downcast.
2. Tap **+** → **Add via URL** (not the built-in search/directory).
3. Paste your `https://<username>.github.io/<repo>/feed.xml` URL.
4. Downcast will pull in the talks with video enclosures, thumbnails, and
   descriptions. New talks appear automatically after each scheduled run.

## Tuning

- `--max-items` (default 40) — how many talks to keep in the feed.
- `--pages` (default 3) — how many listing pages to crawl to find that many
  new talks.
- Cron schedule in the workflow file — change `0 */6 * * *` to run more or
  less often (TED usually posts one talk a day, so every 6–12 hours is
  plenty).

## Notes / caveats

- This depends on TED's page markup (`__NEXT_DATA__` JSON) staying roughly
  the same. If TED redesigns their site, the script may need small updates —
  it's written to skip a talk it can't parse rather than crash the whole run,
  so a partial break degrades gracefully.
- This is for personal use. Please respect TED's Terms of Use — the script
  only links to mp4 files TED itself already serves publicly; it doesn't
  re-host or redistribute anything.
- Video files are TED's full HD/SD encodes, so mobile data usage can add up;
  Downcast's Wi-Fi-only download setting is worth using if you're not on
  unlimited data.
