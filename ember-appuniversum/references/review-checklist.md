# Appuniversum UI review checklist

Use this for a requested UI review/refactor, or the affected portion of an
implementation. Report concrete findings with file/line evidence, user impact,
and a minimal fix. Do not turn an ordinary UI task into an unrelated rewrite.

Source anchors: `appuniversum/src/components/`, `src/modifiers/`,
`src/services/toaster.ts`, `styles/`, corresponding integration tests, and the
consuming app. Reference snapshot: 4.5.1 / 4052636.

## Establish the version and intended behavior

- [ ] Read app package.json and lockfile/resolved addon version; compare with
      the local clone instead of treating the newest source as universal.
- [ ] Read app wrappers/overrides/styles and tests before changing behavior.
- [ ] Verify API against actual implementation/tests. Treat historical stories
      as examples, not higher authority than current source.
- [ ] Identify loading, empty, error, success, and disabled states that matter
      for this task. Mark unverified APIs **“verify in source”**.

## Component choice and contract

- [ ] Use AuButton for actions, AuLink for routes, AuLinkExternal for URLs.
      No invented @onClick/@linkRoute/@href contracts.
- [ ] Prefer existing controls, alert, modal, card/table, and navigation patterns
      over duplicates. Native form/fieldset/table cells/nav markup are valid;
      no AuForm/AuSelect/AuBreadcrumbs/AuSidebar exists in this snapshot.
- [ ] Distinguish native value/type/required/readonly/href from component args.
      AuInput/AuTextarea do not expose @value/@onChange.
- [ ] Match callback payloads: DOM Event versus checked boolean, selected value,
      value array, date pair, time object, or upload result.
- [ ] Check exact yielded names/case and scope; do not destructure separate
      hashes as though all keys were simultaneously present.
- [ ] Do not mix named and contextual modal blocks. AuTable requires named
      title/header/body/footer blocks and consumer-supplied tr/th/td markup.
- [ ] Review classic data-table mutation/query contracts and pagination metadata;
      do not invent modern callbacks or assume it fetches/sorts server data.
- [ ] Import only public package modules in consumers, not #src or private.
      Generated icon components require named imports, not global lookup.

## Accessibility and interaction

- [ ] Every control has an accessible name, preferably a visible label.
      IDs are unique and label for attributes point to the intended input.
- [ ] Group controls have an accessible group name; inspect the rendered
      AuFieldset legend wrapper when depending on native fieldset semantics.
- [ ] Required markers agree with native/application validation. Errors have
      useful text, aria-invalid, and aria-describedby where applicable.
      Conditional descriptions never reference missing elements.
- [ ] Do not assume attributes on a composite component reach its internal
      input: verify AuDatePicker's host/internal control associations.
- [ ] ToggleSwitch uses visible block text, not @label alone. TimePicker labels
      are set and fixed input IDs do not collide across multiple instances.
- [ ] Headings use semantic @level independently of visual @skin; table
      captions, scoped headers, landmarks, and link text are meaningful.
- [ ] Decorative icons stay hidden; meaningful icon-only controls have text.
      Badges are hidden by default and do not replace textual status.
- [ ] Dialog has an accessible name, unique @id and an existing modal container;
      open/close state, initial focus, tab containment, Escape, background
      interaction and return to opener are verified.
- [ ] Keyboard behavior matches the rendered widget. AuDropdown is not an
      automatic fully implemented menu keyboard system; AuTabs is route-nav,
      not an ARIA tab-panel state machine. Test rather than promising behavior.
- [ ] Tooltips appear on focus as well as hover, dismiss with Escape, and are
      linked to their target. Essential instructions are not tooltip-only.
- [ ] Inspect expandable-card keyboard/name behavior, row-click navigation and
      default data-table selection labels; there are source-level caveats.
- [ ] Loading/alerts/toasts use appropriate announcement urgency. No duplicate
      assertive announcements, unexplained focus theft, or lost form data.
- [ ] File upload can be operated without drag/drop and the server validates
      type/size/authentication. UI filtering is not a security boundary.

## Layout and styling

- [ ] MainHeader skip link and navigation focus targets #main/#content exist,
      are focusable, unique, and not obscured. Avoid nested main landmarks.
- [ ] Narrow/desktop layouts, long labels, 200% zoom, scroll containment and
      table overflow work. Sidebar collapsed sizing is not a mobile drawer.
- [ ] Existing object/utility classes and supported component arguments are
      preferred; class names and responsive suffixes are verified in SCSS.
- [ ] App overrides are scoped and token-based, with visible focus and contrast.
      No addon/node_modules patches or unnecessary new CSS dependency.
- [ ] Sass import order is correct, runtime tokens are actually consumed by the
      target rule, and no unverified dark-mode/theme capability is claimed.
- [ ] Check print behavior and that visual reversal does not confuse DOM order.

## Verification and reporting

- [ ] Test meaningful behavior (state updates, error associations, keyboard,
      modal focus, routing/query changes), not only matching CSS classes.
- [ ] Run the app's normal lint/test commands where available; run browser
      checks for responsive/focus behavior when possible.
- [ ] Keep component/wrapper APIs stable and unrelated files untouched.
- [ ] Distinguish verified failures from risks requiring browser validation.
      Report exact commands, results, skipped checks, and missing prerequisites.
- [ ] For no-finding reviews, explicitly note remaining test coverage gaps;
      do not imply source inspection alone established accessibility compliance.
