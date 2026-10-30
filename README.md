# Ember Appuniversum skill

A portable Agent Skill for building, refactoring, and reviewing Ember applications
using `@appuniversum/ember-appuniversum`.

## Install

```sh
npx skills add aatauil/skill-appuniversum --skill ember-appuniversum
```

The skill consists of plain Markdown: `SKILL.md` routes agents to task-specific
references. It does not require a particular agent runtime or add a dependency
to your Ember application. Agent support and installation locations depend on
your installer configuration.

## Contents

```text
ember-appuniversum/
  SKILL.md
  references/
    components.md
    forms.md
    layout-and-navigation.md
    styling-and-theming.md
    ember-patterns.md
    review-checklist.md
```

The catalog covers 62 checked-in public component paths (including three
compatibility aliases), the generated icon family and 620 SVG keys, public
modifiers, and the toaster service. Other references cover accessible forms,
layout/navigation, SCSS and tokens, Ember integration, and UI review.

## Source and version boundaries

The reference snapshot was inspected against the Appuniversum addon repository,
version **4.5.1**, commit **4052636**. Its source is
`https://github.com/appuniversum/ember-appuniversum`.

The consuming app’s installed addon version and intentional app-level overrides
remain authoritative. Verify names, arguments, yields, events, and accessibility
behavior in matching source/tests before implementation. This skill explicitly
calls out legacy APIs and accessibility caveats; it does not guarantee compliance
or claim built-in dark mode.

Generated icon export names beyond the inspected examples must be verified in
the built installed package. Browser interaction and complete form examples
still need validation in the consuming app’s actual environment.

## Maintaining the skill

Keep `SKILL.md` concise and load reference sections on demand. When updating the
catalog, reconcile it with the addon’s component exports, source, and tests;
regenerate the SVG-key inventory from `public/icons/*.svg`, and record the
version/commit inspected. Do not copy obsolete story arguments over current APIs.
