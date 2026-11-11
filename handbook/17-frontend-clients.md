# 17 — Frontend & Client Applications

The client is where users experience every backend decision. It must be fast, accessible, resilient to bad networks, secure, and maintainable as the UI grows.

**Applies when:** building web UIs, mobile/desktop clients, design systems, or anything users interact with directly. Mobile/desktop specifics: [24](24-project-type-addenda.md).

## Key questions
- What rendering model fits the content and interactivity (static, server-rendered, client-rendered, hybrid)?
- Where does state live, and what's the source of truth?
- How does the UI behave on slow, flaky, or offline networks?
- Is it usable by keyboard, screen reader, and on small screens?
- What's the performance budget, and how is it enforced?

## 1. Rendering & architecture [MVP]
| Model | Fits | Trade-offs |
|---|---|---|
| **Static (SSG)** | Marketing, docs, blogs. | Rebuilds for content changes (ISR mitigates). |
| **Server-rendered (SSR / MPA / server components)** | SEO, fast first paint, content-heavy apps, simpler state. | Server cost; hydration complexity. |
| **Client-rendered SPA** | Highly interactive apps behind login. | Slower first load, SEO, bundle size. |
| **Hybrid / islands / streaming** | Mixed content and interactivity. | Framework complexity. |
| **Hypermedia-driven (htmx, Hotwire, LiveView)** | CRUD-heavy apps, small teams. | Less suited to rich offline/client-side interactions. |
| **Local-first** | Offline, collaboration, instant UI. | Sync/conflict complexity. |

- Organize by feature (components, hooks, API calls, tests colocated) ([11](11-code-design.md)).
- Keep business rules on the server; the client validates for UX only.

## 2. State management [MVP]
- Classify state: **server state** (cached remote data — use a data-fetching/cache library: TanStack Query, SWR, Apollo, RTK Query), **URL state** (filters, pagination, tabs — keep in URL for shareability), **form state**, **local UI state**, **global client state** (rarely as much as assumed).
- Avoid duplicating server data into global stores.
- Optimistic updates with rollback for snappy UX where safe.
- Handle loading, empty, error, partial, and stale states explicitly for every data view.

## 3. Design system & UI consistency [MVP]
- Design tokens (color, spacing, typography, radius, motion) as the single source of truth.
- Shared component library (own or built on accessible primitives: Radix, React Aria, Headless UI, Ark, Material, etc.).
- Component documentation/playground (Storybook, Ladle, Histoire) [SCALE].
- Consistent patterns for forms, validation messages, tables, modals, toasts, empty states, confirmations of destructive actions.
- Dark mode / theming if required.

## 4. Accessibility (a11y) [MVP]
- Target **WCAG 2.2 AA** (often legally required — ADA, EAA, Section 508) ([10](10-privacy-compliance.md)).
- Semantic HTML first; ARIA only when needed and correct.
- Full keyboard operability; visible focus; logical focus order; focus management in modals/route changes.
- Labels for all inputs; error messages associated with fields; announcements for dynamic changes (live regions).
- Color contrast; don't rely on color alone; respect `prefers-reduced-motion`; scalable text/zoom to 200–400%.
- Alt text for meaningful images; captions/transcripts for media.
- Automated checks (axe, eslint-plugin-jsx-a11y, Lighthouse) + manual screen reader testing for key flows [PROD].
- Accessibility conformance report (VPAT) [ENT].

## 5. Performance [MVP]
- **Core Web Vitals** targets: LCP < 2.5 s, INP < 200 ms, CLS < 0.1 (at p75 of real users).
- Performance budgets (JS KB, image weight) enforced in CI [PROD].
- Code splitting/lazy loading by route and heavy component; tree-shaking; avoid giant dependencies for small tasks.
- Image optimization (modern formats, responsive sizes, lazy loading, explicit dimensions).
- Font loading strategy (subset, `font-display`, preload).
- Avoid request waterfalls (parallel fetching, server-side data loading, prefetching).
- Virtualize long lists; debounce/throttle expensive handlers; avoid unnecessary re-renders.
- Caching headers and CDN for static assets (hashed filenames, long max-age).
- RUM to measure real users, not just lab tests ([13](13-observability.md)).

