# Upstream pin

| Field | Value |
| --- | --- |
| Package | `@base-ui/react` |
| Version | `1.6.0` |
| Advertised range | `1.6.x` |
| Source boundary | `.base-ui/packages/react/src/` (checkout at the pin, not vendored into this repo) |
| React oracle | `react@19.2.7` / `react-dom@19.2.7`, with `@base-ui/react@1.6.0` installed for differential comparison |

Every ported module cites its upstream path and the pin in its file header, in the form
`// Ported from .base-ui/packages/react/src/<module>/ (v1.6.0): <files>`. That header is the
per-module crosswalk; this file records the package-level pin and the gaps.

## Source boundary

Upstream source is **not vendored** into this repository. It is read from a checkout at the pinned
commit under `.base-ui/`, which is also what the file headers reference. `@base-ui/react@1.6.0` is
installed as a dependency so its React implementation is available as a differential oracle.

This predates the `react-library-port` requirement to vendor the pinned source under
`packages/base-ui/upstream/`. Bringing the package fully up to that requirement — vendored bytes,
LICENSE, prettier-ignored, unpublished — is outstanding and is the main gap in this file.

## Coverage

35 of upstream's 44 component subpaths are ported; the 9 below are not. `status.json`'s `surface` field carries the
prose summary; the table below records only what is NOT done, because that is what silence would
otherwise misrepresent.

### Not ported

| Upstream subpath | Reason |
| --- | --- |
| `scroll-area` | Not started. Its behaviour is almost entirely measurement (`scrollHeight`, `getBoundingClientRect`, `ResizeObserver`, thumb sizing), and upstream itself skips those cases under jsdom (`describe.skipIf(isJSDOM)` on sizing, overflow attributes and interactions) — so a port would land with no meaningful in-repo oracle until a real-browser suite exists. |
| `navigation-menu` | Not started. 3,106 lines across 34 files; the trigger alone is 914 lines of hover-intent, focus-guard and popup auto-sizing logic coordinated across the floating tree. All of its dependencies are already ported except a small local `setSharedFixedSize` util. |
| `select` | Not started. 4,220 lines across 42 files, 212 upstream cases, and the only remaining family integrating with the Field/Form layer. |
| `autocomplete`, `combobox`, `otp-field`, `toolbar`, `drawer` | Not started, no assessment recorded. |
| `global` | Not a component subpath. |

### Partial

| Part | Gap |
| --- | --- |
| `Tabs.Indicator` | Not ported. It positions from measured tab geometry and ships a pre-hydration `<script>` with its own SSR contract. Every other Tabs part (`Root`, `List`, `Tab`, `Panel`) is complete. Adding it is additive. |

## Test dispositions

Upstream's own suites are the parity oracle where they can run. This package's tests execute in
jsdom, which bounds what is portable.

| Upstream suite | Disposition |
| --- | --- |
| `collapsible/**/*.test.tsx` (35 cases) | 16 ported to `tests/upstream/collapsible.test.ts` — every case not requiring real layout measurement, CSS transition/keyframe sequencing, `React.Activity`, or find-in-page `beforematch`. |
| `accordion/**/*.test.tsx` | 15 ported to `tests/upstream/accordion.test.ts`, plus 3 written to close a real gap: upstream exercises single-vs-multiple toggling only inside jsdom-skipped blocks, so removing the single-mode collapse branch passed every ported case. |
| `tabs/**/*.test.tsx` (84 cases: 76 Root, 5 List, 1 Tab, 2 Panel) | 15 ported to `tests/upstream/tabs.test.ts`, covering aria wiring, tab order, the automatic selection policy including the explicit-vs-implicit `defaultValue` distinction, controlled-vs-uncontrolled divergence, and panel mounting. The remainder are unported; most exercise `Tabs.Indicator`, and the `activationDirection` cases read `getBoundingClientRect`, which jsdom reports as zero. |
| All other ported modules | No upstream cases ported. Their coverage is this package's own behavioural tests plus the differential suites. This is the second significant gap in this file. |

## Known divergences

Recorded in `status.json`'s `divergences` field, which `pnpm bindings:status` renders into
`docs/bindings-status.md`. In summary: handlers receive native DOM events with no synthetic layer;
`forwardRef` becomes ref-as-prop; the vendored `floating-ui-react` surface is internal rather than
republished; `NumberField.ScrubArea` and hold-to-repeat stepping are unported.

### Attribute-name skew against newer releases

At this pin, `getStateAttributesProps` converts an `orientation: 'horizontal'` state to
`data-orientation="horizontal"`. Newer Base UI releases — and the current shadcn sources written
against them — expect bare `data-horizontal` / `data-vertical` attributes instead. This is a version
skew, not a porting defect: consuming layers must target what this pin emits. It affects every
oriented family (`slider`, `tabs`, `separator`, `toggle-group`, `menubar`) and is the single most
likely source of silently-unstyled output for a consumer copying class strings from current shadcn
sources.
