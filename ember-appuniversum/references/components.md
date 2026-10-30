# Component catalog

## Contents

Selection guide · Actions · Forms · Feedback · Overlays · Navigation · Data display
(including legacy subcomponents/aliases) · Layout · Utilities · Generated icon family.
Search for `### AuName` to read only the component you need.

## Scope and source

Verified locally: `@appuniversum/ember-appuniversum` **4.5.1**, commit **4052636**.
The 62 checked-in public component paths are the component entries in
`appuniversum/package.json` → `ember-addon.app-js`, backed by
`appuniversum/src/components/`. They include ten nested data-table components and
three flat compatibility aliases (59 implementations). The build additionally
generates import-only icon components; see the final section. Private components
are implementation details, not consuming-app APIs.

Each entry names its repository-relative implementation file. Read that file and
its matching test under `appuniversum/tests/integration/components/` before
changing version-sensitive usage. These are concise API summaries, not promises
that every argument combination has been tested.

**Invocation conventions:** examples below use `.hbs` resolver names. Strict
`.gjs`/`.gts` templates must import values, e.g.
`import AuButton from '@appuniversum/ember-appuniversum/components/au-button';`.
Nested resolver syntax is `<AuDataTable::Content …>`; strict templates may import
`…/components/au-data-table/content` and name that binding `TableContent`.
Examples referring to `this.*`, `person`, or `table` assume app-provided state.
Contextual examples `t`, `c`, and `menu` are scoped by the composition below.
Unless stated otherwise, blocks receive no values, callbacks are absent, and
native attributes are forwarded to the root element. Do not infer arbitrary
attribute support for tagless/classic components.

## Component selection

| UI need                                      | Prefer                                            | Boundary                                                  |
| -------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------- |
| Perform an action                            | AuButton                                          | Native click handler; not navigation                      |
| Route / URL                                  | AuLink / AuLinkExternal                           | @route versus native href                                 |
| Text / multiline / typed date / calendar     | AuInput / AuTextarea / AuDateInput / AuDatePicker | Different value/event contracts                           |
| Multiple / single choices                    | AuCheckboxGroup / AuRadioGroup                    | Accessible group name required                            |
| Boolean setting                              | AuToggleSwitch                                    | Supply visible label block                                |
| Field label / hint / row / group             | AuLabel / AuHelpText / AuFormRow / AuFieldset     | App owns validation                                       |
| Feedback / loading / transient notification  | AuAlert / AuLoader / AuToaster                    | Alert and status have different urgency                   |
| Dialog / action menu / hint                  | AuModal / AuDropdown / AuTooltip                  | Do not interchange focus semantics                        |
| Small static table / metadata-driven listing | AuTable / AuDataTable                             | Legacy table requires query/binding integration           |
| Page shell                                   | AuApp + AuMainHeader + AuMainContainer            | Preserve #main/#content focus targets                     |
| Related content / collapsible details        | AuCard / AuAccordion                              | Audit expandable-card keyboard behavior                   |
| Breadcrumb / sidebar / grid / native form    | Semantic HTML + verified styles                   | No dedicated public Ember component                       |
| Select control                               | Existing app select/PowerSelect integration       | No public AuSelect; styling is not a dependency guarantee |

## Data-table contextual scope

```hbs
<AuDataTable
  @content={{this.people}}
  @hidePagination={{true}}
  @enableSelection={{false}}
  as |t table|
>
  <t.menu as |menu|>
    <menu.general><AuButton
        {{on "click" this.create}}
      >Nieuw</AuButton></menu.general>
  </t.menu>
  <t.content aria-label="Personen" as |c|>
    <c.header><th scope="col">Naam</th></c.header>
    <c.body as |person|><td>{{person.name}}</td></c.body>
  </t.content>
</AuDataTable>
```

The outer block executes separately for menu/content hashes. The classic
subcomponents do not expose a modern controlled callback API: verify installed
version bindings rather than substituting invented handlers.

## Actions

### AuButton

- **Purpose:** Perform an action.
- **Source:** `appuniversum/src/components/au-button.gts`.
- **API / blocks / events:** @skin: primary (default), secondary, naked, link, link-secondary, link-bold; @size="large"; @width="block"; @alert, @disabled, @loading, @loadingMessage, @wrap, @icon, @iconAlignment, @hideText. Default label block; native click event.
- **Use / accessibility / avoid:** Use for actions, not navigation. Defaults to type="button"; set type="submit" explicitly. Loading disables the button; provide a meaningful loadingMessage and keep hidden labels meaningful.

```hbs
<AuButton
  type="submit"
  @loading={{this.saving}}
  @loadingMessage="Opslaan…"
>Opslaan</AuButton>
```

### AuButtonGroup

- **Purpose:** Space related actions.
- **Source:** `appuniversum/src/components/au-button-group.gts`.
- **API / blocks / events:** @inline boolean; default block. No callback.
- **Use / accessibility / avoid:** Use for related buttons, not an automatic ARIA toolbar or unrelated navigation.

```hbs
<AuButtonGroup><AuButton>Bewaren</AuButton><AuButton
    @skin="secondary"
  >Annuleren</AuButton></AuButtonGroup>
```

## Forms

### AuInput

- **Purpose:** Style a native single-line input.
- **Source:** `appuniversum/src/components/au-input.gts`.
- **API / blocks / events:** @disabled, @error, @warning, @width="block", @icon, @iconAlignment="left|right". No block/callback; attributes and modifiers land on input even with an icon wrapper.
- **Use / accessibility / avoid:** Use value= and DOM input/change; not @value/@onChange. Supply a label, native type/required/readonly and explicit error associations.

```hbs
<AuInput
  id="name"
  name="name"
  value={{this.name}}
  {{on "input" this.updateName}}
/>
```

### AuTextarea

- **Purpose:** Style a native multiline control.
- **Source:** `appuniversum/src/components/au-textarea.gts`.
- **API / blocks / events:** @disabled, @error, @warning, @width="block"; default block for initial text; native input/change.
- **Use / accessibility / avoid:** Use for multiline text, not rich editing. Link a label; no @value or @onChange. Test native value updates in your app.

```hbs
<AuTextarea
  id="notes"
  rows="4"
  {{on "input" this.updateNotes}}
>{{this.notes}}</AuTextarea>
```

### AuLabel

- **Purpose:** Render an associated form label.
- **Source:** `appuniversum/src/components/au-label.gts`.
- **API / blocks / events:** @error, @warning, @inline, @required, @requiredLabel; default text block. Native for attribute.
- **Use / accessibility / avoid:** Use for standalone controls; @required is visual only and does not set required on the input.

```hbs
<AuLabel for="name" @required={{true}}>Naam</AuLabel>
```

### AuHelpText

- **Purpose:** Display field guidance or validation text.
- **Source:** `appuniversum/src/components/au-help-text.gts`.
- **API / blocks / events:** @skin="secondary|tertiary", @size="normal|large", @error, @warning; default block; span root.
- **Use / accessibility / avoid:** Use as a linked description, not an automatic validator/live region. Set id and aria-describedby on the control.

```hbs
<AuHelpText id="name-help">Gebruik uw volledige naam.</AuHelpText>
```

### AuFormRow

