<script lang="ts">
    import { FontWeight, FontWeightText, TextType } from '../Text.svelte'
    import Input from './Input.svelte'

    export let inputElement: HTMLInputElement

    export let value: string
    export let hasFocus: boolean
    export let error = ''

    // Text Props
    export let type = TextType.p
    export let fontSize = '11'
    export let lineHeight = '140'
    export let fontWeight: FontWeight = FontWeightText.normal
    export let alignment: 'left' | 'right' | 'center' | 'justify' = 'left'
</script>

<Input
    bind:inputElement
    bind:value
    bind:hasFocus
    type="text"
    {error}
    textProps={{
        type,
        fontSize,
        lineHeight,
        fontWeight,
    }}
    {alignment}
    {...$$restProps}
/>
