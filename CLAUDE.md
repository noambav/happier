# Claude Code

@AGENTS.md

## TypeScript commands

Follow the canonical TypeScript compiler-ownership rules in `AGENTS.md`. Use repository/package typecheck and build scripts; `yarn tsc ...` is safe from the repository root and every TypeScript-owning workspace, and delegates to the native compiler runner. For a direct ad hoc check, use `node scripts/workspaces/runTypeScriptCli.mjs ...`. Do not run bare `tsc`, `npx tsc`, `node_modules/.bin/tsc`, or `typescript/bin/tsc`: compilation must use the centrally resolved native TypeScript 7 compiler, while the `typescript` package remains TypeScript 5.9 only for programmatic API compatibility.

<!-- ===== Bernard's fork — everything below this line is ours, not upstream's. =====
     Kept at EOF so upstream merges conflict minimally. Inlined rather than imported
     because an @import inside a --add-dir'd CLAUDE.md does not resolve. -->

## Bernard's fork

This is Bernard's fork of `happier-dev/happier` (MIT). Everything above this heading is upstream's.

### Branches

- `upstream` is the vendor remote; `origin` is Bernard's fork.
- `main` mirrors **`upstream/dev`**, upstream's default branch. Upstream *also* has its own `main`,
  which we deliberately do not track — the names are confusing, and that is worth knowing before
  you pull the wrong one.
- Never commit to `main`. All Bernard work lands on `bernard`.
- Sync is: fetch `upstream`, fast-forward `main`, merge `main` into `bernard`. **The merge commits
  are the sync log** — do not keep a second one.

### The invariant that matters

**The relay image and the iPhone client ship from the same git sha.** Both halves of an end-to-end
encrypted protocol are forked here, so version skew between them is a Bernard-owned bug rather than
an upstream one. Rebuilding one without the other is the mistake this line exists to prevent.

### Building here

- `node_modules` is a symlink to `~/.cache/happier-build/node_modules`, and Yarn must run with
  `YARN_CACHE_FOLDER=~/.cache/happier-build/yarn`. Both live on the NVMe system disk because
  `/srv/bernard` is a 5400 rpm SMR drive (see `/srv/bernard/home/HOST.md`). They are disposable —
  deleting them costs only rebuild time.
- **Node 22 or newer is required.** `packages/relay-server` declares `engines.node >=22` while the
  root `package.json` declares nothing, so checking only the root reports a false all-clear. That
  is exactly how the first install here failed.
- A failed install can leave `node_modules/@happier-dev/*` behind and the next run dies with
  `EEXIST ... mkdir`. Clear the tree and reinstall rather than retrying in place.
- The relay image builds entirely inside Docker (`ARG NODE_VERSION=22`), so the image build does
  not depend on the host's Node at all. Only host-side `yarn build` does.
- Docker on this host is **rootless**; no `docker` command needs `sudo`. The relay deployment and
  its load-bearing `user: "0:0"` are owned by `/srv/bernard/home/control/happier/CLAUDE.md`.
