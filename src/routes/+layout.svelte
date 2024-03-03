<script lang="ts">
  // Global CSS imports
  import '../styles/tailwind.css';
  import '$lib/static/reset.css';
  import '$lib/static/global.css';
  import '$lib/static/colors.css';
  import '$lib/static/elevation.css';
  import '$lib/static/typography.css';
  import '../styles/app.css';

  import themeStore from '$lib/stores/theme/theme.store';

  import { page } from '$app/stores';
  import { onMount } from 'svelte';
  import scroll from '$lib/stores/scroll';
 

  onMount(() => {
    scroll.attach();
    return scroll.detach;
  });

  const { primaryColor } = themeStore;
</script>

<svelte:head>
  <meta property="og:title" content="Beaaams" />
  <meta
    property="og:description"
    content="Beaaam allows you to stream money without any platform fees.."
  />
</svelte:head>

<div class="main" data-uifont="inter" data-theme={$themeStore.currentTheme}>
  <div class="page" data-primary-color={$primaryColor}>
    <slot />
  </div>
</div>

<style>
  .main {
    min-height: 100vh;
    min-width: 100vw;
    background-color: var(--color-background);
    color: var(--color-foreground);
    overflow: hidden;
    transition: background-color 0.5s;
  }
</style>
