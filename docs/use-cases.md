# Use Cases & Problem Solving

Operational case studies from running **agent-assisted infrastructure work** on a real host:
a Windows 10 workstation, a Linux/Docker layer, an AWS account, and a self-hosted automation
stack. Each case below follows the same bar as the featured migration repo — problem, actual
diagnosis (not a guess), the fix, and **verification output** that proves the fix held.

> Nothing here is a tutorial written after reading docs. Every case is something that broke,
> was measured, was fixed, and was re-tested on this machine.

---

## Case index

| # | Case | Domain | Hardest part |
|:--:|---|---|---|
| 1 | [Recovering a full system drive with zero reinstalls](#1-recovering-a-full-system-drive-with-zero-reinstalls) | Windows / Storage | Moving live app data while apps keep their original paths |
| 2 | [Self-hosted multi-provider AI gateway with cost control](#2-self-hosted-multi-provider-ai-gateway-with-cost-control) | AI infra / Cost | Routing + token accounting across free and paid providers |
| 3 | [GitHub CLI auth on a Windows/MSYS host](#3-github-cli-auth-on-a-windowsmsys-host) | DevOps / Auth | Device flow that never returns in a PTY, and scope drift |
| 4 | [Provisioning an agent skill library under rate limits](#4-provisioning-an-agent-skill-library-under-rate-limits) | Automation | Installing 100+ skills with no token and a 60 req/h API cap |
| 5 | [n8n + Telegram bots: laptop → AWS EC2](#5-n8n--telegram-bots-laptop--aws-ec2) | Cloud | Zero open app ports, state consistency |
| 6 | [Delegation that reported success and shipped nothing](#6-delegation-that-reported-success-and-shipped-nothing) | Agent ops | Verifying sub-agent output instead of trusting it |

---

## 1. Recovering a full system drive with zero reinstalls

**Problem.** The 119 GB system drive (C:) dropped to **9.9 GB free** — Windows update failures and
app crashes. The bulk was app data (Telegram cache, browser profiles, Prisma engines, Docker
images) sitting on the only drive Windows apps know how to use.

**Constraints.** No admin rights for parts of the move, no reinstall, no path changes — breaking a
path would break pinned shortcuts, updaters, background services, and hardcoded config.

**Diagnosis.** `du` on Windows/MSYS hangs for minutes on Chromium profiles, so the tree was walked
in Python with `os.walk` + `st_size` instead. That identified four multi-GB offenders:
`Telegram Desktop`, a Chromium browser profile, an Edge profile, and Prisma engine data.

**Fix.** Directory **junctions** (`mklink /J`) — they work without admin, unlike symlinks, and they
preserve the original path so nothing needs reconfiguring. Crash-safe order per folder:

1. Close the app (graceful, then `/F /T`) — Chromium keeps 7+ background processes alive.
2. **Copy** (`shutil.copytree(..., dirs_exist_ok=True)`), never move first.
3. Verify destination file count ≥ source; abort and leave the source untouched on mismatch.
4. Rename source to `<src>.premigrate` (a rename succeeds even with locked files).
5. `cmd /c mklink /J <src> <dst>`; on failure rename `.premigrate` back.
6. `cmd /c rd /s /q <old>` — `shutil.rmtree` routinely leaves hundreds of MB on Chromium trees.

Deliberately **not** migrated: `EdgeWebView2` and `EdgeCore` (shared system runtimes that other
apps and Edge Update fight over). Pagefile was moved to the second drive separately, and the AI
gateway, Docker images and model weights were relocated to the D: drive.

**Non-obvious failure that cost a cycle.** The copy preserved the read-only attribute, and
Chromium/leveldb needs to write its `LOCK` file — every launch logged
`leveldb_database.cc Unable to open .../LOCK: Access denied` and the browser showed
*"Profile error occurred"*. Fix: strip read-only recursively on the migrated tree before launching
(it is also why `rmtree` left files behind).

**Verification.**
- Junction is real: `st_file_attributes & 0x400` set, `os.readlink()` resolves to the target.
- File counts match source vs destination.
- Browser launched with `--enable-logging=stderr --v=1`; asserted `Access denied` / `LOCK`
  appear **0 times** in the log.
- Visible windows enumerated (`EnumWindows`) to confirm no "Profile error" dialog, rather than
  trusting "the app opened".
- **Result: 9.9 GB → 33 GB free**, measured with `df -h`.

**Lesson.** A migration is not done when the app opens — it is done when the app's own log and the
filesystem metadata both agree. Also: `os_crypt_win.cc Failed to encrypt` appears once on *any*
profile including a brand-new one, so a baseline on the original drive is required before blaming
the migration.

---

## 2. Self-hosted multi-provider AI gateway with cost control

**Problem.** Running an agent platform against several LLM providers meant juggling keys, hitting
paid models for trivial tasks, and having no way to fail over when one provider went down.

**Fix — 9Router, self-hosted, loopback-only.** A Next.js gateway on `127.0.0.1:20128` with state in
a SQLite `DATA_DIR`, run natively (no admin) and kept off any public interface. Only one instance is
allowed on the port at a time (native **or** container), because state lives in a single DB.

**Routing model = tiers, not one model.** Combos are ordered fallback chains:

| Combo | Purpose | Shape |
|---|---|---|
| `free-3tier` | default traffic | free provider lanes, highest quality first |
| `rutin-docs` | documentation / routine edits | cheap, high-context |
| `tugas-kompleks` | hard reasoning | premium models only here |
| `belajar` | study / explanation | reasoning-tuned, moderate budget |

Disconnected providers simply fail over, so it is safe to keep an unreachable lane at the front of
a chain. The default client model is the free-tier combo, which means a misconfigured automation
degrades to free instead of burning premium credits.

**Admin without storing passwords.** Management routes return `401` until dashboard login. Instead
of the shipped default password, the app accepts a machine-scoped CLI token
(`sha256(machineId + "9r-cli-auth" + cliSecret)[:16]`) on a dedicated header — derived locally from
files in `DATA_DIR`, never transmitted, and wrapped in a small `tools/9r.js` helper so it never
appears in shell history. Client API keys are generated by the gateway
(`requireApiKey: true`) and written to a file, never pasted into chat or committed.

**Cost controls, measured not assumed.** Response-side compression helped in a measured run
(−57% output tokens on a reasoning model) but was kept **off** for formal writing. Content-aware
truncation was proven to be non-generic: it reported 60% on `git diff`, 54% on build logs, and
**0%** on a plain numbered file dump — so no blanket "saves 20–40%" claim is made anywhere.

**Verification.** End-to-end probe through the gateway returns `200` with the resolved upstream
model id and token usage, and the token-saving A/B script compares `prompt_tokens` /
`completion_tokens` per feature with a random salt to defeat prompt caching.

**Lesson.** Route by task tier, default to free, and only claim a compression win after measuring
the *shape* of the payload the client actually sends.

---

## 3. GitHub CLI auth on a Windows/MSYS host

**Problem.** Repo automation kept dying on `gh auth status` → *"The token in default is invalid"* —
and re-authenticating interactively was unreliable.

**Diagnosis (three separate failures, all real):**
1. `gh` is not in `winget`/`scoop` on this host; installed from the official zip release into
   `~/bin` instead.
2. `gh auth login --web` in a **PTY** hangs at *"Press Enter to open github.com in your browser"* —
   the prompt cannot be driven by writing a newline. What works is a **non-PTY** background process
   with stdin from `/dev/null`, which keeps polling and prints
   `One-time code (XXXX-XXXX)` to the log.
3. MSYS path translation is disabled here, so native binaries receive `/c/Users/...` as a *relative*
   path. Paths are passed as `C:/Users/...` or resolved by `cd` first.

**Scope drift.** The device-flow token arrives with `gist, read:org, repo` — and **no `workflow`**,
so pushing anything under `.github/workflows/` is rejected until `gh auth refresh -s workflow`.
Checked with `gh auth status` before assuming a push permission problem is a network problem.

**Verification.** `gh auth status` reports the account and scopes; `gh auth setup-git` wires the
credential helper; a read-only `git ls-remote` proves pushes will be authorised without performing
one. Confirmed today: `✓ Logged in to github.com account r3conxploit (keyring)`.

**Operational rule.** Credentials are only handled by the CLI's own flow — a token is never typed
into chat, an issue, or a script argument. Time-boxed device codes are regenerated rather than
debugged when they expire.

**Lesson.** On a mixed Windows/MSYS host, half of "auth is broken" is actually "the prompt layer is
broken". Non-PTY background process + log scraping beats fighting a TTY.

---

## 4. Provisioning an agent skill library under rate limits

**Problem.** Standing up a ~100+ skill library for an agent profile, with **no GitHub token** on
that profile — so the installer hit the unauthenticated API cap (**60 requests/hour**) and stalled
partway through.

**Fix — tarball fallback.** Bypass the API path entirely: fetch
`codeload.github.com/<owner>/<repo>/tar.gz/refs/heads/main`, extract, and copy each skill folder
into `<profile>/skills/<category>/`. The profile auto-registers anything in that tree as a local
skill, so the library is complete without a single authenticated API call.

**Verification.** `skills_list` per category afterwards, plus an audit script that flags skills with
broken frontmatter, missing `description` triggers, or unreachable referenced files.

**Lesson.** Rate limits are an artefact of *how* you fetch, not a hard ceiling. When a distribution
path is quota-bound, switch the transport.

---

## 5. n8n + Telegram bots: laptop → AWS EC2

Full case study lives in its own repository — the pattern is: Docker-pinned n8n on a `t3.micro`,
SQLite state restored into a named volume, ingress exclusively through a **Cloudflare Tunnel** so
the host exposes **zero application ports**, and verification that the stack survives a reboot
without intervention (20 workflows · 7 credentials · 104 executions, ≈ $13/month).

→ [`n8n-telegram-aws-migration`](https://github.com/r3conxploit/n8n-telegram-aws-migration) —
6 docs, 8 ADRs, 5 idempotent scripts, 10-case troubleshooting catalogue.

**Later addition worth noting:** the gateway in Case 2 is what the cloud-hosted workflows can call
for model inference, keeping provider keys on the private host instead of inside n8n credentials.

---

## 6. Delegation that reported success and shipped nothing

**Problem.** A documentation task (this document, in fact) was delegated to a sub-agent. It came
back `status=completed` in **36 seconds** with three API calls, and a summary that sounded like
progress.

**Diagnosis.** Inspecting the full tool trace showed it called `skill_view` and `skills_list`, then
emitted a "searching conversation history" line as its **final** message. Zero files created, zero
commits, zero pushes. The summary described an intention; the trace described reality.

**Fix.** Treat any delegated claim of a side effect as unverified until a **handle** exists — a
commit SHA, a PR number, a file path that can be read back. The work was redone in the parent
context, where the real blocker surfaced immediately: the `gh` token was invalid (Case 3), which is
exactly the kind of finding a summary line would have hidden.

**Verification gate now applied to every delegation.** Require the child to return a verifiable
handle (commit SHA / PR URL / absolute path), then read it back independently before reporting
success upward. `completed` is a process state, not a delivery.

**Lesson.** Sub-agent summaries are self-reports, not facts. The cheapest possible verification —
one `git log`, one `gh pr view`, one `read_file` — is what separates "the agent said so" from
"it is on GitHub".

---

## How to read these cases

Everything above is expressed in terms of **failure → evidence → fix → re-measurement**. Where a
number appears (33 GB free, −57% output tokens, 0% on a plain dump, 60 req/h), it came from a
command that was actually run on this host, not from a vendor datasheet. The same standard applies
to the migration repo and to anything else published here.
