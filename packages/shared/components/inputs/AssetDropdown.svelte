<script lang="ts">
    import { NetworkProtocol } from '@core/network'
    import { selectedAccountAssets } from '@core/wallet'
    import { AssetTile, Icon, Text, AssetIcon } from '..'
    import { FontWeightText } from '../Text.svelte'
    import { clickOutside } from '../../lib/actions'

    export let asset = $selectedAccountAssets?.baseCoin

    let isDropdownOpen = false
    let icon: string

    $: hasMultipleAssets = $selectedAccountAssets?.nativeTokens.length >= 1
    $: switch (asset?.metadata?.name?.toLocaleLowerCase()) {
        case NetworkProtocol.IOTA:
        case NetworkProtocol.Shimmer:
            icon = asset?.metadata?.name?.toLocaleLowerCase()
            break
        default:
            icon = 'tokens'
    }

    function handleDropdownClick() {
        if (hasMultipleAssets) {
            isDropdownOpen = !isDropdownOpen
        }
    }

    function handleAssetClick(_asset) {
        asset = _asset
        isDropdownOpen = false
    }

    function handleOnClickOutside(): void {
        isDropdownOpen = false
    }
</script>

{#if asset}
    <div class="asset-dropdown flex flex-col" use:clickOutside on:clickOutside={handleOnClickOutside}>
        <div
            class="flex flex-row items-center p-2 space-x-2 text-left bg-gray-100 dark:bg-gray-700 rounded-md cursor-default"
            class:cursor-pointer={hasMultipleAssets}
            on:click={handleDropdownClick}
        >
            <AssetIcon small {asset} />
            <div class="w-full relative" style="max-width: 75px;">
                <Text
                    color="gray-600"
                    darkColor="white"
                    fontWeight={FontWeightText.semibold}
                    fontSize="15"
                    classes="overflow-hidden whitespace-nowrap overflow-ellipsis"
                >
                    {asset?.metadata?.name ?? asset?.id}
                </Text>
            </div>
            {#if hasMultipleAssets}
                <div class="transform rotate-0">
                    <Icon height="18" width="18" icon="chevron-down" classes="text-gray-600 dark:text-gray-500" />
                </div>
            {/if}
        </div>
        {#if isDropdownOpen && hasMultipleAssets}
            <div
                class="dropdown bg-white dark:bg-gray-800 absolute flex flex-col top-12 -left-5 -right-5 border border-solid border-blue-500 rounded-xl z-10 p-4 "
            >
                <ul class="overflow-y-auto space-y-2 h-full -mr-2 pr-2 scroll-secondary">
                    <li>
                        <AssetTile
                            onClick={() => handleAssetClick($selectedAccountAssets?.baseCoin)}
                            asset={$selectedAccountAssets?.baseCoin}
                            overrideColor
                            classes="bg-white hover:bg-gray-50 dark:bg-transparent"
                        />
                    </li>
                    {#each $selectedAccountAssets?.nativeTokens as nativeToken}
                        <li>
                            <AssetTile
                                onClick={() => handleAssetClick(nativeToken)}
                                asset={nativeToken}
                                overrideColor
                                classes="bg-white hover:bg-gray-50 dark:bg-transparent"
                            />
                        </li>
                    {/each}
                </ul>
            </div>
        {/if}
    </div>
{/if}

<style lang="scss">
    .asset-dropdown {
        font-feature-settings: 'calt' off;

        .icon-bg {
            background-color: var(--icon-bg-color);
        }
        /* odd margin needed to match the size of the upper parent box */
        .dropdown {
            margin: 0 3px;
            max-height: 233px;
        }
    }
</style>
