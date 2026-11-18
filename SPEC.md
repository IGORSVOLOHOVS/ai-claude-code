# ai-claude-code - software requirements specification

Reverse-engineered on 2026-11-18 from commit `571945c` (branch `release`).
Every requirement below describes behaviour present in the files at that
commit; each carries its source. The documentation hub is
[docs/README.md](docs/README.md).

Source paths are relative to the repository root. Short names stand for:
`gate.ps1`, `log.ps1`, `session-start-stop.ps1` in `.claude/hooks/`;
`commit-msg`, `commit-msg.ps1`, `pre-push`, `pre-push.ps1` in
`.claude/hooks/git/`; `propose.yaml`, `test.yaml`, `release-gate.yaml`,
`release.yaml`, `sweep.yaml` in `.gitea/workflows/`.

## 1. Purpose and scope

The repository provides the public home page and privacy policy of the Google
OAuth application "AI Claude Code", a personal assistant used by one person, the
repository owner (`README.md:3-7`). It also carries the owner's lab tooling that
governs how the repository is changed and released.

In scope:

- `index.html` and `privacy.html`, static pages (`README.md:11-16`).
- `recipe.ps1`, the command interface and metadata (`README.md:23-25`).
- Claude Code hooks and settings under `.claude/`.
- Git hooks under `.claude/hooks/git/`.
- Gitea Actions workflows under `.gitea/workflows/` and the issue and pull
  request templates.

Out of scope:

- The assistant program that calls Google Tasks and Google Calendar: "This
  repository ships no program" (`README.md:23`). Its code, OAuth scopes and
  token storage are not here.
- Hosting of the pages: the README names GitHub Pages (`README.md:16-17`); no
  Pages configuration is in the repository.
- `BEST_REQUIREMENTS.md`, the lab requirements document cited by the README,
  hooks and workflows (`README.md:34`, `session-start-stop.ps1:46`,
  `pre-push.ps1:80`); it is not in the repository.

## 2. Context

Actors:

| Actor | Role | Source |
|---|---|---|
| Owner | The only user of the assistant; the only `$Reviewers` login; posts every pull request | `index.html:16`, `recipe.ps1:22`, `CLAUDE.md:64-65` |
| Claude Code | Works in the repository under `CLAUDE.md`; its tool calls pass the hooks | `CLAUDE.md`, `.claude/settings.json` |
| Tester | Runs manual tests when `tests/manual/tests.json` lists any | `release-gate.yaml:204-213` |
| `lab-bot` | Git identity of the release workflow | `release.yaml:38-41` |
| Visitor / Google OAuth review | Reads the two pages; inference from "These two pages exist for its Google OAuth application" | `README.md:6-7` |

External systems:

| System | Use | Source |
|---|---|---|
| Gitea server with Actions | Hosts the working copy; REST API `/api/v1`, generic packages `/api/packages` | `.gitea/workflows/*.yaml` |
| Gitea Actions runner labelled `windows` | Runs all five workflows in Windows PowerShell | `runs-on: windows`, `shell: powershell` in each workflow |
| GitHub | Hosts the public copy; GitHub Pages serves the pages (per README) | `README.md:16-19` |
| Google | Linked from the privacy policy only (permissions page, API Services User Data Policy) | `privacy.html:42-47` |
| Git LFS | Called by `pre-push` when installed; `lfs: true` in checkouts | `pre-push:4-6`, workflows |

Runtime environment: Windows; Windows PowerShell 5.1 (`powershell.exe`,
`.claude/settings.json:8-18`, `.gitea/pull_request_template.md:16`); Git for
Windows `sh.exe` for the git hooks (`.claude/hooks/git/commit-msg:2`); Python
with pytest and ruff for `recipe.ps1 test` (`recipe.ps1:46-52`). No hardware.

## 3. Functional requirements

### 3.1 Home page

- **FR-1.** The system shall provide a home page that states that AI Claude Code
  is a private tool on its owner's computer which, on the owner's explicit
  command, creates entries in the owner's Google Tasks and Google Calendar
  (including Google Meet links) and reads them back, and that it has no other
  users. Source: `index.html:15-22`. Verification: no test.
- **FR-2.** The home page shall link to the privacy policy. Source:
  `index.html:24`. Verification: no test.
- **FR-3.** The home page shall carry a Russian-language paragraph marked
  `lang="ru"` with the same description, limited to tasks and calendar events.
  Source: `index.html:27-29`. Verification: no test.

