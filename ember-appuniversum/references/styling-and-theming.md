# Styling and theming

## Contents

Entrypoints · Tokens · Verified classes · Flanders assets · Safe overrides · Limits.

Sources: `appuniversum/package.json` exports/public-assets,
`appuniversum/styles/_index.scss`, `_a-*.scss`, `settings/_s-{global,root,colors}.scss`,
`generic/_g-font.scss`, `objects/`, `components/`, `utilities/`, and
`docs-app/app/styles/app.scss`. Snapshot: 4.5.1 / 4052636.

## Entrypoints and build pipeline

This addon ships **SCSS**, not a Tailwind/shadcn registry. Its actual docs-app
entrypoint starts with:

```scss
@import "@appuniversum/ember-appuniversum/styles";
// App-owned styles follow.
```

`styles/_index.scss` imports ordered layers: settings, tools, generic, elements,
objects, components, plugins, utilities. The public exports also support
`styles/a-settings`, `styles/a-tools`, etc., and individual partial groups such
as `styles/components/c-button` or `styles/settings/s-root`.

Use the app's existing Sass/build configuration. The historical getting-started
story describes ember-cli-sass and autoprefixer; inspect the actual app pipeline
before installing anything, particularly in Vite/Embroider projects. The source
still uses Sass `@import`; do not mechanically substitute `@use` and assume
shared variables/mixins remain compatible. Do not import the full bundle and
all layers again, which duplicates output.

## Tokens: runtime CSS versus compile-time Sass

`settings/_s-root.scss` declares CSS custom properties on `:root`:

- Palette: `--au-white`, `--au-gray-100` … `--au-gray-1000`, selected blue,
  yellow, orange, red, green stops; also `--vl-*` Flanders palette names.
  Not every number in every color scale exists: check before using a stop.
- UI: `--au-page-bg`, `--au-select-text-color`, `--au-select-text-bg`,
  `--au-divider-color`, `--au-outline-color`.
- Typography: `--au-font`, `--au-font-secondary`, `--au-font-tertiary`,
  `--au-global-font-size`, `--au-global-line-height`, `--au-para`, `--au-small`,
  `--au-h1` … `--au-h6`, `--au-regular`, `--au-medium`, `--au-bold`.
- Geometry/motion: `--au-icon-size`, `--au-icon-size-medium`,
  `--au-icon-size-large`, `--au-radius`, `--au-border`, `--au-outline`,
  `--au-outline-offset`, `--au-transition`, `--au-duration`, `--au-easing`,
  and `--au-z-index-alpha/beta/gamma`.

`settings/_s-global.scss` retains compile-time `!default` settings, including
`$au-unit-tiny: 0.6rem`, `$au-unit-small: 1.2rem`, `$au-unit: 2.4rem`,
`$au-unit-large: 4.8rem`, `$au-unit-huge: 9.6rem`, layout widths and breakpoint
map. These are **not** automatically runtime `--au-unit` variables. The
Sass colors and many typography settings are marked deprecated in favor of CSS
variables; do not revive those as the preferred new override API.

Override supported `!default` Sass settings **before** the full import:

```scss
$au-layout-width: 1200px;
@import "@appuniversum/ember-appuniversum/styles";
```

Set runtime properties **after** the bundle's root declarations:

```scss
:root {
  --au-page-bg: var(--au-gray-100);
}
```

Inspect where a token is consumed. For example, `.au-c-app` uses `--au-white`,
not `--au-page-bg`, so overriding the latter is not a global repaint guarantee.
`--au-outline` is a separate literal composite token; changing
`--au-outline-color` alone does not reconstruct it.

## Class conventions and verified patterns

| Prefix / pattern          | Role                        | Examples verified in styles                                                                                                    |
| ------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `au-c-*`                  | Component/pattern classes   | `au-c-input`, `au-c-sidebar`, `au-c-list-navigation`                                                                           |
| `__element`, `--modifier` | Component elements/variants | `au-c-sidebar__content`, `au-c-list-horizontal--small`                                                                         |
| `au-o-*`                  | Layout objects              | `au-o-layout`, `au-o-layout--small`, `au-o-box`, `au-o-box--small`, `au-o-grid`, `au-o-grid__item`, `au-o-flow`, `au-o-region` |
| `au-u-*`                  | Utilities                   | `au-u-hidden-visually`, `au-u-text-center`, `au-u-margin-top-tiny`, `au-u-margin-right-tiny`                                   |
| Fraction + breakpoint     | Generated widths            | `au-u-1-1`, `au-u-1-2@medium`                                                                                                  |
| `is-*`                    | State classes               | `is-disabled`, `is-active`, `is-loading`                                                                                       |

Prefer a component's argument to manually setting its internal state/modifier
class. A class can exist without an Ember component: sidebar, horizontal-list,
and navigation-list CSS are valid composition tools, not proof of `AuSidebar`.
Use semantic elements and labels with these styling patterns.

Utilities can use `!important`. Avoid specificity wars and avoid overriding
all `.au-c-button`, `.au-c-input`, or root elements to satisfy one screen.
Inspect generated responsive classes if the app customizes fractions,
breakpoints, or utility settings.

## Flanders assets

`generic/_g-font.scss` declares flanders-sans/flanders-serif WOFF/WOFF2 faces.
The default `$au-font-location` is
`"../@appuniversum/ember-appuniversum/fonts"`; verify asset URLs under the app's
rootURL/build output. The package's public-assets mapping exposes fonts and the
symbolset below `/@appuniversum/ember-appuniversum/`.

`AuBrand` renders the Flanders logo/logotype. `AuIcon` string mode reads the
namespaced symbolset, prefixed with configured rootURL; component mode uses the
generated importable icons. Do not copy logos, fonts, or a symbolset from an
unrelated Appuniversum version into the app.

## Safe app-level extensions

1. Try a supported component argument first.
2. Use existing layout objects/utilities for placement and spacing.
3. If necessary, add a narrowly scoped app class or wrapper. Forward attributes
   intentionally; confirm which element actually receives them.
4. Use existing tokens for colors, typography, focus, and transitions. Keep
   focus visibility and contrast intact and test all interactive states.
5. Keep overrides in the app's style layer; do not patch addon source from a
   consuming app. Avoid undocumented deep selectors when a wrapper works.

Example for a genuinely app-specific notice, not a duplicate alert:

```scss
.app-record-meta {
  border-inline-start: var(--au-border) solid var(--au-divider-color);
  color: var(--au-gray-900);
}
```

## Theming boundaries and verification

No dark-mode selector, `prefers-color-scheme` implementation, theme service,
or complete alternate palette was found in the inspected source/styles.
CSS-variable overrides are possible; they are **not** a verified built-in dark
mode or a guarantee that all components are tokenized consistently. Mark any
release-specific theme API **“verify in source”** until present in that version.

Plugin partials style ember-power-select and Duet date picker. Styling alone
does not install/register a plugin or prove compatibility with arbitrary
versions. Verify the consuming app dependency and integration before using it.
