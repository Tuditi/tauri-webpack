<script lang="ts">
    import { exportStronghold } from '@contexts/settings'
    import { localize } from '@core/i18n'
    import { showAppNotification } from '@lib/notifications'
    import { openPopup } from '@lib/popup'
    import { Button } from '../..'

    export let isBusy = false
    export let message = ''
    export let showNotification = false
    export let overrideTitle = ''

    function handleExportStrongholdResponse(cancelled, error): void {
        setTimeout(
            () => {
                message = ''
            },
            cancelled ? 0 : 5000
        )
        isBusy = false
        if (!cancelled) {
            if (error) {
                message = localize('general.exportingStrongholdFailed')
                showNotification &&
                    showAppNotification({
                        type: 'error',
                        message: localize(error),
                    })
            } else {
                message = localize('general.exportingStrongholdSuccess')
                showNotification &&
                    showAppNotification({
                        type: 'info',
                        message: localize('general.exportingStrongholdSuccess'),
                    })
            }
        }
    }

    function handleExportClick(): void {
        isBusy = false
        message = ''

        openPopup({
            type: 'password',
            props: {
                onSuccess: (password: string) => {
                    isBusy = true
                    message = localize('general.exportingStronghold')
                    exportStronghold(password, handleExportStrongholdResponse)
                },
                returnPassword: true,
                subtitle: localize('popups.password.backup'),
            },
        })
    }
</script>

<Button medium inlineStyle="min-width: 156px;" onClick={handleExportClick} disabled={isBusy} {...$$restProps}>
    {overrideTitle || localize('actions.export')}
</Button>
