# 04 — Patch Talk Integration

> How Artist.ID relates to Patch Talk. v1 is consume-only with manual fallback.
> Bidirectional is the eventual vision but explicitly out of scope for v1.

## The two apps in one sentence each

**Patch Talk** = where the music is *made*. Producer workflow: beat tagging, sample
management, lyric storage, ghostwriter assistance, GenAI project concept generation.
Existing codebase (do not rebuild).

**Artist.ID** = where the release is *managed and published*. Pipeline tool: structured
release records, EPK / DSP / pitch outputs, rollout tracking, credits + merch + timeline.
New build.

## The relationship

Patch Talk produces the **creative material**. Artist.ID consumes that material as
release content. The arrow points one direction in v1: Patch Talk → Artist.ID.

```
PATCH TALK                        ARTIST.ID
─────────────                     ─────────────
beat catalog          ──┐
sample tags             │
lyrics + rhymes         ├─→  Track.lyrics
project concepts        │    Track.notes
chord progressions      │    Release.concept_long
ghostwriter sessions  ──┘    Release.production_signatures

                                  ──→ EPK output
                                  ──→ DSP submission
                                  ──→ Playlist pitch
                                  ──→ Merch listings
                                  ──→ Rollout timeline
```

## V1 integration approach — consume + manual fallback

### Principle

Artist.ID should **always accept manual entry** for every field, regardless of whether
Patch Talk could theoretically provide it. Manual is the floor. Integration is the
convenience layer on top.

This matters because:

1. Patch Talk integration isn't built yet
2. Patch Talk's existing data may not match Artist.ID's schema cleanly
3. Some releases will draw from material made before Patch Talk existed
4. Some releases will use external material (collaborator beats from Danny Fal, etc.)

### Three input modes

Every text/data field on a Release or Track has three input modes:

| Mode | When used | UI hint |
|---|---|---|
| **Manual** | Default, always available | Plain input field |
| **Patch Talk import** | When integration is live and the field has a Patch Talk source | "Import from Patch Talk" button next to the field |
| **External source** | Manual entry with source attribution (e.g. "from Danny Fal session notes") | Source field exposed |

### V1 import shape (manual JSON / clipboard paste)

Until a real API exists, Artist.ID should support **paste-import** from Patch Talk.
Patch Talk can export a project as JSON (this is a small ask on the Patch Talk side
since it's an existing codebase — just add an "Export to Artist.ID" button).

A pasted JSON blob like this:

```json
{
  "patch_talk_project_id": "pt_abc123",
  "title_working": "Never Coast EP",
  "tracks": [
    {
      "patch_talk_track_id": "pt_t1",
      "title_working": "Fixed Gear",
      "bpm": 78,
      "key": "Am",
      "lyrics": "...",
      "samples_used": ["..."],
      "notes": "..."
    }
  ],
  "concept_notes": "...",
  "color_reference": "#1A7A6E",
  "image_reference_url": "..."
}
```

...maps into Artist.ID by:

- Creating or updating a Release with `title_working` (locked in Artist.ID, not Patch Talk)
- Creating Tracks with `patch_talk_ref` populated
- Populating `concept_long` from `concept_notes`
- Importing lyrics into `Track.lyrics`

The mapping should be **suggestive, not destructive**. Artist.ID asks "import these
fields? select which to apply" rather than blindly overwriting locked values.

### Patch Talk side — what we need from it (eventually)

When Patch Talk gets its overhaul (Oct 2026 per the timeline), one feature ask:

- **Export project as JSON** — single button on a project view, copies a normalized
  JSON blob to clipboard. No API needed. Manual paste into Artist.ID.

That's the minimum integration that unlocks the value. Everything beyond it is
convenience.

## V2+ integration (bidirectional, out of scope for v1 build but spec for clarity)

This is the long-term vision. **Do not build this in v1.** Capturing for context only.

### Direction 2: Artist.ID → Patch Talk

Use cases that flow back:

- **Track is locked in Artist.ID** → Patch Talk marks the source project as "released"
  and locks editing on it
- **Release timeline updates** → Patch Talk knows when its source project is going
  live, can show it on the producer dashboard
- **Credits add a collaborator** in Artist.ID → Patch Talk imports that Person into
  its collaborator list

### Shared services

When both apps mature, they should share:

- **Person registry** — Danny Fal is one Person across both apps
- **Asset library** — beats, stems, samples referenced from both apps
- **Brand identity** — Patch Talk's GenAI concept generator should produce concepts
  aligned to D.E.W.E.Y. brand locks

### Implementation approach when ready

Both apps run on Railway with Prisma/Postgres per the locked infra split. The two
options when integration matures:

1. **Shared database, separate schemas** — fastest, simplest, requires Artist.ID
   and Patch Talk to share a Railway project. Schema-namespaced tables. Direct queries
   across schemas.

2. **Separate databases, REST/webhook integration** — cleaner separation, more flexible
   if either app gets open-sourced or used by other artists. Each app owns its data;
   webhooks notify on relevant events.

V2 recommendation: start with option 1 (shared database) because it's a one-person
operation. Move to option 2 only if either app is ever offered beyond Samu=L.

## What v1 looks like in practice

1. Dewey opens Artist.ID, creates a new Release for *gREY oCTOBER*
2. Fills in basic fields manually (title, drop date, genre)
3. Adds Tracks one by one — either typing them in or pasting a Patch Talk export
4. Adds Credits via a person picker that defaults to existing People (Danny Fal,
   El Bassface, daughters) and lets new ones be added
5. Uploads assets (album art, press photo, videos) with host metadata captured
6. Defines merch items and variants
7. Lays out timeline events
8. Clicks "Render EPK" — gets the same kind of artifact the *nEVER cOAST.* EPK is,
   styled to *gREY oCTOBER*'s darker palette via BrandLock overrides
9. Exports HTML, hosts at `d3w3y.com/press/greyoctober`

No Patch Talk integration required for any of that. When Patch Talk gets a paste-export
button, step 3 becomes faster. Everything else still works.

## What success looks like

Patch Talk feeds Artist.ID without either app being aware of the other's internals.
Artist.ID is the canonical place a release lives. Patch Talk remains the canonical
place creation happens. The data flows one way until both apps mature enough to
warrant bidirectional, at which point the path forward is clear.

---

*Next: `05_WORKFLOW_END_TO_END.md` — the full creator workflow Artist.ID sits inside.*
