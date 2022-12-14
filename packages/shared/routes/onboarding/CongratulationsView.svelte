<script lang="ts">
    import { Animation, Button, Icon, OnboardingLayout, Text } from '../../components'
    import { onDestroy, onMount } from 'svelte'
    import { get } from 'svelte/store'
    import { mobile } from '@core/app'
    import { appRouter, ledgerRouter } from '@core/router'
    import { localize } from '@core/i18n'
    import {
        activeProfile,
        addNewProfile,
        getStorageDirectoryOfProfile,
        loadPersistedProfileIntoActiveProfile,
        login,
    } from '@core/profile'
    import { createNewAccount } from '@core/account'
    import { newProfile, cleanupOnboarding } from '@contexts/onboarding'
    import { SetupType } from '@lib/typings/setup'
    import { convertToFiat, currencies, exchangeRates, formatCurrency } from '@lib/currency'
    import { Platform } from '@lib/platform'
    import { promptUserToConnectLedger } from '@lib/ledger'
    import { LOG_FILE_NAME, migration, migrationLog, resetMigrationState, totalMigratedBalance } from '@lib/migration'
    import { LedgerAppName } from '@lib/typings/ledger'
    import { formatUnitBestMatch } from '@lib/units'
    import { walletSetupType } from '@lib/wallet'
    import { AvailableExchangeRates, CurrencyTypes } from '@lib/typings/currency'

    const { didComplete } = $migration
    const wasMigrated = $didComplete

    let localizedBody = 'body'
    let localizedValues: { legacy: LedgerAppName }
    let logExported = false

    const fiatbalance = formatCurrency(
        convertToFiat(
            // Only show actually migrated balance to user
            $totalMigratedBalance,
            get(currencies)[CurrencyTypes.USD],
            get(exchangeRates)[AvailableExchangeRates.USD]
        ),
        AvailableExchangeRates.USD
    )

    function advanceView(): void {
        cleanupOnboarding()
        $appRouter.next()
    }

    function handleContinueClick(): void {
        if (wasMigrated) {
            const _continue = () => {
                /**
                 * We check for the new Ledger IOTA app to be connected after migration
                 * because the last app the user had open was the legacy one
                 */
                if ($walletSetupType === SetupType.TrinityLedger) {
                    promptUserToConnectLedger(false, advanceView)
                } else {
                    advanceView()
                }
            }
            const _exportMigrationLog = () => {
                getStorageDirectoryOfProfile($activeProfile?.id)
                    .then((source) =>
                        $walletSetupType === SetupType.TrinityLedger
                            ? Platform.exportLedgerMigrationLog($migrationLog, `${$activeProfile?.id}-${LOG_FILE_NAME}`)
                            : Platform.exportMigrationLog(
                                  `${source}/${LOG_FILE_NAME}`,
                                  `${$activeProfile?.id}-${LOG_FILE_NAME}`
                              )
                    )
                    .then((result) => {
                        if (result) {
                            logExported = true
                            _continue()
                        }
                    })
                    .catch(console.error)
            }
            if (logExported) {
                _continue()
            } else {
                _exportMigrationLog()
            }
        } else {
            advanceView()
        }
    }

    onMount(async () => {
        if (!wasMigrated) {
            if ($walletSetupType === SetupType.FireflyLedger) {
                localizedBody = 'fireflyLedgerBody'
            }
            // This is the last screen in onboarding for all flows i.e., if you create a new wallet or import stronghold
            // When this component mounts, ensure that the profile is persisted in the local storage.
            addNewProfile($newProfile)
            loadPersistedProfileIntoActiveProfile($newProfile.id)
            newProfile.set(null)
            await createNewAccount()
            const shouldRecoverAccounts = $walletSetupType !== SetupType.New
            void login(shouldRecoverAccounts)
        } else {
            if ($walletSetupType === SetupType.TrinityLedger) {
                localizedBody = 'trinityLedgerBody'
                localizedValues = { legacy: LedgerAppName.IOTALegacy }

                // updateProfile('ledgerMigrationCount', $activeProfile?.ledgerMigrationCount + 1)
            } else {
                localizedBody = 'softwareMigratedBody'
            }
        }
    })

    onDestroy(() => {
        if (wasMigrated) {
            resetMigrationState()
        }
        $ledgerRouter.reset()
    })
</script>

<OnboardingLayout allowBack={false}>
    <div slot="leftpane__content">
        {#if wasMigrated}
            <div class="relative flex flex-col items-center bg-gray-100 dark:bg-gray-900 rounded-2xl mt-10 p-10 pb-6">
                <div class="bg-green-500 rounded-2xl absolute -top-6 w-12 h-12 flex items-center justify-center">
                    <Icon icon="success-check" classes="text-white" />
                </div>
                <Text type="h2" classes="mb-6 text-center">{localize('views.congratulations.fundsMigrated')}</Text>
                <Text type="p" secondary classes="mb-6 text-center">
                    {localize(`views.congratulations.${localizedBody}`, { values: localizedValues })}
                </Text>
                <Text type="h2">{formatUnitBestMatch($totalMigratedBalance, true, 3)}</Text>
                <Text type="p" highlighted classes="py-1 uppercase">{fiatbalance}</Text>
            </div>
        {:else}
            <div class="relative flex flex-col items-center bg-gray-100 dark:bg-gray-900 rounded-2xl mt-10 p-10 pb-6">
                <div class="bg-green-500 rounded-2xl absolute -top-6 w-12 h-12 flex items-center justify-center">
                    <Icon icon="success-check" classes="text-white" />
                </div>
                <Text type="h2" classes="mb-5 text-center">{localize('views.congratulations.title')}</Text>
                <Text type="p" secondary classes="mb-2 text-center"
                    >{localize(`views.congratulations.${localizedBody}`)}</Text
                >
            </div>
        {/if}
    </div>
    <div slot="leftpane__action">
        <Button classes="w-full" onClick={() => handleContinueClick()}>
            {localize(
                `${wasMigrated && !logExported ? 'views.congratulations.exportMigration' : 'actions.finishSetup'}`
            )}
        </Button>
    </div>
    <div slot="rightpane" class="w-full h-full flex justify-center {!$mobile && 'bg-pastel-yellow dark:bg-gray-900'}">
        <Animation classes="setup-anim-aspect-ratio" animation="congratulations-desktop" />
    </div>
</OnboardingLayout>
