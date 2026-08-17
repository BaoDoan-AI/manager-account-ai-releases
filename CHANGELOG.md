# Changelog

What changed in each published version of **Manager Account AI**. Newest
first. Written by the release workflow, which inserts one section directly below
the marker line — do not remove it, and do not reorder what is under it.

<!-- releases -->

## v0.2.3 — 2026-08-17

### Changed

- **T80** — `.claude/gitconfig.yml` is no longer tracked. It holds `auto_push`
  and `protected_branches` for the local commit tooling — per-developer
  behaviour, not a property of the repository — and it had been sitting modified
  and uncommitted across four merges, which is exactly what `scripts/release.mjs`
  refuses to start on.

  Gitignoring alone would not have helped: the file was tracked, so the working
  tree would have stayed dirty. `git rm --cached` untracks it while leaving the
  local copy in place, so nothing changes on this machine.

  The consequence, stated: a fresh clone has no such file, and
  `apply_commits.py` then creates commit-only defaults — `auto_push: false`.
  Failing toward "does not push" is the right direction, but a new machine has
  to opt back in deliberately.

  `.claude/commit-backup.json` is ignored alongside it; it is a crash-recovery
  scratch file the commit tooling deletes on success.

### Documentation

- **T79** — The bank describes an app with no proxy and a feature called the
  server. `behavior/relay-server.md` → `behavior/server.md`; fourteen docs
  touched; `overview.md` and `memory.md` regenerated — 30 docs, validate strict
  PASS.

  Two stale *facts*, not just stale prose, and both would have misled the next
  reader: `data/claude-code-files.md` still pointed its `source:` at
  `claude-settings.ts` and its own deleted test file, and
  `rule/testing-conventions.md` still listed that module as covered. The same
  doc also claimed this app owns "four keys inside `env`" of
  `~/.claude/settings.json` — it now owns **nothing** there and only ever copies
  the file.

  `interface/claude-code-io-api.md` lost its whole `## claude-settings.ts`
  section and went from three modules to two.

  The proxy verification gap is recorded as **closed by deletion**, in those
  words, with a strikethrough rather than being removed. It was the bank's
  oldest open item across four rounds; a gap that vanishes silently reads like
  it was solved.

  Three new known issues in `progress.md`: the vault's dead-key list is now up
  to eleven, `profiles.ts` still propagates proxy keys nothing can edit, and
  `NODE_EXTRA_CA_CERTS` has no replacement. A fourth records that the rename
  turned a running server off — observed by launching the app, not predicted.

  `active-context.md` gets the through-line the four rounds actually share:
  **a capability nobody has watched work is a liability, not an escape hatch.**
  The gateway's return as the server is the same rule producing the opposite
  answer once a need existed, not a reversal of its deletion.

  A blanket `sed` rewrote the historical branch name `feat/relay-server` into
  nonsense inside `active-context.md`. Caught by reading the output. A rename
  pass cannot be trusted on prose, and history in particular must keep the names
  it actually had.

### Changed

- **T78** — The relay is the **server**. `src/main/relay.ts` →
  `src/main/server.ts`, `tests/relay.test.ts` → `tests/server.test.ts`,
  `RelayPage.tsx` → `ServerPage.tsx`, and every identifier, settings key, IPC
  channel value, DOM id, log prefix and user-visible string with it. `rg -in
  relay src/ tests/` is silent.

  It was always a server — the app's one inbound surface, the only direction
  nothing else in it points — and now the name says so. One commit rather than
  two, because splitting the main process from the renderer would have left a
  failing typecheck in between.

  `/_relay/health` → `/_server/health`. `'relay:status'` / `'relay:new-token'` →
  `'server:*'`. The `netsh` rule the page prints is now named
  `"Manager Account AI server"`, so a machine that already ran the old one has a
  stale rule for the same port — harmless, but it is there.

  **The three settings keys renamed with no migration**, as chosen:
  `relayEnabled` / `relayPort` / `relayToken` → `server*`. Confirmed by running
  it: the app came up with the server **off**, the old keys sitting in the vault
  as dead JSON. Ticking *Serve other machines* and pressing Save minted a fresh
  token — the first time that path has actually run, since before this the token
  always existed already. The vault's dead-key list is now eleven.

  Verified against the running app: sidebar reads `Accounts | Usage | Server |
  Settings`, the badge goes `Server off` → `Server :8787`,
  `/_server/health` answers 200 with the token and 401 without it over both
  loopback and the LAN address, and `/_relay/health` is gone — it now falls
  through to the upstream forward like any other unknown path.

  One string the mechanical pass got wrong and a human had to fix: the checkbox
  read "Serve the server". It says "Serve other machines".

