# svelte-currency-input

A form input that converts numbers to localized currency formats as you type

[<img width="962" alt="image" src="https://user-images.githubusercontent.com/1434675/190873948-c0385747-6fa9-4077-8bd5-717e4d1124a0.png">](https://svelte.dev/repl/d8f7d22e5b384555b430f62b157ac503?version=3.50.1)

<p align="center">
  👩‍💻 Play with it on <a href="https://svelte.dev/repl/d8f7d22e5b384555b430f62b157ac503?version=3.50.1" target="_blank">REPL</a>  &nbsp;—&nbsp; 💵 See it in a <a href="https://github.com/Canutin/desktop/blob/master/sveltekit/src/lib/components/FormCurrency.svelte" target="_blank">real project</a>!
</p>

---

## Features

- Formats **positive** and **negative** values
- Leverages [`Intl.NumberFormat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat) for **localizing** currency denominations and masking the input
- Simple [API](#api)
- Minimal [default styling](https://github.com/canutin/svelte-currency-input/blob/main/src/lib/CurrencyInput.svelte#L88-L118), easy to [customize](#styling)

## Usage

```bash
npm install svelte-currency-input --save
```

```html
<script lang="ts">
  import CurrencyInput from '@canutin/svelte-currency-input';
</script>

<CurrencyInput name="total" value={-420.69} locale="nl-NL" currency="EUR" />
```

## How it works

When the form is submitted you get _unformatted_ or _formatted_ values from two `<input />`'s.
This is more or less what `<CurrencyInput />` looks like under the hood:

```html
<div class="currencyInput">
  <!-- Unformatted value -->
  <input
    class="currencyInput__unformatted"
    type="hidden"
    name="total"
    value="-420.69"
  />

  <!-- Formatted value -->
  <input
    class="currencyInput__formatted"
    type="text"
    name="formatted-total"
    value="€ -420,69"
  />
</div>
```

## API

| Option            | Type            | Default     | Description |
| ----------------- | --------------- | ----------- | ----------- |
| value             | `number`        | `undefined` | Initial value. If left `undefined` a formatted value of `0` is visible as a placeholder |
| locale            | `string`        | `en-US`     | Overrides default locale. [Examples](https://gist.github.com/ncreated/9934896) |
| currency          | `string`        | `USD`       | Overrides default currency. [Examples](https://www.xe.com/symbols/) |
| name              | `string`        | `total`     | Applies the name to the [input fields](#how-it-works) for _unformatted_ (e.g `[name=total]`) and _formatted_ (e.g. `[name=formatted-total]`) values |
| required          | `boolean`       | `false`     | Marks the inputs as required |
| disabled          | `boolean`       | `false`     | Marks the inputs as disabled |
| placeholder       | `number` `null` | `0`         | Overrides the default placeholder. Setting the value to a `number` will display it as formatted. Setting it to `null` will not show a placeholder   |
| isNegativeAllowed | `boolean`       | `true`      | If `false`, forces formatting only to positive values and ignores `--positive` and `--negative` styling modifiers                                   |
| fractionDigits    | `number`        | `2`         | Sets `maximumFractionDigits` in [`Intl.NumberFormat()` constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat/NumberFormat#minimumfractiondigits) used for formatting the currency. Supported digits: `0` to `20` |

## Styling

The [default styles](https://github.com/canutin/svelte-currency-input/blob/main/src/lib/CurrencyInput.svelte#L88-L118) use [BEM naming conventions](https://getbem.com/naming/). To override the default styles apply your styles as shown below:

```html
<div class="my-currency-input">
  <CurrencyInput name="total" value="{420.69}" />
</div>

<style>
  /* Container */
  div.my-currency-input :global(div.currencyInput) { /* ... */ }

  /* Formatted input */
  div.my-currency-input :global(input.currencyInput__formatted) { /* ... */ }

  /* Formatted input when the it's disabled */
  div.my-currency-input :global(input.currencyInput__formatted:disabled) { /* ... */ }

  /* Formatted input when the value is zero */
  div.my-currency-input :global(input.currencyInput__formatted--zero) { /* ... */ }

  /* Formatted input when the value is positive */
  div.my-currency-input :global(input.currencyInput__formatted--positive) { /* ... */ }

  /* Formatted input when the value is negative */
  div.my-currency-input :global(input.currencyInput__formatted--negative) { /* ... */ }
</style>
```

## Contributing

Here's ways in which you can contribute:

- Found a bug? Open a [new issue](https://github.com/canutin/svelte-currency-input/issues/new)
- Comment or upvote [existing issues](https://github.com/canutin/svelte-currency-input/issues)
- Submit a [pull request](https://github.com/canutin/svelte-currency-input/pulls)

## Developing

This package was generated with [SvelteKit](https://kit.svelte.dev/). Install dependencies with `npm install`, then start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```