## 6. Resilience & network [MVP]
- Timeouts and retries (idempotent requests) on the client; cancel stale requests.
- Graceful handling of offline/poor connectivity; meaningful error messages with retry.
- Error boundaries so one broken widget doesn't blank the app.
- Handle auth expiry gracefully (refresh, re-login preserving work).
- Prevent double submission; disable buttons during in-flight mutations; idempotency keys for critical actions.
- Version skew: handle the client running older code than the server (chunk load errors after deploy → reload prompt).

## 7. Security (client side) [ALL]
- No secrets in client bundles — anything shipped to the browser/app is public.
- XSS: rely on framework escaping; sanitize any HTML rendering; CSP ([08](08-security.md)).
- Token storage considerations ([09](09-identity-access-tenancy.md)); prefer HttpOnly cookies for web sessions.
- Validate redirects; avoid `target="_blank"` without `rel="noopener"` (modern browsers default this, but be explicit with older targets).
- Minimize and vet third-party scripts (analytics, chat widgets) — they run with full page privileges.
- Dependency hygiene for the frontend supply chain ([20](20-dependencies-supply-chain.md)).

## 8. Internationalization & localization [MVP if multi-locale]
- Externalize all user-facing strings from day one if multiple locales are plausible.
- ICU message format for plurals/gender/selects; no string concatenation for sentences.
- Locale-aware formatting of dates, times, numbers, currencies (Intl APIs); timezone display from UTC.
- RTL layout support (logical CSS properties).
- Text expansion (German ~30% longer); avoid fixed widths.
- Unicode handling (emoji, combining characters, normalization); sorting/collation per locale.
- Translation workflow (TMS, pseudo-localization testing) [SCALE].

## 9. Forms & UX quality [MVP]
- Client validation mirrors server rules (ideally shared schemas) but the server remains authoritative.
- Clear, specific, actionable error messages; preserve user input on error.
- Autosave or unsaved-changes warnings for long forms.
- Confirmation/undo for destructive actions.
- Responsive design across breakpoints; touch target sizes.
- Empty states that guide; loading skeletons over spinners where appropriate.

## 10. Analytics & experimentation [MVP]
- Defined event taxonomy (names, properties, versions); avoid PII in events ([10](10-privacy-compliance.md)).
- Consent-respecting analytics.
- A/B testing infrastructure with proper statistical practice [SCALE].

## 11. SEO (public content) [MVP]
- Server-rendered or pre-rendered content for indexable pages; semantic headings; meta tags; Open Graph; sitemap; robots; canonical URLs; structured data; fast pages.

## 12. Testing the client [MVP]
- Component tests (Testing Library) focused on user behaviour; E2E for critical journeys (Playwright/Cypress); visual regression [SCALE]; a11y checks; cross-browser matrix defined.

## Red flags
- API keys or secrets in frontend env vars shipped to the browser.
- Every piece of server data copied into a global store.
- Missing loading/error/empty states.
- `div` buttons without keyboard support; no focus styles.
- Multi-MB JS bundles for simple pages.
- Hardcoded strings and concatenated sentences in a multi-locale product.
- Business rules implemented only in the client.

## Decisions to record
Rendering model · state management approach · component library/design system · a11y target · performance budgets · i18n approach · supported browsers/devices.

## Evidence of done
- Loading/empty/error states implemented for new views.
- Automated a11y checks pass; keyboard flow verified.
- Bundle/performance budget not regressed.
- Strings externalized (if i18n applies).

## Related
[07](07-performance-scalability.md) · [08](08-security.md) · [12](12-testing-quality.md) · [24](24-project-type-addenda.md)
