## Task 1: Rename local directory to opencode2 and move session

**Description:** Rename filesystem directory from `opencode2-todo` to `opencode2` to match GitHub `ckir/opencode2`, then move the active session so subsequent tool calls run in the new CWD.

**Acceptance criteria:**
- [ ] Directory `/home/user/Development/Typescript/opencode2` exists, old `opencode2-todo` path gone
- [ ] Session CWD is new path (`pwd` confirms)
- [ ] `git status` still clean, `git remote -v` unchanged (already `ckir/opencode2.git`)

**Verification:**
- [ ] Run: `pwd && git remote -v && git status --short --branch`
- [ ] Manual check: `ls /home/user/Development/Typescript/` shows `opencode2/`

**Dependencies:** None

**Files likely touched:**
- (filesystem move only, no file edits)

**Estimated scope:** XS: 0 files, directory move

## Task 2: Verify git remote + GitHub state

**Description:** Confirm GitHub rename is complete and local remote tracks it, so no `gh repo rename` or `git remote set-url` is needed.

**Acceptance criteria:**
- [ ] `gh repo view --json name,nameWithOwner,url` returns `opencode2`, `ckir/opencode2`
- [ ] `git remote -v` fetch/push both `https://github.com/ckir/opencode2.git`
- [ ] `git fetch origin` succeeds

**Verification:**
- [ ] Run: `gh repo view --json name,nameWithOwner,url && git fetch origin && git branch -a`

**Dependencies:** Task 1

**Files likely touched:**
- (none, read-only verification)

**Estimated scope:** XS

## Checkpoint: After Tasks 1-2
- [ ] `pwd` shows `.../opencode2`, remote is `ckir/opencode2.git`, `gh repo view` is `ckir/opencode2`, tree clean
- [ ] Review with human before restructuring

## Task 3: Create category dirs + git mv plugin to plugins/todo/

**Description:** Create monorepo categories `agents/ models/ skills/ themes/ commands/ plugins/` with `.gitkeep` + stub READMEs, then `git mv src/ tool/ index.ts tui.tsx` into `plugins/todo/`.

**Acceptance criteria:**
- [ ] All six category dirs exist; empty ones contain `.gitkeep`
- [ ] `plugins/todo/src/index.ts`, `plugins/todo/src/tui.tsx`, `plugins/todo/tool/*`, `plugins/todo/index.ts`, `plugins/todo/tui.tsx` exist via `git mv` (history preserved)
- [ ] Old top-level `src/ tool/ index.ts tui.tsx` gone
- [ ] `git status` shows renames (not deletes + untracked)

**Verification:**
- [ ] Run: `git status --short && ls plugins/todo/ && git log --follow --oneline -- plugins/todo/tool/store.ts | head -n 3`
- [ ] Manual check: per-category stub README present

**Dependencies:** Tasks 1-2

**Files likely touched:**
- `agents/.gitkeep`, `agents/README.md`
- `models/.gitkeep`, `models/README.md`
- `skills/.gitkeep`, `skills/README.md`
- `themes/.gitkeep`, `themes/README.md`
- `commands/.gitkeep`, `commands/README.md`
- `plugins/todo/*` (moved)

**Estimated scope:** M: 5+ files (moves + stubs)

## Task 4: Convert to bun workspaces (root + plugins/todo/package.json, tsconfig)

**Description:** Make root `package.json` a private workspace root and give `plugins/todo/` its own package manifest; update `tsconfig.json` includes and verify entrypoint re-exports still resolve.

**Acceptance criteria:**
- [ ] Root `package.json`: `name: "opencode2"`, `private: true`, `workspaces: ["plugins/*", "agents/*", "models/*", "skills/*", "themes/*", "commands/*"]`, no runtime deps
- [ ] `plugins/todo/package.json`: keeps `name: "opencode2-todo"`, version, deps, scripts
- [ ] `plugins/todo/tsconfig.json` (or root tsconfig with updated `include`) typechecks
- [ ] `bun install` succeeds; `bun test` still 25 pass; `tsc --noEmit` clean

**Verification:**
- [ ] Tests pass: `bun test` (from root and `plugins/todo/`)
- [ ] Build succeeds: `npx tsc --noEmit`
- [ ] Manual check: `cat package.json` shows workspaces

**Dependencies:** Task 3

**Files likely touched:**
- `package.json` (root rewrite)
- `plugins/todo/package.json` (moved + kept)
- `tsconfig.json` / `plugins/todo/tsconfig.json`
- `bun.lock` (regenerated if needed)

**Estimated scope:** M: 3-5 files

## Checkpoint: After Tasks 3-4
- [ ] All tests pass (`bun test` 25 pass)
- [ ] Typecheck clean (`tsc --noEmit`)
- [ ] Plugin entrypoints `plugins/todo/src/index.ts` + `plugins/todo/src/tui.tsx` resolve

## Task 5: Fix repository/homepage/bugs URLs, README paths, root README

**Description:** Point all repo metadata to `ckir/opencode2`, update README absolute-path example to monorepo path, write minimal root README index linking to `plugins/todo/`.

**Acceptance criteria:**
- [ ] `plugins/todo/package.json` `repository.url` = `git+https://github.com/ckir/opencode2.git`, `homepage` = `https://github.com/ckir/opencode2#readme` (or `.../tree/main/plugins/todo`), `bugs` = `https://github.com/ckir/opencode2/issues`
- [ ] No `gabparrot` or `opencode2-todo-tool` references remain in `*.json`/`*.md` (except historical note if kept)
- [ ] `plugins/todo/README.md` absolute-path example uses `.../opencode2/plugins/todo/src/index.ts`
- [ ] Root `README.md` describes monorepo categories + links to `plugins/todo/`
- [ ] Bare package name `opencode2-todo` install docs unchanged (package name preserved)

**Verification:**
- [ ] Run: `grep -rn "gabparrot\|opencode2-todo-tool" --include="*.json" --include="*.md" . | grep -v node_modules | grep -v tasks/` returns empty (or only intentional history note)
- [ ] Run: `grep -rn "ckir/opencode2" package.json plugins/todo/package.json README.md plugins/todo/README.md`

**Dependencies:** Task 4

**Files likely touched:**
- `plugins/todo/package.json`
- `plugins/todo/README.md`
- `README.md` (root rewrite)

**Estimated scope:** S: 2-3 files

## Task 6: Commit + push (atomic commits)

**Description:** Commit the rename/restructure in atomic, reviewable commits per git-workflow skill and push to `origin`.

**Acceptance criteria:**
- [ ] Commits: (1) monorepo move, (2) workspace conversion, (3) metadata/URL fixes — each one logical thing with `type: subject` message explaining why
- [ ] Pre-commit hygiene: no secrets in diff, `bun test` green, `tsc --noEmit` clean, `.gitignore` covers `node_modules/`
- [ ] Push succeeds to `origin` on current branch (or `main` if agreed)

**Verification:**
- [ ] Run: `git log --oneline -5 && git status --short --branch`
- [ ] Run: `git diff --staged | grep -i "password\\|secret\\|api_key\\|token"` returns empty
- [ ] Tests pass: `bun test`

**Dependencies:** Task 5

**Files likely touched:**
- (git history only)

**Estimated scope:** XS

## Checkpoint: Complete
- [ ] All acceptance criteria met
- [ ] `git log --oneline -5` shows clean atomic history
- [ ] Ready for review
