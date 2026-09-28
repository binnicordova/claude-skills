---
name: play-store-aso
description: Audit and rewrite Google Play store listings (name, short and full description, category, tags, language) to grow store listing visitors, conversion and installed audience, then send the changes for review in Play Console. Use when asked to improve an app's Play Store listing, ASO, keywords, installs or "installed audience".
---

# Play Store ASO (App Store Optimization)

Goal: more **store listing visitors** and a higher **visitor → install conversion**, so the installed audience grows.
Proven formula from this portfolio: CoCap (100+ visitors/day on its best days) wins because of **traffic loops + a listing that explains "how it works" in 3 steps first**.

## 1. Audit first (per app, last 28 days)

In Play Console:

- **Grow users → Store listings**: Visitors, Unique install clicks, Click-through rate (conversion).
- **Grow users → Overview**: impressions, acquisitions, first opens, "explore" acquisitions (Play organic).
- **Store settings**: category + tags.
- **Main store listing**: which language the default slot is, and whether the text actually matches it.

Diagnose:

| Symptom | Meaning | Fix |
|---|---|---|
| Few visitors, high CTR | Listing converts, nobody finds it | Keywords + share loop (step 5) |
| Many visitors, low CTR | People find it but don't install | Screenshots, video, short description, first 3 lines |
| Installs ≫ first opens | Listing promises something the app doesn't deliver right away | Align listing with onboarding |

## 2. Listing rules

**App name (30 chars)**
- One brand word, no colons or "Brand: keyword" patterns (owner preference: e.g. `Saludables`, `PlacaOk`, `CoCap`, `JobYnc`, `Vigiia`).
- Fix casing (`pickpointer` → `PickPointer`). No emojis, "Best", "#1", "Free", or government or third-party brand names.

**Short description (80 chars)**
- 2–3 real search keywords in one natural sentence about the user's benefit.
- Don't repeat the app name. No emojis, ALL CAPS or ranking claims.
- Count characters before saving.

**Full description (4000 chars, all indexed)**: use this exact structure (the CoCap formula):

```
<Hook: 1–2 lines with the main keyword and the core value. Visible above the fold (~167 chars).>

<How it works / Así de fácil …>:
1. <Step>
<one line>
2. <Step>
<one line>
3. <Step>
<one line>

<Purpose paragraph: what the app is for and why it's different, with keyword variants.>

<What you get / Lo que … >

<emoji> <Feature>: <benefit>
<emoji> <Feature>: <benefit>
… (4–6 items)

<Ideal for / Ideal para …: audiences + long-tail keywords>

👉 <Call to action with the app name>
```

- Main keyword 3–5 times total, with natural variants. No stuffing.
- **Only claims the app already makes.** Never invent features, stats, reviews or social proof.
- Keep the brand's own voice lines (e.g. "Dale play, carajo.").

**Language**
- The default listing language must match the text (Spanish text goes in `es-419`, not `en-US`).
- If there's a mismatch, add the matching language listing or change the default under Manage translations.

**Category and tags**
- Pick the category the user would browse (examples: resorts/beaches → Travel & Local; exam practice → Education).
- Up to 5 tags. Remove unrelated ones (e.g. "Web browser" on a mototaxi app, "Watch face" on a social app).

**Graphics**
- 8 phone screenshots with short benefit captions, and a feature graphic.
- A 20–30 s video. It must be public or unlisted with ads off, or Play blocks saving ("cannot be displayed…").

## 3. Policy red flags (fix before anything else)

- Government agency names in the title without authorization (e.g. "MINSA-DIGESA") → remove, and mention data sources only in the description.
- Implying official certification ("CERTIFICADO", "Official Certification") without accreditation → "completion certificate" or remove.
- Invented personas or testimonials ("Soy Carlos Mendoza…") presented as real → remove.
- Third-party brands (TikTok, YouTube, podcast hosts) → need permission or a disclaimer: "X is an independent app and is not affiliated with, sponsored by or endorsed by <Brand>."

## 4. Apply and send for review (Play Console)

1. Edit **Main store listing** → Save. Edit **Store settings → App category** → Save.
2. Go to **Publishing overview** and check that the "Changes not yet submitted" list only contains your changes.
3. Quick checks run for up to ~14 minutes. Then **Submit N changes for review → Send changes for review**.
4. If you see **"Do you want to restart your review?"**, a release is already in review. Restarting adds wait time, so confirm with the owner unless they already agreed.
5. **"1 issue affects all of your changes"** means a blocker outside the listing, usually the target API level of an old build. It needs a new app release.
6. Reviews usually take up to 7 days.

Browser-automation tips: fill fields with `el.focus(); el.select(); document.execCommand('insertText', false, text)` so Angular registers the change. Plain `value =` leaves Save disabled.

## 5. Growth loops (what really drove CoCap)

Add a way for every user to bring the next one through a Play link:

- Events and communities: QR or invite links (CoCap).
- Content: "share this clip" plus a Play link (podcast apps).
- Utilities: a shareable report or certificate ("Revisé este auto con PlacaOk").
- Seasonal apps: update the listing about 6 weeks before the season (Peru's summer starts in December).

## 6. After submitting

- Run **Store listing experiments**, one element at a time: icon, then short description, then screenshots. Run each for at least 7 days.
- Reply to every 1–3★ review. Ratings drive conversion (a 2.5★ rating caps any listing).
- Re-check visitors and CTR after 28 days and compare with the audit.

## Output to the owner

A short table per app: new short description, main keywords, other changes (name, category, tags, video), and review status (sent / blocked and why). Then the top 1–2 manual follow-ups.
