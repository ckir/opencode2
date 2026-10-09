# Implementation Plan: Rename to opencode2 + Monorepo Skeleton

## Overview
GitHub rename `opencode2-todo` → `opencode2` is already done (`ckir/opencode2`, origin URL already `https://github.com/ckir/opencode2.git`). Remaining work: rename local directory to match, convert the single-plugin repo into a monorepo home for opencode v2 developments with top-level categories `agents/ models/ skills/ themes/ commands/ plugins/`, and move the existing todo plugin to `plugins/todo/` without breaking tests or runtime loading. Root `package.json` repository/homepage/bugs URLs still point to old upstream `gabparrot/opencode2-todo-tool` and must point to `ckir/opencode2`.

## Architecture Decisions
- **Keep npm package name `opencode2-todo` for now.** Repo rename ≠ package rename. Renaming the published package would break `plugins: ["opencode2-todo"]` consumers. Monorepo path will be `plugins/todo/` but package name stays until we decide scoped names (e.g. `@opencode2/todo`). Rationale: minimal breakage, reversible.
- **Bun workspaces for monorepo.** Repo already uses `bun.lock` + `bun test`. Root `package.json` becomes `private: true` with `workspaces: ["plugins/*", "agents/*", "models/*", "skills/*", "themes/*", "commands/*"]`. Initially only `plugins/todo` is populated; empty category dirs get `.gitkeep` + per-category README stub.
- **`git mv` for history preservation.** Move `src/ tool/ index.ts tui.tsx` → `plugins/todo/` via `git mv` so `git log --follow` retains history.
- **Local directory rename via filesystem `mv` + `session_move`.** `/home/user/Development/Typescript/opencode2-todo` → `.../opencode2`. Git remote already correct, no remote change needed.
- **Update only repo URLs, not behavior.** `repository.url`, `homepage`, `bugs` → `https://github.com/ckir/opencode2`. No code changes to plugin logic. README absolute-path example updated to new monorepo path.
- **Atomic commits per git-workflow skill.** One logical change per commit: (1) dir rename, (2) monorepo move, (3) metadata/URL fixes, (4) docs.

## Task List

### Phase 1: Rename alignment
- [ ] Task 1: Rename local directory to `opencode2` and move session
- [ ] Task 2: Verify git remote + GitHub state

### Checkpoint: Rename
- [ ] `pwd` shows `.../opencode2`, `git remote -v` shows `ckir/opencode2.git`, `gh repo view` shows `ckir/opencode2`, working tree clean

### Phase 2: Monorepo skeleton + move
- [ ] Task 3: Create category dirs + `git mv` plugin to `plugins/todo/`
- [ ] Task 4: Convert to bun workspaces (root + `plugins/todo/package.json`, tsconfig updates)

### Checkpoint: Structure
- [ ] `bun test` green, `tsc --noEmit` clean from both root and `plugins/todo/`

### Phase 3: Metadata + docs
- [ ] Task 5: Fix repository/homepage/bugs URLs, README paths, root README
- [ ] Task 6: Commit + push (atomic commits)

### Checkpoint: Complete
- [ ] All acceptance criteria met, `git log --oneline -5` shows clean atomic history, ready for review

## Risks and Mitigations
| Risk | Impact | Mitigation |
|------|--------|------------|
| Local `mv` breaks open session CWD | Med | Use `session_move` immediately after `mv`; verify `pwd` |
| `git mv` + tsconfig `include` paths break typecheck | High | Update `include` + entrypoint re-exports; verify with `tsc --noEmit` before commit |
| Bun workspace resolution breaks plugin loading (`@opencode/plugin`) | Med | Keep `plugins/todo` as workspace with own deps; run `bun install` + `bun test` after move |
| README absolute-path example stale | Low | Update to `.../opencode2/plugins/todo/src/index.ts`; keep bare-name `opencode2-todo` install docs unchanged |
| Empty category dirs not tracked by git | Low | Add `.gitkeep` + stub README per category |

## Open Questions
- Scoped package names later (`@opencode2/todo`)? — deferred, keep `opencode2-todo` for now.
- LICENSE copyright stays `gabparrot` (vendored upstream) or add `ckir`? — keep as-is this pass, revisit.
- Root README content depth? — minimal monorepo index + link to `plugins/todo/README.md` this pass.
