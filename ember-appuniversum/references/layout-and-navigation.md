# Layout and navigation

## Contents

Shell contract · Standard content page · Settings/list page · Responsive patterns.

Sources: `appuniversum/src/components/au-{app,main-header,main-container,body-container,main-footer,toolbar,content,content-header,navigation-link,tabs,list}.gts`;
`appuniversum/styles/objects/_o-{layout,grid,box,flow,region}.scss`;
`styles/components/_c-{main-container,sidebar,list-navigation,list-horizontal}.scss`;
`docs-app/stories/7-templates/`. Snapshot: 4.5.1 / 4052636.
Older stories use `@linkRoute`; the current source requires **`@route`**.

## Shell contract

- `AuApp` is a flex application wrapper, not a semantic main.
- `AuMainHeader @appTitle` includes branding, optional home/contact routes,
  navigation actions, a route narrator, and the skip link `href="#content"`.
  Its home-route link focuses `#main`.
- `AuMainContainer` renders `<main id="main" tabindex="-1">` and yields
  **separate** lowercase `sidebar` and `content` hashes. Use `<m.sidebar>`
  and `<m.content>` in its block; do not assume both keys are present in one
  destructured object. The content component renders `id="content"` with
  `tabindex="-1"` and supports `@scroll`.
- `AuNavigationLink @route` focuses `#content` on click. Preserve these IDs
  uniquely. Do not nest another main inside the shell's main.
- `AuMainFooter` includes Flanders branding; `AuBodyContainer @scroll` is a
  body layout alternative, not another main landmark.
- Place `AuModalContainer` before any modal construction and generally once
  outside shell overflow clipping. Mount `AuToaster` once for notifications.
- `AuContent` styles prose. `au-o-layout` constrains width; `au-o-box` supplies
  padding. Do not assume AuContent supplies either.
- `AuContentHeader` yields into a picture and renders title spans; it does
  **not** replace the semantic page heading.

The examples are resolver-style `.hbs`; strict templates must explicitly import
components as described in [ember-patterns.md](ember-patterns.md). Route names
and image URLs are app examples, not addon-provided routes/assets.

## Standard content page

An application template can own the shell; a route template then owns the page.
Keep one shell instance across routes rather than duplicating it in each page.

```hbs
{{! application.hbs }}
<AuModalContainer />
<AuApp>
  <AuMainHeader @appTitle="Digitaal loket" @homeRoute="index">
    <AuLink @route="settings">Instellingen</AuLink>
  </AuMainHeader>
  <AuMainContainer as |m|>
    <m.content @scroll={{true}}>
      {{outlet}}
      <AuMainFooter><p>Een toepassing van de Vlaamse overheid.</p></AuMainFooter>
    </m.content>
  </AuMainContainer>
</AuApp>
<AuToaster />
```

```hbs
{{! index.hbs, rendered inside m.content }}
<div class="au-o-layout au-o-box">
  <nav aria-label="Broodkruimel">
    <ol class="au-c-list-horizontal au-c-list-horizontal--small">
      <li class="au-c-list-horizontal__item"><span
          aria-current="page"
        >Start</span></li>
    </ol>
  </nav>
  <AuHeading @level="1" @skin="2">Welkom bij het loket</AuHeading>
  <AuContent>
    <p>Beheer uw aanvragen en raadpleeg uw documenten.</p>
    <AuLink @route="requests" @skin="button">Bekijk aanvragen</AuLink>
  </AuContent>
</div>
```

There is no public `AuBreadcrumbs` component. Compose a labelled nav with a
semantic list and actual route links; use `aria-current="page"` for its current
item. The horizontal-list class provides layout, not breadcrumb semantics or
automatic separators.

## Settings/list page

For an application with a persistent settings sidebar, use this shell **instead
of** the preceding shell. `this.signOut` is an app action; `settings.profile`,
`settings.notifications`, and `settings.members` must exist in the app router.

