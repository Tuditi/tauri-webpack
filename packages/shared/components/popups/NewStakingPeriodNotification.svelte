<script lang="ts">
    import { formatDate, LocaleArguments, localize } from '@core/i18n'
    import { getAccountParticipationAbility } from '@lib/participation/participation'
    import { assemblyStakingEventState, shimmerStakingEventState } from '@lib/participation/stores'
    import { AccountParticipationAbility } from '@lib/participation/types'
    import { closePopup, openPopup } from '@lib/popup'
    import { selectedAccount } from '@core/account'
    import { Button, Illustration, Text, TextHint } from '..'
    import { ASSEMBLY_EVENT_START_DATE, isStakingPossible, LAST_ASSEMBLY_STAKING_PERIOD } from '@lib/participation'

    function getLocaleArguments(): LocaleArguments {
        return {
            values: {
                periodNumber: LAST_ASSEMBLY_STAKING_PERIOD + 1,
                date: formatDate(ASSEMBLY_EVENT_START_DATE, { format: 'long' }),
            },
        }
    }

    function handleOk(): void {
        const isStakingPossibleForAssembly = isStakingPossible($assemblyStakingEventState)
        const isStakingPossibleForShimmer = isStakingPossible($shimmerStakingEventState)
        const cannotStake =
            getAccountParticipationAbility($selectedAccount) === AccountParticipationAbility.HasDustAmount

        if ((isStakingPossibleForAssembly || isStakingPossibleForShimmer) && !cannotStake) {
            openPopup({
                type: 'stakingManager',
            })
        } else {
            closePopup(true)
        }
    }
</script>

<Illustration illustration="staking-notification" classes="mb-4" />
<Text type="h3" classes="mb-2">{localize('popups.newStakingPeriodNotification.title', getLocaleArguments())}</Text>
<Text type="p" secondary classes="mb-4"
    >{localize('popups.newStakingPeriodNotification.body', getLocaleArguments())}</Text
>
<TextHint info text={localize('popups.newStakingPeriodNotification.info')} classes="mb-4" />
<div class="flex flex-row space-x-2">
    <Button classes="w-full" onClick={handleOk}>
        {localize('actions.okIUnderstand')}
    </Button>
</div>
