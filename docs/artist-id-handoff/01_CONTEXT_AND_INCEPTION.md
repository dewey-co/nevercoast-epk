# 01 — Context & Inception

> Why Artist.ID exists, what the EPK build taught us, and how the trail from
> *nEVER cOAST.* planning to a working EPK distilled into a product specification.

## The arc

The Music arm has a planned app called **Artist.ID** — described in the enterprise
master doc and the Music brief as a "release pipeline / EPK builder / backlog management"
tool. Until this session it was a name with a brief sentence of scope. After this
session, it has a real specification grounded in real product work.

The path from "Samu=L is releasing *nEVER cOAST.* on June 16, 2026" to "Artist.ID needs
to model these entities" went like this:

1. **Release planning** — *nEVER cOAST.* was conceived, tracked, and rolled out through
   loose docs, a master constitution, and arm-level briefs. No tool. No structure.
   Decisions lived in conversation and got promoted to docs after the fact.

2. **Promotional rollout** — the campaign (Posts 1–4, drop day, merch reveal) was
   tracked through memory and references in chat. Hashtags, captions, post timing,
   asset routing all lived in conversation history.

3. **Merch production** — the embroidered towels and collector prints went from spec
   to production with intentional design (SKU system, edition numbering, run-based
   scarcity). The SKU system itself became a CO-owned document.

4. **EPK build** — when it came time to package the release for press / curators /
   playlist editors, every prior decision had to be re-surfaced and re-typed. Credits.
   Tracks. Dates. Roles. Merch detail. Contact. Links. Brand voice locks. The EPK
   wasn't generated *from* the master record — there *was* no master record. The EPK
   *became* the master record by being the first place all those facts assembled.

5. **The realization** — Artist.ID isn't "an EPK builder." It's the missing source
   of truth for every release. The EPK is one of N possible outputs (alongside DSP
   metadata sheets, playlist pitch one-pagers, sync licensing briefs, social posts,
   merch listings, and so on). Build the data model right and the EPK is a render
   pass against it.

## What the EPK build exposed

### Pattern 1 — Credit complexity is non-trivial

A press kit needs to credit every contributor accurately. On *nEVER cOAST.* alone:

- Samu=L: 17 distinct roles (artist, writer, narrator, composer, executive producer,
  producer on tracks 1 + 8, mixing engineer, recording engineer, mastering engineer,
  sound designer, designer, art director, photographer, videographer, editor, colorist,
  stylist, content curator)
- Danny Fal: production on tracks 2, 3, 4, 5, 6, 7
- El Bassface: featured artist (bass) on tracks 2, 4, 5
- Three daughters: voice on tracks 1, 8 (placeholder names pending real entries)

Every one of those roles is **scoped**: enterprise-level (Samu=L is exec producer of
the entire release), track-level (Danny on tracks 2–7), or instrumental (El Bassface
plays bass, not vocals). The data model has to capture role + scope + tracks
simultaneously. A flat "collaborators" string doesn't work.

### Pattern 2 — Brand identity must propagate

Hard-locked typographic conventions surfaced repeatedly:

- `Samu=L` (with equals sign — hyphen only when forced by platform constraints)
- `nEVER cOAST.` (with period, inverted case, only when referring to the EP as a title)
- `D.E.W.E.Y.` (always periods, always uppercase)
- Inverted-case track titles (`fIXED gEAR`, `mAGELLAN`, `tRACK sTANDS`, etc.)

These are not stylistic preferences. They are brand locks. Artist.ID has to enforce
them on output — a release can't be rendered with `Samu-L` in the body unless the
output channel forces it (DSPs).

### Pattern 3 — Releases have stages, not just dates

The release lifecycle has at least four states:

1. **In production** — material exists, not yet locked
2. **Locked / mastered** — final assets ready, drop date set
3. **Bandcamp exclusive window** — 30-day artist-direct period
4. **DSP live** — distributed to streaming platforms

The EPK had to render different content per state (e.g. the rollout timeline shows
"active" vs "future" dates). Artist.ID needs lifecycle awareness, not just a release date.

### Pattern 4 — Merch is part of the release, not a separate concern

Limited runs, edition numbers, hand-numbering, SKU codes, variants — all tied to a
specific release moment. Tracked in the EPK alongside the music. The data model has
to treat merch as a first-class release child, not a separate apparel-arm concern.

### Pattern 5 — Media embeds are hostile environments

Building the EPK ran into:

- YouTube Shorts blocking iframe embedding by default
- Box blob URLs not working as direct sources
- Dropbox `dl=0` vs `dl=1` link patterns
- Hotlinking restrictions across major file hosts

Artist.ID needs to model media assets with **hosting metadata**: where the file lives,
which URL pattern produces a working embed, whether it requires `?dl=1` etc. This isn't
a minor field — it's the difference between an EPK that works and one that shows broken
boxes.

### Pattern 6 — Asset iteration is the norm

The EPK went through eight major styling revisions before locking. Tags were debated
multiple times. The pullquote was added, then removed when the video carried the
same content. Photos arrived after the EPK was structurally complete. Audio embeds
went through three host attempts before settling on Dropbox.

Artist.ID isn't a one-shot generator. It's a **workspace** for iteration. Releases
need draft / locked states on every field. Outputs need re-render on demand.

### Pattern 7 — Cross-app data flow is real

The EPK pulled implicitly from:

- Master enterprise doc (brand standards, voice rules, lockables)
- Music arm brief (project narrative, ride data, references)
- Promotional rollout (campaign timing, post sequence, hashtags)
- Merch production records (SKU system, edition counts)
- Patch Talk (eventually — the beats, the lyrics, the project concepts)

Artist.ID is not an island. The data model has to anticipate inputs from these
sources even if integration is manual at v1.

## The thesis Artist.ID is built on

**A release is a structured record with many faces.** The structured record is the
truth. The faces — EPK, DSP submission sheet, playlist pitch, merch listing, social
post — are renders. Build the record well and every face becomes cheap to produce.

The reverse is what causes pain: each face being typed from scratch, with no shared
truth, drifting from each other, and falling out of sync as decisions change.

## What success looks like

When Artist.ID is built, the next release (*gREY oCTOBER*) should:

1. Start as a new Release record with the same schema *nEVER cOAST.* now retroactively
   conforms to
2. Accept manual data entry for everything Patch Talk can't yet provide
3. Render an EPK with one command, styled to brand-locked defaults, with override
   hooks for project-specific aesthetics (Grey October is darker; the EPK should know)
4. Output other release artifacts (DSP sheet, playlist pitch, merch listing) from the
   same record
5. Track the rollout state and surface what's next on the timeline

The *nEVER cOAST.* EPK is the proof-of-concept that this is possible. Every field on
that page is a data point in this model.

---

*Next: `02_DATA_MODEL.md` — the entities and relationships in full.*