```hbs
<AuModalContainer />
<AuApp>
  <AuMainHeader @appTitle="Instellingen" @homeRoute="index">
    <AuDropdown @title="Mijn account" @alignment="right">
      <AuButton
        @skin="link"
        role="menuitem"
        {{on "click" this.signOut}}
      >Afmelden</AuButton>
    </AuDropdown>
  </AuMainHeader>
  <AuMainContainer as |m|>
    <m.sidebar @size="small">
      <div class="au-c-sidebar">
        <div class="au-c-sidebar__content">
          <nav aria-label="Instellingen">
            <ul class="au-c-list-navigation">
              <li class="au-c-list-navigation__item">
                <AuNavigationLink
                  @route="settings.profile"
                >Profiel</AuNavigationLink>
              </li>
              <li class="au-c-list-navigation__item">
                <AuNavigationLink
                  @route="settings.notifications"
                >Meldingen</AuNavigationLink>
              </li>
            </ul>
          </nav>
        </div>
      </div>
    </m.sidebar>
    <m.content @scroll={{true}}>{{outlet}}</m.content>
  </AuMainContainer>
</AuApp>
<AuToaster />
```

A members route inside that content area can use a simple static list/table:

```hbs
<div class="au-o-layout au-o-box">
  <nav aria-label="Broodkruimel">
    <ol class="au-c-list-horizontal au-c-list-horizontal--small">
      <li class="au-c-list-horizontal__item"><AuLink
          @route="index"
        >Start</AuLink></li>
      <li class="au-c-list-horizontal__item"><span
          aria-current="page"
        >Leden</span></li>
    </ol>
  </nav>
  <AuToolbar @border="bottom" as |Group|>
    <Group><AuHeading @level="1" @skin="2">Leden</AuHeading></Group>
    <Group><AuButton {{on "click" this.invite}}>Lid uitnodigen</AuButton></Group>
  </AuToolbar>
  <AuTable>
    <:title>Leden van uw organisatie</:title>
    <:header><tr><th scope="col">Naam</th><th
          scope="col"
        >Status</th></tr></:header>
    <:body>
      {{#each this.members as |member|}}
        <tr><th scope="row">{{member.name}}</th><td><AuPill
            >{{member.status}}</AuPill></td></tr>
      {{else}}
        <tr><td colspan="2">Er zijn nog geen leden.</td></tr>
      {{/each}}
    </:body>
  </AuTable>
</div>
```

For route-style tabs use `AuTabs as |Tab|` with `<Tab><AuLink …>…</AuLink></Tab>`.
AuTabs is nav/list composition, **not** an interactive ARIA tab-panel system.
For searchable/paginated server listings, inspect AuDataTable's metadata and
classic binding contracts before replacing AuTable.

## Responsive layout and common patterns

- `au-o-grid` wraps; direct `au-o-grid__item` children default to full width.
  Verified responsive width syntax: `au-u-1-1 au-u-1-2@medium`. Width fractions
  are generated from `$au-fractions`; inspect overrides before using one.
- Default breakpoint map: tiny 300px, xsmall 500px, small 767px, medium 1023px,
  large 1600px (`settings/_s-global.scss`). Main-container sidebar/content
  become side-by-side at medium by default; below that they stack.
- `m.sidebar @size="collapsed"` is a desktop width variant, not a toggleable
  mobile drawer. Implement needed mobile disclosure using the app's pattern,
  with its own labelled control and focus handling.
- `m.content @scroll={{true}}` adds medium-and-up content scrolling. Check
  nested scroll containers, viewport height, sticky elements, zoom, and focus
  outlines; a scrollbar in the desktop shell is not proof mobile works.
- Standard full import gives layout widths 1280px / small 768px / large 1600px
  through global settings. The layout partial alone has different fallback
  defaults; import order matters.
- Use box/flow/grid classes for spacing before custom positioning. Use
  `AuToolbar` groups for page heading/actions, allowing wrapping by default.
  CSS visual reversal must not create a confusing keyboard/reading order.
- Check long Dutch labels, narrow viewports, 200% zoom, empty/loading/error
  states, table overflow, and print behavior. Yielded sidebar is hidden on
  print and content made wide through existing utilities.
