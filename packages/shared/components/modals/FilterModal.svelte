<script lang="ts">
    import { FontWeightNumber } from '../Text.svelte'
    import { Modal, Text } from '..'
    import { Filter } from '@core/wallet/interfaces/filter.interface'
    import { localize } from '@core/i18n'

    export let modal: Modal
    export let filter: Filter
    export let onSetFilters: () => void
    export let onClose: () => void

    function clear(): void {
        for (const key in filter) {
            filter[key].active = false
            filter[key].value = undefined
        }
        onSetFilters()
        modal.toggle()
    }

    function confirm(): void {
        onSetFilters()
        modal.toggle()
    }
</script>

<Modal bind:this={modal} on:close={onClose} position={{ absolute: true, right: '0', top: '30px' }}>
    <div class="filter-modal">
        <div class="flex flex-row items-center justify-between bg-gray-50 dark:bg-transparent px-4 py-3">
            <button
                class="action py-1 px-3 border border-solid border-gray-300 text-center rounded-4 w-fit font-normal text-13 bg-white hover:bg-blue-200"
                on:click|stopPropagation={clear}
            >
                {localize('actions.clear')}
            </button>
            <Text fontWeight={FontWeightNumber._600} fontSize="13" classes="text-center flex grow-1"
                >{localize('filters.title')}</Text
            >
            <button
                class="action py-1 px-3 text-center rounded-4 w-fit font-normal text-13 text-white bg-blue-500 hover:bg-blue-600 dark:hover:bg-blue-400"
                on:click|stopPropagation={confirm}
            >
                {localize('actions.done')}
            </button>
        </div>
        <div class="pb-1">
            <slot />
        </div>
    </div>
</Modal>

<style type="text/scss">
    .filter-modal {
        width: 254px;
    }
</style>
