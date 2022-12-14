<script lang="ts">
    import { Text, NodeConfigurationForm, Button, Spinner } from '..'
    import { localize } from '@core/i18n'
    import { INode, INetwork } from '@core/network'
    import { closePopup } from '@lib/popup'
    import { activeProfile, addNodeToActiveProfile } from '@core/profile'
    import { showAppNotification } from '@lib/notifications'

    export let node: INode = { url: '', auth: { username: '', password: '', jwt: '' } }
    export let nodes: INode[] = []
    export let network: INetwork
    export let isEditingNode: boolean = false
    export let onSuccess: (..._: any[]) => void

    let nodeConfigurationForm: NodeConfigurationForm
    let isBusy = false

    async function handleAddNode(): Promise<void> {
        try {
            isBusy = true
            await nodeConfigurationForm.validate({
                validateUrl: true,
                checkSameNetwork: true,
                uniqueCheck: !isEditingNode,
                checkNodeInfo: true,
                validateClientOptions: true,
            })
            await addNodeToActiveProfile(node)
            onSuccess()
        } catch (err) {
            if (err.type !== 'validationError') {
                showAppNotification({
                    type: 'error',
                    message: localize(err?.error ?? 'error.global.generic'),
                })
            }
        } finally {
            isBusy = false
        }
    }
</script>

<div class="flex flex-col space-y-6">
    <Text type="h4">{localize(`popups.node.title${isEditingNode ? 'Update' : 'Add'}`)}</Text>
    <NodeConfigurationForm
        bind:this={nodeConfigurationForm}
        bind:node
        {isBusy}
        {nodes}
        {network}
        isDeveloperProfile={$activeProfile.isDeveloperProfile}
    />
    <div class="flex flex-row justify-between space-x-4 w-full">
        <Button secondary classes="w-1/2" onClick={closePopup} disabled={isBusy}>
            {localize('actions.cancel')}
        </Button>
        <Button
            disabled={!node.url || isBusy}
            type="submit"
            form="node-config-form"
            classes="w-1/2"
            onClick={handleAddNode}
        >
            {#if isBusy}
                <Spinner
                    busy={isBusy}
                    message={localize(`popups.node.${isEditingNode ? 'updatingNode' : 'addingNode'}`)}
                    classes="justify-center"
                />
            {:else}
                {localize(`actions.${isEditingNode ? 'updateNode' : 'addNode'}`)}
            {/if}
        </Button>
    </div>
</div>
