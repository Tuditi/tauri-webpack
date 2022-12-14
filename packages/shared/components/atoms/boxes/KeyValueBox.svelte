<script lang="ts">
    import { Text, Tooltip } from '../..'
    import CopyableBox from './CopyableBox.svelte'

    export let keyText = ''
    export let valueText = ''
    export let textColor = 'gray-600'
    export let darkTextColor = 'gray-500'
    export let backgroundColor = 'gray-50'
    export let darkBackgroundColor = 'gray-850'
    export let padding = 'px-4 py-3.5'
    export let classes = ''
    export let copyValue = ''
    export let isCopyable = false

    let showTooltip = false
    let tooltipAnchor: HTMLElement
    let textContainerWidth: number

    $: isValueTextTruncated = (tooltipAnchor?.firstChild as HTMLElement)?.scrollWidth > textContainerWidth

    function toggleTooltip(show: boolean): void {
        if (isValueTextTruncated && !isCopyable) {
            showTooltip = show
        }
    }
</script>

<div on:mouseleave={() => toggleTooltip(false)} class="w-full">
    <CopyableBox
        value={copyValue ? copyValue : valueText}
        {isCopyable}
        offset={5}
        row
        {backgroundColor}
        {darkBackgroundColor}
        classes="w-full justify-between {padding} {classes}"
    >
        {#if keyText}
            <Text fontSize="14" lineHeight="5" color={textColor} darkColor={darkTextColor} classes="mr-4">
                {keyText}
            </Text>
        {:else}
            <slot name="key" />
        {/if}
        {#if valueText}
            <div
                on:focus={() => toggleTooltip(true)}
                on:mouseover={() => toggleTooltip(true)}
                class="truncate"
                bind:this={tooltipAnchor}
                bind:clientWidth={textContainerWidth}
            >
                <Text fontSize="14" lineHeight="5" color={textColor} darkColor={darkTextColor} classes="truncate">
                    {valueText}
                </Text>
            </div>
            {#if showTooltip}
                <Tooltip anchor={tooltipAnchor} position="top">
                    <div class="max-h-40 overflow-y-scroll pr-0">
                        <Text color={textColor} darkColor={darkTextColor} classes="text-left break-words"
                            >{valueText}</Text
                        >
                    </div>
                </Tooltip>
            {/if}
        {:else}
            <slot name="value" />
        {/if}
    </CopyableBox>
</div>
