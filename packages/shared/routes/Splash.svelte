<script lang="ts">
    import { Animation } from '../components'
    import { appSettings, shouldBeDarkMode, mobile } from '@core/app'
    import { onMount } from 'svelte'

    /**
     * NOTE: This reactive dependency ensures that darkMode is set to the
     * correct value in the case that the system's settings have changed since
     * the app was last opened.
     */
    $: $appSettings.darkMode = shouldBeDarkMode($appSettings.theme)

    onMount(() => {
        console.log('Hey there')
        // void pollChrysalisSnapshot()
    })
</script>

<div class="w-full h-full flex justify-center items-center bg-white dark:bg-gray-900">
    <div class={$mobile ? 'w-full px-4' : 'w-1/3'}>
        <Animation classes="w-full h-auto" animation="splashscreen-desktop" loop={false} renderer="canvas" />
    </div>
</div>
