# Pipeline rules — Enterprise ABP monorepo (React + .NET)

A strict preset for an enterprise ABP monorepo where the backend is C# / ABP with multi-tenant data isolation and PHI/PII handling, and the frontend is React/TypeScript consuming a typed OpenAPI client from a shared package.

Rename identifiers below (`AppRoles`, `AppDbContext`, `@myorg/ui`, `App.slnx`, `TenantId`, etc.) to match your project.

## Backend (`src/App.*` — ABP / C#)

### Authorization stance (mandatory — enforce with a convention test)

- Every controller action MUST carry `[Authorize]` / `[Authorize(Roles=...)]` / `[AllowAnonymous]` (no reliance on the global fallback).
- Roles come from an `AppRoles` constants class. NEVER inline `"role:..."` string literals.
- Any action reachable by an external persona (partner, tenant admin) MUST carry either an "ownership-scoped" attribute (that the framework enforces) OR an explicit `[OwnershipScopeExempt("reason")]` marker.
- Controller classes that grant broad role access (e.g. `TenantAdminController` scoped `AdminOrPartner`) leak that role to every method. New actions must re-narrow at the method level unless the ticket explicitly grants the wider role.
- Any action a partner can call with a `<tenantId>` MUST call the ownership guard (`ITenantOwnershipGuard.CheckAsync(tenantId, level)`).

### Data-layer tenant isolation

- New entity with a direct `TenantId` → implement `ITenantOwned`; the EF global query filter and SaveChanges write-guard then apply automatically.
- Child entity (no `TenantId`, owned via a parent) → add a chain filter in `AppDbContext.TenantScope.cs` + an entry in `s_chainWriteGuards` + coverage in `TenantScopeIsolationTests`.
- Never suppress the filter except via a scoped `IDataFilter.Disable<ITenantOwned>()` block with a comment explaining why.

### New-entity checklist (mandatory)

- [ ] `IHasAuditLog` marker.
- [ ] `[LogMasked]` (or the equivalent PII attribute) on every PHI/PII property: name, email, phone, SSN/EIN, DOB, member ID, claims data.
- [ ] `ITenantOwned` or a chain guard (see above).
- [ ] Table name WITHOUT prefix (`Groups`, not `AppGroups`); index names without `App`.
- [ ] EF migration generated, reviewed, and committed together with the entity.

### Domain

- Status enums are NOT uniform across the codebase (some lifecycles use `Active = 3`, some binary flags use `Active = 1`). ALWAYS read the enum file before writing a filter/badge, do not assume a value from memory.
- User-facing failure → `UserFriendlyException`. Only the outbound-service (Refit/HttpClient) wrapper catches `Refit.ApiException` and rethrows as `UserFriendlyException`.
- Authorization failure → `AbpAuthorizationException`.

### ABP app-service default-deny

- `/api/app/*` auto-routes are OFF by default. A new app service is ONLY exposed when it carries `[ExposeConventionalApi]` — prefer writing a thin manual controller.
- Public methods on an exposed app service become endpoints. Internal helpers MUST be `[RemoteService(false)]` or non-public.

## Web (`apps/web` — React/TypeScript)

### Automated-review recurring findings (checklist BEFORE pushing)

- **List key**: `.map(item => <Row key={item.id} />)` — do not use index if the list can reorder.
- **openapi-fetch `*Api.ts`**: destructure `{ data, error }` and handle `error` — do not throw a generic exception.
- **Null / out-of-range enum guard**: `EnumMap[value] ?? fallback`, do not assume the value is always in the map.
- **Nullish default**: `value ?? default` not `value || default` when `''` / `0` is a valid value.
- **Test assertions**: use positive assertions (`expect(x).toBe(y)`), not just `toHaveBeenCalled()`. Cover the golden path AND an edge case.
- **Nullable enum in OpenAPI**: nullable enums must emit `type: ["string", "null"]`.
- **Design tokens**: use CSS variables from `@myorg/ui`, never hardcoded hex.
- **Paged grid**: when filter/search changes, reset page to 1 and clamp `page ≤ maxPage`.
- **aria-busy** on async regions (loading / fetching).
- **AG Grid**: tokens for colors; standard row height from the design system.

### Accessibility (WCAG 2.2 AA — mandatory)

- Target size ≥ 24×24px (pad if the visual is smaller).
- Focus not obscured by sticky header/footer.
- Drag-and-drop has a single-pointer alternative (Move Up/Down buttons).
- Forms: `<label>` or `aria-labelledby` on EVERY input; error `aria-invalid="true"` + `aria-describedby`.
- Contrast: text ≥ 4.5:1 (large text ≥ 3:1). Do not use color ALONE to convey state.
- Images: meaningful `alt`, or `alt=""` / `aria-hidden="true"` when decorative.
- Toast/loading/route change: `aria-live="polite"` or `aria-live="assertive"`.

### UI conventions

- Reuse components from `@myorg/ui` before writing a new one.
- Empty placeholder is `-` (hyphen), NOT `—` (em dash).
- Date display: use a `formatUsDate(...)` helper, not `new Date().toLocaleDateString()` (the backend sends `Z` which shifts the day for US users).
- PHI mask: gate the field with the PII attribute on the backend and render `***-**-1234` client-side.
- Row kebab / action dropdowns: use an anchored-menu portal (not the DataTable default menu).
- Detail-page pattern: Outlet tabs + breadcrumb + plain title row + Edit only on the overview tab.

## Test

- Backend: xUnit + NSubstitute + Shouldly, direct instantiation (no ABP TestBase).
- Web: vitest + `@testing-library/react`.
- Coverage: **80% on new code** (SonarQube new-code gate, zero new violations).

## Quality gate before the Coder moves a task to review

Backend:
```bash
dotnet build src/App.slnx  # zero warnings
dotnet test src/App.slnx   # zero failures
```

Web:
```bash
pnpm --filter web typecheck
pnpm --filter web lint
pnpm --filter web test --run
```

## Ticket / commit / branch

- Branch: `feature/<ticket>-<kebab-case-title>` (e.g. `feature/123-confirm-user-details`). If the work touches multiple repos, create the same branch in each.
- Commit: conventional commits, NO AI attribution.
- Commit only to the feature branch; do NOT push unless the user asks.
- NEVER stage: secrets, `appsettings.*.local.*`, logs, `.DS_Store`, `.claude/settings.local.json`.
- Code comments must NOT contain ticket numbers, PR numbers, dates, or reviewer names; keep only the timeless reason.

## Notes

- Test frameworks: xUnit backend, vitest web. NO DB mocks on the backend (use EF in-memory sqlite per the `TenantScopeIsolationTests` pattern).
- If graphify is installed (`graphify-out/graph.json` exists), use `graphify query "..."` instead of wide Grep.
- OpenAPI regen: scope to the feature's own files. A full regen must be merged manually to avoid drift.
