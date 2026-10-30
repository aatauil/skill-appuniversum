# Forms

## Contents

Contracts · Labels/validation · Simple form · Complex form · Focus and edge cases.

## Verified contracts

Sources: `appuniversum/src/components/au-{input,textarea,label,help-text,form-row,fieldset,checkbox,checkbox-group,radio,radio-group,toggle-switch,date-input,date-picker,time-picker,file-upload}.gts`,
`appuniversum/src/modifiers/au-date-input.ts`, and corresponding integration tests.
Snapshot: 4.5.1 / 4052636. See [components.md](components.md) for argument summaries.

| Control                     | Value and change contract                                                | Important distinction                                                  |
| --------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| AuInput                     | Native `value=`, `{{on "input" handler}}`                                | Handler receives Event; read `event.target.value`; no @value/@onChange |
| AuTextarea                  | Default text block; native input event                                   | No @value/@onChange; test programmatic reset behavior                  |
| AuCheckbox / AuToggleSwitch | @checked, @onChange(boolean, event)                                      | Not the same callback shape as text inputs                             |
| AuCheckboxGroup             | @selected string[], @onChange(array, event)                              | Use yielded `g.Checkbox @value="…"`                                    |
| AuRadioGroup                | @selected value, @onChange(value, event)                                 | Use yielded `g.Radio`; generated shared name if none supplied          |
| AuDateInput / AuDatePicker  | @value ISO date string or Date; @onChange(isoDate or null, Date or null) | Do not store the displayed DD-MM-JJJJ string as the API value          |
| AuTimePicker                | @hours/@minutes/@seconds, @onChange(timeObject)                          | Supply labels; hard-coded IDs complicate multiple instances            |
| AuFileUpload                | @onUpload({fileId,file}) and queue callbacks                             | Uploads immediately to @endPoint; not a generic local file input       |

There is no public `AuForm`, `AuFormGroup`, `AuError`, `AuSelect`, or validation
service in this snapshot. A native `<form>` is correct. Existing app select or
PowerSelect integration may be appropriate; the addon styles PowerSelect but
lists it as a development dependency, not a guaranteed consuming-app dependency.

## Labels, help text, and errors

- Link `AuLabel for="x"` to `AuInput id="x"`. Use unique IDs per form instance.
- `AuLabel @required={{true}}` and yielded fieldset legend markers are visual;
  also set native `required` or implement equivalent application validation.
- Pair `@error` styling with `aria-invalid` and linked error text. `@warning`
  is a visual warning, not automatically an invalid value.
- Put hint/error IDs in `aria-describedby`. Add an ID only while its element
  exists; do not leave dangling references. Do not rely on red borders alone.
- Controls with built-in wrapping labels (checkbox, radio, toggle) need visible
  block text. `AuToggleSwitch @label` is declared but not rendered as text.
- Label checkbox/radio groups via `aria-labelledby` or fieldset composition.
  `AuFieldset` yields separate lowercase `legend`/`content` hashes. Its legend
  currently sits in a div wrapper: inspect the accessibility tree, and use
  native fieldset/legend composition when needed for reliable semantics.
- Use `@disabled` for the components that implement it. `readonly` is native
  on text inputs/textarea; it is not a universal `@readOnly` argument and does
  not apply to radio/checkbox controls. Disabled inputs are not submitted.

## Complete simple form

Example app component; `@onSave` is **an app-defined wrapper callback**, not an
Appuniversum API. This `.gjs` version demonstrates explicit imports. Conventional
`.hbs` apps can use the same angle-bracket names through the resolver.

```gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { on } from '@ember/modifier';
import AuButton from '@appuniversum/ember-appuniversum/components/au-button';
import AuFormRow from '@appuniversum/ember-appuniversum/components/au-form-row';
import AuInput from '@appuniversum/ember-appuniversum/components/au-input';
import AuLabel from '@appuniversum/ember-appuniversum/components/au-label';
import AuHelpText from '@appuniversum/ember-appuniversum/components/au-help-text';

export default class NameForm extends Component {
  @tracked name = '';
  @tracked submitted = false;

  get invalid() { return this.submitted && !this.name.trim(); }
  get descriptionId() { return this.invalid ? 'name-error' : 'name-help'; }
  updateName = (event) => { this.name = event.target.value; };
  submit = (event) => {
    event.preventDefault();
    this.submitted = true;
    if (this.invalid) {
      event.currentTarget.querySelector('#name').focus();
      return;
    }
    this.args.onSave({ name: this.name.trim() });
  };

  <template>
    <form novalidate {{on "submit" this.submit}}>
      <AuFormRow>
        <AuLabel for="name" @required={{true}} @error={{this.invalid}}>Naam</AuLabel>
        <AuInput id="name" name="name" required value={{this.name}}
          @width="block" @error={{this.invalid}}
          aria-invalid={{if this.invalid "true" "false"}}
          aria-describedby={{this.descriptionId}}
          {{on "input" this.updateName}} />
        {{#if this.invalid}}
          <AuHelpText id="name-error" @error={{true}}>Vul uw naam in.</AuHelpText>
        {{else}}
          <AuHelpText id="name-help">Gebruik uw volledige naam.</AuHelpText>
        {{/if}}
      </AuFormRow>
      <AuButton type="submit">Bewaren</AuButton>
    </form>
  </template>
}
```

The example uses `novalidate` to demonstrate app-controlled errors and focus.
Without it, the browser blocks invalid submission before the submit handler.
Choose deliberately; do not disable native validation without replacing it.
Adapt IDs when rendering more than one instance. Add saving/failure handling
around asynchronous persistence in the consuming app.

