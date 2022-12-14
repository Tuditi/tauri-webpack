<script lang="ts">
    import { Text } from '..'
    import { localize } from '@core/i18n'
    import { getOfficialExplorerUrl } from '@core/network/utils'
    import { Platform } from '../../lib/platform'
    import { FontWeightText } from '../Text.svelte'
    import { TransactionDetails } from '../molecules'
    import {
        Activity,
        ActivityAsyncStatus,
        ActivityDirection,
        claimActivity,
        getAssetFromPersistedAssets,
        hideActivity,
    } from '@core/wallet'
    import { Spinner } from '..'
    import { activeProfile } from '@core/profile'
    import { currencies, exchangeRates } from '@lib/currency'
    import { CurrencyTypes } from '../../lib/typings/currency'
    import { setClipboard } from '@lib/utils'
    import { truncateString } from '@lib/helpers'
    import { closePopup, openPopup } from '@lib/popup'

    export let activity: Activity

    const asset = getAssetFromPersistedAssets(activity?.assetId)
    const explorerUrl = getOfficialExplorerUrl($activeProfile?.networkProtocol, $activeProfile?.networkType)

    let isClaiming = activity.isClaiming

    $: formattedFiatValue = activity.getFiatAmount(
        $currencies[CurrencyTypes.USD],
        $exchangeRates[$activeProfile?.settings?.currency]
    )

    function handleExplorerClick(): void {
        Platform.openUrl(`${explorerUrl}/block/${activity.transactionId}`)
    }

    function handleTransactionIdClick(): void {
        setClipboard(activity.transactionId)
    }

    async function claim() {
        isClaiming = true
        await claimActivity(activity)
        isClaiming = false
        openPopup({
            type: 'activityDetails',
            props: { activity },
        })
    }

    function reject() {
        openPopup({
            type: 'confirmation',
            props: {
                title: localize('actions.confirmRejection.title'),
                description: localize('actions.confirmRejection.description'),
                hint: localize('actions.confirmRejection.node'),
                warning: true,
                confirmText: localize('actions.reject'),
                onConfirm: () => {
                    hideActivity(activity.id)
                    closePopup()
                },
                onCancel: () =>
                    openPopup({
                        type: 'activityDetails',
                        props: { activity },
                    }),
            },
        })
    }
</script>

<activity-details-popup class="w-full h-full space-y-6 flex flex-auto flex-col flex-shrink-0">
    <div class="flex flex-col">
        <Text type="h3" fontWeight={FontWeightText.semibold} classes="text-left">
            {localize('popups.transactionDetails.title')}
        </Text>
        {#if explorerUrl && activity.transactionId}
            <button
                class="action w-fit flex justify-start text-center font-medium text-14 text-blue-500"
                on:click={handleExplorerClick}
            >
                {localize('general.viewOnExplorer')}
            </button>
        {:else if activity.transactionId}
            <button
                class="action w-fit flex justify-start text-center font-medium text-14 text-blue-500"
                on:click={handleTransactionIdClick}
            >
                {truncateString(activity.transactionId, 12, 12)}
            </button>
        {/if}
    </div>
    <TransactionDetails {formattedFiatValue} {...activity} {asset} />
    {#if activity.isAsync && (activity?.direction === ActivityDirection.In || activity.isSelfTransaction) && activity.asyncStatus === ActivityAsyncStatus.Unclaimed}
        <div class="flex w-full justify-between space-x-4">
            <button
                disabled={isClaiming}
                class="action p-4 w-full text-center font-medium text-15 text-blue-500 rounded-lg border border-solid border-gray-300"
                on:click={reject}
            >
                {localize('actions.reject')}
            </button>
            <button
                disabled={isClaiming}
                class="action p-4 w-full text-center rounded-lg font-medium text-15 bg-blue-500 text-white"
                on:click={claim}
            >
                {#if isClaiming}
                    <Spinner busy={true} classes="justify-center" />
                {:else}
                    {localize('actions.claim')}
                {/if}
            </button>
        </div>
    {/if}
</activity-details-popup>
