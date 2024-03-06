<script lang="ts">
  import Balances from './sections/balances.section.svelte';
  import Streams from './sections/streams.section.svelte';
  import Splits from './sections/splits.section.svelte';

  import wallet from '$lib/stores/wallet/wallet.store';
  import guardConnected from '$lib/utils/guard-connected';


  $: userId = $wallet.beamsUserId;

  $: {
    $wallet.connected;
    guardConnected();
  }

</script>

<svelte:head>
  <title>Streams | Beaaams</title>
  <meta name="description" content="Beaaams Streams Page" />
</svelte:head>

<div class="dashboard">
 
  <div class="sections">
    <Balances {userId} disableActions={false} />
    <Streams {userId} disableActions={false} />
    <Splits {userId} disableActions={false} />
  </div>
</div>

<style>
  h4 {
    margin-bottom: 1.5rem;
  }

  .edu-carousel {
    margin: 0 -4rem;
  }

  .edu-carousel h4 {
    margin-left: 4rem;
  }

  .sections {
    margin-top: 4rem;
    display: flex;
    flex-direction: column;
    gap: 4rem;
  }
</style>