### 3.2 Privacy policy

- **FR-4.** The system shall provide a privacy policy page with a "Last updated"
  date. Source: `privacy.html:15-16`. Verification: no test.
- **FR-5.** The policy shall list the accessed data: Google Tasks (create tasks,
  read open tasks) and Google Calendar (create events and Google Meet calls, read
  the calendar). Source: `privacy.html:21-27`. Verification: no test.
- **FR-6.** The policy shall state that data is used only for the owner's
  commands, that the access token is stored only on the owner's computer, that
  the app has no server, that Google user data is not sold, transferred or
  disclosed, and that use adheres to the Google API Services User Data Policy
  including Limited Use. Source: `privacy.html:29-43`. Verification: no test.
- **FR-7.** The policy shall link to the Google account permissions page for
  revocation, to the owner's GitHub profile as contact, and back to the home
  page. Source: `privacy.html:45-52`. Verification: no test.

### 3.3 recipe.ps1

- **FR-8.** The system shall accept exactly one verb from `build`, `install`,
  `uninstall`, `clean`, `run`, `publish`, `info`, `test`, and pass remaining
  arguments as `$RunArgs`. Source: `recipe.ps1:1-7`. Verification: no test.
- **FR-9.** For every verb, the system shall load `KEY=VALUE` lines from
  `.claude/.env` into the process environment, trimming surrounding quotes and
  leaving variables that are already set unchanged. Source: `recipe.ps1:28-38`.
  Verification: no test.