### Removed

- **T77** — The Claude Code proxy. Gone: `src/main/claude-settings.ts`, its 19
  tests, `ProxyPage.tsx`, the `proxy:get` / `proxy:set` / `proxy:clear` channels
  and their handlers, `ProxySettings`, the three preload methods, and the Proxy
  entry in the sidebar.

  The reason is the one this branch line has used three times already: the
  memory bank's oldest open item was that **no value this feature wrote had ever
  been watched working against a real proxy**. A capability nobody has confirmed
  is a liability, not an escape hatch. Deleting it closes that gap by deletion,
  which is at least an honest way to close it.

  `claude-settings.ts` died whole. Its one export that was not about the proxy,
  `claudeSettingsPath`, turned out to be called only by the three proxy handlers
  and its own test, so nothing had to be rehomed. `homedir` left `ipc.ts` with
  them — the deletion made it unused.

  What is **lost**, said plainly: `NODE_EXTRA_CA_CERTS` has no replacement. A
  machine behind a TLS-inspecting corporate proxy now edits
  `~/.claude/settings.json` by hand, or Claude Code fails its handshake with no
  hint from this app.

  What is **not** fixed by this: `profiles.ts` still blind-copies
  `~/.claude/settings.json` into every launcher profile, so proxy variables an
  older build or the user's own hand left in that file keep being mirrored. The
  app stops managing the proxy; it does not stop carrying it. The comment there
  now says so. `tests/profiles.test.ts` keeps its `HTTPS_PROXY` fixture on
  purpose — it asserts mirroring, not the proxy, and swapping the key would be
  churn.

  Two comments in `ipc.ts` still contain the word and are correct: `net.fetch`
  really does resolve the machine's own proxy, and a proxy really is one reason
  the login window can fail.

### Documentation

- **T76** — The memory bank describes an app that serves as well as consumes.
  New `behavior/relay-server.md`; six docs touched; `overview.md` and
  `memory.md` regenerated — 30 docs, validate strict PASS.

  The correction that mattered most was in `behavior/account-rotation.md`. It
  stated flatly that the live-429 fast path "left with the gateway" and that the
  poller was the only source. T74 made that false, so the section is rewritten
  as two sources with the `resetsAt` / `expiresAt` split spelled out — the first
  is never fabricated because the UI shows it, the second is what stops a stale
  429 benching an account after its window reopened.

  `active-context.md` had framed the whole branch as subtraction. Rather than
  recasting the removal as a mistake, the entry keeps both halves of one
  argument: the gateway was deleted for want of a confirmed reason, the reason
  arrived, and the code came back from git history with its guard replaced and
  its verification done as it landed. That is now an active decision in its own
  right — deleting and restoring cost less than a feature flag, and left no dead
  branch in between.

  What is *not* claimed: `progress.md` records the relay as verified by `curl`
  over loopback and this machine's LAN address, and explicitly **not** from a
  second machine. The `netsh` rule has never been run and the one attempted
  round trip to Anthropic died on the active account's dead refresh grant. Four
  ceilings are recorded as known issues rather than argued away: plain HTTP, one
  shared account and quota, a plaintext `relayToken`, and in-memory 429 readings.

- **T75** — The *Relay* page, between Proxy and Settings, plus a `Relay :<port>`
  badge in the StatusBar so a port handing out a live credential is never
  serving unnoticed from any page.

  Four cards. The state and the port. The access token, with *Regenerate* and
  the warning that it cuts off every machine still holding the old one. The
  lines to run on the other machine, one for Command Prompt and one for
  PowerShell, filled in from `RelayStatus.lanUrls` — with a picker when this
  machine is on more than one network, and a warning instead of a snippet when
  it is on none. And the `netsh` rule, with the reason it is printed rather
  than run: silently opening a port to the network is not something software
  should do behind the user's back, and without it the other machine just times
  out with nothing to go on.

  Enabling the relay with no token mints one first rather than surfacing main's
  refusal — there is no decision there for the user to make.

  A last card says what is actually being turned on: a live credential attached
  to any request carrying the token, plain HTTP so the token and every prompt
  cross the network unencrypted, and one account and one quota shared by every
  client with a rotation here changing all of them mid-session.

  One defect found by looking at the running page rather than the diff: the
  token field was masked while the same secret was spelled out in full in the
  two setup lines below it. Masking that does not mask is theatre, so the
  switch is now page-wide — the field and both lines hide together, and Copy
  still puts the real value on the clipboard.

