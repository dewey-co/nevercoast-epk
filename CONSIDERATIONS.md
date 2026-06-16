# nEVER cOAST. EPK — Project Considerations

> Retrospective notes from building the EPK. What was learned, what was painful,
> what is locked, and what this project seeded for Artist.ID.

---

## What this document is

A project-specific retrospective tied to the *nEVER cOAST.* EPK build (June 2026).
Not a generalized spec — that lives in `dewey-enterprise/docs/artist-id/`.
This is the "what we learned from building THIS thing" record.

---

## What went well

- **Self-contained HTML output** — a single `index.html` with embedded base64 assets
  means zero dependencies, zero CDN calls, zero breakage. For a press kit that needs
  to survive indefinitely, this was the right call.
- **GitHub + Netlify deployment** — repo → Netlify → custom domain took under an hour
  once the DNS cache flushed. The pipeline is repeatable.
- **Brand locks held** — `Samu=L`, `nEVER cOAST.` (with period), inverted-case track
  titles, and the `3` motif stayed consistent across 8 revision rounds.

---

## What was painful

### 1. No master record existed before the EPK
Every credit, date, link, and track detail had to be re-surfaced from conversation
history and loose docs. The EPK itself became the first place all facts assembled —
which means it was being built and discovered simultaneously.

**Implication for Artist.ID:** the data entry problem is real. The app must support
incremental, partial records — not force a complete record before anything renders.

### 2. Media embed failures burned significant time
- YouTube Shorts blocked iframe embedding
- Box blob URLs didn't work as direct sources
- Dropbox needed `?dl=1` not the default share URL
- Audio went through three host attempts before settling on Dropbox

Final working embed solution: YouTube standard video for video; Dropbox `?dl=1`
for audio. These are the locked patterns for this release.

**Implication for Artist.ID:** hosting metadata (not just URLs) must be a first-class
field. Where a file lives determines whether the embed works.

### 3. Eight visual revision rounds before lock
Typography, palette, motif density, photo placement, pullquote in/out — all went
through multiple passes. Without a live preview, every round was: edit → refresh →
evaluate → repeat.

**Implication for Artist.ID:** live preview is not optional. It's the difference
between a tool and a punishment.

### 4. Credit complexity was underestimated
One EP, one primary artist, two collaborators — and the credits required:
- 17 roles for Samu=L (scoped at release and track level)
- Collaborator roles scoped to specific tracks
- Instrumental scoping (what instrument, not just "featured")

A flat "collaborators" field would have been wrong. The EPK forced a real credits
data model to emerge.

---

## Decisions locked for this release

| Area | Decision |
|---|---|
| Video hosting | YouTube (standard, non-Shorts) |
| Audio hosting | Dropbox with `?dl=1` |
| Static hosting | Netlify via `dewey-co/nevercoast-epk`, domain `press.d3w3y.com` |
| HTML format | Single self-contained `index.html`, base64 assets embedded |
| EPK version | v8 (locked June 2026) |
| Bandcamp window | 30-day exclusive from June 16, 2026 |
| DSP target | LANDR submission after Bandcamp window closes |

---

## What this EPK seeded

This build is the inception case for **Artist.ID** — the release pipeline / EPK builder
app planned for the Music arm. The full spec handoff (data model, open decisions,
workflow) lives in `docs/artist-id-handoff/` alongside this file, and is mirrored
into the enterprise repo at `dewey-enterprise/docs/artist-id/`.

The *nEVER cOAST.* EPK is the reference artifact Artist.ID will be built to produce
on demand. Every field on the page is a data point in the model.

---

## What to do before the next release (*gREY oCTOBER*)

1. Resolve open decisions D1, D3, D10 in `docs/artist-id-handoff/06_OPEN_DECISIONS.md`
2. Fill `APP_SPEC_TEMPLATE.md` to produce the Artist.ID build spec
3. Start *gREY oCTOBER* as a new Release record in Artist.ID (once built) rather
   than repeating the manual EPK assembly process

---

*D.E.W.E.Y. — Do Everything Without Erasing Yourself.*
