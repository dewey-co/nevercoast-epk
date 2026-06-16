# 07 — Claude Code Processing

> How to hand this package off to Claude Code for repo commit. Includes the exact
> prompt patterns and the file structure that should land in `dewey-enterprise`.

## What Claude Code is being asked to do

Take the seven markdown files in this handoff package and commit them into the
`dewey-enterprise` repo at a specific location, with a specific commit message,
following the version-control conventions established by the master strategy doc.

This is **not** a code-writing task. It's a structured file-commit task with some
light directory creation.

## Target file structure

After Claude Code finishes, the repo should look like this:

```
dewey-enterprise/
├── docs/
│   ├── (existing docs at this level — DEWEY_MASTER.md, briefs, etc.)
│   └── artist-id/
│       ├── 00_README.md
│       ├── 01_CONTEXT_AND_INCEPTION.md
│       ├── 02_DATA_MODEL.md
│       ├── 03_EPK_AS_OUTPUT.md
│       ├── 04_PATCH_TALK_INTEGRATION.md
│       ├── 05_WORKFLOW_END_TO_END.md
│       ├── 06_OPEN_DECISIONS.md
│       └── 07_CLAUDE_CODE_PROCESSING.md
└── assets/
    └── music/
        └── epk-reference/
            └── nevercoast_epk_v8.html
```

The `assets/music/epk-reference/` directory should hold the working EPK as the
reference artifact alongside the spec.

## Exact prompt to give Claude Code

Paste this verbatim into a Claude Code session. The placeholder `<HANDOFF_PATH>`
is wherever you've saved this handoff package locally.

```text
I have an Artist.ID specification handoff package at <HANDOFF_PATH> consisting of
8 markdown files (00_README.md through 07_CLAUDE_CODE_PROCESSING.md) plus a
reference EPK HTML file at <PATH_TO_NEVERCOAST_EPK_V8.html>.

Please commit them into the dewey-enterprise repo following these rules:

1. Create directory `docs/artist-id/` if it does not exist
2. Copy all 8 markdown files from the handoff package into `docs/artist-id/`,
   preserving their numbered prefixes and filenames exactly
3. Create directory `assets/music/epk-reference/` if it does not exist
4. Copy `nevercoast_epk_v8.html` into `assets/music/epk-reference/`
5. Do not modify any file contents during the copy
6. Do not generate any additional files
7. Verify the file list after copying — all 8 markdown files + 1 HTML file
   should be present at the target paths

Commit the changes in a single commit with this message:

    docs(artist-id): add inception spec handoff package

    Captures Artist.ID specification derived from building the nEVER cOAST. EPK
    as the inception use case. Includes data model, EPK output spec, Patch Talk
    integration approach (v1 = consume + manual), end-to-end workflow, and 10
    open decisions for resolution before the build spec is written against
    APP_SPEC_TEMPLATE.md.

    Reference artifact: nevercoast_epk_v8.html committed to
    assets/music/epk-reference/.

Then push to the main branch.

Do not ask for confirmation on any step. Proceed with the full sequence and
report when complete.
```

## Why this prompt works

A few details worth noting in case the prompt needs to be adapted:

- **Verbatim file paths** — Claude Code is told exactly where to put things. No
  inference about "the right place" — the right place is named.
- **No modification clause** — explicit "do not modify any file contents" prevents
  Claude Code from helpfully reformatting markdown.
- **Single commit** — keeps git history clean. The whole handoff is one logical
  change.
- **Verification step** — Claude Code is told to check its own work.
- **No questions** — the request is complete; Claude Code shouldn't pause to confirm.

## After the commit lands

Once the commit is in, the next moves:

1. **Resolve open decisions** (file 06). Each decision needs a status flip from
   `OPEN` or `LEANING` to `DECIDED`.
2. **Update file 06 with resolutions** — small follow-up commit:
   `docs(artist-id): resolve open decisions D1-D10`
3. **Fill in `APP_SPEC_TEMPLATE.md`** for Artist.ID, drawing content from this
   handoff package. New file at `docs/software/specs/artist-id.md` or wherever the
   Software project's specs live.
4. **Tag the handoff** as the spec inception:
   `git tag artist-id-spec-inception` so the commit is findable.

## Variant prompts

### If the repo doesn't exist yet locally

```text
[Same prompt as above, but add this line at the top:]

If dewey-enterprise is not cloned locally, clone it from
git@github.com:[your-username]/dewey-enterprise.git first, then proceed.
```

### If you want a PR instead of direct commit to main

```text
[Same prompt as above, but replace the commit/push instructions with:]

Create a branch named `feat/artist-id-handoff`, commit the changes there with
the message above, push the branch, and open a pull request to main with the
commit message body as the PR description.
```

### If you want to also update the master doc's open decisions

```text
[After the main prompt:]

After the commit lands, also update docs/DEWEY_MASTER.md by:
1. Adding the following row to the §12 Open Decisions table:
   "- [ ] Artist.ID spec resolution — see docs/artist-id/06_OPEN_DECISIONS.md"
2. Adding this entry to the §14 Changelog:
   "| 1.0.3 | YYYY-MM-DD | Artist.ID inception handoff added at docs/artist-id/. |"
   (Use today's date in YYYY-MM-DD format.)
Commit those changes separately with the message:
"docs(master): reference Artist.ID handoff package"
```

## Sanity check before handing off

Run through this list before pasting the prompt:

- [ ] All 8 markdown files are present in the handoff package locally
- [ ] `nevercoast_epk_v8.html` is at a known local path
- [ ] You know the local path to the dewey-enterprise repo (or want Claude Code to clone)
- [ ] You've replaced `<HANDOFF_PATH>` and `<PATH_TO_NEVERCOAST_EPK_V8.html>` with real paths
- [ ] You're authenticated to push to the repo (SSH key, gh CLI, etc.)
- [ ] You actually want this in `main` (or you've adapted to PR variant)

## Why this is in a separate file

Two reasons:

1. **Operational vs informational** — files 00–06 are the specification. File 07 is
   the operational handoff. Mixing them clutters the spec.
2. **Future reuse** — when the *gREY oCTOBER* spec gets written, a similar handoff
   pattern can reuse this file as a template.

---

*End of handoff package.*

*D.E.W.E.Y. — Do Everything Without Erasing Yourself*
