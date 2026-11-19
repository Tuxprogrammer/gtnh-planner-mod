# AGENTS.md

GTNH Planner: the in-game production planner for GregTech: New Horizons (mod id `gtnhplanner`, package
`com.gtnhplanner`). Minecraft 1.7.10, Forge, client side only, built with the GTNH Gradle convention plugin.

This repository is an independent fork (`Tuxprogrammer/gtnh-planner-mod`) of `jackwrichards/gtnh-planner-mod`, kept
for one private server's client pack. It carries bug fixes on top of upstream and merges upstream regularly. The repo
is public.

This file holds the fork's rules and is loaded with `CLAUDE.md`, which imports it. `CLAUDE.md` and everything under
`docs/` were written upstream: they describe the code, its invariants and the dev harness, and are edited here only
where a passage is about upstream's own machines, repos or release process. Where one of them still describes
upstream's branches, releases or hosting, this file wins.

## Upstream

- Never contact upstream. No issues, pull requests, comments, reviews, discussions or mentions on `jackwrichards/*`,
  and no pushes there. The same goes for the repos further up the fork chain. Reading is fine: fetch, browse issues,
  read code.
- A pushed commit, issue or comment that names an upstream issue as `#123`, `jackwrichards/...#123` or a URL leaves a
  visible backlink on that issue. Refer to one in plain words only: "upstream issue 123". No `@name` mentions
  anywhere.
- Push only to `origin` (`Tuxprogrammer/gtnh-planner-mod`). The `upstream` push URL is disabled in a correct
  checkout; if `git remote -v` shows a real push URL for it, run `git remote set-url --push upstream DISABLED`.
- `gh` on a fork acts on the parent by default. Every `gh` command that writes names
  `--repo Tuxprogrammer/gtnh-planner-mod`. Never run `gh pr create`; this fork uses no pull requests.
- The built mod still talks to upstream's services: the library, accounts, cloud plans and the live service at the
  site in `library/CommunityApi` (`-Dgtnhplanner.library.url` moves all of them), the Feedback box and the crash
  offer through that site's `/api/feedback`, which files a public issue on upstream's tracker, and the update check
  in `client/UpdateCheck`, which reads upstream's releases. Where these point is the owner's decision: do not change
  them unasked, and add no new call to anything upstream runs.
- So a dev game never runs against the real site. Start it with `PLANNH_LIBRARY_URL` set to the local stand-in
  (`node tools/dev/mock-library.mjs`, `tools/dev/mock-live.mjs`). Never press Send in the Feedback box, accept a crash
  offer, sign in, publish or share while the game points at the real site.
- Upstream merges into `main` regularly, so a fix must stay easy to merge over: change the fewest lines that fix the
  bug, in place. No reformatting beyond what `spotlessApply` does to the lines you touched, no renames, file moves or
  drive-by cleanups in files upstream owns. New code goes in new files where that is natural.
- When a merge conflicts, upstream's code wins unless it brings back the bug; then re-apply the fix on upstream's new
  shape. Keep this fork's `AGENTS.md`, its edits to `CLAUDE.md` and its `.github/workflows/` changes.
- When upstream fixes a bug this fork also fixed, take upstream's fix and drop the fork's.

## Working rules

- One branch per fix, `fix/<short-name>`, cut from `main`. Merge it into `main` once it is verified and delete it.
  `main` is what gets handed off, so it stays shippable.
- Commit and push finished work unless told otherwise.
- CI (`.github/workflows/build-and-test.yml`) builds the mod and runs the checks and the full test suite on every
  push to `main` and to `fix/**`. No tolerated failures. Run `./gradlew spotlessApply` before committing: CI checks
  formatting.
- A test that pins player-facing text changes in the same commit as the text.
- Nothing generated goes in the repo root. Screenshots, probe scripts, captures and logs go in the session scratchpad
  or `scratch/` (gitignored); the build and the dev game keep theirs in `build/` and `run/`. Delete screenshots once
  you have looked at them.
- Comments state the rule and why. No dates, names or "used to" stories; git has those.
- The repo is public. No hostnames, addresses, private paths or credentials of the private server, its build machine
  or its planner site in code, docs, commits, issues or this file.
- Setup, fixes and merges only: no refactors and no dependency updates unless the task is one. The pinned mod
  versions in `dependencies.gradle` move only with the target pack.

## Player-facing copy

- Plain and short: the number, the rule, the consequence. No quips, no internal jargon, no em dashes.
- `CLAUDE.md` has the rest: what the UI may name, and how release notes read.

## Target pack

- Only the GTNH 2.9.0 full release is targeted. Betas, release candidates and other pack versions are not: a report
  that only concerns one of them is not worked.
- The pack's own mod versions decide what the code may call. `tools/dev/compat/manifests/` holds each pack build's
  list and `./gradlew compatCheck` checks the jar against them; a fix must keep that check passing.
- Game source for checking a mechanic is the source at the refs the target pack ships, kept by the pack operator's
  tooling outside this repo. Ask for it; do not read a newer public branch and call it truth.

## Versions and release notes

- The build takes the mod's version from git: the nearest tag, plus a commit count and hash when `HEAD` is past it.
  Nothing in the tree holds a version number.
- A fork version is upstream's tag plus a fork suffix: `0.2.0-beta.3-tux.1`, then `-tux.2`. The base is always
  upstream's latest tag that `main` contains; never invent a base number here.
- Tag once per handoff, not per commit: an annotated tag on the handed-off commit of `main`, pushed to `origin`. Build
  the jar at the tag with a clean tree so it carries that version.
- After an upstream merge that brings a new upstream tag, restart the suffix at `-tux.1` on the new base.
- The fork's player notes build up in `.changelogs/unreleased-tux.md` (New / Better / Fixed, one line a change, in a
  player's words): a commit a player would notice adds its line in the same commit. At a handoff, rename it to
  `.changelogs/<fork tag>.md` and start a fresh one. Upstream's `.changelogs/unreleased.md` and its release notes are
  upstream's; leave them as they come.

## Handoffs

- This repo releases nothing and agents here never publish. No GitHub release, no upload to a mod site, no Maven
  publish. The tag-release workflow is upstream's and is disabled on this fork; pushing a tag builds nothing.
- A finished jar is handed to the pack operator (the Dave-Agent project), who puts it in the client pack. Hand over:
  the jar built at the tag, the commit SHA and tag, the notes for players, and anything the operator must do with it
  (a changed config file, a new JVM argument, a changed dependency on a pack mod).
- Batch fixes into one handoff.
- Never test against the private server or its planner site with real accounts.

## Build and test

```bash
./gradlew spotlessApply       # format; CI fails on unformatted code
./gradlew build               # compile, downgrade, checks and the full JUnit suite; jars land in build/libs/
./gradlew test                # the suite alone, headless, no Minecraft
./gradlew compatCheck         # the jar against each pack build's mod versions
```

- Sources are Java 25 syntax, downgraded to Java 8 bytecode for the game. Gradle needs JDK 25 to run and JDKs 8, 17
  and 21 as toolchains; it downloads what is missing unless the installations are listed in the Gradle user home.
- The jar for the pack is `build/libs/gtnhplanner-<version>.jar`. The `-dev`, `-sources` and `-predowngrade` jars are
  not for players.
- If `test` reports `NO-SOURCE`, nothing ran: see Build and test in `CLAUDE.md`.
- The dev game (`tools/dev/mc.sh`) opens a window on the desktop it runs on. Run it only where the owner has said a
  window is welcome, and only against the local stand-ins (see Upstream).
