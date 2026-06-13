---
tags:
  - npm
  - offline
  - verdaccio
  - mimo2codex
  - better-sqlite3
  - native-module
  - prebuild
  - dependency-tree
  - node-gyp
  - air-gap
  - package-caching
  - 20260613
  - npm_src
---

# npm_src

> Last updated: 17:24:30

### Core Work
- **Added `mimo2codex` (v0.5.27) to offline npm Verdaccio cache** — a MiMo → OpenAI Codex proxy CLI tool
- **Iteratively fixed missing transitive dependencies**: first attempt only cached 4 direct deps, missing 37 transitive deps including `bindings@1.5.0` (pulled by `better-sqlite3`). Second attempt cached the full 42-package dependency tree.
- **Solved `better-sqlite3` native compilation failure in offline env**: `better-sqlite3` is a native C++ module. In offline, `prebuild-install` times out (no GitHub access) and `node-gyp rebuild` fails (no nodejs.org header download). Solution: bundle precompiled `.node` binary matching target Node.js ABI.
- **Final deliverable**: `mimo2codex-offline-20260613-160859.tar.gz` (6.8MB) containing 42 npm packages + prebuilt `better-sqlite3.node` for Node.js v24 (ABI 141) linux-x64 + `install-mimo2codex.sh` script

### Supporting Work
- Created online Verdaccio config with upstream proxy to fetch packages from npm
- Started HTTP server to transfer package to Windows (SCP failed — Windows no SSH server)
- Updated local Node.js to v24.13.1 (via `n`) to match target offline machine's ABI
- Deleted incomplete first package (`npm-new-mimo2codex-20260612-205203.tar.gz`, 4.7MB)

### Key Decisions & Rationale
- **Decision**: Not packaging full Verdaccio, only the 5 new packages' storage → **Rationale**: User explicitly asked to only package the incremental new packages, not the entire ~500MB Verdaccio storage
- **Decision**: Bundle precompiled native binary rather than source-compile on target → **Rationale**: Offline target machine cannot download node-gyp headers or prebuilt binaries; shipping the binary is the only reliable approach
- **Decision**: Use `--ignore-scripts` during npm install on offline machine + manually place `.node` file → **Rationale**: `npm install` triggers `prebuild-install || node-gyp rebuild` which both fail offline; skipping scripts + manual binary placement avoids the compile step entirely

### User Notes
- Target offline machine: `hjsw` (Linux, Node.js v24.13.1, Python 3.10.12)
- Target Verdaccio: `http://192.168.10.20:4873`
- User's Windows machine: `10.72.104.56`, username `jialong.yang`
- This is a recurring pattern: `tiktoken` offline cache problem was solved previously (similar approach — pre-cache the blob file)
- Always verify with `--ignore-scripts` + offline install test before declaring success for npm packages with native modules

### Version Log
- `mimo2codex-offline-20260613-160859.tar.gz` (6.8MB) — Final: 42 npm packages + prebuilt `better-sqlite3` binary for Node.js v24 ABI 141, with install script
- `npm-new-mimo2codex-20260613-105952.tar.gz` (5.7MB) — Second: 42 packages (complete dependency tree), but no native binary
- `npm-new-mimo2codex-20260612-205203.tar.gz` (4.7MB) — First: only 5 packages (missing transitive deps), deleted