- **Purpose:** Arrange a label/control row.
- **Source:** `appuniversum/src/components/au-form-row.gts`.
- **API / blocks / events:** @alignment="inline|pre|post"; default block. No validation/events.
- **Use / accessibility / avoid:** Use inside a native form, not as a replacement for form submission semantics.

```hbs
<AuFormRow><AuLabel for="city">Gemeente</AuLabel><AuInput
    id="city"
  /></AuFormRow>
```

### AuFieldset

- **Purpose:** Group related controls with contextual legend/content.
- **Source:** `appuniversum/src/components/au-fieldset.gts`.
- **API / blocks / events:** @alignment="inline"; yields separate hashes with lowercase legend and content. Legend accepts @skin="1…6|functional", @error, @warning, @required, @requiredLabel, @inline; content takes a default block.
- **Use / accessibility / avoid:** Use for related questions; not a generic section. Legend currently has a div wrapper; inspect resulting semantics. Its visual required marker does not enforce validation.

```hbs
<AuFieldset as |f|><f.legend>Voorkeur</f.legend><f.content><AuCheckbox
    >Nieuws</AuCheckbox></f.content></AuFieldset>
```

### AuCheckbox

- **Purpose:** Select a boolean or group member.
- **Source:** `appuniversum/src/components/au-checkbox.gts`.
- **API / blocks / events:** @checked, @disabled, @indeterminate, @name, @value; @onChange(checked, event); default label block. Group plumbing: @inGroup, @groupValue, @onChangeGroup.
- **Use / accessibility / avoid:** Use for independent choices, not mutually exclusive options. Prefer group-yielded Checkbox instead of wiring group plumbing. Attributes target input.

```hbs
<AuCheckbox @checked={{this.agreed}} @onChange={{this.setAgreed}}>Ik ga akkoord</AuCheckbox>
```

### AuCheckboxGroup

- **Purpose:** Manage several selected string values.
- **Source:** `appuniversum/src/components/au-checkbox-group.gts`.
- **API / blocks / events:** @selected string[], @name, @disabled, @alignment="inline", @onChange(updatedValues, event); yields { Checkbox }. Root role="group".
- **Use / accessibility / avoid:** Use for multi-select questions, not a single choice. Name the group through aria-labelledby or a fieldset.

```hbs
<AuCheckboxGroup
  @selected={{this.channels}}
  @onChange={{this.setChannels}}
  aria-label="Kanalen"
  as |g|
><g.Checkbox @value="email">E-mail</g.Checkbox></AuCheckboxGroup>
```

### AuRadio

- **Purpose:** Select a single radio option.
- **Source:** `appuniversum/src/components/au-radio.gts`.
- **API / blocks / events:** @checked, @disabled, @name, @value; @onChange(selectedString, event); default label block. Group-only args: @inGroup, @groupValue, @onChangeGroup.
- **Use / accessibility / avoid:** Prefer AuRadioGroup for shared names/state. Do not handle the callback as a boolean. Attributes target input.

```hbs
<AuRadio
  @name="delivery"
  @value="email"
  @checked={{true}}
  @onChange={{this.setDelivery}}
>E-mail</AuRadio>
```

### AuRadioGroup

- **Purpose:** Coordinate mutually exclusive choices.
- **Source:** `appuniversum/src/components/au-radio-group.gts`.
- **API / blocks / events:** @selected, @name (generated if omitted), @disabled, @alignment="inline", @onChange(value, event); yields { Radio }. Root role="group".
- **Use / accessibility / avoid:** Use for a single selection, not checkbox-style sets. Supply an accessible group name; selected value can be undefined.

```hbs
<AuRadioGroup
  @selected={{this.delivery}}
  @onChange={{this.setDelivery}}
  aria-label="Levering"
  as |g|
><g.Radio @value="email">E-mail</g.Radio><g.Radio
    @value="post"
  >Post</g.Radio></AuRadioGroup>
```

### AuToggleSwitch

- **Purpose:** Toggle a boolean setting.
- **Source:** `appuniversum/src/components/au-toggle-switch.gts`.
- **API / blocks / events:** @checked, @disabled, @identifier, @name, @alignment="right", @onChange(checked,event); default label block. @label exists in signature but only affects labelless styling.
- **Use / accessibility / avoid:** Use for an on/off setting, not a submit action. Supply visible block text: @label alone does not render a label. Input receives attributes.

```hbs
<AuToggleSwitch
  @identifier="notifications"
  @checked={{this.enabled}}
  @onChange={{this.setEnabled}}
>Meldingen</AuToggleSwitch>
```

### AuDateInput

- **Purpose:** Enter a masked Belgian-format date.
- **Source:** `appuniversum/src/components/au-date-input.gts`.
- **API / blocks / events:** @value ISO date string or Date, @prefillYear, @onChange(isoDate|null, Date|null); @disabled, @error, @warning, @width="block". No block.
- **Use / accessibility / avoid:** Use for typed dates without a calendar. Add a label and description; callback fires on complete/cleared mask, not every keystroke.

```hbs
<AuDateInput
  id="birthday"
  @value={{this.birthday}}
  @onChange={{this.setBirthday}}
/>
```

### AuDatePicker

- **Purpose:** Select a date through a Duet calendar.
- **Source:** `appuniversum/src/components/au-date-picker.gts`.
- **API / blocks / events:** @value, @min, @max: ISO date string or Date; @onChange(isoDate|null, Date|null); @id, @label, @buttonLabel, @disabled, @error, @warning, @alignment="top", @firstDay, legacy @first-day, @adapter, @localization. No block.
- **Use / accessibility / avoid:** Use when a calendar is needed. @id identifies the internal input; native attributes target duet-date-picker, not that input. Dutch-Belgian defaults; verify shadow/internal accessibility. firstDay uses a truthy fallback, so test Sunday (0).

```hbs
<AuDatePicker
  @id="due"
  @label="Uiterste datum"
  @value={{this.due}}
  @onChange={{this.setDue}}
/>
```

### AuTimePicker

- **Purpose:** Edit hours, minutes, and seconds.
- **Source:** `appuniversum/src/components/au-time-picker.gts`.
- **API / blocks / events:** @hours, @minutes, @seconds; @hoursLabel, @minutesLabel, @secondsLabel, @nowLabel; @showNow and @showSeconds default true; @onChange({hours,minutes,seconds}). No block or ...attributes.
- **Use / accessibility / avoid:** Use for time entry, not dates. Labels must be supplied. Hard-coded input-hour/input-minute/input-second IDs mean multiple instances need special review; no generic disabled argument.

```hbs
<AuTimePicker
  @hours={{9}}
  @minutes={{30}}
  @hoursLabel="Uren"
  @minutesLabel="Minuten"
  @secondsLabel="Seconden"
  @nowLabel="Nu"
  @showSeconds={{false}}
  @onChange={{this.setTime}}
/>
```

### AuFileUpload

- **Purpose:** Upload selected/dropped files.
- **Source:** `appuniversum/src/components/au-file-upload.gts`.
- **API / blocks / events:** @accept, @endPoint (default /files), @multiple, @maxFileSizeMB (20), @minFileSizeKB (0), @title, @helpTextDragDrop, @helpTextFileNotSupported; @onUpload({fileId,file}), @onFinishUpload(id,{isQueueEmpty}), @onQueueUpdate({isQueueEmpty}). No block.
- **Use / accessibility / avoid:** Use only with a compatible upload endpoint: multipart response is {data:{id}}. Not a generic file picker; server must validate too. Attributes target dropzone; audit keyboard access to the hidden input.

