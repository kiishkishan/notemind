# Decision log

## Pin TypeScript to 6.0 (web)

- **Decision:** `apps/web` uses `typescript@~6.0.3`, not 7.x.
- **Why:** `typescript-eslint@8.72.1` caps its peer range at `<6.1.0` and throws at startup on TS 7, so `eslint .` fails. Vite's template had installed 7.0.2.
- **Alternatives considered:**
  - `peerDependencyRules` to allow TS 7: only hides the warning; lint still crashes.
  - Side-by-side aliasing (`typescript` -> `@typescript/typescript6`, `@typescript/native` -> `typescript@^7`): works and keeps the native compiler, but makes `package.json` non-obvious.
- **Cost:** Loses TS 7's native compiler speed. A clean `tsc -b` takes about 1.3s today, and Vite does not use `tsc` to transpile, so build and dev speed are unaffected.
- **Revisit when:** typescript-eslint supports TS >=7.1 (tracking: https://github.com/typescript-eslint/typescript-eslint/issues/10940), or when `tsc -b` exceeds about 10s.

## Branching strategy: lightweight Git Flow

- **Decision:** `main` is production (deployed, tagged with semver). `develop` is the default integration branch. Work happens on short-lived `<type>/<phase>-<topic>` branches off `develop`, with hotfixes branching from `main`.
- **Why:** `main` becomes a live deployment at the end of Phase 0 and should not break while features are in flight. `develop` gives a stable place to integrate, and Vercel PR previews cover testing before merge.
- **Merging:** topic into `develop` with rebase-and-merge (keeps atomic commits, no squash). `develop` into `main` with a merge commit plus a tag.
- **Commits:** Conventional Commits, atomic, no attribution trailers. Full rules live in `CLAUDE.md`.
- **Trade-off:** Git Flow is more overhead than trunk-based development for a solo project. Accepted because the deployed `main` needs protecting and the habit is worth practicing.
- **Revisit when:** the two-branch overhead outweighs the protection, for example if `develop` and `main` never diverge in practice.