- **T74** — The relay is wired into the app: `syncRelay()` in `ipc.ts`, started
  at boot next to the updater, stopped on `will-quit`, and re-synced by
  `settings:set` whenever `relayEnabled`, `relayPort` or `relayToken` changes.
  The re-sync is forced rather than compared by port, because the token is
  captured at bind time and a port comparison would miss a token that changed
  under an unchanged port.

  `relay:new-token` mints 24 random bytes as base64url and persists them. It
  restarts a running relay on purpose: every client holding the old token is cut
  off the moment the user regenerates. The renderer never chooses this value —
  it is the only lock on a port that carries a live credential.

  `RelayStatus.lanUrls` comes from `os.networkInterfaces()`, IPv4 and
  non-internal only. Loopback is deliberately excluded: it is the one address
  that works here and nowhere else, so printing it in the list a user copies to
  another machine would be the most misleading thing the page could do.

  **Rotation gets its fast path back.** `410e570` recorded losing it when the
  gateway went: a 429 on a real request used to override the poller immediately,
  and after the removal an account out of quota sat unnoticed for up to one poll
  interval — five minutes by default. `relayRateLimits` restores it, entries
  expiring after their reset or after five minutes when upstream named no time,
  and `accountsRemove` clears an account's entry so a re-imported id does not
  arrive already benched.

  Verified against the running app, not just the suite: `npm run dev` with the
  relay on, `/_relay/health` answered 200 with the token and 401 without it,
  over loopback and over the machine's LAN address alike. The log line for the
  refusal names the path and no token.

- **T73** — `src/main/relay.ts`: an HTTP port other machines point
  `ANTHROPIC_BASE_URL` at. A request arrives with the shared token, this process
  attaches whichever account is active, forwards to Anthropic and streams the
  answer back. It is `src/main/gateway.ts` from T63/T64, restored from history
  and opened up — the per-request credential swap, the refresh-when-stale path,
  the chunk-by-chunk stream and the `parseRateLimitHeaders` read are unchanged,
  because they were right.

  Two things did change. It binds `0.0.0.0` rather than `127.0.0.1`, which is
  the whole point: a machine that is not this one has to be able to reach it.
  And the loopback `Host` guard is **replaced**, not joined, by a mandatory
  shared token. That guard existed for DNS rebinding — a page resolving to
  127.0.0.1 reaches a loopback port from the user's own browser — and the token
  covers the same attack better, because such a page cannot read it. It also
  covers the case the Host guard never had to: the neighbour on the LAN.

  The comparison is `timingSafeEqual` behind a length check, not `===`. The
  length check is not decoration: `timingSafeEqual` *throws* on unequal
  buffers, so without it a wrong-length guess becomes a 500 that tells the
  attacker the length was wrong. A test pins that.

  The token stops at this hop. `authorization` and `x-api-key` were already
  stripped from what gets forwarded; a test now asserts the relay's own secret
  never appears in the request to Anthropic.

  16 tests in `tests/relay.test.ts`, over a real socket. The streaming one
  counts `data` events rather than comparing the joined body — a relay that
  buffered would pass the body assertion and fail the users.

  Not encrypted, and said out loud in the module header: plain HTTP, so the
  token and every prompt cross the LAN in the clear.

- **T72** — The settings, the status shape and the two channel names the relay
  will need. `Settings` gains `relayEnabled`, `relayPort` (8787) and
  `relayToken`; `RelayStatus` carries `running`, `port`, `baseUrl`, `lanUrls`
  and `error`; `IPC.relayStatus` and `IPC.relayNewToken` join the map.

  `relayStatus` is one name in both directions, the way `gateway:status` was:
  the renderer invokes it on mount, main pushes the same shape on it when the
  relay starts, stops or fails. A second broadcast-only channel would have been
  a second name for one fact.

  Two refusals in `store.ts setSettings`, not in the form. A relay port outside
  1..65535 throws, and `relayEnabled: true` with an empty `relayToken` throws —
  the relay binds every interface and attaches a live Claude credential to
  whatever it accepts, so an unlocked port must not be reachable through a bad
  `settings:set`. The renderer is a trust boundary.

### Fixed