```hbs
<AuFileUpload @accept=".pdf" @endPoint="/files" @onUpload={{this.attachFile}} />
```

## Feedback

### AuAlert

- **Purpose:** Announce contextual feedback.
- **Source:** `appuniversum/src/components/au-alert.gts`.
- **API / blocks / events:** @skin="info|success|warning|error", @title, @size="tiny|small", @icon, @iconVisible, @closable, @onClose(); default message block.
- **Use / accessibility / avoid:** Use for important feedback; role="alert" is assertive, so avoid filling static pages with alerts. Closing hides local content then calls onClose.

```hbs
<AuAlert @skin="error" @title="Opslaan mislukt"><p>Probeer opnieuw.</p></AuAlert>
```

### AuBadge

- **Purpose:** Display an icon/count badge.
- **Source:** `appuniversum/src/components/au-badge.gts`.
- **API / blocks / events:** @icon, @number, @iconVisible, @size="small", @skin="action|border|brand|error|success|warning". No block/event.
- **Use / accessibility / avoid:** Use supplementary indicators, not standalone actions. Root is aria-hidden by default; zero is not rendered because number is tested for truthiness. Provide equivalent text for meaningful state.

```hbs
<AuBadge
  @number={{3}}
  @skin="action"
  @iconVisible={{true}}
  aria-label="3 meldingen"
/>
```

### AuPill

- **Purpose:** Display status, a tag, link, or removable token.
- **Source:** `appuniversum/src/components/au-pill.gts`.
- **API / blocks / events:** @skin="action|border|error|link|ongoing|success|warning", @size="small", @draft, @icon, @iconAlignment, @hideText; @route/@model/@models/@query or @href; @onClickAction(), @actionIcon, @actionText; default block.
- **Use / accessibility / avoid:** Use for compact status/tags, not a main CTA. actionText supplies the adjacent action button name; onClickAction takes precedence over route/href rendering.

```hbs
<AuPill @skin="success">Goedgekeurd</AuPill>
```

### AuLoader

- **Purpose:** Announce loading progress.
- **Source:** `appuniversum/src/components/au-loader.gts`.
- **API / blocks / events:** @inline, @hideMessage, @centered (default true); default message block. role="status".
- **Use / accessibility / avoid:** Use for asynchronous waiting, not an empty decorative spinner. Supply status text even when visually hidden.

```hbs
<AuLoader @hideMessage={{true}}>Gegevens laden…</AuLoader>
```

### AuToaster

- **Purpose:** Render notifications from the toaster service.
- **Source:** `appuniversum/src/components/au-toaster.gts`.
- **API / blocks / events:** @position="top|bottom"; no block. Uses toaster.toasts and AuAlert, or a custom toast component.
- **Use / accessibility / avoid:** Mount once in the app shell, not per row. Service options/timeouts and custom components are in ember-patterns.md; do not use as durable inline validation.

```hbs
<AuToaster @position="bottom" />
```

## Overlays

### AuModal

- **Purpose:** Present a focus-trapped dialog.
- **Source:** `appuniversum/src/components/au-modal.gts`.
- **API / blocks / events:** @modalOpen, @closeModal(), @closable (true), @id, @title, @initialFocus selector, @size="large|fullscreen", @overflow, @padding="none". Named title/body/footer OR default { Body, Footer }.
- **Use / accessibility / avoid:** Requires AuModalContainer before construction, even if closed. Do not mix named and contextual modes. Close callback must change app state. Source sets aria-describedby to title, not aria-labelledby; provide a verified accessible name and test focus return.

```hbs
<AuModal
  @id="confirm"
  @modalOpen={{this.open}}
  @closeModal={{this.close}}
  @title="Bevestigen"
  aria-label="Bevestigen"
><:body>Wilt u doorgaan?</:body></AuModal>
```

### AuModalContainer

- **Purpose:** Provide the modal portal destination.
- **Source:** `appuniversum/src/components/au-modal-container.gts`.
- **API / blocks / events:** No arguments/block/events; div with data-au-modal-container; attributes forwarded.
- **Use / accessibility / avoid:** Mount before any AuModal and outside unwanted clipping. Not the modal itself; avoid multiple competing destinations.

```hbs
<AuModalContainer />
```

### AuDropdown

- **Purpose:** Open an action menu.
- **Source:** `appuniversum/src/components/au-dropdown.gts`.
- **API / blocks / events:** @title, @alignment="left|right", @skin (naked default), @size="large", @alert, @disabled, @hideText, @icon, @iconAlignment, @onClose(); default menu block.
- **Use / accessibility / avoid:** Use for actions, not an invented select control. Menu clicks close it; root attributes do not target trigger. Supplies aria-expanded and focus trap, but no automatic roving menu-item keyboard system.

```hbs
<AuDropdown @title="Acties"><AuButton
    @skin="link"
    role="menuitem"
    {{on "click" this.archive}}
  >Archiveren</AuButton></AuDropdown>
```

### AuTooltip

- **Purpose:** Attach supplemental hover/focus text.
- **Source:** `appuniversum/src/components/au-tooltip.gts`.
- **API / blocks / events:** @placement="top|right|bottom|left"; yields {target modifier, Content component, isShown}. No root element.
- **Use / accessibility / avoid:** Use nonessential hints, not the only label or interactive popup. Escape hides; link target aria-describedby to Content id yourself.

```hbs
<AuTooltip as |t|><AuButton
    {{t.target}}
    aria-describedby="save-tip"
  >Bewaren</AuButton><t.Content id="save-tip">Slaat uw wijzigingen op.</t.Content></AuTooltip>
```

## Navigation

### AuLink

- **Purpose:** Navigate to an Ember route.
- **Source:** `appuniversum/src/components/au-link.gts`.
- **API / blocks / events:** Required @route; @model/@models, @query, @active; @skin="primary|secondary|bold|button|button-secondary|button-naked", @width="block", @icon, @iconAlignment, @hideText; default link text.
- **Use / accessibility / avoid:** Use internal navigation, not actions or @linkRoute from obsolete stories. Preserve meaningful link text, including icon-only hidden text.

```hbs
<AuLink @route="settings">Instellingen</AuLink>
```

### AuLinkExternal

- **Purpose:** Render a URL anchor.
- **Source:** `appuniversum/src/components/au-link-external.gts`.
- **API / blocks / events:** Native href; @newTab (default true), @skin="primary|secondary|bold|button|button-secondary|button-naked", @width="block", @icon, @iconAlignment, @hideText; default text.
- **Use / accessibility / avoid:** Use URLs, not route names or actions. newTab sets _blank and noopener noreferrer; choose false for same-tab navigation and communicate new windows where needed.

```hbs
<AuLinkExternal
  href="https://www.vlaanderen.be/"
  @newTab={{false}}
>Vlaanderen</AuLinkExternal>
```

### AuNavigationLink

- **Purpose:** Render a sidebar/list route link and focus content.
- **Source:** `appuniversum/src/components/au-navigation-link.gts`.
- **API / blocks / events:** Required @route; @model/@models, @query; default text. Click handler focuses #content.
- **Use / accessibility / avoid:** Use within application navigation, not arbitrary external URLs. Requires an actual focusable #content; no @linkRoute.

