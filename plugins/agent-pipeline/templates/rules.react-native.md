# Pipeline rules — React Native (bare) / TypeScript

For a bare React Native app (no Expo). If the project uses Expo, most of this still
applies but the native-build and autolinking sections do not.

**Playwright cannot test this app.** It drives browsers over the DevTools protocol;
React Native renders native views with no DOM to attach to. This is an architectural
mismatch, not a configuration gap — if a task proposes Playwright for the mobile app,
reject it. (Playwright is correct for a web app in the same monorepo.)

## Architecture / conventions

- Style guide: ESLint flat config + Prettier.
- Layout: one folder per screen under `src/screens/<Screen>/`, colocating its components
  and tests. Shared UI in `src/components/`, navigators and route param types in
  `src/navigation/`, platform wrappers in their own folders (`src/storage/`, `src/media/`).
- Business rules live apart from UI — in `src/domain/`, or in a shared package if a web
  app or backend needs the same rules. Three copies of "is this token still valid?" is
  how three behaviours diverge.
- Do NOT modify: `ios/Pods/`, `android/.gradle/`, `android/app/build/`, `*.generated.ts`,
  anything under `node_modules/`.
- `ios/` and `android/` are source, not build output. Changes there are reviewable code
  and must be explained in the task, not treated as incidental churn.

## Monorepo traps (pnpm + Metro)

Skip this section if the app is not in a workspace. If it is, these are not style
preferences — each one is an app that does not start.

- **`node-linker=hoisted` in `.npmrc` is mandatory.** pnpm's default symlinked layout
  breaks Metro resolution and breaks native autolinking (Gradle and CocoaPods scan
  `node_modules` for native code and do not understand pnpm's structure — they silently
  link nothing and fail at runtime, not at build time).
- **Do NOT set `resolver.disableHierarchicalLookup: true`** in `metro.config.js`, even
  alongside an explicit `nodeModulesPaths`. Hoisting only hoists *direct* dependencies;
  a transitive-only package like `invariant` (required by react-native itself) stays in
  the store and is reachable only by walking up from react-native's own directory. That
  walk is hierarchical lookup. Disabling it produces
  `Unable to resolve module invariant from .../react-native/index.js` at startup.
- Metro needs `watchFolders: [workspaceRoot]` and `nodeModulesPaths` listing the app's
  own `node_modules` before the workspace root's.
- **Jest's `transformIgnorePatterns` default does not work under pnpm.** The stock
  pattern anchors the package name right after `node_modules/`, but the real path is
  `node_modules/.pnpm/<name>@<ver>_<hash>/node_modules/<name>/…`, so every RN package is
  skipped and the preset throws
  `SyntaxError: Cannot use import statement outside a module`. Match anywhere in the
  path instead: `node_modules/(?!.*(?:react-native|@react-navigation))`.

## Test

