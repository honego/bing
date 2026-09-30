# bing Engineering Rules

**MUST**, **MUST NOT**, and **REQUIRED** define the agent contract. Ownership rules apply to needed behavior, not future feature requirements.

## Project Boundaries

- The project name MUST remain **`bing`**. The stack is **Vite + React + TypeScript + pnpm + ESLint**.
- Production builds MUST emit only static files suitable for Cloudflare Pages or Cloudflare Static Assets.
- Code changes use normal builds. Daily wallpaper changes MUST NOT depend on daily builds; fetch wallpaper data at runtime instead of synchronizing it into the repository.
- MUST NOT introduce Next.js, Astro, SSR, databases, cron, daily data synchronization (including GitHub Actions), R2, KV, or unnecessary backend infrastructure.

## Required Agent Workflow

1. Run `git status`; preserve existing user changes.
2. Read this file and applicable instructions for affected paths.
3. Inspect relevant source, configuration, and existing naming conventions.
4. Before creating any component, function, hook, type, constant, API function, formatter, validator, URL builder, or preload helper, use `rg` to find matching names, synonymous implementations, callers, raw API field usage, and related constants.
5. Identify the canonical owner and check dependency direction before editing.
6. Modify or extend that owner and update its callers. Consolidate duplicates within the task's scope; MUST NOT add another implementation.
7. Remove superseded implementations and dead code in the affected scope.
8. Run the required verification commands below.
9. Review the final diff and new files; search again for obsolete callers and competing implementations. A refactor is incomplete while both versions remain.
10. Report actual changes, verification results, warnings, and unavailable checks.

Unrelated refactors and file edits are prohibited. MUST NOT automatically `git commit` or `git push`; either requires an explicit user request.

## Naming Contract

Each category MUST use one convention, never mixed kebab-case, snake_case, camelCase, or PascalCase. Existing violations do not authorize new ones; correct affected names within scope.

| Category                       | REQUIRED convention                                       | Good                                                         | Bad                                                       |
| ------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| React components               | PascalCase                                                | `WallpaperViewer`, `WallpaperInfo`                           | `wallpaperViewer`, `wallpaper_viewer`                     |
| React component files          | PascalCase, matching the main component                   | `WallpaperViewer.tsx`, `WallpaperInfo.tsx`                   | `wallpaper-viewer.tsx`, `wallpaperViewer.tsx`             |
| Ordinary TypeScript functions  | camelCase; describe an action                             | `fetchWallpapers`, `buildImageUrl`, `preloadImage`           | `FetchWallpapers`, `fetch_wallpapers`                     |
| Ordinary variables             | camelCase, including runtime `const` values               | `currentIndex`, `imageUrl`                                   | `current_index`, `CurrentIndex`                           |
| Boolean values                 | camelCase; prefer `is`, `has`, `can`, or `should`         | `isLoading`, `hasError`, `canRetry`, `shouldPreload`         | `IsLoading`, `has_error`, `flag1`                         |
| Types and interfaces           | PascalCase with a specific domain name                    | `BingWallpaper`, `BingApiResponse`                           | `bingWallpaper`, `bing_response`                          |
| Props types                    | Component name followed by `Props`                        | `WallpaperViewerProps`                                       | `wallpaperViewerProps`, `ViewerData`                      |
| Callback props                 | `on` followed by a PascalCase action                      | `onNext`, `onPrevious`, `onRetry`                            | `nextCallback`, `handleRetry` as a prop                   |
| Component event handlers       | `handle` followed by a PascalCase action                  | `handleNext`, `handlePrevious`, `handleRetry`                | `onNext` as an internal handler, `handle_next`            |
| Custom hooks                   | camelCase starting with `use`; camelCase file             | `useWallpapers`, `useKeyboardNavigation`, `useWallpapers.ts` | `wallpapersHook`, `UseWallpapers.ts`, `use-wallpapers.ts` |
| Non-component TypeScript files | camelCase                                                 | `bing.ts`, `imageUrl.ts`, `preloadImage.ts`                  | `image-url.ts`, `image_url.ts`, `ImageUrl.ts`             |
| Component CSS files            | PascalCase, matching their component                      | `WallpaperViewer.css`                                        | `wallpaper-viewer.css`                                    |
| Shared/global CSS files        | camelCase                                                 | `global.css`, `designTokens.css`                             | `Global.css`, `design-tokens.css`                         |
| Test files, if introduced      | Match the tested file's base name; use only `.test`       | `WallpaperViewer.test.tsx`, `imageUrl.test.ts`               | `wallpaperViewer.test.tsx`, `imageUrl.spec.ts`            |
| Asset files                    | Lowercase/kebab-case                                      | `bing-logo.svg`, `wallpaper-placeholder.webp`                | `BingLogo.svg`, `wallpaper_placeholder.webp`              |
| CSS classes                    | kebab-case                                                | `.wallpaper-viewer`, `.wallpaper-info`                       | `.wallpaperViewer`, `.wallpaper_info`                     |
| Directories                    | Lowercase; prefer short responsibility names              | `components`, `hooks`, `lib`, `styles`, `types`              | `Components`, `Hooks`, `Lib`                              |
| Module constants               | UPPER_SNAKE_CASE for fixed configuration or policy values | `DEFAULT_MARKET`, `REQUEST_TIMEOUT_MS`                       | `default_market`, `RequestTimeoutMs`                      |

