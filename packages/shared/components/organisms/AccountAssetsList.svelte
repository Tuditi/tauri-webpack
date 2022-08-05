<script lang="ts">
    import { AssetList } from '..'
    import { selectedAccountAssets } from '@core/wallet'
</script>

<AssetList assets={$selectedAccountAssets} />
