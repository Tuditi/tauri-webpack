<script lang="ts">
    import { Pane } from '../../../components'
    import { assemblyStakingEventState, participationAction, shimmerStakingEventState } from '@lib/participation/stores'
    import { ParticipationEventState, StakingAirdrop as _StakingAirdrop } from '@lib/participation/types'
    import { closePopup, popupState } from '@lib/popup'
    import { onMount } from 'svelte'
    import { StakingAirdrop, StakingInfo, StakingSummary } from './views'

    // const handleNewStakingEvent = (): void => {
    //     openPopup({
    //         type: 'newStakingPeriodNotification',
    //         hideClose: true,
    //         preventClose: false,
    //     })

    //     updateActiveProfileSettings({'hasVisitedStaking': true})
    // }

    // TODO: This is an exact copy of a method defined in Wallet.svelte. Need to move it to shared.
    // const handleTransactionEventData = (eventData: TransferProgressEventData): TransactionEventData => {
    //     if (!eventData) return {}

    //     const remainderData = eventData as GeneratingRemainderDepositAddressEvent
    //     if (remainderData?.address) return { remainderAddress: remainderData?.address }

    //     const txData = eventData as PreparedTransactionEvent
    //     if (!(txData?.inputs && txData?.outputs) || txData?.inputs.length <= 0 || txData?.outputs.length <= 0) return {}

    //     const numOutputs = txData.outputs.length
    //     if (numOutputs === 1) {
    //         return {
    //             toAddress: txData.outputs[0].address,
    //             toAmount: txData.outputs[0].amount,
    //         }
    //     } else if (numOutputs > 1) {
    //         return {
    //             toAddress: txData.outputs[0].address,
    //             toAmount: txData.outputs[0].amount,

    //             remainderAddress: txData.outputs[numOutputs - 1].address,
    //             remainderAmount: txData.outputs[numOutputs - 1].amount,
    //         }
    //     } else {
    //         return txData
    //     }
    // }

    // let transactionEventData: TransferProgressEventData = null
    // let ledgerAwaitingConfirmation = false

    // const handleTransferState = (state: TransferState): void => {
    //     if (!state) return

    //     const _onCancel = () => {
    //         // transferState.set(null)

    //         closePopup(true)
    //     }

    //     const { data, type } = state
    //     switch (type) {
    //         // If a user presses "Accept" on ledger, this is the next transfer progress item.
    //         case TransferProgressEventType.PerformingPoW:
    //             // Close the current pop up i.e., the one with ledger transaction details
    //             closePopup(true)
    //             // Re-open the staking manager pop up
    //             openPopup({ type: 'stakingManager' }, true)
    //             break

    //         case TransferProgressEventType.SigningTransaction:
    //             ledgerAwaitingConfirmation = true
    //             openPopup(
    //                 {
    //                     type: 'ledgerTransaction',
    //                     hideClose: true,
    //                     preventClose: true,
    //                     props: {
    //                         // TODO: this was duplicated in this file (see Send.svelte)
    //                         ...handleTransactionEventData(transactionEventData),
    //                         onCancel: _onCancel,
    //                     },
    //                 },
    //                 true
    //             )

    //             break

    //         case TransferProgressEventType.PreparedTransaction:
    //             transactionEventData = data

    //             break

    //         default:
    //             break
    //     }
    // }

    $: if (!$participationAction && $popupState.type === 'ledgerTransaction') {
        closePopup(true)
    }

    $: if (
        $popupState.type === 'stakingManager' &&
        $assemblyStakingEventState === ParticipationEventState.Inactive &&
        $shimmerStakingEventState === ParticipationEventState.Inactive
    ) {
        closePopup(true)
    }

    onMount(() => {
        // if (!$activeProfile?.hasVisitedStaking) {
        //     handleNewStakingEvent()
        // }
        // await getParticipationEvents()
        // await getParticipationOverview(ASSEMBLY_EVENT_ID)
    })

    /** Subscribe to transfer state */
    // const unsubscribeFromTransferState = transferState.subscribe((state) => {
    //     if (!$isSoftwareProfile) {
    //         handleTransferState(state)
    //     }
    // })

    // onDestroy(() => {
    //     unsubscribeFromTransferState()
    // })
</script>

<div class="staking-wrapper w-full h-full flex flex-col flex-nowrap p-10 flex-1 bg-gray-50 dark:bg-gray-900">
    <div class="w-full h-full grid grid-cols-3 gap-x-4 min-h-0">
        <div class="h-full flex flex-col space-y-3 overflow-hidden">
            <Pane classes="w-full flex-shrink-0">
                <StakingSummary />
            </Pane>
            <Pane classes="w-full flex-grow">
                <StakingInfo />
            </Pane>
        </div>
        <Pane classes="h-full">
            <StakingAirdrop airdrop={_StakingAirdrop.Assembly} />
        </Pane>
        <Pane classes="h-full">
            <StakingAirdrop airdrop={_StakingAirdrop.Shimmer} />
        </Pane>
    </div>
</div>
