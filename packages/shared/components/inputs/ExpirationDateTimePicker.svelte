<script lang="ts">
    import { createEventDispatcher } from 'svelte'
    import { DateTimePicker } from '..'
    import { localize } from '@core/i18n'
    import { isValidExpirationDateTime } from '@core/utils'
    import { showAppNotification } from '@lib/notifications'

    export let value: Date

    const dispatch = createEventDispatcher()

    function handleCancelClick(): void {
        dispatch('cancel')
    }

    function handleConfirmClick(): void {
        if (isValidExpirationDateTime(value)) {
            dispatch('confirm')
        } else {
            showAppNotification({
                type: 'warning',
                message: localize('warning.transaction.invalidExpirationDateTime'),
            })
        }
    }
</script>

<DateTimePicker {...$$restProps} bind:value on:cancel={handleCancelClick} on:confirm={handleConfirmClick} />
