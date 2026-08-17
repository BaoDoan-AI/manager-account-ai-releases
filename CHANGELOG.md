# Changelog

What changed in each published version of **Manager Account AI**. Newest
first. Written by the release workflow, which inserts one section directly below
the marker line — do not remove it, and do not reorder what is under it.

<!-- releases -->

## v0.2.1 — 2026-08-17

### Added

- **T60** — `npm run verify:token` answers, from a laptop, the question that cost
  the v0.2.0 release two minutes and twenty seconds of building before GitHub
  refused the push: can `RELEASES_TOKEN` publish to the artifact repository?

  The write check is not an inference about permissions. It sends the request
  `git push` sends first — `GET /<repo>.git/info/refs?service=git-receive-pack` —
  which is the one that answers 403 for a token without write access, and which
  pushes nothing. On a refusal it prints the four things to check on the token, in
  the order they are most often wrong.

  It also compares `build.publish` in `package.json` against `env.RELEASES_REPO`
  in the release workflow. Those two are what the installed app reads and what CI
  writes; if they disagree, every release lands somewhere nothing ever looks. That
  half needs no token and runs without one.

  The token is never printed — last four characters, in output and errors alike.

### Changed

- **T59** — This repository publishes before the public one, reversing what T55
  set up. The argument for going public first was that a failure there, after the
  private release had landed, leaves an installer people can reach and nothing can
  update into. T58 removed that argument in the same round it was written: the
  token failure that motivated it is now caught before the build, so it can no
  longer reach either publish step.

  What is left points the other way. The private release is one API call on a
  token that is always valid and never waits on an organisation's approval;
  everything fragile — an external token, a clone, a commit, a push — sits in the
  step after it. Put the reliable one first and *was this version released at all*
  has an answer even when the second step never runs.

### Fixed

- **T58** — A token that cannot write to the releases repository now fails the
  run in seconds instead of after the build. The job opens with a
  `git push --dry-run` against that repository, which authenticates and asks the
  server for write access without pushing anything, and reports what to check on
  the token when it is refused. Before this, the same refusal cost two and a half
  minutes of building first.

- **T57** — The release workflow's one ordering guarantee was not enforced. It
  publishes the changelog commit *before* creating the release, so the tag names
  the commit it describes — but a failed `git push` did not stop the step, and
  `gh release create` ran anyway. The v0.2.0 run proved it: the push was refused
  with a 403 and the release attempt went ahead regardless.

  Nothing was published that time, because the token could not create a release
  either. Had the push failed for a passing reason instead, the result would have
  been a release whose tag names a commit that never left the runner — the exact
  thing the ordering exists to prevent.

  The cause was known and written down one comment away: pwsh does not turn a
  native command's non-zero exit into an error. Every `git` call in that step now
  goes through a wrapper that throws.