Runtime `const` values remain camelCase, such as `const currentWallpaper = wallpapers[currentIndex]`. Only genuine fixed module configuration or policy uses uppercase names.

Preserve prescribed tool filenames, raw external field names, `main.tsx`, and asset source names that must remain intact. These exceptions MUST NOT change application naming conventions.

Domain types MUST be specific: `BingWallpaper`, `BingArchiveResponse`, `ImageLoadResult`, not standalone `Data`, `Item`, `Info`, or `Result`.

## Single Source of Truth

Two or more callers of the same behavior MUST use one authoritative implementation. Equivalent algorithms, raw-field mappings, fallback orders, constants, and domain types count as duplication even without copy/paste.

Required operations MUST have one owner; this table does not prescribe a file or function per entry.

| Canonical owner      | Operations it owns                                                                                 |
| -------------------- | -------------------------------------------------------------------------------------------------- |
| Bing data access     | API endpoint and query construction, response validation and normalization, request timeout policy |
| Image URL            | Bing CDN URLs, resolution/format mappings, UHD URL construction                                    |
| Dates                | Bing date parsing, internal date conversion, display and filename formatting                       |
| Selection/navigation | Wallpaper selection, random selection, index navigation, related state transitions                 |
| Image loading        | Preloading, reusable load-failure handling                                                         |
| Fallback policy      | Failure triggers, fallback order, retry/stop limits                                                |

Caller variations MUST extend the owner, not copy its rules. **Bad:** viewer/download each concatenate URLs. **Good:** both call `buildBingImageUrl()`.

### Reuse Without Over-Abstraction

Unify logic only when meaning and responsibility match and multiple callers need it. Surface similarity alone MUST NOT trigger abstraction. MUST NOT introduce speculative service layers, repository patterns, factories, dependency injection, event buses, plugin systems, or generic utility frameworks.

## Module Boundaries and Dependency Direction

Data MUST flow through these boundaries:

```text
External API → data access → validation → normalization → domain model → React state/hooks → components/UI
```

Recommended `src` roles; application imports MUST follow this direction:

| Role         | Responsibility                                         | May depend on                                                   |
| ------------ | ------------------------------------------------------ | --------------------------------------------------------------- |
| `components` | React UI and interactions                              | Components, hooks, lib, domain types, styles                    |
| `hooks`      | React state/lifecycle coordination                     | Hooks, lib, domain types; MUST NOT import components            |
| `lib`        | API access and framework-independent domain operations | Lib, domain types, browser APIs; MUST NOT depend on React or UI |
| `types`      | Cross-module domain types                              | Domain types; MUST NOT depend on React or UI                    |
| `styles`     | Global/shared styles                                   | Owned styles and design tokens                                  |

- Validators, parsers, normalizers, and URL builders MUST remain framework-independent. Circular dependencies are prohibited.
- A dependency-direction exception requires a documented technical reason; caller convenience is insufficient.
- `main.tsx` MUST contain only Vite/React bootstrap. `App.tsx` owns application composition, not unrelated helpers or data pipelines.
- Create directories only when needed. Keep local types with their module, Props with their component, and shared domain types with their canonical owner.
- Catch-all modules (`utils.ts`, `helpers.ts`, `common.ts`, `misc.ts`) and `constants.ts` mixing unrelated policies are prohibited. Bing, image, and date logic belong to those owners; domain-neutral utilities MUST be named for their actual responsibility.

### Public / Private Module API

- Symbols, including parser helpers and internal mappers, MUST remain private unless another module needs them. Tests, anticipated reuse, or extension do not justify exporting implementation details.
- Expose one public entry per domain action; separate entries require distinct semantics.
- Synonymous APIs are prohibited even when they delegate to one helper: `buildImageUrl` / `getImageUrl` / `createImageUrl` / `resolveImageUrl`, or wrappers named `getWallpaperUrl` / `buildWallpaperUrl` / `resolveWallpaperUrl`, MUST NOT represent the same action.
- `fetchBingData`, `requestBing`, and `getBingArchive` likewise MUST NOT alias the same request flow.
- Use `index.ts` only for a deliberate module public API, never merely to shorten imports.

