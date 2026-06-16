# Artist.ID — Handoff Package

> Version: 1.0.0 | Date: 2026-06-14
> Owner: Dewey | Arm: Music (built in Software project)
> Status: Specification handoff — pre-build

This package captures everything learned from building the *nEVER cOAST.* EPK as the
inception use case for Artist.ID. It is the bridge between "we built a one-off EPK
artifact" and "we build the app that produces these on demand."

## What this package is

A complete specification handoff for **Artist.ID** — the release pipeline / EPK builder /
backlog management app in the Music arm. Built to inherit the standard `APP_SPEC_TEMPLATE.md`
from the Software project, but the *content* of that spec lives here: the data model, the
workflows, the integration points, the open decisions.

This is not the app spec itself. This is the **input** to the app spec — distilled from
real product work (the *nEVER cOAST.* EPK build, which exposed every data point Artist.ID
needs to model).

## Files in this package

| # | File | Purpose |
|---|------|---------|
| 00 | `00_README.md` | This file. Index and how to use the handoff. |
| 01 | `01_CONTEXT_AND_INCEPTION.md` | The story of how we got here. What was learned by building the EPK. Why Artist.ID matters. |
| 02 | `02_DATA_MODEL.md` | The complete data model derived from the EPK. Every entity, every field, every relationship. |
| 03 | `03_EPK_AS_OUTPUT.md` | The EPK treated as the first deliverable Artist.ID must produce. Specification, styling system, embed handling, hosting considerations. |
| 04 | `04_PATCH_TALK_INTEGRATION.md` | How Artist.ID consumes from Patch Talk now (manual input + future hooks) and the bidirectional vision. |
| 05 | `05_WORKFLOW_END_TO_END.md` | The full creator workflow from concept → release → press kit → post-launch. Where Artist.ID sits in it. |
| 06 | `06_OPEN_DECISIONS.md` | Everything not yet decided. Flagged with options and recommendations. |
| 07 | `07_CLAUDE_CODE_PROCESSING.md` | How to hand this off to Claude Code for repo commit. Includes exact prompting. |

## How to use this package

**If you're Dewey reviewing:** read 01, then 06 (open decisions need your call before
the spec gets written). Everything else is reference.

**If you're handing this to Claude Code:** open `07_CLAUDE_CODE_PROCESSING.md` and follow
the prompt patterns there. The package is designed to commit cleanly into
`dewey-enterprise/docs/artist-id/`.

**If you're a future builder picking this up cold:** read 01 → 02 → 05 → 03 → 04 → 06
in that order. That's the narrative arc: why this exists, what it manages, how it's
used, what it outputs, what it integrates with, what's still open.

## What this package does NOT include

- **The Artist.ID app spec itself** (that's `APP_SPEC_TEMPLATE.md` filled in — happens
  after this handoff is reviewed and open decisions are closed).
- **The EPK HTML source** (lives in the repo as the reference artifact —
  `nevercoast_epk_v8.html`).
- **The Patch Talk spec** (lives in the Software project, separately).
- **UI/UX wireframes** (deferred to build phase — this is a content/architecture handoff).

## What was extracted from the EPK build to inform this package

The *nEVER cOAST.* EPK was built in a long iterative session that surfaced:

- Every credit role a release can have (17+ roles for one person on one EP)
- The release lifecycle: completion → Bandcamp window → DSP submission → live
- Merch as part of the release model (limited runs, edition numbers, hand-numbering)
- Media embed mechanics (video hosting that survives third-party restrictions)
- Asset versioning (the EPK went through 8 visual revisions before locking)
- Brand identity rules that must propagate to every release (typographic locks like
  `Samu=L` / `nEVER cOAST.`, the inverted-case treatment, the "3" motif)
- Cross-artist credit ambiguity (executive producer vs producer vs featured artist —
  the data model has to support all of them per-track)

Every one of those learnings is encoded in the files that follow.

---

*D.E.W.E.Y. — Do Everything Without Erasing Yourself*