```hbs
<AuNavigationLink @route="settings">Instellingen</AuNavigationLink>
```

### AuTabs

- **Purpose:** Lay out route-navigation tabs.
- **Source:** `appuniversum/src/components/au-tabs.gts`.
- **API / blocks / events:** @reversed; yields a Tab li component directly; nav > ul structure.
- **Use / accessibility / avoid:** Use linked navigation tabs, not a complete ARIA tablist with automatic tab-panel state/arrow navigation. Label the nav.

```hbs
<AuTabs aria-label="Instellingen" as |Tab|><Tab><AuLink
      @route="settings"
    >Algemeen</AuLink></Tab></AuTabs>
```

### AuList

- **Purpose:** Lay out a semantic list.
- **Source:** `appuniversum/src/components/au-list.gts`.
- **API / blocks / events:** @direction="horizontal" (otherwise vertical), @divider; yields ListItem directly.
- **Use / accessibility / avoid:** Use for list items, not as automatic navigation or a table. Preserve li structure with yielded Item.

```hbs
<AuList as |Item|><Item>Eerste item</Item><Item>Tweede item</Item></AuList>
```

## Data display

### AuTable

- **Purpose:** Render a static semantic table.
- **Source:** `appuniversum/src/components/au-table.gts`.
- **API / blocks / events:** @size="small"; named title (caption), header (thead), body (tbody), footer (tfoot). Attributes target table, not wrapper.
- **Use / accessibility / avoid:** Use for static/tabular data; supply tr/th/td and scope yourself. No default block or automatic pagination/sorting.

```hbs
<AuTable><:title>Gemeenten</:title><:header><tr><th
        scope="col"
      >Naam</th></tr></:header><:body><tr><td>Gent</td></tr></:body></AuTable>
```

### AuDataTable

- **Purpose:** Compose a classic Ember data listing.
- **Source:** `appuniversum/src/components/au-data-table.gjs`.
- **API / blocks / events:** @content, @fields (array or space-separated string), @page (zero-based), @size, @sort, @filter, @autoSearch, @hidePagination, @showPaginationBoundaryLinks, @isLoading, @lineNumbers, @enableSelection, @selection, @enableSizes, @sizes, @noDataMessage, @link, @tableClass, @onClickRow(item). Yields separate {menu} and {content} hashes, then table instance as second param.
- **Use / accessibility / avoid:** Use when existing app data/query contracts match. It does not fetch data. Pagination reads content.meta.count and .pagination links; classic write-back needs version-matched integration. Row clicks are not keyboard links; selection inputs have a source TODO for labels. Prefer AuTable for simple tables. searchDebounceTime exists but is not wired to TextSearch wait.

```hbs
<AuDataTable
  @content={{this.people}}
  @fields="name"
  @hidePagination={{true}}
  @enableSelection={{false}}
/>
```

### AuAccordion

- **Purpose:** Reveal a collapsible section.
- **Source:** `appuniversum/src/components/au-accordion.gts`.
- **API / blocks / events:** @buttonLabel, @subtitle, @isOpenInitially, @loading, @reverse, @skin="border", @iconOpen, @iconClosed; default content.
- **Use / accessibility / avoid:** Use for collapsible content, not route navigation. Internal state only; opening focuses content; button has aria-expanded. No @isOpen controlled API.

```hbs
<AuAccordion @buttonLabel="Meer informatie"><p>Extra uitleg.</p></AuAccordion>
```

### AuCard

- **Purpose:** Group related content in an article.
- **Source:** `appuniversum/src/components/au-card.gts`.
- **API / blocks / events:** @size="small|tiny|flush", @divided, @flex, @shadow, @standOut, @textCenter, @expandable, @isOpenInitially; controlled @manualControl/@isExpanded/@openSection(). Yields separate lowercase header/content/footer hashes. Header badgeIcon/badgeNumber/badgeSize/badgeSkin args supported.
- **Use / accessibility / avoid:** Use related summaries, not a generic clickable div. Expandable clickable wrapper has role=button without its own keyboard handler; audit keyboard/name behavior. Content/footer disableAuContent is declared but unused.

```hbs
<AuCard as |c|><c.header><AuHeading
      @level="2"
      @skin="4"
    >Dossier</AuHeading></c.header><c.content><p
    >Details.</p></c.content></AuCard>
```

### AuFileCard

- **Purpose:** Display an uploaded/downloadable file.
- **Source:** `appuniversum/src/components/au-file-card.gts`.
- **API / blocks / events:** Required @filename; @fileSize string, @downloadLink, @onDelete(); no block.
- **Use / accessibility / avoid:** Use an existing file summary, not upload selection. Delete control has hidden text; filename is rendered as h4, so review heading hierarchy.

```hbs
<AuFileCard
  @filename="verslag.pdf"
  @fileSize="120 KB"
  @downloadLink="/files/verslag.pdf"
/>
```

### AuDataTable::Content

- **Purpose:** Build the table body/header composition layer.
- **Source:** `appuniversum/src/components/au-data-table/content.gjs`.
- **API / blocks / events:** @content, @noDataMessage, @enableSelection, @enableLineNumbers, @onClickRow, @data-table; tableClass aliases data-table.tableClass. Default yields {header,body}; ...attributes target table.
- **Use / accessibility / avoid:** Prefer t.content from AuDataTable, not constructing private-looking table state by hand. Supply a table name/caption and semantic cells.

```hbs
<t.content aria-label="Personen" as |c|><c.header><th
      scope="col"
    >Naam</th></c.header><c.body as |person|><td
    >{{person.name}}</td></c.body></t.content>
```

### AuDataTable::ContentHeader

- **Purpose:** Render a data-table thead and header row.
- **Source:** `appuniversum/src/components/au-data-table/content-header.gjs`.
- **API / blocks / events:** @data-table (sort/parsedFields), @enableSelection, @enableLineNumbers; default block or generated sortable fields.
- **Use / accessibility / avoid:** Prefer contextual c.header inside t.content. Default header labels come from fields; no heading content outside th cells.

```hbs
<c.header><th scope="col">Naam</th></c.header>
```

### AuDataTable::ContentBody

- **Purpose:** Render table rows with optional selection/line numbers.
- **Source:** `appuniversum/src/components/au-data-table/content-body.gjs`.
- **API / blocks / events:** @content, @data-table, @noDataMessage, @enableSelection, @enableLineNumbers, @onClickRow(item); default yields one item per row, expects td cells.
- **Use / accessibility / avoid:** Prefer contextual c.body; do not add a second tr around cells. Default selection checkboxes lack labels; row-click alone is not keyboard navigation.

```hbs
<c.body as |person|><td>{{person.name}}</td></c.body>
```

### AuDataTable::DefaultDataTableContentBody

- **Purpose:** Render default field cells.
- **Source:** `appuniversum/src/components/au-data-table/default-data-table-content-body.gjs`.
- **API / blocks / events:** @item, @data-table (parsedFields/link); default block after cells; first field may become AuLink.
- **Use / accessibility / avoid:** Normally let content-body create this. Not a tbody or row; use only inside a tr with correctly matched field headings.

```hbs
<AuDataTable::DefaultDataTableContentBody
  @item={{person}}
  @data-table={{table}}
/>
```

