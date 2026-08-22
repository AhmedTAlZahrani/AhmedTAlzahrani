# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repository.

## What this is

This is the **GitHub profile repository** for `AhmedTAlZahrani` — the repo whose
`README.md` renders on <https://github.com/AhmedTAlZahrani>. It is not an
application. It contains prose, one generator script, and one workflow.

```
README.md                             # the profile page itself
scripts/update_readme.py              # regenerates the projects catalog in README.md
.github/workflows/update-profile.yml  # runs the script and commits the result
```

Note the casing mismatch: the remote repo is `AhmedTAlZahrani/AhmedTAlzahrani`
(lowercase `z` in the repo name, capital `Z` in the owner). `PROFILE_REPO_NAME`
in `scripts/update_readme.py` must match the **repo name** spelling
(`AhmedTAlzahrani`) or the profile repo will list itself in its own catalog.
`GH_USER` uses the **owner** spelling (`AhmedTAlZahrani`).

## The generated block — read this before editing README.md

`README.md` is part hand-written and part generated. Everything between

```
<!-- PROJECTS:START -->
...
<!-- PROJECTS:END -->
```

is produced by `scripts/update_readme.py` and **will be overwritten** on the
next run. Do not hand-edit inside the markers, and do not remove or reword the
markers — `splice()` locates them with a bare `str.index()` and will raise
`ValueError` if either is missing.

Everything outside the markers (the intro, "How I think about this work",
"Tools I work with", "Contact") is hand-written and safe to edit directly.

## How the generator works

`scripts/update_readme.py`:

1. `fetch_repos()` — GETs `https://api.github.com/users/{GH_USER}/repos`
   (`per_page=100`, public only). `GITHUB_TOKEN` is optional and only raises the
   rate limit. Only `requests` is needed; there is no requirements file, so the
   workflow does a bare `pip install requests`.
2. Filters out forks, archived repos, and the profile repo itself.
3. `build_block()` — groups repos using the hard-coded `CATEGORIES` dict
   (*Saudi Transport & Infrastructure*, *NEOM Smart City*, *ML & Data Science*,
   *Early Projects*), preserving the order the names are listed in. Anything
   public that isn't in `CATEGORIES` lands in a trailing **Other** section,
   sorted case-insensitively by name.
4. `splice()` — replaces the marker block, re-inserting the
   "Auto-generated … do not edit by hand" comment header.
5. Writes only if the text actually changed; prints `no changes` to stderr
   otherwise.

Descriptions in the table come from each repo's **GitHub description field**,
not from this repo. To change a project's blurb, edit the description on that
repo and re-run the workflow — editing the table here is pointless, it will be
reverted. A repo with no description renders as `—`.

**Adding a project**: add its name to the right list in `CATEGORIES`. Leaving it
out is not an error — it just falls into *Other*.

Run it locally with:

```bash
GH_USER=AhmedTAlZahrani python scripts/update_readme.py
```

It writes to `README.md` in place, so check `git diff` afterwards. There are no
tests and no CI for this repo; the generator's correctness is verified by
reading the diff.

## The workflow

`.github/workflows/update-profile.yml` runs on `workflow_dispatch` and on a
`repository_dispatch` with type `refresh` — **not** on push and not on a
schedule. It needs `contents: write`, runs the script with the repo's
`GITHUB_TOKEN`, and commits any README change as `github-actions[bot]` with the
message `chore: refresh project list`. If the README is unchanged it exits 0
without committing.

Bot commits land on `main` directly. When you branch off `main`, be aware a bot
refresh may have moved it since.

## Conventions

- **Voice.** The hand-written prose is first-person, plain, and specific —
  short declarative sentences, concrete examples, no marketing adjectives, no
  emoji, no badge rows. Lines are hard-wrapped at roughly 72 characters. Match
  all of that. If asked to add a section, write it in that register rather than
  a generic profile-README template.
- **Python style** in `scripts/`: module docstring explaining how to run the
  file, small single-purpose functions, `pathlib` for paths, progress messages
  to `stderr` via `print(..., file=sys.stderr)` so stdout stays clean.
  Configuration lives in module-level constants at the top, not scattered
  inline.
- No linter, formatter, or type checker is configured. Match the surrounding
  style by hand.

## Git workflow

- Default branch: `main`. Remote: `https://github.com/AhmedTAlZahrani/AhmedTAlzahrani`.
- Commit messages are short lowercase imperatives ("trim gitignore",
  "chore: refresh project list", "rewrite profile README and add auto-update
  workflow").
- Work on a feature branch and push with `git push -u origin <branch>`.
- Only open a pull request when explicitly asked.
- Anything committed here is public and renders on the owner's profile page.
  Treat every change as published content.
