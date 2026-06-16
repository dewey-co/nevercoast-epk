# 02 — Data Model

> The complete data model derived from building the *nEVER cOAST.* EPK.
> Every entity exists because the EPK exposed it. Every field is one that mattered.

## Entity overview

```
Artist ──┬── Release ──┬── Track ──── Credit (track-scoped)
         │             │
         │             ├── Credit (release-scoped)
         │             ├── Asset (artwork, photo, video, audio, doc)
         │             ├── MerchItem ── MerchVariant
         │             ├── TimelineEvent
         │             ├── Link (external platform)
         │             ├── Tag
         │             └── BrandLock (project-specific overrides)
         │
         └── BrandIdentity (artist-wide locks)

Person ─── Role ─── Credit
```

## 1. Artist

Represents an artist project. There can be more than one (Dewey could add another
artist persona; the system should accommodate).

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `name` | string | The canonical name (e.g. `Samu=L`) |
| `name_platform_safe` | string | Fallback for DSPs that reject special chars (e.g. `Samu-L`) |
| `parent_brand` | string | e.g. `D.E.W.E.Y.` |
| `arm` | enum | `music`, `apparel`, `software`, `lifestyle` |
| `bio_long` | text | Full bio (used in EPK) |
| `bio_short` | text | One-paragraph bio (DSP / playlist pitch) |
| `bio_oneliner` | text | One-sentence bio |
| `home_location` | string | Display only (e.g. `Bowie, MD / DMV`) |
| `references` | string[] | Sonic / artistic references (Alchemist, Ka, Oddisee, Smino) |
| `instagram_artist` | string | Handle |
| `instagram_brand` | string | Handle |
| `bandcamp_url` | url | |
| `youtube_url` | url | |
| `website_url` | url | |
| `contact_email` | email | Press / booking |
| `created_at` | timestamp | |
| `updated_at` | timestamp | |

**Validation rules:**
- `name` is brand-locked. Once set, edits require confirmation.
- `name_platform_safe` is the only acceptable substitution when a platform rejects `name`.
- Bio fields can have draft / locked states.

## 2. BrandIdentity (1:1 with Artist)

Artist-wide brand locks that propagate to every release.

| Field | Type | Notes |
|---|---|---|
| `artist_id` | uuid → Artist | |
| `voice_rules` | json | Documentary / warm-but-gritty / no preaching / etc. |
| `palette_primary` | hex[] | Brand colors |
| `palette_secondary` | hex[] | |
| `typographic_locks` | json | Title-casing rules (inverted case for tracks, periods on titles, etc.) |
| `motif_assets` | url[] | The "3", the bike, etc. |
| `forbidden_terms` | string[] | "rebrand", borrowed-wisdom clichés, etc. |
| `signature_phrases` | string[] | `Do Everything Without Erasing Yourself`, `Keep Pedaling.` |

These should be enforced on every render output. EPK, DSP sheet, social post — all
inherit the same identity.

## 3. Release

The central entity. *nEVER cOAST.* is one Release record.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `artist_id` | uuid → Artist | |
| `project_code` | string | e.g. `D3W3Y-86-MU` |
| `title` | string | e.g. `nEVER cOAST.` |
| `title_typographic` | string | Render form (preserves inverted case + period) |
| `title_platform_safe` | string | DSP fallback (e.g. `Never Coast`) |
| `release_type` | enum | `single`, `ep`, `album`, `compilation`, `mixtape` |
| `release_state` | enum | `concept`, `in_production`, `mastered`, `bandcamp_window`, `dsp_live`, `archived` |
| `bandcamp_drop_date` | date | |
| `dsp_submission_date` | date | |
| `dsp_live_date` | date | |
| `runtime_minutes` | int | Total |
| `genre_primary` | string | |
| `genre_secondary` | string[] | |
| `concept_tagline` | text | One-line description |
| `concept_long` | text | Multi-paragraph project narrative |
| `production_signatures` | text[] | Sonic signatures (GoPro air, watch prompts, etc.) |
| `easter_eggs` | text[] | Hidden details (crash sequence, etc.) |
| `dedication_note` | text | Birthday anchors, family ties, etc. |
| `label` | string | e.g. `D.E.W.E.Y. / Independent` |
| `location` | string | Where it was made |
| `parent_series` | uuid → Release | For trilogies — points to the umbrella release |
| `sequence_in_series` | int | 1, 2, 3 |
| `next_release` | uuid → Release | For "what's next" rollout entries |
| `is_locked` | bool | Hard-lock — no edits without explicit confirmation |
| `notes` | text | Internal notes |

