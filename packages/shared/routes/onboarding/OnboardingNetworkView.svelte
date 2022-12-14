<script lang="ts">
    import { Animation, OnboardingButton, OnboardingLayout, Text } from '../../components'
    import { TextType } from '../../components/Text.svelte'
    import features from '../../features/features'
    import { mobile } from '@core/app'
    import { localize } from '@core/i18n'
    import { NetworkType } from '@core/network'
    import { appRouter } from '@core/router'
    import {
        cleanupOnboarding,
        newProfile,
        updateNewProfile,
        initProfileManagerFromNewProfile,
    } from '@contexts/onboarding'

    const networkProtocol = $newProfile.networkProtocol

    const networkIcon: Readonly<{ [key in NetworkType]: string }> = {
        [NetworkType.Mainnet]: networkProtocol,
        [NetworkType.Devnet]: 'settings',
        [NetworkType.PrivateNet]: 'settings',
    }

    async function onClick(networkType: NetworkType): Promise<void> {
        if (networkType === NetworkType.PrivateNet) {
            updateNewProfile({ networkType })
        } else {
            updateNewProfile({ networkProtocol, networkType: networkType })
            await initProfileManagerFromNewProfile()
        }
        $appRouter.next({ networkType })
    }

    async function onBackClick(): Promise<void> {
        const isDeveloperProfile = $newProfile.isDeveloperProfile
        await cleanupOnboarding(true)
        updateNewProfile({ isDeveloperProfile })
        $appRouter.previous()
    }
</script>

<OnboardingLayout {onBackClick}>
    <div slot="title">
        <Text type={TextType.h2}>{localize('views.network.title')}</Text>
    </div>
    <div slot="leftpane__content">
        <Text secondary classes="mb-8">{localize('views.network.body')}</Text>
    </div>
    <div slot="leftpane__action" class="flex flex-col space-y-4">
        {#each Object.values(NetworkType) as networkType}
            <OnboardingButton
                primaryText={localize(`views.network.${networkProtocol}.${networkType}.title`)}
                secondaryText={!$mobile ? localize(`views.network.${networkProtocol}.${networkType}.body`) : ''}
                icon={networkIcon[networkType]}
                iconColor={networkType === NetworkType.Mainnet ? `${networkProtocol}-highlight` : 'blue-500'}
                hidden={features?.onboarding?.[networkProtocol]?.[networkType]?.hidden}
                disabled={!features?.onboarding?.[networkProtocol]?.[networkType]?.enabled}
                onClick={() => onClick(networkType)}
            />
        {/each}
    </div>
    <div slot="rightpane" class="w-full h-full flex justify-center {!$mobile && 'bg-pastel-yellow dark:bg-gray-900'}">
        <Animation classes="setup-anim-aspect-ratio" animation="onboarding-network-desktop" />
    </div>
</OnboardingLayout>
