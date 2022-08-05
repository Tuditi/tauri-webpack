<script lang="ts">
    import { Button, Text } from '../../../../../components'
    import { localize } from '@core/i18n'
    import { navigateToNewIndexMigration } from '../../../../../lib/ledger'
</script>

<Text type="h4" classes="mb-3">{localize('views.settings.migrateLedgerIndex.title')}</Text>
<Text type="p" secondary classes="mb-5">{localize('views.settings.migrateLedgerIndex.description')}</Text>
<Button medium inlineStyle="min-width: 156px;" onClick={navigateToNewIndexMigration}>
    {localize('views.settings.migrateLedgerIndex.title')}
</Button>
