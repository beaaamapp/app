<script lang="ts">
  import ChevronRight from 'radicle-design-system/icons/ChevronRight.svelte';
  import type { ComponentType } from 'svelte';

  export let disabled = false;
  export let icon: ComponentType | undefined = undefined;
  export let href: string | undefined = undefined;
</script>

<a {href} on:click class:disabled class:clickable={href} class="account-menu-item-wrapper">
  {#if !icon}
    <slot name="left" />
  {:else}
    <div class="icon-wrapper">
      {#if icon}<svelte:component this={icon} style="fill: var(--color-background)" />{/if}
    </div>
  {/if}
  <div class="description typo-text">
    <slot name="title" />
  </div>
  <slot name="right">
    <ChevronRight />
  </slot>
</a>

<style>
  .account-menu-item-wrapper {
    display: flex;
    gap: 0.5rem;
    align-items: center;
    padding: 0.5rem;
    border-radius: 1rem 0 1rem 1rem;
    transition: background-color 0.3s;
  }

  .account-menu-item-wrapper > * {
    min-width: 0;
  }

  .account-menu-item-wrapper.clickable {
    cursor: pointer;
  }

  .account-menu-item-wrapper.clickable:hover {
    background-color: var(--color-foreground-level-1);
  }

  .account-menu-item-wrapper.clickable:focus {
    background-color: var(--color-foreground-level-2);
    outline: none;
  }

  .account-menu-item-wrapper.disabled {
    opacity: 0.5;
  }

  .icon-wrapper {
    height: 3rem;
    width: 3rem;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: var(--color-foreground);
    border-radius: 1.5rem;
  }

  .description {
    flex: 1;
    color: var(--color-foreground);
  }
</style>
