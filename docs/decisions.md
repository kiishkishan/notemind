# Decision log

## Pin TypeScript to 6.0 (web)

- **Decision:** `apps/web` uses `typescript@~6.0.3`, not 7.x.
- **Why:** `typescript-eslint@8.72.1` caps its peer range at `<6.1.0` and throws at startup on TS 7, so `eslint .` fails. Vite's template had installed 7.0.2.
- **Alternatives considered:**
  - `peerDependencyRules` to allow TS 7: only hides the warning; lint still crashes.
  - Side-by-side aliasing (`typescript` -> `@typescript/typescript6`, `@typescript/native` -> `typescript@^7`): works and keeps the native compiler, but makes `package.json` non-obvious.
- **Cost:** Loses TS 7's native compiler speed. A clean `tsc -b` takes about 1.3s today, and Vite does not use `tsc` to transpile, so build and dev speed are unaffected.
- **Revisit when:** typescript-eslint supports TS >=7.1 (tracking: https://github.com/typescript-eslint/typescript-eslint/issues/10940), or when `tsc -b` exceeds about 10s.