### AuDataTable::Menu

- **Purpose:** Compose general/selected table actions.
- **Source:** `appuniversum/src/components/au-data-table/menu.gjs`.
- **API / blocks / events:** @data-table; default yields {general,selected}.
- **Use / accessibility / avoid:** Prefer t.menu from AuDataTable. This is a classic wrapper, not an ARIA menu widget; label actions normally.

```hbs
<t.menu as |menu|><menu.general><AuButton
    >Nieuw</AuButton></menu.general><menu.selected as |selection|><p
    >{{selection.length}} geselecteerd</p></menu.selected></t.menu>
```

### AuDataTable::MenuGeneral

- **Purpose:** Show actions only for empty selection.
- **Source:** `appuniversum/src/components/au-data-table/menu-general.gjs`.
- **API / blocks / events:** @data-table; default block, no yielded values/events.
- **Use / accessibility / avoid:** Prefer menu.general inside t.menu; not a standalone dropdown menu.

```hbs
<menu.general><AuButton>Nieuw dossier</AuButton></menu.general>
```

### AuDataTable::MenuSelected

- **Purpose:** Show actions for selected rows.
- **Source:** `appuniversum/src/components/au-data-table/menu-selected.gjs`.
- **API / blocks / events:** @data-table; init enables selection; default yields copied selection and table instance.
- **Use / accessibility / avoid:** Prefer menu.selected. Provide named/keyboard-accessible row selection; a conditional/nested composition can affect initialization.

```hbs
<menu.selected as |selection table|><AuButton
    {{on "click" table.clearSelection}}
  >Selectie wissen ({{selection.length}})</AuButton></menu.selected>
```

### AuDataTable::NumberPagination

- **Purpose:** Render classic page navigation.
- **Source:** `appuniversum/src/components/au-data-table/number-pagination.gjs`.
- **API / blocks / events:** @page, @size, @nbOfItems, @total, @links (first/last/prev/next with number), @showBoundaryLinks; @sizeOptions passed by parent but not rendered. No block; writes page via classic set.
- **Use / accessibility / avoid:** Normally let AuDataTable configure it. Do not invent @onPageChange; inspect link numbering and write-back before standalone use. Disabled controls guard missing boundary links.

```hbs
<AuDataTable::NumberPagination
  @page={{mut this.page}}
  @size={{10}}
  @total={{this.total}}
  @links={{this.links}}
/>
```

### AuDataTable::TextSearch

- **Purpose:** Render the legacy debounced search input.
- **Source:** `appuniversum/src/components/au-data-table/text-search.gjs`.
- **API / blocks / events:** @filter, @placeholder (Search), @wait (debounce delay), @auto (declared but not used in template); no block/callback. Writes filter via classic set.
- **Use / accessibility / avoid:** Normally use parent filter integration. Not a modern @onChange API; verify classic write-back and debounce in the app. Placeholder becomes aria-label.

```hbs
<AuDataTable::TextSearch
  @filter={{mut this.filter}}
  @placeholder="Zoeken"
  @wait={{300}}
/>
```

### AuDataTable::ThSortable

- **Purpose:** Render a sortable column header.
- **Source:** `appuniversum/src/components/au-data-table/th-sortable.gjs`.
- **API / blocks / events:** @field, @label, @currentSorting; no block/callback. Converts field to dash-case; cycles field, -field, empty and sets currentSorting.
- **Use / accessibility / avoid:** Use in a table header with classic bindings, not an invented @onSort. Supplies a button with hidden Dutch text but no aria-sort attribute; review announcement.

```hbs
<AuDataTable::ThSortable
  @field="name"
  @label="Naam"
  @currentSorting={{mut this.sort}}
/>
```

### AuDataTableNumberPagination

- **Purpose:** Compatibility alias for AuDataTable::NumberPagination.
- **Source:** `appuniversum/src/components/au-data-table-number-pagination.js`.
- **API / blocks / events:** @page, @size, @nbOfItems, @total, @links (first/last/prev/next with number), @showBoundaryLinks; @sizeOptions passed by parent but not rendered. No block; writes page via classic set.
- **Use / accessibility / avoid:** Existing apps may retain this flat alias; prefer the nested export for new direct imports. Normally let AuDataTable configure it. Do not invent @onPageChange; inspect link numbering and write-back before standalone use. Disabled controls guard missing boundary links.

```hbs
<AuDataTableNumberPagination
  @page={{mut this.page}}
  @size={{10}}
  @total={{this.total}}
  @links={{this.links}}
/>
```

### AuDataTableTextSearch

- **Purpose:** Compatibility alias for AuDataTable::TextSearch.
- **Source:** `appuniversum/src/components/au-data-table-text-search.js`.
- **API / blocks / events:** @filter, @placeholder (Search), @wait (debounce delay), @auto (declared but not used in template); no block/callback. Writes filter via classic set.
- **Use / accessibility / avoid:** Existing apps may retain this flat alias; prefer the nested export for new direct imports. Normally use parent filter integration. Not a modern @onChange API; verify classic write-back and debounce in the app. Placeholder becomes aria-label.

```hbs
<AuDataTableTextSearch
  @filter={{mut this.filter}}
  @placeholder="Zoeken"
  @wait={{300}}
/>
```

### AuDataTableThSortable

- **Purpose:** Compatibility alias for AuDataTable::ThSortable.
- **Source:** `appuniversum/src/components/au-data-table-th-sortable.js`.
- **API / blocks / events:** @field, @label, @currentSorting; no block/callback. Converts field to dash-case; cycles field, -field, empty and sets currentSorting.
- **Use / accessibility / avoid:** Existing apps may retain this flat alias; prefer the nested export for new direct imports. Use in a table header with classic bindings, not an invented @onSort. Supplies a button with hidden Dutch text but no aria-sort attribute; review announcement.

```hbs
<AuDataTableThSortable
  @field="name"
  @label="Naam"
  @currentSorting={{mut this.sort}}
/>
```

## Layout

### AuApp

- **Purpose:** Provide the application layout root.
- **Source:** `appuniversum/src/components/au-app.gts`.
- **API / blocks / events:** No args/events; default block; div.au-c-app.
- **Use / accessibility / avoid:** Use a shell wrapper, not multiple nested app roots or a main landmark.

```hbs
<AuApp>{{outlet}}</AuApp>
```

### AuMainHeader

- **Purpose:** Render the branded application header.
- **Source:** `appuniversum/src/components/au-main-header.gts`.
- **API / blocks / events:** Required @appTitle; @brandLink, @homeRoute, @contactRoute, @contactLabel, @navigationAriaLabel; default actions block.
- **Use / accessibility / avoid:** Use at shell level. Includes skip link to #content, narrator using router service, and home link focus to #main. No generic @title.

```hbs
<AuMainHeader @appTitle="Loket" @homeRoute="index"><AuLink
    @route="settings"
  >Instellingen</AuLink></AuMainHeader>
```

### AuMainContainer

- **Purpose:** Provide main/sidebar/content application regions.
- **Source:** `appuniversum/src/components/au-main-container.gts`.
- **API / blocks / events:** No root args; yields separate lowercase sidebar/content hashes. sidebar @size="collapsed|small|large"; content @scroll boolean.
- **Use / accessibility / avoid:** Use once for main landmark; fixed #main and #content focus targets must stay unique. Sidebar precedes content on small screens; not an automatic mobile drawer.