- **FR-10.** `test` shall set `core.hooksPath` to `.claude/hooks/git`, run
  `python -m pytest .claude` accepting exit codes 0 and 5, then
  `python -m ruff check .claude` and `python -m ruff format .claude` in place,
  and exit 1 at the first failing step, else 0. Source: `recipe.ps1:41-55`.
  Verification: no test (the CI steps "Tests and formatting" and "Unit tests,
  lint, format" run it but do not check its behaviour).
- **FR-11.** `info` shall print `Name`, `Code`, `Version` and `Summary` and exit
  0. Source: `recipe.ps1:56-62`. Verification: no test (manual run 2026-11-18,
  four lines, exit 0).
- **FR-12.** `clean` shall delete every `__pycache__`, `.pytest_cache` and
  `.ruff_cache` directory under the repository root. Source:
  `recipe.ps1:63-68`. Verification: no test.
- **FR-13.** `build`, `install`, `uninstall`, `run` and `publish` shall print
  `nothing to <verb>` and exit 0. Source: `recipe.ps1:69-73`. Verification: no
  test (manual run of `build` 2026-11-18).

### 3.4 Claude Code hooks

- **FR-14.** The system shall register hooks for SessionStart, UserPromptSubmit,
  PreToolUse (matcher `Bash|PowerShell`, gate), PostToolUse, Notification, Stop,
  SubagentStop, PreCompact and SessionEnd, each run as
  `powershell.exe -NoProfile -ExecutionPolicy Bypass -File <script> -Event <name>`.
  Source: `.claude/settings.json:7-19`. Verification: no test.
- **FR-15.** The gate shall deny a PreToolUse call whose stdin is empty, is not
  JSON, or has no `tool_name`. Source: `gate.ps1:2-4`, `gate.ps1:34-36`.
  Verification: no test (manual run 2026-11-18, empty payload denied).
- **FR-16.** The gate shall remove all quote characters from the command, split
  it on `||`, `&&`, `;`, `|`, `&` and newlines, strip leading `sudo`, `env` and
  `NAME=value` words, unwrap `xargs`, `bash`, `sh`, `cmd`, `powershell`, `pwsh`
  wrappers, and judge each segment by its verb. Source: `gate.ps1:38-57`,
  `gate.ps1:135-144`. Verification: no test.
- **FR-17.** The gate shall deny a recursive delete (`rm`, `Remove-Item`, `ri`,
  `rd`, `rmdir`, `del`, `erase` with `-r`/`-R`/`-Recurse`/`--recursive`, `rd`/
  `rmdir`/`del`/`erase` with `/s`, or any `rimraf`) unless every path resolves
  under `$env:TEMP`; `$TEMP`, `${TEMP}`, `$TMP`, `/tmp` and `$env:TEMP` prefixes
  are expanded, and `$env:TEMP` itself is not under it. Source:
  `gate.ps1:83-119`, `gate.ps1:145-154`. Verification: no test (manual run
  2026-11-18: delete of `build` denied, delete under `$TEMP` allowed,
  `Remove-Item -Recurse build` denied).
- **FR-18.** The gate shall deny `git push` with `--force`, `--mirror`, a short
  flag bundle containing `f`, or a `+<ref>` refspec, and allow
  `--force-with-lease`. Source: `gate.ps1:155-163`. Verification: no test
  (manual run 2026-11-18).
- **FR-19.** When the current branch is `test` or `release`, the gate shall deny
  `git reset --hard` (except to `origin/<branch>`, `@{u}` or `@{upstream}`),
  `git clean` with `-f`/`--force`, `git commit --amend`, `git rebase` and
  `git filter-branch`. Source: `gate.ps1:121-129`, `gate.ps1:164-177`.
  Verification: no test.
- **FR-20.** A denial shall be printed as JSON
  `hookSpecificOutput.permissionDecision = "deny"` with a reason, exit code 0;
  an allowed call prints nothing. Source: `gate.ps1:25-32`, `gate.ps1:180-181`.
  Verification: no test (manual run 2026-11-18).
- **FR-21.** Every hook shall append one JSON object per event to
  `.claude/logs/ai.jsonl` with `ts`, `event` and, when present, `session`,
  `tool`, `decision` and `detail`. Source: `log.ps1:4-24`, `gate.ps1:6-23`,
  `session-start-stop.ps1:8-23`. Verification: no test.
- **FR-22.** `log.ps1` shall set `detail` to the first 200 characters of the
  first present field among `prompt`, `message`, `reason`, `source`,
  `file_path`. Source: `log.ps1:7-13`. Verification: no test.
- **FR-23.** At SessionStart the system shall move `ai.jsonl` larger than 10 MB
  to `ai.jsonl.<yyyyMMddHHmmss>.bak`, set `core.hooksPath` to
  `.claude/hooks/git`, write a warning to stderr when `recipe.ps1` is missing,
  parse the issue number from a branch matching `^[a-z][a-z0-9]*-<n>/`, and
  return `additionalContext` naming the branch and the issue (or stating that
  there is none). Source: `session-start-stop.ps1:26-48`. Verification: no test.
- **FR-24.** At Stop and SessionEnd the system shall log `turn finished` and
  `session ended: <reason>`. Source: `session-start-stop.ps1:49-54`.
  Verification: no test.

### 3.5 Git hooks

- **FR-25.** `commit-msg` shall refuse a message file that starts with a UTF-8
  BOM. Source: `commit-msg.ps1:21-28`. Verification: no test.
- **FR-26.** `commit-msg` shall require the first non-comment line to match
  `<type>(<scope>): <text>` case-sensitively, with types and scopes read from
  `$CommitTypes` and `$CommitScopes` in `recipe.ps1`, and refuse when either list
  cannot be read; a subject starting `Merge ` passes only while `MERGE_HEAD`
  exists. Source: `commit-msg.ps1:7-47`. Verification: no test (manual run
  2026-11-18: wrong case and unknown scope refused).
- **FR-27.** `commit-msg` shall require a `Signed-off-by: <name> <<email>>`
  trailer, as parsed by `git interpret-trailers --parse`. Source:
  `commit-msg.ps1:48-53`. Verification: no test (manual run 2026-11-18).
- **FR-28.** `pre-push` shall pass the push to `git lfs pre-push` first when
  `git-lfs` is on PATH. Source: `.claude/hooks/git/pre-push:3-7`. Verification:
  no test.
- **FR-29.** `pre-push` shall refuse a pushed branch whose name does not match
  `^<Code>-<issue>/<BranchType>/<slug>$` (issue without leading zero, slug of
  lower-case words joined by `-`), and shall not judge tags, `test`, `from-*`
  branches or deletions of branches other than `release`. Source:
  `pre-push.ps1:39-41`, `pre-push.ps1:52`, `pre-push.ps1:87-93`.
  Verification: no test (manual run 2026-11-18 on five ref lines).
- **FR-30.** `pre-push` shall refuse deleting `release`, pass creating
  `release`, and refuse any other push to `release` unless one first-parent
  commit in the pushed range that is not on `origin/test` has `$OverridePhrase`
  alone on a line and a line `Why: <text>`. Source: `pre-push.ps1:54-85`.
  Verification: no test.
- **FR-31.** `pre-push` shall pass, with a message, when `recipe.ps1` is
  missing or `$BranchTypes` is not a literal list, and refuse when `$Code` or
  `$BranchTypes` is absent. Source: `pre-push.ps1:10-38`. Verification: no test.

### 3.6 Workflow `propose` (pull request into `test`)

- **FR-32.** On a pull request into `test` (opened, reopened, synchronize) the
  system shall fail unless the head branch matches the branch grammar read from
  `recipe.ps1`; `from-*` branches are exempt. Source: `propose.yaml:1-5`,
  `propose.yaml:33-75`. Verification: no test.
- **FR-33.** It shall fail when no commit arrived since the merge base, and
  otherwise check each non-merge commit not already on `release` for the subject
  grammar and a `Signed-off-by` trailer. Source: `propose.yaml:77-142`.
  Verification: no test.
- **FR-34.** It shall run `.\recipe.ps1 test` and fail if `git diff --exit-code`
  finds changes afterwards. Source: `propose.yaml:144-152`. Verification: no
  test.
- **FR-35.** On opened or reopened, it shall request review from `$Reviewers`
  minus the poster and comment that the gate passed; without `LAB_BOT_TOKEN` it
  warns and notifies nobody. Source: `propose.yaml:154-193`. Verification: no
  test.

### 3.7 Workflow `test` (push to `test`, daily, manual)

- **FR-36.** On push to `test`, on schedule `0 3 * * *` and on manual dispatch,
  the system shall fail when drive C: has under 5 GB free (warn under 20 GB) or
  when `python`, `ruff`, `pytest` or `mypy` does not answer `--version`. Source:
  `test.yaml:2-7`, `test.yaml:32-45`. Verification: no test.
- **FR-37.** It shall run `.\recipe.ps1 test`, `git diff --exit-code` and
  `.\recipe.ps1 build`; when `recipe.ps1` names `$App` it shall also prove an
  install, `info`, uninstall, `info` cycle; when it names `$App` or a non-empty
  `$Provides` it shall regenerate the wiki with `tools/wiki.py`; it always ends
  with `.\recipe.ps1 clean`. Source: `test.yaml:46-120`. Verification: no test.

### 3.8 Workflow `release-gate` (pull request into `release`)

- **FR-38.** The `checks` job shall fail unless the head branch is `test`, then
  run `recipe.ps1 test` with `git diff --exit-code` and `recipe.ps1 build`, and
  for an app require an `.exe` in `dist\` and an install/uninstall cycle. Source:
  `release-gate.yaml:16-85`. Verification: no test.
- **FR-39.** The `manual-gate` job shall read `tests/manual/tests.json`; when it
  is absent or lists no tests, set commit status `manual/verdict` to success
  with "no manual tests defined"; it shall fail on a file that does not parse,
  lacks a `tests` array, or has an illegal or duplicate id. Source:
  `release-gate.yaml:107-149`. Verification: no test.
- **FR-40.** With manual tests, it shall set `manual/<id>` to pending where no
  status exists, keep an earlier green verdict, post tester instructions, and
  poll every 30 s for up to `WAIT_MINUTES` until `manual/verdict` is success or
  failure, or until a `$Reviewers` login comments, after the head commit and its
  first status, with `$OverridePhrase` alone on a line and a `Why:` line, which
  sets `manual/verdict` to success as "override by <login>: <why>". Source:
  `release-gate.yaml:151-346`. Verification: no test.
- **FR-41.** Without an override it shall reject a verdict while any blocking
  test is not success (an optional test may be `warning` with description
  `n/a: ...`) by setting `manual/verdict` to failure. Source:
  `release-gate.yaml:348-390`. Verification: no test.
- **FR-42.** It shall write `manual-report.md` (a table of id, title, state,
  note, plus any override note), upload it to the generic package
  `<name>-manual-reports/<sha>/` and as artifact `manual-report`. Source:
  `release-gate.yaml:392-446`. Verification: no test.

### 3.9 Workflow `release` (push to `release`)

- **FR-43.** On push to `release` the system shall read `$Version` and publish
  nothing (warning) when tag `v<Version>` already exists. Source:
  `release.yaml:45-60`. Verification: no test.
- **FR-44.** It shall write release notes: non-merge commits since the last
  `v*` tag excluding changelog commits, and the issues and pull requests their
  subjects reference. Source: `release.yaml:76-124`. Verification: no test.
- **FR-45.** It shall record overrides: a first-parent commit not from a pull
  request into `release` counts as a direct push; a pull request merged without
  `manual/verdict` and `release-gate / checks (pull_request)` both success is
  recorded with its `Why:` (from the body of a PR posted by a `$Reviewers`
  login, or from a `$Reviewers` comment); either
  marks the release as pre-release, and a record without `Why:` opens or
  comments the issue "release changed past the gate without the owner's
  override". Source: `release.yaml:126-277`. Verification: no test.
- **FR-46.** It shall create Gitea release `v<Version>` at the pushed commit and
  attach the first `dist\*.exe` if any, `manual-report.md` (fetched from the
  package or a placeholder) and `sbom.md` built from the `$Needs`, `$Tools`,
  `$Version` lines and `uv.lock` if present. Source: `release.yaml:279-336`.
  Verification: no test.
- **FR-47.** It shall comment "Shipped in v<Version>." on, and close, each open
  issue whose number came from a merge subject `Merge ... from <Code>-<n>/`.
  Source: `release.yaml:93-116`, `release.yaml:338-354`. Verification: no test.
- **FR-48.** It shall merge `release` into `test` with `--no-ff` and push; on a
  conflict or a refused push it shall open or comment the issue "test is behind
  release". Source: `release.yaml:356-400`. Verification: no test.
- **FR-49.** When `recipe.ps1` names `$App` or a non-empty `$Provides`, it shall
  prepend the release notes to `CHANGELOG.md` on `test` in a commit
  `docs(docs): changelog for v<Version>`. Source: `release.yaml:402-422`.
  Verification: no test.

### 3.10 Workflow `sweep` (weekly)

- **FR-50.** On schedule `0 4 * * 1` the system shall delete remote work
  branches (`<Code>-<n>/<type>/<slug>`) already merged into `origin/test`.
  Source: `sweep.yaml:3-4`, `sweep.yaml:23-44`. Verification: no test.
- **FR-51.** It shall comment on open pull requests idle 30 days or more and
  close those idle 60 days or more, idleness counted from creation, head commit
  date and comments other than `lab-bot`'s. Source: `sweep.yaml:45-80`.
  Verification: no test.

### 3.11 Repository hygiene

- **FR-52.** Git shall ignore `.claude/logs/`, `.claude/settings.local.json`,
  `.claude/.env`, root build output and tool caches. Source: `.gitignore:1-24`.
  Verification: no test.
- **FR-53.** Git shall check out text with LF endings, `.ps1`/`.bat`/`.cmd`
  with CRLF (`.ps1` working tree UTF-8), the two shell git hooks with LF, and
  store `*.pdf`, `*.png`, `*.jpg`, `*.zip`, `*.7z` through Git LFS. Source:
  `.gitattributes:1-13`. Verification: no test.

### 3.12 Declared but not implemented

| Item | Where declared | State |
|---|---|---|
| The assistant's access to Google Tasks and Calendar, local token storage, no server, Limited Use | `index.html:18-20`, `privacy.html:21-43` | No code for it in this repository (`README.md:23`) |
| GitHub Pages serves the pages | `README.md:16-17` | No Pages configuration in the repository |
| AI usage report `.claude/logs/report-<date>.md` | `.claude/skills/lab-ai-env/SKILL.md:39-69` | Instructions for Claude Code; no script implements it |
| Wiki generation | `test.yaml:85`, `test.yaml:106` | `tools/wiki.py` does not exist; step inactive for this repository |
| Manual test runner at `http://127.0.0.1:8765/` | `release-gate.yaml:210-211` | `tests/manual/serve.py` and `tests/manual/tests.json` do not exist |
| Installer build and install/uninstall proof | `test.yaml:59-72`, `release-gate.yaml:55-74` | `build`/`install`/`uninstall` print `nothing to <verb>`; no `$App` |
| `--force-with-lease` allowed "on your own branch" | `CLAUDE.md:33` | The gate does not check the branch (`gate.ps1:159-160`) |
| Branch protection stages 1 and 2 | `CLAUDE.md:68-70` | Server setting; not in the repository |
| "Pin ruff and pytest" | subject of commit `f80022e` | Only ruff is pinned (`.claude/ruff.toml:2`) |

## 4. Interfaces

### 4.1 Command line

| Command | Exit code | Output / effect | Source |
|---|---|---|---|
| `.\recipe.ps1 test` | 0 or 1 | pytest, ruff check, ruff format; sets `core.hooksPath` | `recipe.ps1:41-55` |
| `.\recipe.ps1 info` | 0 | Four `Key : value` lines | `recipe.ps1:56-62` |
| `.\recipe.ps1 clean` | 0 | Deletes cache directories | `recipe.ps1:63-68` |
| `.\recipe.ps1 build\|install\|uninstall\|run\|publish` | 0 | `nothing to <verb>` | `recipe.ps1:69-73` |

### 4.2 Hooks

| Hook | Invoked by | Input | Output |
|---|---|---|---|
| `gate.ps1 -Event PreToolUse` | Claude Code, Bash and PowerShell tools | stdin JSON: `tool_name`, `tool_input.command`, `cwd`, `session_id` | deny JSON or nothing; exit 0 |
| `log.ps1 -Event <name>` | Claude Code, 5 events | stdin JSON: `session_id`, `tool_name`, `prompt`, `message`, `reason`, `source`, `file_path` | log line; exit 0 |
| `session-start-stop.ps1 -Event <name>` | Claude Code, SessionStart / Stop / SessionEnd | stdin JSON: `session_id`, `reason` | SessionStart: JSON `additionalContext`; exit 0 |
| `commit-msg <file>` | git | path of the message file | messages on stdout; exit 1 refuses |
| `pre-push` | git | stdin lines `<local ref> <local sha> <remote ref> <remote sha>` | messages on stdout; exit 1 refuses |

Sources: `.claude/settings.json:7-19`, `gate.ps1:1-36`, `log.ps1:1-13`,
`session-start-stop.ps1:1-48`, `commit-msg.ps1:1`, `pre-push.ps1:1`.

### 4.3 Workflow triggers

| Workflow / job | Trigger | Timeout | Source |
|---|---|---|---|
| `propose` / `gate` | pull request into `test`: opened, reopened, synchronize | 60 min | `propose.yaml:2-15` |
| `test` / `prove` | push to `test`; cron `0 3 * * *`; manual | 60 min | `test.yaml:2-17` |
| `release-gate` / `checks`, `manual-gate` | pull request into `release`: opened, reopened, synchronize | 60 min, 170 min | `release-gate.yaml:2-5`, `:19`, `:91` |
| `release` / `publish` | push to `release` | 60 min | `release.yaml:2-14` |
| `sweep` / `sweep` | cron `0 4 * * 1` | 15 min | `sweep.yaml:2-14` |

### 4.4 Gitea API calls

| Method and path (under `/api/v1/repos/<repo>`) | Workflow | Source |
|---|---|---|
| `POST /pulls/<pr>/requested_reviewers`, `POST /issues/<n>/comments` | propose | `propose.yaml:177-191` |
| `PATCH` (repository, `has_wiki`), `POST /wiki/new` | test | `test.yaml:96-102` |
| `GET /commits/<sha>/status`, `POST /statuses/<sha>`, `GET /statuses/<sha>`, `GET` and `POST /issues/<pr>/comments` | release-gate | `release-gate.yaml:143-315` |
| `GET /issues/<n>`, `GET /commits/<sha>/pull`, `GET /commits/<sha>/status`, `GET` and `POST /issues/<n>/comments`, `GET /issues?state=open&type=issues`, `POST /issues`, `POST /releases`, `POST /releases/<id>/assets`, `PATCH /issues/<n>` | release | `release.yaml:104-397` |
| `DELETE /branches/<b>`, `GET /pulls?state=open`, `GET /git/commits/<sha>`, `GET` and `POST /issues/<n>/comments`, `PATCH /pulls/<n>` | sweep | `sweep.yaml:40-76` |
| `PUT`/`DELETE`/`GET` `/api/packages/<owner>/generic/<name>-manual-reports/<sha>/manual-report.md` | release-gate, release | `release-gate.yaml:438-440`, `release.yaml:308-309` |

### 4.5 Commit status contexts

| Context | Set by | Meaning | Source |
|---|---|---|---|
| `manual/<id>` | tester (pending seeded by release-gate) | One manual test | `release-gate.yaml:182-186` |
| `manual/verdict` | release-gate, tester | Overall manual result or override | `release-gate.yaml:143-147`, `:311-315`, `:383-387` |
| `release-gate / checks (pull_request)` | Gitea for the `checks` job | Read by release to judge a merge | `release.yaml:194` |

### 4.6 Pages

| File | Title | Links | Source |
|---|---|---|---|
| `index.html` | "AI Claude Code — personal assistant" | `privacy.html` | `index.html:6`, `:24` |
| `privacy.html` | "AI Claude Code — privacy policy" | Google policy, Google permissions, GitHub profile, `index.html` | `privacy.html:6`, `:42-52` |

## 5. Data

| Entity | Storage and format | Source |
|---|---|---|
| Recipe metadata | `recipe.ps1`; each value a literal on one line; read as text by regex in hooks and workflows | `recipe.ps1:9-23`, `propose.yaml:164` |
| Keys | `.claude/.env`, `KEY=VALUE` lines, gitignored, never committed | `recipe.ps1:30-38`, `CLAUDE.md:20-24` |
| Hook event log | `.claude/logs/ai.jsonl`, JSON lines, UTF-8 without BOM, fields `ts`, `event`, `session`, `tool`, `decision`, `detail`; rotated at 10 MB to `.bak` | `log.ps1:4-24`, `session-start-stop.ps1:27-30`, `.claude/skills/lab-ai-env/SKILL.md:18-27` |
| Manual test list | `tests/manual/tests.json`: `{"tests": [{"id", "title", "blocking"}]}`, id `^[a-z0-9][a-z0-9_-]{0,60}$`, not `verdict` (file absent) | `release-gate.yaml:111-134`, `:417-426` |
| CI work files | `manual-report.md`, `release-body.md`, `issues-to-close.txt`, `sbom.md`, in the runner checkout | `release-gate.yaml:148`, `release.yaml:116-124`, `:332` |
| Changelog | `CHANGELOG.md` on `test`, written only when the repository ships | `release.yaml:402-422` |
| Versions | git tags `v<Version>`; `v1.0.0` exists | `release.yaml:53`, `git tag -l` |

The pages hold no data beyond their text. No database.

## 6. Non-functional requirements

- **NFR-1.** Platform: Windows PowerShell 5.1 for every script; pull requests
  require new PowerShell to parse in 5.1, have no BOM and check
  `$LASTEXITCODE` after native calls. Source: `.claude/settings.json:8-18`,
  `.gitea/pull_request_template.md:16-17`.
- **NFR-2.** Log writes shall retry up to 20 times on `IOException`, waiting
  10, 20, ... ms; hook failures to log are swallowed and the hook exits 0.
  Source: `log.ps1:20-26`, `gate.ps1:17-22`, `session-start-stop.ps1:17-22`.
- **NFR-3.** `detail` logged by `log.ps1` is at most 200 characters (the
  `detail` of `session-start-stop.ps1` is not truncated). Source: `log.ps1:10`,
  `session-start-stop.ps1:43`, `session-start-stop.ps1:53`.
- **NFR-4.** Job timeouts: 60 min (`prove`, `gate`, `checks`, `publish`),
  170 min (`manual-gate`), 15 min (`sweep`). Source: workflow files, §4.3.
- **NFR-5.** `manual-gate` waits `WAIT_MINUTES` = 150, polls every 30 s, and
  fails after 10 consecutive failed polls. Source: `release-gate.yaml:101`,
  `:279-341`.
- **NFR-6.** Status lists are read 50 per page for at most 40 pages. Source:
  `release-gate.yaml:161-166`, `release.yaml:166-171`.
- **NFR-7.** Concurrency: `prove` and `checks` share group
  `installer-<repo>` without cancelling; `release-gate` cancels an older run of
  the same pull request; `publish` runs one at a time per repository. Source:
  `test.yaml:18-20`, `release-gate.yaml:7-9`, `:20-22`, `release.yaml:15-17`.
- **NFR-8.** Runner disk: error below 5 GB free on C:, warning below 20 GB.
  Source: `test.yaml:34-36`.
- **NFR-9.** Secrets: the workflows use only `secrets.LAB_BOT_TOKEN`; `test` and
  `release` check out with `persist-credentials: false` and pass the token to
  git through `GIT_CONFIG_*` variables; keys are never committed. Source:
  `test.yaml:24-30`, `:75-78`, `release.yaml:26-35`, `CLAUDE.md:20-24`,
  `.gitignore:24`.
- **NFR-10.** Encoding: files the scripts write are UTF-8 without BOM; a commit
  message with a BOM is refused. Source: `log.ps1:16-19`,
  `commit-msg.ps1:21-28`, workflow `UTF8Encoding $false` calls.
- **NFR-11.** The pages contain no script; CSS is inline; body width at most
  640 px; a viewport meta tag is set. Source: `index.html:5-12`,
  `privacy.html:5-12`, `README.md:15-16`.
- **NFR-12.** Language of everything written in the repository and its Gitea
  pages is English. Source: `CLAUDE.md:27-29`.

## 7. Constraints and dependencies

No lock file (`uv.lock` is absent; the workflows use it only if present,
`test.yaml:37-41`, `release.yaml:329-331`).

| Dependency | Version | Source |
|---|---|---|
| ruff | `==0.16.1` | `.claude/ruff.toml:2` |
| pytest | not pinned | `recipe.ps1:46`, `.claude/pytest.ini` |
| Python | not pinned | `recipe.ps1:46` |
| mypy | not pinned; only probed by runner health | `test.yaml:38` |
| Windows PowerShell | 5.1 | `.gitea/pull_request_template.md:16` |
| Git, Git for Windows `sh.exe` | not stated | `.claude/hooks/git/commit-msg:2` |
| Git LFS | not stated | `.claude/hooks/git/pre-push:4`, workflows `lfs: true` |
| `curl.exe` on the runner | not stated | `release-gate.yaml:439-440`, `release.yaml:298-334` |
| `actions/checkout` | `v4` | all workflows |
| `actions/upload-artifact` | `v3` | `release-gate.yaml:442` |
| Licence | MIT | `LICENSE` |

Constraints: branches `acc-<issue>/<type>/<slug>` (`CLAUDE.md:8-9`); commits
`<type>(<scope>): <text>` with `git commit -s` (`CLAUDE.md:10`); `release`
accepts pull requests only from `test` (`release-gate.yaml:27-31`); changes to
`release` only by a `$Reviewers` login (`CLAUDE.md:48-56`).

## 8. Acceptance

1. `.\recipe.ps1 test` exits 0 and `git status` stays clean
   (`CLAUDE.md:15-19`, `test.yaml:54-58`).
2. `.\recipe.ps1 info` prints the four metadata lines and exits 0
   (`recipe.ps1:56-62`).
3. On Gitea: `propose / gate` is green on a pull request into `test`;
   `release-gate / checks` and `manual-gate` are green on the pull request from
   `test` into `release` (`CLAUDE.md:42-46`).
4. Manual, no test exists: open `index.html` and `privacy.html`, compare their
   text with FR-1 to FR-7, and follow each link.
5. Manual, no test exists: feed sample PreToolUse JSON to a copy of `gate.ps1`
   outside the repository and sample messages and ref lines to the git hooks,
   and compare with FR-15 to FR-31 (done for a subset on 2026-11-18; see
   [docs/README.md](docs/README.md#about-this-document)).

## 9. Open questions

1. Which branch and settings GitHub Pages uses. The GitHub copy has `main`
   (the first commit only), `test` and `release` (its default); the two pages are
   identical on `main` and `release` (`git diff`).
2. How commits reach the GitHub copy: `README.md:17-19` says nothing pushes from
   Gitea to GitHub, yet the GitHub copy carries `test`, `release` and tag
   `v1.0.0`.
3. Where the assistant program lives, which OAuth scopes it requests, and where
   its token file is; none of this is in the repository, so the claims of
   `privacy.html` cannot be checked here.
4. What R-37, R-41 and R-42 require and why they are excepted
   (`README.md:34`); the cited `BEST_REQUIREMENTS.md` is reachable only at a
   local address.
5. Whether a Gitea runner labelled `windows` currently serves this repository,
   and at which branch-protection stage it is (`CLAUDE.md:44-46`,
   `CLAUDE.md:68-70`).
6. In which time zone the cron schedules `0 3 * * *` and `0 4 * * 1` fire.
7. Why runner health requires `mypy` (`test.yaml:38`) when nothing uses it, and
   whether pytest was meant to be pinned (commit `f80022e`). Partly answered:
   the body of `f80022e` names a `required-version` only for ruff and adds
   `.claude/pytest.ini` without one; the reason for `mypy` is not stated.
8. Who reads the pages besides the owner; the README implies Google's OAuth
   process (`README.md:6-7`) but does not say whether verification was
   requested. Partly answered: the message of commit `3d30744` says Google
   does not let the OAuth application leave Testing for production without
   home page and privacy policy addresses, and that keys last 7 days in
   Testing; whether verification was requested is still not stated.