- **T67** — Three stale claims in `docs/memory-ai/progress.md`. The installer has
  been per-machine since T61, not per-user. The public releases repository and
  `RELEASES_TOKEN` were listed as not yet existing; both exist and `v0.2.1` and
  `v0.2.2` are published there — what is actually true is narrower and worth
  keeping, that `v0.2.0` has no release page in either repository because its
  publish failed on a 403 and a published tag is never moved. And with two
  releases in the public repository the self-update loop is exercisable for the
  first time, which is now recorded as open work rather than as done.

### Documentation

- **T71** — The memory bank and the README match an app with one proxy and no
  tunable Anthropic addresses. Eleven durable docs touched, `overview.md` and
  `memory.md` regenerated: 29 docs, validate strict PASS.

  Two docs the plan listed were opened and **left alone**, which is the useful
  part of the result: `data/claude-code-files.md` and
  `behavior/account-switching.md` both talk about "the proxy this app writes into
  `~/.claude/settings.json`" — that was always Claude Code's proxy, so every
  sentence survived the deletion word for word.

  Where the removed thing carried a reason, the reason had to be re-homed rather
  than deleted with it. `behavior/app-lifecycle.md` used to flag that
  `applyProxy()` was not awaited; the flag is gone and the paragraph now says why
  there is no ordering left to get wrong. `behavior/app-update.md` explained the
  `electron-updater` partition through the proxy that had to be pushed onto it; it
  now explains the partition and records that nothing needs to reach into it.
  `interface/anthropic-endpoints.md` states the cost of fixed addresses instead of
  claiming an override that no longer exists.

  `README.md` also loses its "in development, visible but not runnable yet"
  paragraph — a stale claim from before T65, advertising the deleted gateway. The
  proxy it mentioned in the same breath is now listed under what the app does.

- **T66** — The memory bank matches an app with no gateway. Two docs deleted
  (`interface/gateway-api.md`, `behavior/gateway-request-flow.md`), fifteen
  rewritten, `overview.md` and `memory.md` regenerated: 29 durable docs, validate
  strict PASS.

  Not a find-and-delete pass. Where the gateway explained *why* something is the
  way it is, the reason had to survive without it — `account-rotation.md` now
  states plainly that the poller is the only source and what that costs in
  latency, and `account-switching.md` says there is no longer any way to land a
  switch inside a running Claude Code session. `ui-conventions.md` keeps both
  work-in-progress patterns as history and extracts the rule that outlived them:
  a flag is deleted by the change it was waiting for.

### Removed

- **T69** — The three Anthropic addresses are constants in the app, not settings.
  `usageEndpoint`, `oauthTokenUrl` and `oauthAuthorizeUrl` are gone from
  `Settings` and `DEFAULT_SETTINGS`, the URL-validation loop in `store.ts` goes
  with them (nothing left in `Settings` is a URL), `oauthOverrides()` is deleted
  along with both call sites, the poller's `getEndpoint` returns
  `ANTHROPIC_USAGE_URL` outright, and the Settings page loses its "Usage endpoint"
  field.

  The two OAuth keys had no UI at all — they arrived with the gateway, to point a
  login at a stand-in server, and nothing has pointed one anywhere since. The
  usage endpoint had a field for a real reason, recorded in the comment above it:
  the address is undocumented, so the day Anthropic moves it every account reads
  as "no figures". That reason has not gone away — it has been accepted. A moved
  endpoint now costs a build rather than a setting, and the comment above
  `ANTHROPIC_USAGE_URL` says so.

  What is deliberately **kept**: the `tokenUrl?` / `authorizeUrl?` parameters on
  `refreshAccessToken`, `exchangeAuthorizationCode`, `startLoopbackLogin` and
  `startManualLogin`, and `getEndpoint` on the poller. Those are injection points
  the tests use to stand up a local server — 25 tests across `tests/oauth.test.ts`
  and `tests/usage.test.ts` pass a URL through them. Only the settings plumbing
  that read from the vault is gone.

