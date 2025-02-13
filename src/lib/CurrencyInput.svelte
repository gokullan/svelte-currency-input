<script lang="ts">
	const DEFAULT_LOCALE = 'en-US';
	const DEFAULT_CURRENCY = 'USD';
	const DEFAULT_NAME = 'total';
	const DEFAULT_VALUE = 0;
	const DEFAULT_FRACTION_DIGITS = 2;

	export let value: number = DEFAULT_VALUE;
	export let locale: string = DEFAULT_LOCALE;
	export let currency: string = DEFAULT_CURRENCY;
	export let name: string = DEFAULT_NAME;
	export let required: boolean = false;
	export let disabled: boolean = false;
	export let placeholder: number | null = DEFAULT_VALUE;
	export let isNegativeAllowed: boolean = true;
	export let fractionDigits: number = DEFAULT_FRACTION_DIGITS;

	// Formats value as: e.g. $1,523.00 | -$1,523.00
	const formatCurrency = (
		value: number,
		maximumFractionDigits?: number,
		minimumFractionDigits?: number
	) => {
		return new Intl.NumberFormat(locale, {
			currency: currency,
			style: 'currency',
			maximumFractionDigits: maximumFractionDigits || 0,
			minimumFractionDigits: minimumFractionDigits || 0
		}).format(value);
	};

	// Checks if the key pressed is allowed
	const handleKeyDown = (event: KeyboardEvent) => {
		const isDeletion = event.key === 'Backspace' || event.key === 'Delete';
		const isModifier = event.metaKey || event.altKey || event.ctrlKey;
		const isArrowKey = event.key === 'ArrowLeft' || event.key === 'ArrowRight';
		const isInvalidCharacter = !/^\d|,|\.|-$/g.test(event.key); // Keys that are not a digit, comma, period or minus sign

		if (!isDeletion && !isModifier && !isArrowKey && isInvalidCharacter) event.preventDefault();
	};

	let inputTarget: HTMLInputElement;
	const currencyDecimal = new Intl.NumberFormat(locale).format(1.1).charAt(1); // '.' or ','
	const isDecimalComma = currencyDecimal === ',';
	const currencySymbol = formatCurrency(0, 0)
		.replace('0', '') // e.g. '$0' > '$'
		.replace(/\u00A0/, ''); // e.g '0 €' > '€'

	// Updates `value` by stripping away the currency formatting
	const setUnformattedValue = (event: KeyboardEvent) => {
		// Don't format if the user is typing a `currencyDecimal` point
		if (event.key === currencyDecimal) return;

		// Pressing `.` when the decimal point is `,` gets replaced with `,`
		if (isDecimalComma && event.key === '.')
			formattedValue = formattedValue.replace(/\.([^.]*)$/, currencyDecimal + '$1'); // Only replace the last occurence

		// Pressing `,` when the decimal point is `.` gets replaced with `.`
		if (!isDecimalComma && event.key === ',')
			formattedValue = formattedValue.replace(/\,([^,]*)$/, currencyDecimal + '$1'); // Only replace the last occurence

		// Don't format if `formattedValue` is ['$', '-$', "-"]
		const ignoreSymbols = [currencySymbol, `-${currencySymbol}`, '-'];
		const strippedUnformattedValue = formattedValue.replace(' ', '');
		if (ignoreSymbols.includes(strippedUnformattedValue)) return;

		// Set the starting caret positions
		inputTarget = event.target as HTMLInputElement;

		// Remove all characters that arent: numbers, commas, periods (or minus signs if `isNegativeAllowed`)
		let unformattedValue = isNegativeAllowed
			? formattedValue.replace(/[^0-9,.-]/g, '')
			: formattedValue.replace(/[^0-9,.]/g, '');

		// Reverse the value when minus is pressed
		if (isNegativeAllowed && event.key === '-') value = value * -1;

		// Finally set the value
		if (Number.isNaN(parseFloat(unformattedValue))) {
			value = 0;
		} else {
			// The order of the following operations is *critical*
			unformattedValue = unformattedValue.replace(isDecimalComma ? /\./g : /\,/g, ''); // Remove all group symbols
			if (isDecimalComma) unformattedValue = unformattedValue.replace(',', '.'); // If the decimal point is a comma, replace it with a period
			value = parseFloat(unformattedValue);
		}
	};

	const setFormattedValue = () => {
		// Previous caret position
		const startCaretPosition = inputTarget?.selectionStart || 0;
		const previousFormattedValueLength = formattedValue.length;

		// Apply formatting to input
		formattedValue = isZero ? '' : formatCurrency(value, fractionDigits, 0);

		// New caret position
		const endCaretPosition =
			startCaretPosition + formattedValue.length - previousFormattedValueLength;

		// HACK:
		// Delay setting the new caret position until the input has been formatted
		setTimeout(() => {
			inputTarget?.setSelectionRange(endCaretPosition, endCaretPosition);
		}, 0.1);
	};

	let formattedValue = '';
	let formattedPlaceholder =
		placeholder !== null ? formatCurrency(placeholder, fractionDigits, fractionDigits) : '';
	$: isZero = value === 0;
	$: isNegative = value < 0;
	$: value, setFormattedValue();
</script>

<div class="currencyInput">
	<input class="currencyInput__unformatted" type="hidden" {name} {disabled} bind:value />
	<input
		class="
			currencyInput__formatted
			{isNegativeAllowed && !isZero && !isNegative && 'currencyInput__formatted--positive'}
			{isZero && 'currencyInput__formatted--zero'}
			{isNegativeAllowed && isNegative && 'currencyInput__formatted--negative'}
		"
		type="text"
		inputmode="numeric"
		name={`formatted-${name}`}
		required={required && !isZero}
		placeholder={formattedPlaceholder}
		{disabled}
		bind:value={formattedValue}
		on:keydown={handleKeyDown}
		on:keyup={setUnformattedValue}
	/>
</div>

<style>
	input.currencyInput__formatted {
		border: 1px solid #e2e2e2;
		padding: 10px;
		box-sizing: border-box;
	}

	input.currencyInput__formatted--zero {
		color: #333;
	}

	input.currencyInput__formatted--positive {
		color: #00a36f;
	}

	input.currencyInput__formatted--negative {
		color: #e75258;
	}

	input.currencyInput__formatted:disabled {
		color: #999;
		background-color: #e2e2e2;
		pointer-events: none;
		cursor: default;
	}
</style>
