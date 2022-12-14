<script lang="ts">
    import { time } from '@core/app'
    import { localize } from '@core/i18n'
    import {
        Activity,
        ActivityAsyncStatus,
        ActivityDirection,
        claimActivity,
        ActivityType,
        hideActivity,
        InclusionState,
        VerificationStatus,
        selectedAccountAssets,
        getAssetFromPersistedAssets,
        IPersistedAsset,
    } from '@core/wallet'
    import { truncateString } from '@lib/helpers'
    import { closePopup, openPopup } from '@lib/popup'
    import { ActivityAsyncStatusPill, ClickableTile, HR, Icon, Text, Spinner, AssetIcon } from '../..'
    import { FontWeightText } from '../../Text.svelte'

    export let activity: Activity

    let asset: IPersistedAsset

    $: activity?.assetId, $selectedAccountAssets, (asset = getAssetFromPersistedAssets(activity?.assetId))
    $: title = activity?.getTitle()
    $: subject = activity?.getFormattedSubject()
    $: isIncomingActivityUnclaimed =
        (activity?.direction === ActivityDirection.In || activity.isSelfTransaction) &&
        activity.asyncStatus === ActivityAsyncStatus.Unclaimed
    $: timeDiff = activity?.getTimeDiffUntilExpirationTime($time)

    function handleTransactionClick(): void {
        if (asset?.verification === VerificationStatus.New) {
            openPopup({
                type: 'tokenInformation',
                overflow: true,
                props: {
                    activity,
                    asset,
                },
            })
        } else {
            openPopup({
                type: 'activityDetails',
                props: { activity },
            })
        }
    }

    function handleRejectClick() {
        openPopup({
            type: 'confirmation',
            props: {
                title: localize('actions.confirmRejection.title'),
                description: localize('actions.confirmRejection.description'),
                hint: localize('actions.confirmRejection.node'),
                warning: true,
                confirmText: localize('actions.reject'),
                onConfirm: () => {
                    hideActivity(activity?.id)
                    closePopup()
                },
            },
        })
    }

    function handleClaimClick() {
        claimActivity(activity)
    }
</script>

<ClickableTile
    onClick={handleTransactionClick}
    classes={activity?.inclusionState !== InclusionState.Confirmed ? 'opacity-50' : ''}
>
    <div class="w-full flex flex-col space-y-4">
        <div class="flex flex-row items-center text-left space-x-4">
            <AssetIcon {asset} showVerifiedBadgeOnly />
            <div class="flex flex-col w-full space-y-0.5">
                <div class="flex flex-row justify-between space-x-1">
                    <Text
                        fontWeight={FontWeightText.semibold}
                        lineHeight="140"
                        classes="overflow-hidden overflow-ellipsis multiwrap-line2"
                    >
                        {localize(title)}
                    </Text>
                    <Text
                        fontWeight={FontWeightText.semibold}
                        lineHeight="140"
                        color={activity?.direction === ActivityDirection.In ? 'blue-700' : ''}
                        classes="whitespace-nowrap"
                    >
                        {activity?.getFormattedAmount(true)}
                    </Text>
                </div>

                <div class="flex flex-row justify-between">
                    <Text fontWeight={FontWeightText.normal} lineHeight="140" color="gray-600">
                        {#if activity?.type === ActivityType.Minting}
                            {asset?.metadata?.name
                                ? truncateString(asset?.metadata?.name, 20, 0)
                                : truncateString(asset?.id, 6, 7)}
                        {:else}
                            {localize(
                                activity?.direction === ActivityDirection.In
                                    ? 'general.fromAddress'
                                    : 'general.toAddress',
                                { values: { account: subject } }
                            )}
                        {/if}
                    </Text>
                    <Text
                        fontWeight={FontWeightText.normal}
                        lineHeight="140"
                        color="gray-600"
                        classes="whitespace-nowrap"
                    >
                        {activity?.getFiatAmount()}
                    </Text>
                </div>
            </div>
        </div>
        {#if activity?.isAsync && (activity.direction === ActivityDirection.Out || !activity?.isClaimed)}
            <HR />
            <div class="flex w-full justify-between space-x-4">
                <div class="flex flex-row justify-center items-center space-x-2">
                    {#if !activity?.isClaimed}
                        <Icon width="16" height="16" icon="timer" classes="text-gray-600" />
                        <Text fontSize="13" color="gray-600" fontWeight={FontWeightText.semibold}
                            >{timeDiff ?? localize('general.none')}</Text
                        >
                    {/if}
                </div>
                <div class="flex flex-row justify-end w-1/2 space-x-2">
                    {#if isIncomingActivityUnclaimed}
                        <button
                            disabled={activity.isClaiming}
                            class="action px-3 py-1 w-1/2 text-center rounded-4 font-normal text-14 text-blue-500 bg-transparent hover:bg-blue-200"
                            on:click|stopPropagation={handleRejectClick}
                        >
                            {localize('actions.reject')}
                        </button>
                        <button
                            class="action px-3 py-1 w-1/2 h-8 text-center rounded-4 font-normal text-14 text-white bg-blue-500 hover:bg-blue-600 dark:hover:bg-blue-400"
                            on:click|stopPropagation={handleClaimClick}
                        >
                            {#if activity.isClaiming}
                                <Spinner busy={true} classes="justify-center h-fit" />
                            {:else}
                                {localize('actions.claim')}
                            {/if}
                        </button>
                    {:else}
                        <ActivityAsyncStatusPill asyncStatus={activity.asyncStatus} />
                    {/if}
                </div>
            </div>
        {/if}
    </div>
</ClickableTile>