- **T68** — The app's own proxy is gone: the `proxyRules` / `proxyBypass` settings,
  `applyProxy` / `applyProxyTo`, the bootstrap step in `index.ts`, the
  `applyProxyTo(loginSession)` call, the `settings:set` re-apply branch, and the
  `UPDATER_SESSION_PARTITION` constant that existed only to be given a proxy. The
  Proxy page keeps one section instead of two and loses its second Save button.

  There were two proxies and only one of them did anything. Claude Code's proxy
  makes the CLI reachable from behind a corporate proxy; the app's was a second
  set of fields for the same job, aimed at the app's own requests, and it had
  never been verified against a real proxy.

  **This is a behaviour change, not only a deletion.** With the field empty,
  `applyProxy` pushed `{ mode: 'direct' }` onto every session — which
  deliberately *ignored* whatever proxy the machine was configured with. Removing
  it hands proxy resolution back to Chromium's default, so on a machine behind a
  proxy the usage poller, the sign-in window and the update check now go through
  it instead of being forced past it. Better on such a machine, and identical on
  one connecting directly.

  Dropping the bootstrap step also closes the race the memory bank recorded: the
  proxy was applied with `void`, not `await`, so a request could reach the network
  before it landed. There is no step left to lose.

  A vault already on disk keeps its two `proxy*` values as dead JSON, the same way
  the `gateway*` values were left in T65. Forgetting a proxy the user set is what
  removing the feature means; no migration is added for a file the user owns.

- **T65** — The gateway is gone: `src/main/gateway.ts`, its page, its 17 tests,
  the `gateway:status` channel and both preload methods, the StatusBar badge,
  and the `gatewayEnabled` / `gatewayPort` / `gatewayUpstream` settings with
  their validation in `store.ts`.

  It was a loopback port that handed a live Claude credential to whatever
  reached it, and it had never been verified against a real Claude Code. The
  guard work in T63 made it defensible, not necessary.

  **What is lost, stated plainly.** Rotation had a fast path: a 429 the gateway
  saw on a real request overrode the poller's reading and triggered
  `evaluateRotation` immediately. That path is gone, so an account that runs out
  of quota is now noticed by the poller alone — up to one poll interval later,
  five minutes by default. Switching accounts is untouched: it writes two files
  and never went through the gateway. What it never did, and still does not, is
  land inside a Claude Code session that is already running; the credential is
  read once, at startup.

  A vault already on disk keeps its three `gateway*` values as dead JSON —
  `store.ts` merges what it finds over the defaults, nothing reads them, and no
  migration is added for a file the user owns.

### Changed

- **T70** — Claude Code's proxy is now just "the proxy", in every layer. There is
  no second one left to tell it apart from, so the qualifier went: `ClaudeProxy` →
  `ProxySettings`, `ClaudeProxyConfig` → `ProxyConfig`, `readClaudeProxy` /
  `writeClaudeProxy` / `clearClaudeProxy` / `hasClaudeProxy` → `readProxy` /
  `writeProxy` / `clearProxy` / `hasProxy`, the IPC channels `claude-proxy:get` /
  `:set` / `:clear` → `proxy:get` / `:set` / `:clear`, the preload methods
  `getClaudeProxy` / `setClaudeProxy` / `clearClaudeProxy` → `getProxy` /
  `setProxy` / `clearProxy`, and the `[claude-proxy]` log tag → `[proxy]`.

  `src/main/claude-settings.ts` keeps its filename: it is the
  `~/.claude/settings.json` module, and the proxy is one thing it writes there.

  Two pieces of prose had gone stale the moment the app's proxy was deleted, and
  both are user-facing. The SOCKS refusal used to end "the app's own proxy in the
  section above can still be SOCKS", pointing at a section that no longer exists;
  it now stops at the advice that is still true. And the Proxy page's single card
  loses the "Claude Code proxy" heading that merely repeated the page title,
  taking the `Section` helper with it — one section needs no section component.

- **T64** — Proxy has its own page, in the sidebar where Gateway used to be, and
  `SHOW_NETWORK_FEATURES` is gone with the collapsed "coming in a later release"
  card it gated.

  Both proxies moved there, because they are the two things users mix up: this
  app's own proxy (`proxyRules` / `proxyBypass`, applied to the Chromium session
  the poller and the sign-in window travel through) and Claude Code's proxy (env
  variables written into `~/.claude/settings.json`). Separate drafts and separate
  Save buttons — they are separate writes to separate files, and one Save would
  suggest a single setting.

  The Settings page's whole-draft save now deletes `proxyRules` and `proxyBypass`
  from its patch, alongside the `theme` it already deleted. Without that, a
  Settings form left open since before a proxy was saved would write its stale
  copy back over it. That rule was written on the Gateway page; it had to outlive
  the page it was written on.

### Added

