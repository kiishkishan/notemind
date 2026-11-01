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

## UI components: shadcn/ui on Base UI (not Radix)

- **Decision:** shadcn/ui with the `base` component library (Base UI, `@base-ui/react`), preset Nova (Lucide icons, Geist font), neutral base color. Components live in `apps/web/src/components/ui`.
- **Why:** Base UI ships Combobox, Autocomplete and Number Field natively; Radix does not. Those are likely needed for thread/document pickers and model settings.
- **Trade-offs:** fewer community examples than Radix, and some API differences (for example a `render` prop instead of `asChild`), so Radix snippets may not copy over. Components are copied into the repo, so they can be adjusted later.
- **Lint:** `react-refresh/only-export-components` is off for `src/components/ui/**` only, because shadcn files export variants (for example `buttonVariants`) alongside the component. It stays on for hand-written code.
- **Revisit when:** a needed component is missing or awkward in Base UI.