## React Rules

- Use function components only; class components are prohibited.
- State MUST live at its lowest owning common ancestor. Store minimal state and derive other values. **Good:** store `currentIndex`, derive `wallpapers[currentIndex]`. **Bad:** also store `currentWallpaper` and synchronize it with an effect. Handle empty data and invalid indices.
- A component MUST NOT combine API/normalization, URL/preload policy, navigation state, and large UI. Split independent reasons to change, not simple components into trivial wrappers.
- Hooks are React-specific boundaries for state, lifecycle, refs, context, or other hooks. Pure builders, parsers, validators, formatters, random selectors, and transformers belong in `lib`; `useBuildImageUrl()` is prohibited.
- Effects synchronize requests, listeners, timers, observers, subscriptions, and browser APIs. Derived-state effects and dependency suppressions concealing lifecycle problems are prohibited.
- Effects MUST remove listeners, clear timers, disconnect observers, unsubscribe, and release obsolete request ownership.
- `useMemo` and `useCallback` require an identified performance or reference stability need.

## Async Ownership and Cancellation

- Stale responses MUST NOT overwrite newer state. Wallpaper, market, or parameter changes MUST abort obsolete work or invalidate it through request identity/generation checks.
- Prefer `AbortController` / `AbortSignal` with a clear request owner. Intentional abort MUST NOT appear as an ordinary user error; genuine failures remain distinguishable.
- Image preload completion MUST verify current selection before committing. Work that cannot be aborted still needs identity checks. Unmounted components MUST NOT receive subsequent state updates.
- MUST NOT scatter boolean race guards across callers. Shared cancellation/race rules belong to one coordinating hook or operation, without a concurrency framework.

## TypeScript and External Data

- TypeScript strict mode is REQUIRED; `any` is prohibited.
- External unknown values MUST enter as `unknown` and be explicitly narrowed. Unsafe `as SomeType` assertions MUST NOT substitute for runtime validation.
- Infer local types naturally; explicitly type public function boundaries, Props, domain models, and external data boundaries.
- Prefer string unions for simple states, such as `type LoadStatus = 'idle' | 'loading' | 'ready' | 'error'`, rather than unnecessary enums.

### Raw API Model / Domain Model

- Raw Bing types and fields (`url`, `urlbase`, `startdate`, `copyright`, and other Bing-specific fields) belong only to the Bing data layer. MUST NOT expose raw responses to React or shared domain types.
- The data layer MUST validate structure and required field types/constraints before normalization. Guards MUST check data, not repeat assertions or always return `true`.
- React MUST consume the canonical domain model, such as `BingWallpaper`, the internal Single Source of Truth. Components MUST NOT maintain raw-field mappings, read `images[0].urlbase` / `images[0].startdate`, or build Bing API endpoints.
- Confine external contract adaptations to the data layer unless the internal domain contract intentionally changes.

```ts
// Bad: raw data is trusted without validation.
const uncheckedResponse = (await response.json()) as BingApiResponse

// Good: validate inside the data layer before normalization.
const responseData: unknown = await response.json()
if (!isBingApiResponse(responseData)) {
  throw new Error('Invalid Bing response')
}
```

## Network and Error Ownership

- Bing requests MUST use the canonical data owner, such as `src/lib/bing.ts`. Components/hooks MUST NOT independently `fetch` Bing endpoints.
- Requests MUST check HTTP status, handle network failure and malformed JSON/data, apply needed timeout policy, and accept `AbortSignal` when required.
- Distinguish network error, HTTP failure, invalid JSON/data, abort, and image load failure. Preserve operation, status, and cause needed for recovery decisions; do not require an error class hierarchy.
- Catch only where recovery, contextual translation, or presentation belongs. Empty `catch {}`, silent failures, and collapsing every failure into `new Error('Something went wrong')` are prohibited.
- UI MUST offer retry for recoverable failures. User messages may be concise; preserve diagnostic context internally.

## Image, Date, and Fallback Ownership

### Images

- Keep metadata, canonical URL generation, preload, browser image loading, and download/open-original behavior as distinct responsibilities.
- Viewer, download, preload, share, and metadata MUST use the same URL owner and its resolution/UHD/format mappings.
- Preload MUST be bounded without duplicate image loads. Browser load events use owned failure and async identity policies.

### Dates

- The date owner MUST define any required Bing `YYYYMMDD`, internal date, UI `YYYY-MM-DD`, locale display, and filename conversions.
- Parsing and formatting may be separate operations within that owner. Components MUST NOT slice Bing dates, reparse normalized dates independently, or recreate conversions for downloads.
- Use native date APIs; libraries require actual timezone/calendar complexity and dependency justification.