- **T63** — The gateway is open. Its four defects are fixed and
  `GATEWAY_READY` is gone — deleted rather than flipped to `true`, since every
  branch behind it was then dead.

  **A `Host` guard, in front of every path including health.** Binding 127.0.0.1
  keeps other machines out; it does not keep a web page out. A page on evil.com
  whose domain resolves to 127.0.0.1 reaches this port from the user's own
  browser, and a browser sends the site's own name in `Host`, never ours — so
  that is the header worth checking. `127.0.0.1`, `localhost`, `[::1]` and `::1`
  pass, with or without a port; anything else is 403.

  **One rate-limit reader instead of two.** The response is now read through
  `parseRateLimitHeaders`, which accepts both `rate_limited` (what live responses
  send) and `rejected` (what a third-party proxy sends), and puts the reset
  through `toEpochMs`. The hand-rolled check matched only `rejected` and read the
  reset with `Number()`, so an ISO-8601 value became `NaN` and fed the next bug.

  **A bench that ends.** `gatewayRateLimits` entries carry `expiresAt` separately
  from `resetsAt`: upstream's own time when it gave one, otherwise five minutes
  (`GATEWAY_LIMIT_FALLBACK_MS`). An entry with no end benched an account until
  the app restarted, and a fabricated `resetsAt` in the UI is a number the user
  would plan around. Entries are also dropped when the account is deleted.

  `tests/gateway.test.ts` drives all of it over a real loopback socket with
  upstream injected — 17 cases. One of them records that an HTTP/1.1 request with
  no `Host` never reaches the handler at all: Node answers 400 first, so the
  guard's missing-`Host` branch is there for HTTP/1.0, where the header is
  optional.

  Not yet verified end to end against a real Claude Code — that is the next step.

### Changed

- **T62** — `npm run release patch` now carries the whole release in one pass:
  the cut as before, then the branch push, the CI wait, the tag push, and the
  `--no-ff` merge of `release/<x>` into `developing` and `developing` into `main`.

  No stage flags. A release that stops halfway is a release somebody has to
  remember to finish, which is why `main` sat at the initial commit through the
  whole 0.2 line. Instead every stage is skipped when it is already done, so a run
  interrupted anywhere is finished by `npm run release` with no version at all: an
  existing tag is not re-cut, a tag already on origin is not re-pushed, a merge
  already made is `Already up to date`.

  What it refuses. Any branch that is not `release/*`, checked before a byte is
  written rather than three stages later. A merge whose tag is not on origin — a
  merged commit for a version nobody can install is a lie in the history. A target
  branch that has diverged from its `origin/` ref, which is fast-forwarded first
  and never rebuilt from a local fork. And the tag push and every hop into a
  protected branch are confirmed at the terminal, `auto_push` notwithstanding,
  unless `--yes` says otherwise; answering `n` to the tag push is how a release is
  cut without publishing it.

  CI is matched on the pushed commit's sha, not on the branch: `gh run list
  --branch` also returns the previous push's run, and watching that one reports
  green for work this commit never contained. Without `gh`, or when `gh` cannot
  answer, CI becomes a question for the human rather than a guess.

## v0.2.2 — 2026-08-17

### Changed

- **T61** — The installer is per-machine. It installs for every user of the
  machine, into `C:\Program Files (x86)\Manager Account AI`, and Windows asks for
  an administrator to do it — at install time, and again at every update.

  electron-builder's default for a per-machine 64-bit application is
  `C:\Program Files`. The `customInit` macro in `build/installer.nsh` rewrites it
  to the x86 tree, and only while `$INSTDIR` still holds exactly the default
  `initMultiUser` computed: a path that came from an installation already on the
  machine, or from the installer's `/D` switch, is a deliberate choice and
  survives an upgrade untouched. `customInit` rather than `preInit`, which runs
  before `initMultiUser` and would have to seed `InstallLocation` in HKLM —
  writing to the registry before the user has agreed to anything, and making a
  machine that has the old per-user install look like it has both.

  `packElevateHelper: true` is set for a reason its name does not suggest.
  `elevate.exe` is packed for any value but a literal `false`, so the line reads
  as redundant — but electron-builder gates the `isAdminRightsRequired: true`
  field of `latest.yml` on the *raw* option being truthy, and left undefined it
  writes no field at all. electron-updater would then launch the update installer
  unelevated. The first build of this change shipped exactly that `latest.yml`;
  the flag is what puts the field back.

  Two things to know before upgrading. A user without administrator rights can no
  longer install or update at all — that is the price of all users. And accounts
  do not become shared: `%APPDATA%\manager-account-ai` is per Windows user, so
  every user of the machine still keeps their own vault. A machine carrying the
  old per-user install is not left with two copies; the installer removes the
  `HKCU` one as part of installing for all users.

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