## Complete complex form

A preferences form combines named groups, a calendar, notes, and submission
errors. `@onSave` remains an app callback. The example intentionally uses native
fieldset/legend for direct semantic grouping rather than assuming that styling
alone guarantees the fieldset accessibility tree.

```gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { on } from '@ember/modifier';
import AuAlert from '@appuniversum/ember-appuniversum/components/au-alert';
import AuButton from '@appuniversum/ember-appuniversum/components/au-button';
import AuCheckboxGroup from '@appuniversum/ember-appuniversum/components/au-checkbox-group';
import AuDatePicker from '@appuniversum/ember-appuniversum/components/au-date-picker';
import AuFormRow from '@appuniversum/ember-appuniversum/components/au-form-row';
import AuHelpText from '@appuniversum/ember-appuniversum/components/au-help-text';
import AuLabel from '@appuniversum/ember-appuniversum/components/au-label';
import AuRadioGroup from '@appuniversum/ember-appuniversum/components/au-radio-group';
import AuTextarea from '@appuniversum/ember-appuniversum/components/au-textarea';

export default class PreferencesForm extends Component {
  @tracked channels = [];
  @tracked delivery = 'email';
  @tracked due = undefined;
  @tracked notes = '';
  @tracked error = '';
  @tracked saving = false;

  setChannels = (values) => { this.channels = values; };
  setDelivery = (value) => { this.delivery = value; };
  setDue = (isoDate) => { this.due = isoDate ?? undefined; };
  setNotes = (event) => { this.notes = event.target.value; };
  submit = async (event) => {
    event.preventDefault();
    if (this.saving) return;
    const form = event.currentTarget;
    this.error = '';
    this.saving = true;
    try {
      await this.args.onSave({
        channels: this.channels, delivery: this.delivery,
        due: this.due, notes: this.notes,
      });
    } catch {
      this.error = 'Bewaren mislukt. Uw invoer is behouden; probeer opnieuw.';
      // Existing form is already in the DOM; no render-timing assumption needed.
      form.focus();
    } finally {
      this.saving = false;
    }
  };

  <template>
    <form aria-label="Communicatievoorkeuren" tabindex="-1" {{on "submit" this.submit}}>
      {{#if this.error}}<AuAlert @skin="error">{{this.error}}</AuAlert>{{/if}}
      <fieldset class="au-c-fieldset" disabled={{this.saving}}>
        <legend id="channels-label">Meldingen ontvangen via</legend>
        <AuCheckboxGroup @name="channels" @selected={{this.channels}}
          @disabled={{this.saving}} @onChange={{this.setChannels}}
          aria-labelledby="channels-label" as |g|>
          <g.Checkbox @value="email">E-mail</g.Checkbox>
          <g.Checkbox @value="sms">Sms</g.Checkbox>
        </AuCheckboxGroup>
      </fieldset>
      <fieldset class="au-c-fieldset" disabled={{this.saving}}>
        <legend id="delivery-label">Documenten ontvangen</legend>
        <AuRadioGroup @name="delivery" @selected={{this.delivery}}
          @disabled={{this.saving}} @onChange={{this.setDelivery}}
          aria-labelledby="delivery-label" as |g|>
          <g.Radio @value="email">Digitaal</g.Radio>
          <g.Radio @value="post">Per post</g.Radio>
        </AuRadioGroup>
      </fieldset>
      <AuFormRow>
        <AuDatePicker @id="preferences-due" @label="Einddatum (optioneel)"
          @value={{this.due}} @disabled={{this.saving}} @onChange={{this.setDue}} />
      </AuFormRow>
      <AuFormRow>
        <AuLabel for="preferences-notes">Opmerkingen</AuLabel>
        <AuTextarea id="preferences-notes" name="notes" @width="block"
          @disabled={{this.saving}} aria-describedby="notes-help"
          {{on "input" this.setNotes}}>{{this.notes}}</AuTextarea>
        <AuHelpText id="notes-help">Vermeld geen gevoelige gegevens.</AuHelpText>
      </AuFormRow>
      <AuButton type="submit" @loading={{this.saving}} @loadingMessage="Bewaren…">Bewaren</AuButton>
    </form>
  </template>
}
```

## Focus, error messaging, and specialized controls

- On failed client validation, focus the first invalid control or an error
  summary with working links, not every error message. When a summary is
  conditionally inserted, wait using the app's render-aware mechanism before
  focusing it. Keep typed data after server errors; let the user retry.
- Use live feedback intentionally: AuAlert already has `role="alert"`. Do not
  stack redundant assertive live regions or steal focus on every keystroke.
- Check date-picker internal input associations in the rendered component;
  forwarded attributes are on the Duet host. Do not promise `aria-describedby`
  on that host reaches the internal input without verification.
- AuDateInput only reports complete/cleared mask values. Validate incomplete
  dates on submit, rather than interpreting the last reported date as proof
  that the current input is complete.
- TimePicker normalizes hours to 0–23 and minutes/seconds to 0–59. It has no
  general disabled/read-only API and uses fixed IDs; review multiple instances.
- FileUpload checks type/extension/size and displays errors but still requires
  server validation/authentication. It expects JSON `{data:{id}}`, not an
  arbitrary response. Verify keyboard file selection; drag-and-drop alone is
  insufficient. Do not invent disabled/read-only/upload cancellation APIs.
