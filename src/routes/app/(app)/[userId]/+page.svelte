<script lang="ts">
  import { page } from '$app/stores';
  import IdentityBadge from '$lib/components/identity-badge/identity-badge.svelte';
  import LargeEmptyState from '$lib/components/large-empty-state/large-empty-state.svelte';
  import ens from '$lib/stores/ens';
  import streams from '$lib/stores/streams';
  import wallet from '$lib/stores/wallet/wallet.store';
  import { onMount } from 'svelte';
  import Balances from '../streams/sections/balances.section.svelte';
  import Splits from '../streams/sections/splits.section.svelte';
  import Streams from '../streams/sections/streams.section.svelte';
  import SocialLink from '$lib/components/social-link/social-link.svelte';
  import unreachable from '$lib/utils/unreachable';
  import SectionSkeleton from '$lib/components/section-skeleton/section-skeleton.svelte';
  import { fade, fly } from 'svelte/transition';
  import decodeUserId from '$lib/utils/decode-user-id';

  $: userId = $page.params.userId;

  let beamsUserId: string | undefined;
  let address: string | undefined;
  let error: Error | undefined;

  const ensRecords = ['description', 'url', 'com.twitter', 'com.github'] as const;
  const socialLinks = ['com.twitter', 'com.github', 'url'] as const;

  let socialLinkValues: { [key in (typeof socialLinks)[number]]: string } | undefined = undefined;
  let description: string | undefined;

  async function fetchEnsRecords(
    ensName: string,
  ): Promise<{ [key in (typeof ensRecords)[number]]: string } | undefined> {
    try {
      const { provider } = $wallet;

      const resolver = await provider.getResolver(ensName);

      const promises = ensRecords.map(async (recordName) => [
        recordName,
        await resolver?.getText(recordName),
      ]);

      const result = Object.fromEntries(await Promise.all(promises));

      return result;
    } catch {
      return undefined;
    }
  }

  async function updateEnsRecords(name: string) {
    const records = await fetchEnsRecords(name);

    if (!records) return;

    description = records.description;

    socialLinkValues = {
      url: records.url,
      'com.github': records['com.github'],
      'com.twitter': records['com.twitter'],
    };
  }

  onMount(async () => {
    try {
      const decodedUserId = await decodeUserId(userId);

      address = decodedUserId.address;
      beamsUserId = decodedUserId.beamsUserId;
    } catch (e) {
      error = e instanceof Error ? e : new Error();
    }
  });

  $: {
    const name = address && $ens[address]?.name;
    if (name) updateEnsRecords(name);
  }

  function isNetwork(input: string): input is (typeof socialLinks)[number] {
    return socialLinks.includes(input as (typeof socialLinks)[number]);
  }

  async function fetchRequestedAccount(userId: string) {
    try {
      await streams.fetchAccount(userId);
    } catch (e) {
      // eslint-disable-next-line no-console
      console.error(e);
      error = e instanceof Error ? e : new Error('Unable to fetch account');
    }
  }

  $: beamsUserId && fetchRequestedAccount(beamsUserId);

  
</script>

<svelte:head>
  <title>{(address && $ens[address]?.name) ?? address ?? userId} | Beaaaams</title>
  <meta name="description" content="Beaaaams User Profile" />
</svelte:head>

{#if error}
  <LargeEmptyState

    headline="Unable to show profile"
    description="We weren't able to find that profile."
  />
{:else}
  <div class="profile">
    <SectionSkeleton placeholderOutline={false} loaded={Boolean(address)}>
      {#if address}
        <div class="identity">
          <div class="avatar-and-name">
            <IdentityBadge
              disableLink
              {address}
              size="gigantic"
              showIdentity={false}
              
            />
            <div class="w-full">
              <IdentityBadge
                disableLink
                {address}
                size="gigantic"
                showAvatar={false}
                
              />
            </div>
          </div>
          <div class="social-links">
            <div in:fade><SocialLink network="ethereum" value={address} /></div>
            {#each Object.entries(socialLinkValues ?? {}) as [network, value]}
              {#if value}<div in:fade>
                  <SocialLink network={isNetwork(network) ? network : unreachable()} {value} />
                </div>{/if}
            {/each}
          </div>
          {#if description}<p class="description" in:fade>{description}</p>{/if}
        </div>
      {/if}
    </SectionSkeleton>
    <Balances userId={beamsUserId} />
    <Streams userId={beamsUserId} />
    <Splits userId={beamsUserId} />
    
  </div>
{/if}

<style>
  .identity {
    display: flex;
    gap: 1.5rem;
    flex-direction: column;
  }

  .identity > .avatar-and-name {
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }

  .profile {
    display: flex;
    flex-direction: column;
    gap: 4rem;
  }

  .social-links {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
  }

  .social-links * {
    display: flex;
    gap: 0.5rem;
  }
  .social-links *:not(:last-child)::after {
    content: '•';
  }

  .description {
    max-width: 40rem;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
  }
</style>
