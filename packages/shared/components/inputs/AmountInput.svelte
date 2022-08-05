<script lang="ts">
    import { NumberInput } from '..'
    import { localize } from '@core/i18n'
    import { FontWeightText } from '../Text.svelte'

    export let inputElement: HTMLInputElement
    export let disabled = false
    export let hasFocus = false
    export let amount: string
</script>

<NumberInput
    bind:inputElement
    bind:value={amount}
    bind:hasFocus
    {disabled}
    placeholder={localize('general.amount')}
    fontSize="24"
    alignment="right"
    fontWeigth={FontWeightText.semibold}
    {...$$restProps}
/>
