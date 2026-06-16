# 05 — End-to-End Workflow

> The full creator workflow from concept to post-launch. Where Artist.ID lives in
> the larger Music arm operation and how it interacts with adjacent tools and
> processes.

## The workflow at a glance

```
1. CONCEPT          → Patch Talk (existing) — beats, lyrics, project idea
                                ↓
2. PRODUCTION       → Patch Talk + Ableton — beats become tracks, lyrics written
                                ↓
3. RELEASE PLANNING → Artist.ID — Release record created, drop date set
                                ↓
4. ASSET CREATION   → Lightroom / Photoshop / Premiere / embroidery rig
                                ↓
5. PRESS KIT        → Artist.ID — EPK rendered from Release record
                                ↓
6. ROLLOUT          → Artist.ID timeline + Instagram / YouTube / Bandcamp manual posting
                                ↓
7. DROP DAY         → Bandcamp goes live, EPK URL distributed
                                ↓
8. DSP SUBMISSION   → Artist.ID generates DSP sheet; manual submission to distributor
                                ↓
9. DSP LIVE         → Artist.ID flips state, EPK auto-updates timeline + links
                                ↓
10. POST-LAUNCH     → Artist.ID tracks engagement metrics manually entered;
                       backlog for next release surfaces
```

## Where Artist.ID is the source of truth vs handoff

| Activity | Tool of record | Artist.ID role |
|---|---|---|
| Beat / sample creation | Patch Talk | Consumes references |
| Lyric writing | Patch Talk | Consumes finalized lyrics |
| Mixing / mastering | Ableton + plug-ins | Stores file paths / version notes |
| Project concept | Patch Talk (or notebook) | Stores `Release.concept_long` |
| Track-level metadata (BPM, key) | Patch Talk | Imports |
| Release-level metadata (title, dates, type) | **Artist.ID** | Source of truth |
| Credits | **Artist.ID** | Source of truth |
| Merch design / production | Embroidery rig + Photoshop + Cricut | Stores listings, photos, SKUs |
| Promotional content | Premiere / Instagram | Stores assets + timeline events + captions |
| Press kit | **Artist.ID** | Source of truth — renders EPK |
| DSP submission | LANDR or equivalent | Artist.ID generates the input sheet |
| Bandcamp listing | Bandcamp | Artist.ID renders the copy |
| Social posting | Manual on each platform | Artist.ID stores caption + hashtag bundle, "marks as posted" |
| Engagement metrics | Manual entry | Stored against Release for future analysis |

## The flow in detail

### Phase 1 — Concept

**Tools:** Patch Talk (existing), notebook, voice notes

**Artist.ID involvement:** none yet. Concept lives in Patch Talk or wherever Dewey
likes to think.

**Trigger to move forward:** an idea is sticky enough to commit to a release record.

### Phase 2 — Production

**Tools:** Patch Talk for organization, Ableton for production, Maschine for sketches

**Artist.ID involvement:** still minimal. Once tracks are clearly going to be a
specific release, create a Release record in Artist.ID with a working title and
`release_state = "in_production"`. Add Track records as material solidifies.

**This is the bridge moment** — when something stops being a vibe and starts being
a release. Artist.ID gets created at this transition.

### Phase 3 — Release Planning

**Tools:** Artist.ID + the master enterprise doc

**Artist.ID activities:**
- Lock title, type, drop date
- Set up parent_series if part of a trilogy
- Define genre, tags, concept_tagline
- Add Bandcamp drop date and projected DSP go-live
- Create initial timeline events (Bandcamp drop, DSP submission, DSP live, future
  next-release teaser if applicable)
- Set `release_state = "mastered"` when material locks

**Decision required:** does this release have project-specific BrandLocks (darker
palette, different motif)?

### Phase 4 — Asset Creation

**Tools:** Lightroom / Photoshop for stills, Premiere for video, embroidery rig
for merch, Cricut for print SKU cuts

**Artist.ID activities:**
- Upload assets as they're produced
- Tag each with `asset_type` and `host_provider`
- Mark `is_primary` on the canonical version of each type
- Create MerchItem + MerchVariant records as physical pieces come together

**Critical:** the `embed_url_pattern` must be captured at upload time, not at EPK
render time. The lesson from the *nEVER cOAST.* EPK build: figuring out embed
patterns at render time is too late.

### Phase 5 — Press Kit

**Tools:** Artist.ID

