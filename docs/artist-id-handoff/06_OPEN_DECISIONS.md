# 06 — Open Decisions

> Everything not yet decided. Each has options and a recommendation.
> Read carefully — these need Dewey's call before the spec gets written.

## How to use this doc

For each open decision below:

- **Question** — what needs deciding
- **Why it matters** — what depends on this
- **Options** — what the realistic choices are
- **Recommendation** — what I'd pick and why
- **Status** — `OPEN`, `LEANING [option]`, `DECIDED — [option]`

When Dewey resolves a decision, the status changes and the rationale gets noted.
Once all are `DECIDED`, the Artist.ID spec is ready to be written against the
`APP_SPEC_TEMPLATE.md`.

---

## D1 — Asset hosting strategy

**Question:** Where do EPK media assets live long-term?

**Why it matters:** the *nEVER cOAST.* EPK hit embed failures across Box, YouTube
Shorts, Dropbox. Artist.ID needs a default hosting answer or every release will
re-run that experiment.

**Options:**

1. **Self-host on the same domain as the site** — assets live in the
   `dewey-enterprise` repo, served from `d3w3y.com`. Free, reliable, version-controlled.
   Limited by GitHub file-size constraints (50–100MB per file for non-LFS).

2. **Cloudinary free tier** — purpose-built for media delivery, CDN-backed, no
   hotlink restrictions. Free up to 25GB / 25k transformations a month. Survives
   when files are large.

3. **Hybrid** — small assets (album art, stills, brand assets) self-hosted; heavy
   media (videos, audio) on Cloudinary. Best of both, slightly more setup.

**Recommendation:** option 3 (hybrid). Matches the existing asset architecture from
the master doc.

**Status:** `OPEN`

---

## D2 — Where Artist.ID is hosted / what stack precisely

**Question:** Is Artist.ID a confirmed Railway + Prisma + Postgres + Next.js build?

**Why it matters:** the master doc locks this in §11 (decision 3) but the brief
hasn't been filled in via `APP_SPEC_TEMPLATE.md` yet.

**Options:**

1. **Standard Railway stack** — per the locked decision. No reason to deviate.

2. **Bolt-on to the Samu=L site** — host Artist.ID under `d3w3y.com/admin/artist-id`
   on the GCP stack. Tighter coupling. Faster v1.

**Recommendation:** option 1 (Railway). The locked decision exists for a reason —
keeping the GCP environment limited to the public site means the rest of the app
factory stays homogeneous.

**Status:** `LEANING 1` (deferring to the locked decision)

---

## D3 — Live EPK preview architecture

**Question:** How does Artist.ID preview an EPK while editing?

**Why it matters:** the iteration cycle on the *nEVER cOAST.* EPK was 8+ rounds.
Without live preview, every iteration is "click render, wait, look, change, click
render." Brutal.

**Options:**

1. **Side-by-side editor + iframe preview** — edit data on the left, see rendered
   EPK on the right. Reloads on save.

2. **Inline rich-text-on-render** — edit copy directly in the rendered EPK preview
   (contenteditable-style).

3. **Tabbed editor with refresh button** — simplest. Tab between "Edit" and "Preview".

**Recommendation:** option 1. Inline editing (option 2) is too complex for v1 and
gets weird with structured fields. Tabbed (option 3) is fine but slows iteration.

**Status:** `OPEN`

---

## D4 — Template flexibility model

**Question:** How rigid vs flexible should the EPK template be across releases?

**Why it matters:** *gREY oCTOBER* should look darker than *nEVER cOAST.* without
the EPK being a completely different page. The styling system needs to bend without
breaking.

**Options:**

1. **Fully fixed template** — same structure, same styling, only data changes. Cleanest.
   Limits expression.

2. **BrandLock overrides** — template is fixed but palette, motif, and typography can
   be overridden per release. Matches the data model.

3. **Section-level templates** — each section has multiple template variants the user
   can pick from per release.

**Recommendation:** option 2 (BrandLock overrides). It's what the data model already
supports. Section variants (option 3) is a v2 idea once we know what variants matter.

**Status:** `LEANING 2`

---

## D5 — Manual vs API-based DSP integration

**Question:** When generating the DSP submission sheet, do we hand-render a printable
PDF / markdown, or eventually integrate with a DSP API?

**Why it matters:** LANDR isn't going away as the distributor. There may be an API
worth integrating with eventually but not now.

**Options:**

1. **Hand-render markdown / PDF** — v1. Artist.ID outputs the sheet; Dewey copies
   the values into LANDR manually.

2. **LANDR API integration** — v2+. Submit directly from Artist.ID.

**Recommendation:** option 1 for v1. Hard-skip option 2 from v1 scope. Worth noting:
LANDR's API maturity should be checked before this is queued for v2.

**Status:** `LEANING 1`

---

## D6 — Person registry: artist-scoped or enterprise-scoped?

**Question:** When Dewey adds Danny Fal as a person, is that a Person within Samu=L's
namespace, or a Person within the whole D.E.W.E.Y. enterprise?

