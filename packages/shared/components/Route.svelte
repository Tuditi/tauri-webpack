<script lang="ts">
    import { appRoute, AppRoute } from '@core/router'
    import { Transition } from '.'

    export let transition = true
    export let route: AppRoute
</script>

{#if $appRoute === route}
    {#if transition}
        <Transition>
            <slot />
        </Transition>
    {:else}
        <slot />
    {/if}
{/if}
