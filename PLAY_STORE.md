# Publishing Pulse Counter to Google Play

The Android app is a Trusted Web Activity (TWA): a thin wrapper that opens the hosted PWA full screen. It is built with PWABuilder and hosted on Cloudflare Pages, which serves `/.well-known/assetlinks.json` at the domain root (GitHub Pages project sites cannot).

Placeholders used below. Change them consistently if you pick different values:

| Value | Placeholder |
|---|---|
| Site URL | `https://meditation-pulse.pages.dev` |
| Android package ID | `com.greiye.pulsecounter` |
| Contact email | `REPLACE_WITH_CONTACT_EMAIL` in `privacy-policy.html` |

The package ID is permanent once uploaded to Play. The host is baked into the app too: if you later move to a custom domain, rebuild the package with the new URL and serve `assetlinks.json` there.

## 1. Deploy on Cloudflare Pages

1. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** → **Connect to Git** → pick `greiye/meditation-pulse`.
2. Project name `meditation-pulse`, production branch `main`.
3. Build settings: framework preset **None**, build command **empty**, build output directory **`/`**.
4. Deploy. Every push to `main` redeploys; other branches get preview URLs.
5. Check:
   - `https://meditation-pulse.pages.dev/` loads and installs.
   - `curl -i https://meditation-pulse.pages.dev/.well-known/assetlinks.json` returns `200` with `content-type: application/json`.
   - `_headers` is applied (`curl -I .../sw.js` shows `cache-control: no-cache`).
6. Fill in the contact email in `privacy-policy.html`, then commit.

The GitHub Pages workflow (`.github/workflows/static.yml`) can stay or be deleted; Cloudflare does not use it.

## 2. Build the Android package with PWABuilder

1. Go to https://www.pwabuilder.com, enter the Cloudflare URL, and review the report. The manifest has `id`, `start_url`, `scope`, `display: standalone`, 192/512 icons, maskable icons and screenshots, so it should pass.
2. **Package for stores** → **Android** → **Google Play**. Set:
   - Package ID: `com.greiye.pulsecounter`
   - App name `Pulse Counter`, launcher name `Pulse`
   - Version code `1`, version name `1.0.0`
   - Signing key: **Create new**
3. Download the zip. It contains the `.aab` (upload this), an `.apk` (side-load test), `signing.keystore`, `signing-key-info.txt` and an `assetlinks.json`.
4. **Back up `signing.keystore` and `signing-key-info.txt` somewhere safe and outside this repo.** You need them for every future update. Never commit them.

## 3. Update the SHA-256 fingerprints

Android only hides the browser URL bar if `/.well-known/assetlinks.json` lists the fingerprint of the key that signed the installed app. With Play App Signing there are two keys, so list both:

1. **Upload key**: copy the fingerprint from the `assetlinks.json` in the PWABuilder zip (or `signing-key-info.txt`).
2. **App signing key**: after the first upload (step 4), Play Console → your app → **Test and release → Setup → App signing** (older UI: **Setup → App integrity**) → copy **SHA-256 certificate fingerprint** under *App signing key certificate*. Play Console also offers a ready-made Digital Asset Links JSON on that page.
3. Edit `.well-known/assetlinks.json`: replace `REPLACE_WITH_PLAY_APP_SIGNING_SHA256` and `REPLACE_WITH_UPLOAD_KEY_SHA256` (format `AA:BB:CC:...`, 32 colon-separated bytes) and set `package_name` if you changed it. Commit and push; Cloudflare redeploys.
4. Verify with Google's tester: `https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://meditation-pulse.pages.dev&relation=delegate_permission/common.handle_all_urls`
5. Install from the testing track. If a URL bar still shows, the fingerprint or package name does not match; clear Chrome's data or reinstall after fixing, since verification is cached.

## 4. Play Console setup

1. Create a developer account at https://play.google.com/console (one-time US$25, identity verification). A personal account is enough.
2. **Create app**: name `Pulse Counter`, default language English, App, Free.
3. **App content** (Policy → App content):
   - Privacy policy: `https://meditation-pulse.pages.dev/privacy-policy.html`
   - Ads: No
   - App access: all functionality available without restrictions
   - Content rating: fill the questionnaire (category *Utility, Productivity, Communication or Other*; all answers No) → expect Everyone / PEGI 3
   - Target audience: 18+ (or 13+). Avoid under-13 groups unless you want the Families policy requirements
   - Data safety: **No** data collected, **No** data shared
   - Health apps declaration, if shown: pick the meditation / mindfulness option, or none if no option fits
   - Government app, financial features, news: No
4. **Store settings**: category **Health & Fitness**, contact email.
5. **Main store listing**: use the draft below, plus these assets from the repo:
   - App icon 512×512: `maskable-512.png` (full square; Play applies its own mask)
   - Feature graphic 1024×500: `store/feature-graphic.png`
   - Phone screenshots: `screenshots/screenshot-1.png`, `screenshots/screenshot-2.png` (add more of a running timer for a better listing)

## 5. Closed testing (required for new personal accounts)

Personal developer accounts created after 13 Nov 2023 need a closed test with **at least 12 testers opted in for 14 consecutive days** before they can apply for production.

1. **Test and release → Testing → Closed testing** → create a track → upload the `.aab` → add release notes.
2. Testers: create an email list of 12+ Google accounts (aim for 15 to absorb drop-outs). Countries: select yours.
3. Send review. Once approved, share the opt-in link. Each tester must open it, tap **Become a tester**, and install from Play.
4. Keep at least 12 testers opted in for the full 14 consecutive days.
5. Meanwhile, grab the app signing fingerprint (step 3) and confirm the URL bar is gone on a tester's device.
6. After 14 days: **Dashboard → Apply for production**, answer the questions about the test, wait for review (often up to 7 days), then promote the release to Production.

## 6. Updating later

- Web changes (HTML/JS/CSS): push to `main`. Installed apps pick them up; no Play release needed. Bump `CACHE` in `sw.js` when cached files change.
- Manifest/icon/package changes: rebuild in PWABuilder **with the same keystore**, increment the version code, upload a new release.

---

## Store listing draft

**App name** (≤30): Pulse Counter

**Short description** (≤80):
> Meditation timer that vibrates each second so you can keep time eyes closed.

**Full description** (≤4000):

> Pulse Counter is a minimalist meditation timer you can follow without looking at the screen. Your phone gives a gentle vibration once every second, so you always know how much time has passed with your eyes closed.
>
> HOW IT WORKS
> • Tap the circle to start, tap again to stop.
> • Open mode counts up for as long as you sit.
> • Pick 1, 5, 10 or 20 minutes, or enter your own, to count down. The session ends with three long buzzes and a soft chime.
> • Optional longer buzz at each full minute, so you can count minutes by touch.
> • Optional soft tick sound for each second.
> • The screen stays awake while the timer runs.
>
> GOOD FOR
> • Breath counting and box breathing
> • Silent sitting meditation
> • Yoga holds, stretching and planks
> • Any timed practice where you'd rather not watch a clock
>
> PRIVATE BY DESIGN
> No account, no ads, no tracking, no data collected. It works offline once installed.
>
> Light and dark themes follow your system setting.
>
> Tip: vibration stops if the screen locks, so leave the app open while you practise.

**Release notes (v1.0.0)**: First release.
