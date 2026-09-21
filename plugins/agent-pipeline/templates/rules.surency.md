# Pipeline rules — Surency Admin Hub

Surency-specific rules distilled from `.claude/rules/`, `.claude/skills/coderabbit-review-guard/SKILL.md`, `.claude/skills/add-feature/SKILL.md`, and long-standing repo memory.

## Backend (`src/Surency.*` — ABP / C#)

### Authorization stance (mandatory — enforced by convention tests)

- Every controller action MUST carry `[Authorize]` / `[Authorize(Roles=...)]` / `[AllowAnonymous]` (no reliance on the global fallback).
- Roles come from `SurencyRoles` constants: `SurencyAdmin`, `Broker`, `Employer`, `AdminOrBroker`, `AdminOrEmployer`. NEVER inline `"role:..."` string literals.
- Any action reachable by a broker or employer MUST carry `[EmployerOwnershipScoped]` OR `[OwnershipScopeExempt("reason")]`.
- `EmployerController` is class-scoped `AdminOrBroker` — a new action defaults to broker-reachable. Re-narrow to `SurencyAdmin` at the method level unless the ticket explicitly grants brokers access.
- Broker-reachable actions with an `employerId` MUST call `IEmployerOwnershipGuard.CheckAsync(employerId, level)`.

### Data-layer employer isolation (US-1336)

- New entity with a direct `EmployerId` → implement `IEmployerOwned`.
- Child entity (no EmployerId, owned via a parent) → add a chain filter in `SurencyDbContext.EmployerScope.cs` + an entry in `s_chainWriteGuards` + coverage in `EmployerScopeIsolationTests`.

### New-entity checklist (mandatory)

- [ ] `IHasAuditLog` marker.
- [ ] `[LogMasked]` on every PHI/PII property (name, email, phone, SSN/EIN, DOB, member ID, claims data).
- [ ] `IEmployerOwned` or a chain guard (see above).
- [ ] Table name WITHOUT prefix (`Groups`, not `AppGroups`); index names without `App`.
- [ ] EF migration generated, reviewed, and committed together with the entity.

### Domain

- Status enums are NOT uniform: Employer/Group use `Active = 3` (5-state lifecycle); BrokerAgency uses `Active = 1` (binary). ALWAYS read the enum file before writing a filter/badge.
- User-facing failure → `UserFriendlyException`. The SIS/Refit wrapper is the ONLY place that catches `Refit.ApiException`.
- Authorization failure → `AbpAuthorizationException`.

### ABP app-service default-deny (US-1335/1336)

- `/api/app/*` auto-routes are OFF by default. A new app service is ONLY exposed when it carries `[ExposeConventionalApi]` — prefer writing a thin manual controller.
- Public methods on an exposed app service become endpoints. Internal helpers MUST be `[RemoteService(false)]` or non-public.

## Web (`apps/web` — React/TypeScript)

### CodeRabbit recurring findings (run this checklist BEFORE pushing)

- **List key**: `.map(item => <Row key={item.id} />)` — do not use index if the list can reorder.
- **openapi-fetch `*Api.ts`**: destructure `{ data, error }` and handle `error` — do not throw a generic exception.
- **Null / out-of-range enum guard**: `EnumMap[value] ?? fallback`, do not assume the value is always in the map.
- **Nullish default**: `value ?? default` not `value || default` when `''` / `0` is a valid value.
- **Test assertions**: use positive assertions (`expect(x).toBe(y)`), not just `toHaveBeenCalled()`. Cover the golden path AND an edge case.
- **Nullable enum in OpenAPI**: nullable enums must emit `type: ["string", "null"]`.
- **Design tokens**: use CSS variables from `@surency/ui` (charcoal, sky, etc.), never hardcoded hex.
- **Paged grid**: when filter/search changes, reset page to 1 and clamp `page ≤ maxPage`.
- **aria-busy** on async regions (loading / fetching).
- **AG Grid**: tokens for colors; row height per the standard (`rowHeightStandard`).

### Accessibility (WCAG 2.2 AA — mandatory)

- Target size ≥ 24×24px (pad if the visual is smaller).
- Focus not obscured by sticky header/footer.
- Drag-and-drop has a single-pointer alternative (Move Up/Down buttons).
- Forms: `<label>` or `aria-labelledby` on EVERY input; error `aria-invalid="true"` + `aria-describedby`.
- Contrast: text ≥ 4.5:1 (large text ≥ 3:1). Do not use color ALONE to convey state.
- Images: meaningful `alt`, or `alt=""` / `aria-hidden="true"` when decorative.
- Toast/loading/route change: `aria-live="polite"` or `aria-live="assertive"`.

### UI conventions

- Reuse components from `@surency/ui` before writing a new one.
- Empty placeholder is `-` (hyphen), NOT `—` (em dash).
- Date display: `formatUsDate(...)`, not `new Date().toLocaleDateString()` (backend sends `Z`, which shifts the day for US users).
- PHI mask: gate the field with `[Phi(Ssn)]` on the backend and render `***-**-1234` client-side.
- Row kebab / action dropdowns: use the `AnchoredMenu` portal (not the DataTable default menu).
- Charcoal contrast: `/70` in white cards, `/80` on gradient-start surfaces, `/60` always for placeholders.
- Detail-page pattern: Outlet tabs + breadcrumb + plain title row + Edit only on the overview tab.

## Test

- Backend: xUnit + NSubstitute + Shouldly, direct instantiation (no ABP TestBase).
- Web: vitest + `@testing-library/react`.
- Coverage: **80% on new code** (Sonar new-code gate — three projects, zero new violations).

## Quality gate before the Coder moves a task to review

Backend:
```bash
dotnet build src/Surency.slnx  # zero warnings
dotnet test src/Surency.slnx   # zero failures
```

Web:
```bash
pnpm --filter web typecheck
pnpm --filter web lint
pnpm --filter web test --run
```

## Ticket / commit / branch

- Branch: `feature/us-<n>-<kebab-case-title>` (e.g. `feature/us-593-confirm-user-details-on-first-login`). If the work touches both admin-hub and identity → create the same branch in BOTH repos.
- Commit: conventional commits, NO AI/Claude attribution.
- Commit only to the feature branch; do NOT push unless the user asks.
- NEVER stage: secrets, `appsettings.*.local.*`, logs, `.DS_Store`, `.claude/settings.local.json`.
- Code comments must NOT contain `US-XXXX / PR#XXXX / date / BA name`; keep only the timeless reason.
- Code and comments are English; chat replies to the user are Vietnamese.

## Notes

- Test frameworks: xUnit backend, vitest web. NO DB mocks on the backend (use EF in-memory sqlite per the `EmployerScopeIsolationTests` pattern).
- A graphify graph exists at `graphify-out/graph.json` — use `graphify query "..."` instead of wide Grep.
- OpenAPI regen (`pnpm run generate`): scope to the feature's own files. A full regen must be merged manually to avoid drift.
