<script lang="ts">
    import { onMount } from 'svelte'
    import { Transition } from '../../components'
    import { EnterPinView, SelectProfileView } from './views'
    import { FireflyEvent, loginRoute, LoginRouter, LoginRoute } from '@core/router'

    let loginRouter: LoginRouter

    onMount(() => {
        loginRouter = new LoginRouter()
    })

    const next = (event: CustomEvent<FireflyEvent>): void => loginRouter.next(event.detail)
    const previous = (): void => loginRouter.previous()
</script>

{#if $loginRoute === LoginRoute.SelectProfile}
    <Transition>
        <SelectProfileView on:next={next} on:previous={previous} />
    </Transition>
{:else if $loginRoute === LoginRoute.EnterPin}
    <Transition>
        <EnterPinView on:next={next} on:previous={previous} />
    </Transition>
{/if}