- Framework: **Jest** with **`@testing-library/react-native`** (not `@testing-library/react`).
- Suite command: `pnpm test` (or the repo's equivalent).
- Recent RNTL + React 19 needs `await render(...)` and `await fireEvent.press(...)`.
  A test that forgets the `await` usually fails for a confusing unrelated reason.
- Register matchers by importing the package root; the old `/extend-expect` subpath was
  removed in v14.
- Native modules with no JS fallback (Keychain, MMKV, camera, bootsplash) must be mocked
  globally in a setup file, or importing them throws under Jest.
- Coverage target: 80% on new code, but prefer one test that would actually fail against
  the bug over three that raise the number.
- Every test needs a positive assertion. `expect(fn).toHaveBeenCalled()` alone proves
  almost nothing.
- **A test only counts if it fails against the broken version.** When fixing a bug, watch
  the new test fail before the fix lands. When reviewing, be suspicious of any test added
  alongside a fix with no evidence it discriminates.
- E2E: **Maestro** (YAML flows in `.maestro/`), not Detox — no native build step and no
  test-runner integration, so far less to maintain. Reserve E2E for what only a device
  can prove: camera capture, secure-storage persistence across a real app kill, airplane
  mode. Anything a component test covers does not need a flow.

## Quality gate before the Coder moves a task to review

Run in order, all must pass clean:

```bash
pnpm typecheck
pnpm lint
pnpm test
```

If the task touched anything under `ios/` or `android/`, a native build must also succeed
— a JS-only check cannot catch a broken pod or a missing autolink.

## State and storage

- Server state belongs to a query library (React Query or similar). Do not copy it into
  a local store; read from the query, mutate through the mutation.
- Local store (Zustand or similar) holds UI-only state, plus durable preferences via its
  persist middleware.
- **Storage is not interchangeable.** Auth tokens go to Keychain / EncryptedSharedPreferences
  and nowhere else — phones get lost and rooted, and an unencrypted token is a live
  session for whoever finds it. Caches and non-secret preferences go to MMKV or
  AsyncStorage, where the worst case is stale data.
- **One retry authority.** If the app has an offline mutation queue, the query library's
  own mutation retry must be off (`retry: false`, no `resumePausedMutations`). Two retry
  mechanisms means every write can fire twice.
- Every offline-queued write carries a client-generated idempotency key, sent unchanged
  on each retry, so a retry after a flaky timeout cannot double-apply.

## Review checklist (extra)

- Lists use `FlatList`/`FlashList` with a stable `keyExtractor` and memoized rows — not
  `.map()` inside a `ScrollView`.
- Store selectors pick the narrowest slice needed; a component subscribed to the whole
  store re-renders on every unrelated change.
- Animations use `useNativeDriver: true` for transform and opacity, so they survive a
  busy JS thread.
- Every `setTimeout`/`setInterval`/subscription is cleared on unmount and on every early
  return. A timer outliving its screen fires into a torn-down tree.
- Async work started in an effect is guarded so a superseded run cannot write state or
  navigate after the component moved on.
- No `aria-*` props and no `dangerouslySetInnerHTML` — neither exists in React Native.
  Use `accessibilityRole`, `accessibilityLabel`, `accessibilityLiveRegion`.
- `value ?? fallback`, not `value || fallback`, when `0` or `''` is a legitimate value.
- No hardcoded secrets, no production URLs, no default `React` import in `.tsx`.

## Field and accessibility rules

Assume the reader is outdoors, one-handed, possibly gloved, possibly in direct sun.

- Touch targets at least 44×44 pt.
- **Status is never conveyed by colour alone** — always pair colour with an icon or a
  text label.
- Body text meets WCAG AA contrast (4.5:1). Check it rather than assuming: white on a
  mid-tone brand colour frequently fails, and the screens most likely to fail are error
  states, which is exactly when legibility matters most.
- Screens respect OS font scaling without clipping. Centred fixed-height blocks overflow
  at large accessibility sizes.
- State changes that matter (an error appearing, a submission confirmed) are announced,
  not only rendered.

## Verification standards

Two rules that exist because their absence has shipped real bugs:

- **Verify Metro with the real entry point.** Bundling a synthetic file that imports only
  the module under test proves that module resolves; it does not pull `react-native`
  into the graph, so it cannot catch a broken RN resolution. Bundle `index.js`.
- **Colour matches are measured, never eyeballed.** Two surfaces meant to be identical
  (a native launch screen and the JS screen replacing it) must be verified by sampling
  the same pixel in a screenshot of each and comparing RGB triples. "Both look blue" is
  how a visible seam ships. Note that a native storyboard colour and a JS hex can render
  to different pixels from the same value, because one path is colour-managed and the
  other is not.

## Security

- Tokens never reach logs, crash reports, analytics, or breadcrumb HTTP headers. If a
  crash reporter is wired up, scrub before send rather than trusting defaults, and match
  key names by normalized substring — `access_token`, `Access-Token`, and `accessToken`
  are the same secret.
- User-captured media (photos, documents) may have its file *path* logged as context;
  the file *contents* must never be uploaded to a third-party service.
- Personal data belonging to the end user or their customers is not ours to ship
  off-device for debugging convenience.
- Do not put personal data in URL query strings or deep links.

## Ticket / commit

- Branch: `feature/<short-slug>` or `fix/<short-slug>`.
- Commit: conventional commits (`feat:`, `fix:`, `chore:`, `test:`, `refactor:`, `docs:`,
  `build:`). Scope by workspace package where the repo defines scopes.
- Never `--no-verify`. A failing hook is a finding, not an obstacle.
- Anything a human reads outside the repo — ticket comments, PR descriptions, QA handoff
  notes — goes through the `human-ticket-voice` skill.

## Notes

- If the repo has its own conventions document (a `SKILL.md`, a `CONTRIBUTING.md`, an
  architecture note), that file is the source of truth for code style and layout, and
  this file should point at it rather than restate it. Two copies of a convention drift.
- Native dependency changes need `pod install` on iOS and a Gradle sync on Android.
  Neither happens automatically after `pnpm install`.