```hbs
<AuMainContainer as |m|><m.sidebar><nav
      aria-label="Menu"
    >Menu</nav></m.sidebar><m.content
    @scroll={{true}}
  >{{outlet}}</m.content></AuMainContainer>
```

### AuBodyContainer

- **Purpose:** Wrap an application body region.
- **Source:** `appuniversum/src/components/au-body-container.gts`.
- **API / blocks / events:** @scroll; default block.
- **Use / accessibility / avoid:** Use when body-container layout matches the shell. Not a semantic main; avoid competing nested scroll regions.

```hbs
<AuBodyContainer @scroll={{true}}><AuContent><p
    >Inhoud</p></AuContent></AuBodyContainer>
```

### AuMainFooter

- **Purpose:** Render a branded footer.
- **Source:** `appuniversum/src/components/au-main-footer.gts`.
- **API / blocks / events:** No args/events; default content; includes AuBrand with tagline.
- **Use / accessibility / avoid:** Use for footer content, not another main landmark or unrelated inline branding.

```hbs
<AuMainFooter><p>Een toepassing van de Vlaamse overheid.</p></AuMainFooter>
```

### AuBrand

- **Purpose:** Render the Flanders logo/logotype.
- **Source:** `appuniversum/src/components/au-brand.gts`.
- **API / blocks / events:** @link URL, @tagline; no block/events. Renders a or div.
- **Use / accessibility / avoid:** Use branding, not a replacement for the application title. Link branch has labelledby; SVG is hidden from assistive technology.

```hbs
<AuBrand @link="https://www.vlaanderen.be/" @tagline="verbeelding werkt" />
```

### AuContentHeader

- **Purpose:** Render an image-backed page introduction.
- **Source:** `appuniversum/src/components/au-content-header.gts`.
- **API / blocks / events:** @titlePartOne, @titlePartTwo, @pictureSize="large"; default block is inside picture.
- **Use / accessibility / avoid:** Use for a visual intro, not the page h1: title parts are spans. Provide appropriate image alt and a real heading elsewhere.

```hbs
<AuContentHeader @titlePartOne="Welkom" @titlePartTwo="bij het loket"><img
    src="/images/banner.jpg"
    alt=""
  /></AuContentHeader>
```

### AuContent

- **Purpose:** Apply rich-content typography.
- **Source:** `appuniversum/src/components/au-content.gts`.
- **API / blocks / events:** @skin="tiny|small|large"; default block; div root.
- **Use / accessibility / avoid:** Use for prose, not as an automatic container width or heading outline.

```hbs
<AuContent><h2>Over deze toepassing</h2><p>Uitleg.</p></AuContent>
```

### AuHeading

- **Purpose:** Render a semantic heading with independent visual scale.
- **Source:** `appuniversum/src/components/au-heading.gts`.
- **API / blocks / events:** @level="1|2|3|4|5|6" (default 1); @skin="1|2|3|4|5|6|functional" (default 1); default text.
- **Use / accessibility / avoid:** Choose level for outline and skin for appearance; do not skip semantic levels to get smaller text.

```hbs
<AuHeading @level="2" @skin="4">Instellingen</AuHeading>
```

### AuPanel

- **Purpose:** Group content into panel sections.
- **Source:** `appuniversum/src/components/au-panel.gts`.
- **API / blocks / events:** @active; yields Section directly; default section blocks.
- **Use / accessibility / avoid:** Use a visual panel, not an automatic dialog, accordion, or tabpanel. Add semantics according to content.

```hbs
<AuPanel as |Section|><Section><p>Details</p></Section></AuPanel>
```

### AuToolbar

- **Purpose:** Lay out groups of page actions/content.
- **Source:** `appuniversum/src/components/au-toolbar.gts`.
- **API / blocks / events:** @reverse, @border="top|bottom", @skin="tint", @size="small|medium|large", @nowrap; yields Group directly.
- **Use / accessibility / avoid:** Use for grouped layout, not an automatic role=toolbar with arrow-key behavior. Do not force nowrap on narrow screens without testing.

```hbs
<AuToolbar @border="bottom" as |Group|><Group><AuHeading
      @level="2"
    >Dossiers</AuHeading></Group><Group><AuButton>Nieuw dossier</AuButton></Group></AuToolbar>
```

## Utilities

### AuHr

- **Purpose:** Render a styled thematic break.
- **Source:** `appuniversum/src/components/au-hr.gts`.
- **API / blocks / events:** @size="large|huge"; no block/event; hr root.
- **Use / accessibility / avoid:** Use semantic section breaks, not spacing that should be handled by layout utilities.

```hbs
<AuHr @size="large" />
```

### AuIcon

- **Purpose:** Render a symbol sprite icon or supplied icon component.
- **Source:** `appuniversum/src/components/au-icon.gts`.
- **API / blocks / events:** Required @icon string or component; @alignment="left|right", @size="large", @ariaHidden (default true). No block/events.
- **Use / accessibility / avoid:** Use decoration with a text label; for meaningful standalone imagery provide an accessible name and @ariaHidden=false. Do not guess symbol names; inspect public/icons.

```hbs
<AuIcon @icon="check" @alignment="left" />
```

## Generated icon family (620 SVG assets)

Sources: `appuniversum/lib/generate-icon-components.mjs`,
`appuniversum/public/icons/*.svg`, `tests/integration/components/icon-components-test.gts`,
and `rollup.config.mjs`. The `build:icons` script creates one
`src/components/icons/<svg-basename>.gts` for each SVG and a symbol sprite.
These generated files are absent in the clean source checkout; do not mistake
that for absence from a built package. They are deliberately excluded from app
reexports, so there is **no global `<Icons::…>` resolver contract**.

The generator uses `change-case`'s `pascalCase(basename,
{ mergeAmbiguousCharacters: true })` plus `Icon` for a **named** export:

```gts
import AuIcon from '@appuniversum/ember-appuniversum/components/au-icon';
import { AddIcon } from '@appuniversum/ember-appuniversum/components/icons/add';

