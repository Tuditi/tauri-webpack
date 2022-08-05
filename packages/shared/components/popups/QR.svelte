<script lang="ts">
    import { Text, QR } from '..'
    import { Locale } from '@core/i18n'

    export let locale: Locale

    export let data
</script>

<Text type="h4" classes="mb-5">{locale('popups.password.title')}</Text>
<div class="flex justify-center w-full">
    <QR {data} />
</div>
