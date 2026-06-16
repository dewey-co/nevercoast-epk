# 03 — EPK as Output

> The EPK as the first deliverable Artist.ID must produce. Specification, styling
> system, embed handling, hosting considerations. Derived directly from the working
> *nEVER cOAST.* EPK (`nevercoast_epk_v8.html`).

## What an EPK is, in this system

An EPK is **one render target** of a Release record. It is a single-page HTML
artifact intended to be shared with press, playlist curators, sync supervisors,
and booking contacts. It must be hostable on a custom domain or as a static file
and must work without third-party dependencies that could break (CDNs are okay;
hotlink-restricted asset hosts are not).

It is not a CMS. It is not a marketing site. It is a press kit — a structured,
beautiful single page that answers every question a curator might ask before
deciding whether to listen, pitch, or place the music.

## Mandatory sections (and what each one renders)

The reference EPK established these sections in this order. Artist.ID should treat
them as the default template, all individually toggleable.

| # | Section | Renders from | Notes |
|---|---|---|---|
| 1 | Link page (landing) | Artist.links + Release.links + scroll trigger | Black background. Linktree-style. Last button scrolls to EPK. |
| 2 | Spacer | Static phrase ("KEEP PEDALING.") | Black. Visual breathing room between link page and EPK. |
| 3 | Hero | Artist + Release.title + Release.concept_tagline + Release meta | Dark. Glitch + ambient glow. Meta row pills. |
| 4 | Press Photo | Asset where type = `press_photo` and is_primary | Dark. Falls back to album art if no press photo. |
| 5 | Artist | Artist.bio_long + Release.tags | Light (off-white). Display title treatment. |
| 6 | Concept | Release.concept_long + Asset(teaser_video) at top | Dark. Video leads. |
| 7 | Stream / Watch | Asset(audio_stream or promo_video) | Light. |
| 8 | Ride Data / Real Data | Release.custom_data (project-specific numeric fields) | Dark. Counts up on scroll. |
| 9 | Tracklist | Tracks + Credits grouped by track | Light. Inverted case preserved. |
| 10 | Credits | Credits grouped by Person | Dark. Pill block for primary artist, plain rows for others. |
| 11 | Merch | MerchItem + MerchVariant | Light. Photo + edition number watermark. |
| 12 | Rollout | TimelineEvent[] | Dark. Active date highlighted gold. |
| 13 | Contact | Artist.contact_email + parent brand | Light. Email in gold mono. |
| 14 | Footer | Artist + Release.title + year | Dark. Teal-to-gold rule. |

Alternating dark / light is intentional. Sections 4, 6, 8, 10, 12, 14 are dark;
sections 5, 7, 9, 11, 13 are light. Artist.ID should support this as a template
attribute, not hard-code it — *gREY oCTOBER* may want a different rhythm.

## Brand styling system

These are the locks that propagated through eight EPK revisions. They should be
codified as a styling system Artist.ID inherits from Artist.BrandIdentity.

### Color palette

| Token | Value | Use |
|---|---|---|
| `navy` | `#080C10` | Dark section background |
| `offwhite` | `#F0EDE8` | Light section background |
| `teal` | `#1A7A6E` | Primary brand accent (UI) |
| `teal-lt` | `#26A99A` | Bright teal (hover, glow, eyebrows) |
| `gold` | `#C9922A` | Support accent (numbers, active dates, hidden detail) |
| `gold-lt` | `#E0A83A` | Bright gold (hover, email, data numbers) |
| `slate` | `#3A3F4B` | Muted text |
| `ash` | `#6E7480` | Body muted |
| `ash-lt` | `#9AA0AA` | Body |
| `white` | `#F0EDE8` | White-text fallback (matches offwhite) |

**Balance rule** (locked through iteration): teal leads, gold supports. Gold is
jewelry — it appears on numbers, active timeline dates, the EPK button on landing,
contact email, merch badges. It does not lead UI hierarchy.

### Typography

| Element | Family | Treatment |
|---|---|---|
| Body | Space Grotesk | 300–700 weight |
| Mono / labels / data | DM Mono | 300–500 weight |
| Track titles | Inverted case preserved as literal text | Never CSS-uppercased |
| Section titles | Uppercase by CSS | Except tracklist section title (`nEVER cOAST.`) |
| Display titles | Larger size class | For Artist, Concept, Ride Data sections |

### Motion system

| Effect | Where | Notes |
|---|---|---|
| Particle field | Full page, gold particles drifting upward | Canvas, opacity 0.6 |
| Cursor glow | Hover state across the page | Teal radial |
| Glitch | Hero artist name | Teal + gold ghost layers, infinite slow loop |
| Grain + scanlines | Full page overlay | Low opacity |
| Hero glow | Project title text-shadow + ambient pulse | Teal |
| Sweep lines | Before each section reveal | Teal-to-gold gradient, left-to-right |
| Reveal | Sections enter viewport | Fade + 24px slide up |
| Count up | Data numbers on view | Cubic ease, 1.4s |
| Parallax | Hero text, section headers, pullquote, merch cards | Per-element speed (-0.05 to -0.24) |
| Sticky nav | Appears after scroll past hero | Frosted glass, section label |
| Back to top | After 50vh scroll | Gold border button |
| Section sweep cut | Hard cut between dark / light | No fade — abrupt is intentional |
| Long EPK scroll | EPK button → 300vh spacer → EPK | Cubic ease scaled to distance, 1–2.4s |

All motion has a `prefers-reduced-motion` fallback that disables particles, cursor
glow, parallax, glitch, and sweeps. The page still works without any motion.

## Embed handling — the painful real-world lesson

The EPK build hit every major embed restriction. Artist.ID's data model treats
each asset with a `host_provider` + `embed_url_pattern` field for this reason.

