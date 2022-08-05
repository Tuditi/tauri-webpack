<script lang="ts">
    import { setClipboard } from '../lib/utils'
    import { Icon } from '.'

    export let itemToCopy: string
</script>

<button class="ml-2" on:click={() => setClipboard(itemToCopy.toLowerCase())}>
    <Icon icon="copy" classes="text-gray-500" />
</button>