**Why it matters:** if Dewey ever launches another artist project, Danny Fal might
work with both. Enterprise-scoped People avoid re-entry.

**Options:**

1. **Enterprise-scoped** — one Person registry across all artists. Danny Fal is one
   record, referenced by both artists' releases.

2. **Artist-scoped** — Person belongs to an Artist. Re-entry required for cross-artist
   collaborators.

3. **Both via tagging** — Person has an optional `primary_artist_id` but can be
   referenced by any release.

**Recommendation:** option 1 (enterprise-scoped). The data is small, the dedup is
worth it, and the master doc's "house of brands" model supports it.

**Status:** `LEANING 1`

---

## D7 — Versioning model: snapshots or git-style history?

**Question:** How are EPK iterations preserved?

**Why it matters:** the 8 EPK revisions on *nEVER cOAST.* should be retrievable.
Future releases will have similar iteration counts.

**Options:**

1. **Snapshot on lock** — every time the EPK is "locked," a frozen snapshot is saved.
   Edits since the last snapshot are tracked in a working draft.

2. **Full git-style history** — every field change is its own commit, full revert
   to any point.

3. **No versioning** — current state only. Manual export saves are the user's problem.

**Recommendation:** option 1 (snapshots). Simple, useful, doesn't over-engineer.
Full history (option 2) is v2+ if it ever matters.

**Status:** `LEANING 1`

---

## D8 — Multi-artist support: v1 or v2?

**Question:** Does Artist.ID support more than one Artist record from v1?

**Why it matters:** today there's only Samu=L. But the data model supports multiple,
and Dewey may launch sister projects.

**Options:**

1. **Multi-artist from v1** — schema supports it; UI surfaces it.

2. **Single-artist with multi-artist-ready schema** — schema supports it; UI hides
   it for simplicity. Easy to enable later.

3. **Hardcoded single artist** — Samu=L is the only artist Artist.ID will ever know.

**Recommendation:** option 2. Schema flexibility costs nothing; UI complexity costs
attention. Hide the multi-artist UI until it's needed.

**Status:** `LEANING 2`

---

## D9 — Apparel arm coordination: who owns merch records?

**Question:** Apparel is its own arm. Merch released with a music drop — is it owned
by Artist.ID or by an Apparel-arm tool?

**Why it matters:** the SKU system is already CO-owned. Merch listings appear in
both the EPK and (eventually) a standalone shop. Splitting ownership creates drift.

**Options:**

1. **Artist.ID owns release-tied merch** — Apparel arm owns standalone apparel drops
   not tied to a music release. SKUs come from the CO-owned system.

2. **Apparel-arm tool owns all merch** — Artist.ID links to merch records owned
   elsewhere.

3. **Shared schema, both apps read/write** — same database tables, different UIs.

**Recommendation:** option 1. Release-tied merch is conceptually part of the release;
non-tied apparel is its own thing. This matches existing precedent (the master doc
notes "release-tied merch routing: files for music-release merch go to the music arm
path, not apparel arm").

**Status:** `LEANING 1`

---

## D10 — Spec template inheritance and naming

**Question:** When Artist.ID's spec gets written, does it inherit `APP_SPEC_TEMPLATE.md`
entirely, or are there release-pipeline-specific sections to add?

**Why it matters:** the template is the build standard. Artist.ID is a release pipeline
tool, not a generic CRUD app. Some sections of the template may not fit cleanly.

**Options:**

1. **Inherit fully** — fill in every section of the template as-is.

2. **Inherit + extend** — fill in every section + add release-pipeline-specific
   sections (render targets, asset embed handling, brand identity inheritance).

3. **Inherit but mark non-applicable** — some sections may say "n/a — see additional
   section X."

**Recommendation:** option 2. The template is the build floor; Artist.ID has extra
real concerns (rendering, embeds, brand locks) that deserve their own sections.
Discuss with Software arm whether those sections should be promoted into the
template for all future apps.

**Status:** `OPEN`

---

## Summary

| # | Decision | Status | Default if not decided |
|---|---|---|---|
| D1 | Asset hosting strategy | OPEN | Hybrid (self-host + Cloudinary) |
| D2 | Stack confirmation | LEANING 1 | Railway standard |
| D3 | Live preview architecture | OPEN | Side-by-side editor + iframe |
| D4 | Template flexibility model | LEANING 2 | BrandLock overrides |
| D5 | DSP integration | LEANING 1 | Manual sheet rendering |
| D6 | Person registry scope | LEANING 1 | Enterprise-scoped |
| D7 | Versioning model | LEANING 1 | Snapshot on lock |
| D8 | Multi-artist support | LEANING 2 | Schema-ready, single in UI |
| D9 | Merch ownership | LEANING 1 | Artist.ID owns release-tied |
| D10 | Spec template inheritance | OPEN | Inherit + extend |

---

*Next: `07_CLAUDE_CODE_PROCESSING.md` — how to commit this handoff into the repo.*
