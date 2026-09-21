# Pipeline rules — Generic React / TypeScript project

## Architecture / conventions

- Style guide: ESLint (project config) + Prettier default.
- Pattern: feature-folder (`src/features/<feature>/*`), colocate tests next to source (`Foo.test.tsx` next to `Foo.tsx`).
- Files that must NOT be modified: `dist/`, `build/`, `*.generated.ts`, `openapi.json` (if generated), `node_modules/`.

## Test

- Framework: **vitest** with `@testing-library/react` for components.
- Suite command: `pnpm test` (or `npm test` / `yarn test` depending on the project).
- Coverage target: **80% on new code** (`pnpm test --coverage`).
- Mocking: mock APIs via MSW or `vi.fn()`; do not mock React Router or internal hooks unless necessary.
- Each test must contain a **positive assertion** (not just `expect(fn).toHaveBeenCalled()`), and cover both the golden path and one edge case.

## Quality gate before the Coder moves a task to review

Run in order, must pass with zero warnings:
```bash
pnpm typecheck
pnpm lint
pnpm test --run
```

## Review checklist (extra)

- Rendered lists have unique, stable `key`s (do not use index if the list can reorder).
- Async regions expose `aria-busy` while loading.
- Nullish default: `value ?? defaultValue`, not `value || defaultValue` when `''` or `0` is a valid value.
- Enum / lookup map: null-guard before access.
- Do not default-import `React` in `.tsx` (the new JSX transform doesn't need it).
- No hardcoded secrets or production URLs.

## Security

- Sanitize user input before using `dangerouslySetInnerHTML`.
- Do not put PII in URL query strings.

## Ticket / commit

- Branch: `feature/<short-slug>` or `fix/<short-slug>`.
- Commit: conventional commits (`feat:`, `fix:`, `chore:`, `test:`, `refactor:`, `docs:`).
- No AI attribution in commit messages.

## Notes

- Component reuse: check `packages/ui` first (in a monorepo) before writing a new one.