### Fallback

- Fallback MUST be a deliberate policy with one owner and order. Two callers MUST NOT use `A → B → C` and `A → C → B` for the same policy.
- Each added fallback MUST state why it exists, which failures trigger it, the next action, and its stopping condition. Automatic retries MUST be bounded.
- MUST NOT catch errors and invent alternate behavior merely to display something, conceal bugs, or discard the original failure context.

## Functions, Parameters, and Constants

- Names MUST describe behavior; prefer `fetch`, `get`, `build`, `create`, `parse`, `normalize`, `validate`, `resolve`, `select`, `load`, `preload`, `format`, or `handle`. `doStuff`, `processData`, `handleThing`, `func1`, and `data2` are prohibited.
- Extraction MUST improve ownership, reuse, readability, testability, or domain clarity. Split by responsibility, cohesion, and reason to change, never mechanical component/function/file line limits or trivial-wrapper targets.
- Avoid long positional lists; use an object for a coherent configuration, not automatically for two simple arguments.
- Opaque boolean switches are prohibited: **Bad:** `buildImageUrl(base, true, false)`. **Good:** explicit options or domain types for resolution, format, or behavior. A self-explanatory boolean need not become an object.
- Business values such as wallpaper limits, timeout, and default market MUST be declared once near their policy owner. Incidental literals do not require named constants.

## Imports and Styling

- Import order MUST be consistent: React/third-party, internal modules, then styles. Use type-only imports, such as `import type { BingWallpaper } from '../types/bing'`.
- Follow checked-in formatting configuration; MUST NOT change it for an individual edit.
- Ordinary CSS is the default. Use CSS custom properties for repeated design tokens and give component styles an explicit owner.
- Avoid unjustified `!important`, deep selectors, and broad globals. Stable UI design belongs in CSS; inline styles require dynamic values or a specific local need.
- MUST NOT add a large CSS/UI dependency for a few styles.

## Dependencies and Accessibility

Use pnpm and keep its lockfile in sync. Before adding a dependency, the agent MUST state the concrete problem, why React/TypeScript/browser APIs or existing dependencies are insufficient, and why the package is better than a small local implementation.

Convenience alone MUST NOT justify packages for fetch, random selection, simple date formatting, class name joining, image preload, basic validation, or URL manipulation. Prefer `fetch`, `URL`, `URLSearchParams`, `AbortController`, `Image`, and `Intl`. Axios requires a documented need that `fetch` cannot solve clearly and simply; large schema libraries require more than a few fields.

- Actions MUST use `<button>`; navigation uses semantic elements and links. `<div onClick>` button substitutes are prohibited.
- Meaningful images need appropriate `alt`; decorative images use `alt=""`.
- Keyboard operation and visible focus MUST remain usable; removing focus styling requires an accessible replacement.

## Performance, Comments, and Dead Code

Prevent duplicate API requests and clearly expensive render work; optimizations require a concrete need.

Comments explain Bing behavior, compatibility, workarounds, or constraints, not obvious operations such as `// increment index` before `index += 1`.

Unused imports, variables, functions, obsolete components, unreachable branches, commented-out code, and superseded helpers MUST be removed in the affected scope. MUST NOT retain code for possible future use.

## Configuration and Verification

Changes to `tsconfig`, ESLint, or Vite configuration require an independent engineering reason. MUST NOT weaken strictness, lint rules, or build validation to bypass failing code.

For each development task, these checks are REQUIRED:

```sh
pnpm typecheck
pnpm lint
pnpm build
git diff --check
```

Actual repository scripts define equivalent commands if names change. Report missing scripts as **unavailable**; MUST NOT fabricate passes or create scripts to satisfy this file. Run relevant existing tests when needed for changed behavior.

Known TypeScript/lint errors, build failures, broken imports, or whitespace errors block completion. Warnings MUST be investigated and resolved or explained.

For relevant UI tasks, verify affected states in a real browser when available: Console, Network, loading, success, error, retry, image loading, keyboard, and responsive layout. Report verification limits; pure code changes do not require an unrelated full browser checklist.

Diff review MUST cover scope, naming, ownership, duplicates, dead code, and configuration. Check untracked file whitespace separately; `git diff --check` omits it. Do not stage solely for verification.

## Rule Priority

Apply these priorities within higher-level agent instructions:

1. Explicit user requirements.
2. Correctness.
3. Single Source of Truth.
4. Type safety.
5. Project architecture, ownership, and conventions consistent with this file.
6. Simplicity.
7. Reusability.
8. Performance.