**Artist.ID activities:**
- Render EPK preview
- Iterate on:
  - Section selection (do we have a Watch section?)
  - Copy adjustments (this is where bio overrides happen if needed)
  - Asset replacement (better press photo when available)
- Lock EPK and export HTML
- Host at stable URL

**The reference cycle** to watch for: the EPK going through 8+ revisions before
locking. Artist.ID should support this iteration cleanly without making it feel like
starting over each time.

### Phase 6 — Rollout

**Tools:** Artist.ID for tracking + Instagram + Threads + YouTube + Bandcamp for posting

**Artist.ID activities:**
- Each promo post is a TimelineEvent
- Caption + hashtag bundle stored against the event
- Asset associations set (the video for that post)
- After posting, `posted_url` updated to where the post lives
- Cross-post coordinated: same caption, platform-specific edits made manually

**This is the busiest period.** Artist.ID's value here is keeping all the post-prep
material organized so Dewey isn't reassembling captions from chat threads.

### Phase 7 — Drop Day

**Tools:** Bandcamp + Artist.ID

**Artist.ID activities:**
- Flip `release_state = "bandcamp_window"`
- TimelineEvent for Bandcamp drop marked posted, Bandcamp URL recorded
- Drop day post recorded
- EPK link distributed to press list / playlist contacts

### Phase 8 — DSP Submission

**Tools:** Artist.ID + LANDR (or equivalent distributor)

**Artist.ID activities:**
- Render DSP submission sheet — metadata, ISRC codes (when assigned), genre,
  contributor splits, artwork in correct dimensions
- Use the sheet to populate the LANDR submission

**This is where the split agreements from the credit data become real.** If Danny
Fal's production split isn't in writing before this point, the legal function flagged
in the master doc gets exercised. Artist.ID surfaces the missing splits as warnings.

### Phase 9 — DSP Live

**Tools:** Artist.ID + manual link gathering

**Artist.ID activities:**
- `release_state = "dsp_live"`
- Each DSP link added as a Link record with `is_live = true`
- EPK auto-updates — the Links section now shows live URLs instead of "coming soon"
- Timeline event for "all DSPs live" marked active

### Phase 10 — Post-Launch

**Tools:** Artist.ID + manual analytics review

**Artist.ID activities:**
- Engagement metrics manually entered (Bandcamp sales, Spotify saves, IG reach)
- Backlog surfaces what's next:
  - Are there unsold merch units? (warning to promote)
  - Is the next release in the trilogy ready for Phase 1?
  - Are there post-mortem learnings to capture against the Release record?

## What Artist.ID is *not* doing in any phase

To keep v1 scoped:

- **Not** posting to social media directly (manual posting per phase 6)
- **Not** managing DSP credentials or submitting via API
- **Not** processing Bandcamp orders or fulfillment
- **Not** running ads or boost campaigns
- **Not** generating creative content (that's Patch Talk + Premiere + Photoshop)
- **Not** acting as a CMS for the artist site (Sanity handles that, eventually)

Artist.ID is the **release-record source of truth** and the **press kit / submission
sheet generator**. Everything else stays in its existing tool.

## Interactions with other arms

| Arm | Interaction |
|---|---|
| Apparel | When a release has project-specific merch (Grey October apparel drop), the MerchItem records are created in Artist.ID but the production process lives in the Apparel arm. SKUs are CO-owned and route via the CO gather pass. |
| Software | Artist.ID *is* a Software arm product (built to the Railway standard). The Software project owns the build template; this handoff feeds into the spec that gets dropped into the template. |
| Lifestyle | Tangential — ride logs / workout data that informed *nEVER cOAST.* came from Lifestyle-adjacent activity (the Samsung watch). For now, captured as text in Release fields. Future: a Ride record could be linked to a Release. |

## What success looks like for v1

When *gREY oCTOBER* is ready to ship, Dewey can:

1. Open Artist.ID
2. Create a Release record in 5 minutes (manually)
3. Paste Patch Talk tracks (if Patch Talk export is built; otherwise enter manually)
4. Add credits via the existing People registry
5. Upload assets with proper host metadata
6. Render an EPK to brand standards with one click
7. Tweak section visibility / copy for the darker palette
8. Lock and export
9. Generate the DSP sheet with another click
10. Run the rollout from the timeline view

Total time from "ready to plan release" to "EPK live and DSP submitted": **under
two hours**, versus the multi-day session that produced the *nEVER cOAST.* EPK.

---

*Next: `06_OPEN_DECISIONS.md` — what's not decided yet.*