**Validation:**
- `title` and `title_typographic` cannot diverge in meaning, only in formatting.
- `release_state` transitions are unidirectional (can't go from `dsp_live` back to `mastered`).
- When `is_locked` is true, the EPK and other renders should treat the record as final.

## 4. Track

Belongs to a Release. Ordered.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `sequence` | int | 1, 2, 3... |
| `title` | string | Plain |
| `title_typographic` | string | Render form (`fIXED gEAR`, `nEVER cOAST.`) |
| `runtime_seconds` | int | |
| `bpm` | int | |
| `key` | string | Musical key |
| `track_type` | enum | `intro`, `interlude`, `outro`, `standard`, `hidden`, `bonus` |
| `is_instrumental` | bool | |
| `lyrics` | text | (Patch Talk source) |
| `notes` | text | |
| `patch_talk_ref` | string | External ID to Patch Talk record (when integrated) |

## 5. Person

A real human credited on releases. Stored once, referenced many times.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `display_name` | string | e.g. `Danny Fal`, `El Bassface`, `[Daughter 1]` |
| `is_placeholder` | bool | True until a real name is provided |
| `bio` | text | Optional |
| `links` | json | Optional — IG, Bandcamp, etc. |
| `notes` | text | |

## 6. Role

A controlled list of credit roles. Hardcoded with seed values.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `name` | string | Display label |
| `category` | enum | `performance`, `production`, `engineering`, `visual`, `editorial`, `business` |
| `default_scope` | enum | `release`, `track` — whether this role typically applies to the whole release or specific tracks |

**Seed roles** (extracted from EPK):

Performance: Artist, Writer, Narrator, Composer, Featured Artist
Production: Executive Producer, Producer
Engineering: Mixing Engineer, Recording Engineer, Mastering Engineer, Sound Designer
Visual: Designer, Art Director, Photographer, Videographer, Editor, Colorist
Editorial: Stylist, Content Curator
Business: Manager, A&R, Label Contact

## 7. Credit (the join table — the heart of the model)

This is what makes the credit complexity tractable.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `person_id` | uuid → Person | |
| `role_id` | uuid → Role | |
| `scope` | enum | `release`, `track`, `instrument` |
| `track_ids` | uuid[] | Populated when scope = `track` |
| `instrument` | string | Populated when scope = `instrument` (e.g. `bass`) |
| `display_order` | int | For ordering in renders |
| `notes` | text | |

**This is how the EPK's credit complexity gets handled:**
- Samu=L → Release → Executive Producer (release scope)
- Samu=L → Release → Producer (track scope, track_ids = [track1, track8])
- Danny Fal → Release → Producer (track scope, track_ids = [t2, t3, t4, t5, t6, t7])
- El Bassface → Release → Featured Artist (instrument scope, instrument = "bass", track_ids = [t2, t4, t5])
- Daughters → Release → Voice/Vocals (track scope, track_ids = [t1, t8])

A render groups by Person, then collapses scopes into the right phrasing automatically.

## 8. Asset

Files attached to a Release: artwork, photos, video, audio, documents.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `asset_type` | enum | `album_art`, `press_photo`, `promo_video`, `teaser_video`, `audio_stream`, `audio_master`, `lyric_sheet`, `social_still`, `other` |
| `file_path` | string | Local repo-relative path when self-hosted |
| `hosted_url` | string | Direct URL when externally hosted |
| `host_provider` | enum | `self_hosted`, `dropbox`, `youtube`, `bandcamp`, `vimeo`, `cloudinary`, `box`, `google_drive` |
| `embed_url_pattern` | string | The transformation needed for embed (e.g. `dl=0` → `dl=1` for Dropbox) |
| `aspect_ratio` | enum | `1:1`, `16:9`, `9:16`, `4:5`, `other` |
| `dimensions_w` | int | |
| `dimensions_h` | int | |
| `caption` | text | |
| `alt_text` | text | |
| `version` | string | e.g. `v1.0`, `final` |
| `is_primary` | bool | Per asset_type — the one to use by default |
| `is_draft` | bool | |
| `created_at` | timestamp | |

**Critical:** the `host_provider` + `embed_url_pattern` fields exist because the EPK
build hit real-world embed failures. The model has to know that Dropbox needs `dl=1`,
that YouTube Shorts need embedding enabled in YouTube Studio, that Box blob URLs
don't work externally. Future renders should auto-resolve embed-friendly URLs.

## 9. MerchItem

Physical merchandise tied to a Release.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `item_type` | enum | `apparel`, `print`, `physical_media`, `accessory`, `bundle` |
| `name` | string | e.g. `Embroidered Workout Towel` |
| `description` | text | |
| `total_units` | int | Run size |
| `is_hand_numbered` | bool | |
| `is_artist_made` | bool | |
| `price_usd` | decimal | |
| `sku_pattern` | string | The SKU format (e.g. `D3W3Y-86-MU-NC-PR-01-[UNIT]-V[VARIANT]`) |
| `bandcamp_listing_url` | url | |
| `production_status` | enum | `concept`, `in_production`, `complete`, `sold_out` |
| `primary_image_asset_id` | uuid → Asset | |
| `notes` | text | |

## 10. MerchVariant

Variants of a MerchItem (the 5 collector print variants).

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `merch_item_id` | uuid → MerchItem | |
| `variant_code` | string | A, B, C, D, E |
| `variant_name` | string | e.g. "Tunnel / Rings" |
| `aspect_ratio` | string | e.g. `9:16`, `16:9`, `1:1` |
| `unit_count` | int | Units in this variant |
| `dimensions` | string | e.g. `5.25 × 8.5"` |
| `image_asset_id` | uuid → Asset | |
| `notes` | text | |

## 11. TimelineEvent

The rollout timeline — release-scoped events.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `event_date` | date | |
| `event_type` | enum | `bandcamp_drop`, `dsp_submission`, `dsp_live`, `promo_post`, `merch_reveal`, `interview`, `live_show`, `next_release_teaser`, `other` |
| `event_title` | string | Display |
| `event_note` | text | Sub-detail |
| `state` | enum | `past`, `active`, `upcoming`, `future` (computed from date + today) |
| `assets` | uuid[] → Asset | Optional — assets associated with the event (a post's video, a flyer, etc.) |
| `caption` | text | For promo posts |
| `hashtags` | string[] | For promo posts |
| `posted_url` | url | After posting, where it lives |

## 12. Link

External platform links for a release.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `platform` | enum | `bandcamp`, `spotify`, `apple_music`, `youtube_music`, `tidal`, `amazon_music`, `instagram_post`, `tiktok_post`, `other` |
| `url` | url | |
| `is_primary` | bool | The lead link for this platform |
| `is_live` | bool | False until the release actually drops there |
| `display_label` | string | e.g. `samu-l.bandcamp.com` |
| `display_order` | int | |

## 13. Tag

Genre / mood / category tags.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `tag_value` | string | e.g. `Lofi`, `Hip Hop`, `Boom Bap`, `Maryland`, `DMV`, `EP`, `Self-Produced`, `Independent`, `Mellow` |
| `tag_category` | enum | `genre`, `mood`, `geography`, `style`, `format` |

## 14. BrandLock (project-specific overrides)

When a release has a darker / different aesthetic than the artist's default (Grey
October will), it gets project-specific brand locks.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `release_id` | uuid → Release | |
| `lock_type` | enum | `palette`, `typography`, `voice`, `motif` |
| `override_value` | json | The override |
| `notes` | text | Why this differs from artist default |

## Cross-cutting requirements

### Versioning

Every text field on Release, Track, Artist (bio), and brand fields should support
draft / locked states. Locked content is what renders; draft content is in flight.

### Audit trail

Every change to a locked field should be logged with who / when / old / new. This
came up because the EPK had decisions reopened multiple times — the system should
make it visible when a "locked" decision actually changes.

### Source attribution

When a field is populated from Patch Talk or another external source, the source
should be recorded. Manual entries should also be flagged as such.

### Render targets

The data model exists to support multiple render targets. Each render target is
basically a templated view over the data:

- **EPK** (web HTML, the *nEVER cOAST.* artifact)
- **DSP Submission Sheet** (metadata for LANDR or equivalent)
- **Playlist Pitch One-Pager** (PDF or web)
- **Sync Licensing Brief** (PDF — for film/TV music supervisors)
- **Bandcamp Listing Copy** (markdown)
- **Merch Listing Copy** (per item)
- **Social Caption Bundle** (per timeline event)

The data model supports all of these because all of these were implicitly assembled
during the EPK build.

---

*Next: `03_EPK_AS_OUTPUT.md` — the EPK as the first render target Artist.ID must produce.*