<template><AuIcon @icon={{AddIcon}} /><span>Toevoegen</span></template>
```

All generated icons share this contract: an SVG element with forwarded native
attributes, **no arguments, blocks, or events**. Purpose: render the glyph named
by the source SVG. Use through AuIcon for standard size/alignment/aria-hidden
handling; do not use an unlabelled glyph as an action or status. Direct
`<AddIcon aria-hidden="true" />` is possible after import but does not itself
supply AuIcon styling or hidden semantics. For every other named icon export,
**verify in source** of the built installed package; punctuation/numeric names
must not be converted by guesswork. The sprite alternative uses the verified
SVG basename: `<AuIcon @icon="add" />`.

The following is the complete asset/sprite-key inventory, not 620 additional
global Ember component names. Each key is also its generated module basename;
search it before choosing an icon. The shared contract/example above applies to
every row. A glyph's suitability is visual/contextual: inspect the SVG rather
than infer meaning solely from its name.

| SVG / module basename        | SVG / module basename        | SVG / module basename        | SVG / module basename       |
| ---------------------------- | ---------------------------- | ---------------------------- | --------------------------- |
| `add`                        | `add-user`                   | `ai-search`                  | `ai-write`                  |
| `airplane`                   | `alarm`                      | `alarm-silent`               | `alert-circle`              |
| `alert-circle-filled`        | `alert-naked`                | `alert-small`                | `alert-triangle`            |
| `alert-triangle-filled`      | `align-center`               | `align-justify`              | `align-left`                |
| `align-right`                | `apple`                      | `archive`                    | `area`                      |
| `arrange-1-to-9`             | `arrange-9-to-1`             | `arrange-a-to-z`             | `arrange-z-to-a`            |
| `arrow-down`                 | `arrow-freemove`             | `arrow-left`                 | `arrow-right`               |
| `arrow-top-right`            | `arrow-up`                   | `asbest`                     | `asterisk`                  |
| `attachment`                 | `audio-description`          | `baby-milk-bottle`           | `back`                      |
| `ban`                        | `banking-bill-euro`          | `barcode`                    | `battery-charging`          |
| `big-bullet`                 | `bike`                       | `bike-closed-criterium`      | `bike-open-criterium`       |
| `bin`                        | `binoculars`                 | `boat-ship`                  | `bold`                      |
| `book`                       | `bookmark`                   | `bookmark-alt-1`             | `bookmark-alt-2`            |
| `briefcase`                  | `briefcase-pause`            | `building`                   | `building-2`                |
| `building-big`               | `building-refresh`           | `bullet`                     | `burgerprofiel`             |
| `bus`                        | `bus-2`                      | `bus-handicap`               | `bus-refresh`               |
| `business-deal-handshake`    | `business-graph-bar`         | `business-graph-pie`         | `business-scale`            |
| `cabinet`                    | `cake`                       | `calculator`                 | `calendar`                  |
| `calendar-add`               | `calendar-check`             | `calendar-new`               | `calendar-subtract`         |
| `calendar-timeout`           | `camera`                     | `car`                        | `car-refresh`               |
| `car-wheel`                  | `caret-down`                 | `caret-up`                   | `caret-up-down`             |
| `chat`                       | `chat-bubble-square`         | `chat-bubble-square-1`       | `chat-bubble-square-alert`  |
| `chat-bubble-square-talk`    | `chat-help`                  | `check`                      | `check-bubble`              |
| `check-circle`               | `check-filled`               | `check-naked`                | `check-shield`              |
| `check-small`                | `check-thin`                 | `checkbox-checked`           | `checkbox-indeterminate`    |
| `checkbox-unchecked`         | `chevron-down`               | `chevron-left`               | `chevron-right`             |
| `chevron-up`                 | `child`                      | `circle`                     | `circle-check`              |
| `circle-full`                | `circle-info`                | `circle-pause`               | `circle-question`           |
| `circle-step-1`              | `circle-step-2`              | `circle-step-3`              | `circle-step-4`             |
| `circle-step-5`              | `circle-step-6`              | `circle-three-dots`          | `circle-x`                  |
| `clock`                      | `clock-rewind`               | `close`                      | `close-circle`              |
| `close-filled`               | `close-light`                | `close-naked`                | `close-small`               |
| `cloud`                      | `cloud-download`             | `cloud-upload`               | `code-branch`               |
| `coffee-cup`                 | `cog`                        | `coin-receive`               | `coin-stack`                |
| `color-picker`               | `comment`                    | `compass`                    | `computer-screen`           |
| `confluence`                 | `construction-crane`         | `construction-shack`         | `contactbook`               |
| `contacts`                   | `content-book`               | `content-book-favorite-star` | `content-box`               |
| `content-filter`             | `content-layers-hide`        | `content-layers-show`        | `content-note`              |
| `content-view-column`        | `content-view-module`        | `contract`                   | `contract-education`        |
| `control-cross-over`         | `copy`                       | `copy-paste`                 | `copyright`                 |
| `credit-card`                | `crop`                       | `cross`                      | `cursor-arrow-big`          |
| `cursor-arrow-small`         | `cursor-finger-down`         | `cursor-finger-left`         | `cursor-finger-right`       |
| `cursor-finger-up`           | `cursor-hand`                | `cursor-hold`                | `dash`                      |
| `dating-target`              | `decided`                    | `demonstration`              | `digitale-assistent`        |
| `direction-sign`             | `document`                   | `document-added`             | `documents`                 |
| `download`                   | `download-harddisk`          | `download-user`              | `draft`                     |
| `drag`                       | `drag-handle`                | `drag-handle-2`              | `drawer`                    |
| `drawer-down`                | `drink-bottle`               | `dropdown`                   | `eco-solar-panel`           |
| `eco-throw-trash`            | `eloket`                     | `email`                      | `email-read`                |
| `email-user`                 | `envelope`                   | `expand`                     | `expand-horizontal`         |
| `expand-horizontal-4`        | `expand-horizontal-alt`      | `expand-vertical`            | `expand-vertical-2`         |
| `expand-vertical-4`          | `export`                     | `external`                   | `external-link`             |
| `eye`                        | `facebook`                   | `family`                     | `fastback`                  |
| `fastforward`                | `fax`                        | `file`                       | `file-1`                    |
| `file-accept`                | `file-audio`                 | `file-copy`                  | `file-download`             |
| `file-hand`                  | `file-images`                | `file-images-heic`           | `file-images-jpg`           |
| `file-images-png`            | `file-images-svg`            | `file-new`                   | `file-office-doc`           |
| `file-office-pdf`            | `file-office-ppt`            | `file-office-xls`            | `file-reject`               |
| `file-swap`                  | `file-tasks-bacteria-check`  | `file-tasks-check`           | `file-tasks-favorite-heart` |
| `file-video`                 | `file-work`                  | `file-zipped-new`            | `file-zipped-vice`          |
| `files-coding`               | `files-coding-html`          | `film`                       | `filter`                    |
| `fingerprint`                | `flickr`                     | `focus`                      | `folder`                    |
| `folder-open`                | `font`                       | `food-apple`                 | `food-apple-core`           |
| `food-cookie`                | `food-grain`                 | `garbage-bag`                | `garbage-bin`               |
| `gear`                       | `gender-female`              | `gender-female-male`         | `gender-male`               |
| `gender-transgender`         | `github`                     | `globe`                      | `globe2`                    |
| `google-play`                | `googleplus`                 | `graduation-euro`            | `graduation-hat`            |
| `graduation-refresh`         | `graduation-time`            | `hammer`                     | `hammer-anvil`              |
| `hand-hint`                  | `handicap`                   | `handicap-money`             | `handicap-tools`            |
| `harddisk`                   | `headphone`                  | `headphones-1`               | `headphones-2`              |
| `health-first-aid-kit`       | `health-heart-pulse`         | `health-hospital`            | `health-syringe`            |
| `health-syringe-certificate` | `hide`                       | `hierarchal-back`            | `hierarchical-back`         |
| `hierarchy`                  | `hotel-bath-shower`          | `hotel-bed`                  | `hotel-fire-alarm`          |
| `hotel-shower`               | `html`                       | `html-dot`                   | `html-plus`                 |
| `id-card`                    | `image`                      | `images`                     | `images-copy`               |
| `import`                     | `inbox`                      | `indent`                     | `indent-left`               |
| `indent-right`               | `info`                       | `info-circle`                | `info-filled`               |
| `info-naked`                 | `info-small`                 | `instagram`                  | `interface-share-time`      |
| `italic`                     | `jeugdhulp`                  | `jeugdhulp-saving`           | `jira`                      |
| `key`                        | `keyboard`                   | `label`                      | `laptop`                    |
| `laundry`                    | `leisure-drama`              | `leisure-ticket`             | `lightbulb`                 |
| `link`                       | `link-broken`                | `link-external`              | `linkedin`                  |
| `list-add`                   | `list-bullets`               | `list-bullets-1`             | `list-number`               |
| `live-purpose`               | `loader`                     | `location`                   | `location-direction-arrow`  |
| `location-gps`               | `location-map`               | `location-path`              | `location-pin-information`  |
| `location-pin-recycle`       | `location-pin-recycle-close` | `lock`                       | `lock-closed`               |
| `lock-open`                  | `lock-unlock`                | `login`                      | `logout`                    |
| `magic-wand`                 | `mail`                       | `manual`                     | `market`                    |
| `menu`                       | `message`                    | `messenger`                  | `microphone`                |
| `microphone-off`             | `milk-carton`                | `minimize`                   | `minus-circle`              |
| `mobile-phone`               | `money-note`                 | `motorbike`                  | `motorcycle`                |
| `move-down`                  | `move-left-right`            | `move-up`                    | `moving-elevator`           |
| `music-note`                 | `nature-bacteria`            | `nature-clover`              | `nature-leaf`               |
| `nature-tree`                | `nav-down`                   | `nav-down-double`            | `nav-down-light`            |
| `nav-left`                   | `nav-left-double`            | `nav-left-light`             | `nav-right`                 |
| `nav-right-double`           | `nav-right-light`            | `nav-show-more-horizontal`   | `nav-show-more-vertical`    |
| `nav-up`                     | `nav-up-double`              | `nav-up-down`                | `nav-up-light`              |
| `newsletter`                 | `newspaper`                  | `next`                       | `no-sign`                   |
| `non-garbage-bag`            | `non-garbage-bin`            | `not-visible`                | `ordered-list`              |
| `organization-network`       | `other-annoyances`           | `other-annoyances-alt`       | `paint-brush`               |
| `paint-palette`              | `paperclip`                  | `paperplane`                 | `paragraph`                 |
| `pause`                      | `pdf`                        | `pdf-dot`                    | `pdf-plus`                  |
| `pencil`                     | `pencil-ruler`               | `pencil-write`               | `pennants`                  |
| `phone`                      | `phone-incoming`             | `phone-off`                  | `phone-outgoing`            |
| `phone-record`               | `phone-signal-low`           | `phone-speaker`              | `pickup`                    |
| `pin-paper`                  | `pin-paper-filled`           | `pinterest`                  | `places-factory`            |
| `places-home`                | `places-home-2`              | `places-home-3`              | `play`                      |
| `playstreet`                 | `plug`                       | `plus`                       | `plus-circle`               |
| `plus-circle-filled`         | `plus-naked`                 | `plus-text`                  | `power-button`              |
| `printer`                    | `printer-view`               | `programming-bug`            | `programming-jigsaw 1`      |
| `publication`                | `question-circle`            | `question-mark`              | `question-mark-filled`      |
| `question-mark-naked`        | `question-mark-small`        | `real-estate-building-1`     | `real-estate-building-2`    |
| `real-estate-hotel`          | `recreation`                 | `redirect-user`              | `redo`                      |
| `remove`                     | `remove-user`                | `renew`                      | `reply`                     |
| `reply-all`                  | `report`                     | `reverse-indent`             | `rewards-badge`             |
| `rewards-certified-badge`    | `rewards-gift`               | `road`                       | `road-alert`                |
| `road-block`                 | `romance-marriage-license`   | `ruler`                      | `safe`                      |
| `save`                       | `save-care`                  | `save-work`                  | `scaffold`                  |
| `scissors`                   | `search`                     | `search-folder`              | `server`                    |
| `settings`                   | `share`                      | `share-megaphone`            | `share-rss-feed`            |
| `shipping-truck`             | `shopping-basket`            | `shopping-basket-add`        | `shopping-basket-subtract`  |
| `shrink`                     | `sign`                       | `sign-disable`               | `sign-recycle`              |
| `signinghub`                 | `sitemap`                    | `skype`                      | `smiley-poker-face`         |
| `smiley-smile`               | `snapchat`                   | `snippet`                    | `sort`                      |
| `speaker-volume-decrease`    | `speaker-volume-high`        | `speaker-volume-increase`    | `speaker-volume-low`        |
| `speaker-volume-medium`      | `speaker-volume-off`         | `sport-football-field`       | `sports-competition`        |
| `spotify`                    | `stop`                       | `store-repair`               | `strikethrough`             |
| `subscript`                  | `subtract`                   | `subtract-filled`            | `subtract-naked`            |
| `subway`                     | `suitcase`                   | `superscript`                | `switch`                    |
| `switches`                   | `symbol-wifi`                | `synchronize`                | `synchronize-timeout`       |
| `t-shirt`                    | `table`                      | `table-column-end-add`       | `table-column-remove`       |
| `table-column-start-add`     | `table-insert`               | `table-row-end-add`          | `table-row-remove`          |
| `table-row-start-add`        | `tablet`                     | `tag`                        | `tag-add`                   |
| `tag-check`                  | `tag-close`                  | `tag-double`                 | `tag-edit`                  |
| `tag-subtract`               | `tag-view`                   | `taxi`                       | `television`                |
| `terrace`                    | `text-cursor`                | `text-eraser`                | `text-input`                |
| `text-redo`                  | `text-undo`                  | `three-dots`                 | `tiles`                     |
| `timeline`                   | `tint`                       | `tractor`                    | `train`                     |
| `trash`                      | `trophy`                     | `truck`                      | `truck-danger`              |
| `twitter`                    | `uitpas`                     | `underline`                  | `underlined`                |
| `undo`                       | `universal-access`           | `university`                 | `unordered-list`            |
| `upload`                     | `upload-harddisk`            | `usb-head`                   | `user`                      |
| `user-add`                   | `user-box`                   | `user-car`                   | `user-check`                |
| `user-contact`               | `user-female`                | `user-fill`                  | `user-group`                |
| `user-group-add`             | `user-group-circle`          | `user-group-home`            | `user-group-refresh`        |
| `user-male`                  | `user-popup`                 | `user-reception`             | `user-setting`              |
| `users`                      | `users-four-of-four`         | `users-one-of-four`          | `users-single`              |
| `users-three-of-four`        | `users-two-of-four`          | `vacuum-cleaner`             | `video-subtitle`            |
| `view-add`                   | `view-hide`                  | `visible`                    | `vlaanderen`                |
| `vote`                       | `vote-flag`                  | `vote-heart`                 | `vote-heart-filled`         |
| `vote-star`                  | `vote-star-filled`           | `vote-thumbs-down`           | `vote-thumbs-up`            |
| `website`                    | `whatsapp`                   | `whiskey-bottle-glass`       | `wifi-signal-check`         |
| `wifi-signal-close`          | `window`                     | `window-forward`             | `word`                      |
| `word-dot`                   | `word-plus`                  | `work-receive`               | `wrench`                    |
| `x`                          | `youtube`                    | `zoom-in`                    | `zoom-out`                  |
