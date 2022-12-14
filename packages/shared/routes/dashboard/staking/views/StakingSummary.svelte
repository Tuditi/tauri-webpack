<script lang="ts">
    import { Button, Icon, Spinner, Text, Tooltip } from '../../../../components'
    import { localize } from '@core/i18n'
    import { showAppNotification } from '../../../../lib/notifications'
    import { getAccountParticipationAbility, isNewStakingEvent, isStakingPossible } from '../../../../lib/participation'
    import {
        assemblyStakingEventState,
        isPartiallyStaked,
        partiallyUnstakedAmount,
        participationAction,
        shimmerStakingEventState,
        stakedAmount,
        unstakedAmount,
    } from '../../../../lib/participation/stores'
    import { AccountParticipationAbility, ParticipationAction } from '../../../../lib/participation/types'
    import { openPopup } from '../../../../lib/popup'
    import { formatUnitBestMatch } from '../../../../lib/units'
    import { isSyncing } from '../../../../lib/wallet'
    import { selectedAccount } from '@core/account'

    $: showSpinner = !!$participationAction || $isSyncing

    $: canParticipateInEvent =
        isStakingPossible($assemblyStakingEventState) || isStakingPossible($shimmerStakingEventState)

    $: cannotStake = getAccountParticipationAbility($selectedAccount) === AccountParticipationAbility.HasDustAmount

    $: isStakedAndCanParticipate = $stakedAmount > 0 && canParticipateInEvent

    $: isPartiallyStakedAndCanParticipate = $isPartiallyStaked && canParticipateInEvent

    const canParticipateWithNode = false
    // $: $networkStatus, (canParticipateWithNode = doesNodeHavePlugin(NodePlugin.Participation))

    let showTooltip = false
    // hide tooltip if tooltipAnchor destroys
    $: {
        if (!isPartiallyStakedAndCanParticipate) {
            showTooltip = false
        }
    }

    let tooltipAnchor
    function toggleTooltip(): void {
        showTooltip = !showTooltip
    }

    function handleStakeFundsClick(): void {
        if (cannotStake) {
            showAppNotification({
                type: 'warning',
                message: localize('warning.participation.noFunds'),
            })

            return
        }

        const showNotice = isNewStakingEvent($assemblyStakingEventState) || isNewStakingEvent($shimmerStakingEventState)

        const type = !isStakedAndCanParticipate && showNotice ? 'newStakingPeriodNotification' : 'stakingManager'

        openPopup({ type })
    }

    function getSpinnerMessage(): string {
        if ($participationAction) {
            return $participationAction === ParticipationAction.Stake ? 'general.staking' : 'general.unstaking'
        } else if ($isSyncing) {
            return 'general.syncingAccounts'
        }
    }
</script>

<div class="p-6 flex flex-col justify-between space-y-6 w-full h-full">
    <div class="flex flex-col justify-between">
        <div class="flex flex-row justify-between items-start">
            <Text type="p" smaller overrideColor classes="mb-3 text-gray-700 dark:text-gray-500">
                {localize('views.staking.summary.stakedFunds')}
            </Text>
            {#if isPartiallyStakedAndCanParticipate}
                <div bind:this={tooltipAnchor} on:mouseenter={toggleTooltip} on:mouseleave={toggleTooltip}>
                    <Icon icon="exclamation" classes="fill-current text-yellow-600" />
                </div>
            {/if}
        </div>
        <Text type="h2">{formatUnitBestMatch(canParticipateInEvent ? $stakedAmount : 0)}</Text>
        {#if canParticipateInEvent}
            <Text type="p" smaller secondary classes="mt-2">
                {formatUnitBestMatch($unstakedAmount)}
                {localize('general.unstaked')}
            </Text>
        {/if}
    </div>
    <Button
        classes="w-full text-14"
        disabled={showSpinner || !canParticipateInEvent}
        caution={isStakedAndCanParticipate && isPartiallyStakedAndCanParticipate}
        secondary={isStakedAndCanParticipate && !isPartiallyStakedAndCanParticipate}
        onClick={() =>
            canParticipateWithNode
                ? handleStakeFundsClick()
                : showAppNotification({
                      type: 'warning',
                      message: localize('error.node.pluginNotAvailable'),
                  })}
    >
        {#if showSpinner}
            <Spinner busy message={localize(getSpinnerMessage())} classes="mx-2 justify-center" />
        {:else}{localize(`actions.${isStakedAndCanParticipate ? 'manageStake' : 'stakeFunds'}`)}{/if}
    </Button>
</div>
{#if showTooltip}
    <Tooltip anchor={tooltipAnchor} position="right">
        {#if isPartiallyStakedAndCanParticipate}
            <Text type="p" classes="text-gray-900 bold mb-1 text-left">
                {localize(
                    `tooltips.partiallyStakedFunds.title${$isPartiallyStaked ? '' : 'NoFunds'}`,
                    $isPartiallyStaked ? { values: { amount: formatUnitBestMatch($partiallyUnstakedAmount) } } : {}
                )}
            </Text>
            <Text type="p" secondary classes="text-left">
                {localize('tooltips.partiallyStakedFunds.preBody')}
                {localize('tooltips.partiallyStakedFunds.body')}
            </Text>
        {/if}
    </Tooltip>
{/if}
