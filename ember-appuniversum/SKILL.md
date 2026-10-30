---
name: ember-appuniversum
description: Build, refactor, and review Ember apps using @appuniversum/ember-appuniversum. Use when creating pages, forms, dialogs, tables, navigation, notifications, or Flanders-style UI with Appuniversum components.
---

# Ember Appuniversum

## Purpose and scope

Build, refactor, and review Appuniversum UI in Ember applications. Appuniversum is
Flanders/Webuniversum-aligned; `@appuniversum/ember-appuniversum` provides the Ember
components and styles for that design system. Apply this skill to Appuniversum
pages, components, forms, modals, alerts, tables, navigation, and accessibility.
This skill is plain Markdown and does not require a particular agent runtime.

## Source of truth

- Locate the consuming app and read its `package.json`, lockfile, existing UI,
  style entrypoint, and test setup. Resolve the **installed** addon version;
  a semver range alone does not identify its exact implementation.
- Prefer the local `ember-appuniversum` clone (often `../ember-appuniversum`).
  This repository is a monorepo: implementation is in `appuniversum/src`, styles
  in `appuniversum/styles`, tests in `appuniversum/tests`, and stories in
  `docs-app/stories`. Older releases may use `addon/` and `app/` instead.
- This reference snapshot was verified against version **4.5.1**, commit
  **4052636**. Re-check installed source when versions differ. A newer clone
  must not override an older installed API.
- Before using a component, search the component directory. Verify its exact
  exported name, arguments, native attributes, blocks/yields, event payloads,
  attribute forwarding, and accessibility behavior in implementation and tests.
  Never assume an `Au…` name exists simply because a CSS class exists.
- App-level wrappers, component overrides, and styles are the final authority
  for the consuming app's behavior. Explain conflicts instead of silently
  replacing intentional app patterns. Stories are useful examples, but source
  and tests win when they disagree.
- If an API cannot be confirmed, say **“verify in source”** and identify the
  relevant file. Do not manufacture a prop, callback, CSS class, or capability.

Useful local searches (adapt paths):

```sh
rg --files appuniversum/src/components
rg -n 'Args:|Blocks:|yield|attributes|on "' appuniversum/src/components/au-input.gts
rg --files appuniversum/tests/integration/components
rg -n 'au-o-grid|au-u-hidden-visually' appuniversum/styles
```

## Core rules

- Prefer existing Appuniversum components over duplicate buttons, alerts, modals,
  controls, or navigation. Native semantic HTML remains necessary for forms,
  table cells, navigation landmarks, and patterns with CSS but no component.
- Use angle-bracket invocation, tracked state, `{{on}}` for DOM events, and
  `{{fn}}` for currying. Use `{{mut}}` only for an API that actually requires
  classic two-way binding. Follow the app's Glimmer/colocated-template patterns;
  do not rewrite unrelated components just to modernize them.
- Distinguish component arguments from native attributes: `AuInput` takes
  `value=`, `required`, `readonly`, and `{{on "input" …}}`, not `@value` or
  `@onChange`. `AuButton` uses `{{on "click" …}}`, not `@onClick`.
- Compose through blocks and contextual components; preserve the **case** of
  yielded keys. Several components yield separate hashes on separate passes,
  so do not destructure every key from one assumed complete object.
- Preserve semantic HTML, visible labels, linked descriptions/errors, keyboard
  interaction, focus management, and accessible names. Use ARIA only to supply
  missing semantics. Inspect actual DOM; a design-system component is not an
  accessibility guarantee.
- Prefer supported arguments, existing object/utility classes, and tokens over
  ad-hoc CSS. Check the stylesheet before writing any `au-*` class.
- In a consuming app, do not patch addon files or `node_modules`. Justified
  extensions belong in app wrappers, modifiers, services, or scoped app styles.
  Work on addon implementation only when the task explicitly targets the addon.
- Keep consuming component APIs stable; do not invent props/events or copy an
  API from a different Appuniversum release.

## Read only the relevant references

- [Components](references/components.md): complete checked-in public component
  inventory, selection guide, examples, and generated icon family.
- [Forms](references/forms.md): inputs, validation, field groups, layout, errors,
  and complete simple/complex form examples.
- [Layout and navigation](references/layout-and-navigation.md): application
  shells, headers, menus, breadcrumbs, responsive layouts, and page examples.
- [Styling and theming](references/styling-and-theming.md): SCSS entrypoints,
  tokens, verified classes, Flanders assets, overrides, and theming boundaries.
- [Ember patterns](references/ember-patterns.md): imports, state/events, yielded
  blocks, modifiers, toaster service, modal integration, and testing.
- [Review checklist](references/review-checklist.md): reviewing/refactoring UI.

For the large component catalog, search for the target heading rather than
loading unrelated entries. Load additional references only as needed.

## Working workflow

1. Identify the UI task and affected interactions.
2. Inspect existing app patterns and the resolved addon version.
3. Search addon source for suitable components and inspect their tests.
4. Read the matching reference section; confirm version-sensitive details.
5. Implement with existing components and idiomatic Ember composition.
6. Validate names, imports/invocation, callback payloads, labels, focus,
   keyboard behavior, responsive/print layout, and loading/error/empty states.
7. Run the app's normal lint/test commands when available. In this monorepo,
   root scripts include `pnpm lint` and `pnpm test`; narrower addon commands
   are documented in the Ember reference. Report commands not run and why.

## Anti-patterns

- Replacing an available component with duplicate raw button/alert/modal UI.
- Hard-coding Webuniversum classes without checking Appuniversum styles.
- Assuming every CSS pattern has an Ember component (there is no checked-in
  `AuBreadcrumbs`, `AuSidebar`, `AuSelect`, or `AuForm`).
- Ignoring keyboard and screen-reader behavior or relying only on color.
- Mixing addon versions, copying components from another project, or importing
  `#src`/`private` modules into consuming apps.
- Adding dependencies, a CSS framework, or custom CSS before checking existing
  capabilities. Do not prescribe shadcn registry/CLI workflows for this addon.