### Working patterns (use these)

| Provider | Direct URL pattern | Embed type |
|---|---|---|
| Self-hosted | `./assets/file.mp4` | `<video>` tag |
| Dropbox | `?dl=1` appended to share link | `<video>` tag (direct source) |
| Vimeo | Standard iframe code | `<iframe>` |
| YouTube (regular) | `youtube.com/embed/VIDEO_ID` | `<iframe>`, embedding must be allowed |
| Cloudinary | Direct asset URL | `<video>` / `<img>` tag |
| Bandcamp | Standard embed snippet from track page | `<iframe>` |

### Broken patterns (avoid these)

| Provider | Issue |
|---|---|
| YouTube Shorts (default) | Block embedding by default in YouTube Studio. Must be enabled per-video. |
| Box (blob URLs) | Blob URLs only valid in browser session. Useless externally. |
| Instagram | No iframe embedding outside Instagram's own platform. |
| TikTok | Limited; embedding works but is restrictive and brittle. |
| Direct file hotlinks on file hosts | Most block hotlinking. |

### The pattern Artist.ID should follow

1. When an Asset is added, record `host_provider` explicitly.
2. Auto-transform the URL to embed-friendly form on render based on provider.
3. Validate embeds on save — fetch HEAD on the URL, flag if it returns 403 / blob / etc.
4. Surface a clear "this won't work in your EPK" warning before the user finalizes.

### Recommended hosting strategy (locked decision needed)

For the long term, asset hosting should follow the enterprise asset model from
the master strategy doc:

- **In-repo assets** (under ~50MB): album art, smaller stills, brand assets → committed to
  the `dewey-enterprise` repo and served from the same domain as the EPK
- **Heavy media** (videos, audio masters): hosted on a single dependable provider
  (recommendation: **Cloudinary free tier** — built for this exact use case, no
  hotlink restrictions, CDN delivery)
- **Bandcamp / YouTube** for distribution-channel embeds only, not for EPK media sources

## EPK render specification (the contract)

When Artist.ID renders an EPK, it must:

1. **Resolve typography locks first.** Title rendering, inverted case, period suffixes,
   `Samu=L` vs `Samu-L` — apply brand rules before content rendering.
2. **Order credits intelligently.** Primary artist (the release's artist) renders as
   pill block, all others as plain rows. Within each Person, group by scope (release-level
   first, then track-level, then instrument-level).
3. **Filter by `is_live` on links.** Links that aren't live yet show "Coming Soon" state,
   not broken URLs.
4. **Compute timeline states.** `active` = today; `upcoming` = within 14 days; `future` =
   beyond. Style each differently.
5. **Validate every asset embed before render.** Don't ship a render with broken videos
   or 404'd images.
6. **Honor BrandLock overrides.** If a release has palette overrides, they take precedence
   over the artist's default brand identity.
7. **Output a single self-contained HTML file.** No build step required for the consumer.
   Inline what makes sense (CSS, small SVG icons); reference what must be external
   (large media, fonts via CDN).

## Hosting the EPK

The EPK is meant to live at a stable URL the artist controls. Three viable patterns:

### Pattern A — Subpage of the artist site (recommended once site is live)

`d3w3y.com/press/nevercoast` — lives alongside the main brand site, version-controlled,
free, professional URL.

### Pattern B — Static file in `dewey-enterprise` repo, served via GitHub Pages

Free, immediate, version-controlled. URL would be `dewey.github.io/nevercoast-epk` or a
custom domain pointed at the Pages site.

### Pattern C — Drag-and-drop on Netlify (immediate, temporary)

Two-minute deploy. Free subdomain. Good for "I need a link right now"; migrate to A
or B when ready.

Artist.ID should support exporting the rendered HTML for any of these — the system
doesn't need to host. It just needs to produce a clean artifact.

## EPK iteration cycle (what Artist.ID has to support)

The reference EPK went through eight major revisions. Every release will. Artist.ID
must support:

- **Live preview** — see the rendered EPK while editing the underlying record
- **Versioned snapshots** — every "lock" creates a snapshot you can roll back to
- **Section toggling** — turn sections on/off per release (an instrumental EP may not
  have a Watch section if there's no promo video)
- **Component overrides** — replace section copy without changing the data model
  (some releases need bespoke language a curator pitch wouldn't pull from bio fields)
- **Export targets** — HTML for hosting, PDF for emailing, plain markdown for
  copy/paste into other systems

## What the *nEVER cOAST.* EPK proved is hard

These were the genuinely tricky parts. Artist.ID must solve them by design, not
by hand-editing every time:

1. **Credit phrasing.** "Production — Tracks 2, 3, 4, 5, 6, 7" vs "Featured Artist (Bass) —
   Tracks 2, 4, 5" vs "Voice — Tracks 1, 8" vs "Producer (Tracks 1, 8)". Every variation
   has a logic but each was hand-crafted in the EPK. The render needs to generate these
   from Credit records with no manual phrasing.

2. **Typographic discipline.** `nEVER cOAST.` rendered correctly *except* where CSS
   uppercased it, *except* when used mid-sentence as a title. The render needs context-aware
   typography.

3. **Asset hosting fallthroughs.** When a press photo doesn't exist, what fills the slot?
   In v8 it fell back to the album art. Artist.ID needs to define these fallbacks per
   render target.

4. **State-aware rendering.** A release in `bandcamp_window` shows different copy than
   one that's `dsp_live`. The render must read state and conditionally include / exclude
   sections.

5. **Multi-source data assembly.** Bio is artist-level, tracks are release-level, credits
   join both, merch is release-level, contact is artist-level. The render has to traverse
   these relationships cleanly.

---

*Next: `04_PATCH_TALK_INTEGRATION.md` — how Artist.ID consumes from Patch Talk.*
