<script lang="ts">
  import TokenStreamIcon from '$lib/static/icons/TokenStreams.svelte';
  import PlusIcon from '$lib/static/icons/Plus.svelte';

  import SectionHeader from '$lib/components/section-header/section-header.svelte';
  import Table, { type RowClickEventPayload } from '$lib/components/table/table.svelte';
  import { getCoreRowModel, type ColumnDef, type TableOptions } from '@tanstack/svelte-table';
  import AmountCell, { type AmountCellData } from '$lib/components/table/cells/amount.cell.svelte';
  import streams from '$lib/stores/streams/streams.store';
  import IdentityBadgeCell from '$lib/components/table/cells/identity-badge.cell.svelte';
  import SectionSkeleton from '$lib/components/section-skeleton/section-skeleton.svelte';
  import modal from '$lib/stores/modal';
  import Stepper from '$lib/components/stepper/stepper.svelte';
  import balances from '$lib/stores/balances';
  import mapFilterUndefined from '$lib/utils/map-filter-undefined';
  import TokenCell, { type TokenCellData } from '$lib/components/table/cells/token.cell.svelte';
  import { onMount } from 'svelte';
  import type { Stream } from '$lib/stores/streams/types';
  import NameAndBadgeCell, {
    type NameAndBadgeCellProps,
  } from '$lib/components/table/cells/name-and-badge-cell.svelte';
  import ChevronRightCell from '$lib/components/table/cells/chevron-right-cell.svelte';
  import { decodeStreamId } from '$lib/stores/streams/methods/make-stream-id';
  import onClickGoto from '$lib/utils/on-click-goto';
  import accountFetchStatussesStore from '$lib/stores/account-fetch-statusses/account-fetch-statusses.store';
  import createStreamFlowSteps from '$lib/flows/create-stream-flow/create-stream-flow-steps';

  export let userId: string | undefined;
  export let disableActions = true;
  export let tokenAddress: string | undefined = undefined;

  interface OutgoingStreamTableRow {
    streamId: string;
    name: NameAndBadgeCellProps;
    toAddress: string;
    amount: AmountCellData;
    token: TokenCellData;
  }

  interface IncomingStreamTableRow {
    streamId: string;
    name: NameAndBadgeCellProps;
    fromAddress: string;
    amount: AmountCellData;
    token: TokenCellData;
  }

  let outgoingTableData: OutgoingStreamTableRow[] = [];
  let incomingTableData: IncomingStreamTableRow[] = [];

  let ownStreams: ReturnType<typeof streams.getStreamsForUser> = { outgoing: [], incoming: [] };

  function updateTable() {
    ownStreams = userId ? streams.getStreamsForUser(userId) : { outgoing: [], incoming: [] };

    // filter by tokenAddress ?
    if (tokenAddress) {
      const byToken = (stream: Stream) =>
        stream.beamsConfig.amountPerSecond.tokenAddress.toLowerCase() ===
        tokenAddress?.toLowerCase();
      ownStreams = {
        outgoing: ownStreams.outgoing.filter(byToken),
        incoming: ownStreams.incoming.filter(byToken),
      };
    }

    outgoingTableData = mapFilterUndefined(ownStreams.outgoing, (stream) => {
      const estimate = balances.getEstimateByStreamId(stream.id);
      if (!estimate) return undefined;

      const { tokenAddress } = stream.beamsConfig.amountPerSecond;

      return {
        streamId: stream.id,
        name: {
          name: stream.name ?? 'Unnamed stream',
          streamId: stream.id,
          paused: stream.paused,
          durationSeconds: stream.beamsConfig.durationSeconds,
          startDate: stream.beamsConfig.startDate,
          senderId: stream.sender.userId,
          tokenAddress: tokenAddress,
        },
        toAddress: stream.receiver.address,
        amount: {
          amount: {
            amount: estimate.totalStreamed,
            tokenAddress,
          },
          amountPerSecond: {
            amount: estimate.currentAmountPerSecond,
            tokenAddress,
          },
          showSymbol: false,
        },
        token: {
          address: tokenAddress,
          size: 'small',
          show: 'symbol',
        },
      };
    });

    incomingTableData = mapFilterUndefined(ownStreams.incoming ?? [], (stream) => {
      const estimate = balances.getEstimateByStreamId(stream.id);
      if (!estimate) return undefined;

      const { tokenAddress } = stream.beamsConfig.amountPerSecond;

      return {
        streamId: stream.id,
        name: {
          name: stream.name ?? 'Unnamed stream',
          streamId: stream.id,
          paused: stream.paused,
          durationSeconds: stream.beamsConfig.durationSeconds,
          startDate: stream.beamsConfig.startDate,
          senderId: stream.sender.userId,
          tokenAddress: tokenAddress,
        },
        fromAddress: stream.sender.address,
        amount: {
          amountPerSecond: {
            amount: estimate.currentAmountPerSecond,
            tokenAddress,
          },
          amount: {
            amount: estimate.totalStreamed,
            tokenAddress,
          },
          showSymbol: false,
        },
        token: {
          address: tokenAddress,
          size: 'small',
          show: 'symbol',
        },
      };
    });
  }

  $: {
    userId;
    $balances;
    updateTable();
  }

  onMount(updateTable);

  const outgoingTableColumns: ColumnDef<OutgoingStreamTableRow>[] = [
    {
      accessorKey: 'name',
      header: 'Name',
      cell: () => NameAndBadgeCell,
      enableSorting: false,
      size: (100 / 24) * 8,
    },
    {
      accessorKey: 'toAddress',
      header: 'To',
      cell: () => IdentityBadgeCell,
      enableSorting: false,
      size: (100 / 24) * 5,
    },
    {
      accessorKey: 'amount',
      header: 'Total streamed',
      cell: () => AmountCell,
      enableSorting: false,
      size: (100 / 24) * 5,
    },
    {
      accessorKey: 'token',
      header: 'Token',
      cell: () => TokenCell,
      enableSorting: false,
      size: (100 / 24) * 2,
    },
    {
      accessorKey: 'chevron',
      header: '',
      cell: () => ChevronRightCell,
      enableSorting: false,
      size: (100 / 24) * 2,
    },
  ];

  const incomingTableColumns: ColumnDef<OutgoingStreamTableRow>[] = [
    {
      accessorKey: 'name',
      header: 'Name',
      cell: () => NameAndBadgeCell,
      enableSorting: false,
      size: (100 / 24) * 8,
    },
    {
      accessorKey: 'fromAddress',
      header: 'From',
      cell: () => IdentityBadgeCell,
      enableSorting: false,
      size: (100 / 24) * 5,
    },
    {
      accessorKey: 'amount',
      header: 'Received',
      cell: () => AmountCell,
      enableSorting: false,
      size: (100 / 24) * 5,
    },
    {
      accessorKey: 'token',
      header: 'Token',
      cell: () => TokenCell,
      enableSorting: false,
      size: (100 / 24) * 2,
    },
    {
      accessorKey: 'chevron',
      header: '',
      cell: () => ChevronRightCell,
      enableSorting: false,
      size: (100 / 24) * 2,
    },
  ];

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  let optionsOutgoing: TableOptions<any>;
  $: optionsOutgoing = {
    data: outgoingTableData,
    columns: outgoingTableColumns,
    getCoreRowModel: getCoreRowModel(),
  };

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  let optionsIncoming: TableOptions<any>;
  $: optionsIncoming = {
    data: incomingTableData,
    columns: incomingTableColumns,
    getCoreRowModel: getCoreRowModel(),
  };

  // As soon as the given account has been fetched at least once, display content.
  let loaded = false;
  $: if (userId && ['error', 'fetched'].includes($accountFetchStatussesStore[userId]?.all ?? '')) {
    loaded = true;
  }

  $: error = Boolean(userId && $accountFetchStatussesStore[userId]?.all === 'error');
  $: empty = ownStreams.incoming.length === 0 && ownStreams.outgoing.length === 0;

  function onRowClick(
    tableData: OutgoingStreamTableRow[] | IncomingStreamTableRow[],
    event: CustomEvent<RowClickEventPayload>,
  ) {
    // go to token page by address
    const streamId = tableData[event.detail.rowIndex].streamId;
    const parsedId = decodeStreamId(streamId);

    onClickGoto(
      `/app/${parsedId.senderUserId}/tokens/${parsedId.tokenAddress}/streams/${parsedId.beamId}`,
      event.detail.event,
    );
  }
</script>

<div class="section">
  <SectionHeader
    icon={TokenStreamIcon}
    label="Streams"
    actionsDisabled={!loaded}
    actions={disableActions
      ? []
      : [
          {
            handler: () => modal.show(Stepper, undefined, createStreamFlowSteps(tokenAddress)),
            icon: PlusIcon,
            label: 'Create stream',
          },
        ]}
  />
  <div class="content">
    <SectionSkeleton
      horizontalScroll
      emptyStateHeadline="No streams"
      emptyStateText="This is where incoming and outgoing streams for your account will appear."
      {loaded}
      {error}
      {empty}
    >
      {#if optionsOutgoing.data.length > 0}
        <div class="table-container">
          <h4 class="table-group-header">↑ Outgoing</h4>
          <Table
            rowHeight={76}
            options={optionsOutgoing}
            isRowClickable
            on:rowClick={(e) => onRowClick(outgoingTableData, e)}
          />
        </div>
      {/if}
      {#if optionsIncoming.data.length > 0}
        <div class="table-container">
          <h4 class="table-group-header">↓ Incoming</h4>
          <Table
            rowHeight={76}
            options={optionsIncoming}
            isRowClickable
            on:rowClick={(e) => onRowClick(incomingTableData, e)}
          />
        </div>
      {/if}
    </SectionSkeleton>
  </div>
</div>

<style>
  .section {
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }

  .table-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .table-container:first-child:not(:last-child) {
    margin-bottom: 2rem;
  }

  .table-group-header {
    color: var(--color-foreground-level-6);
    margin-left: calc(0.75rem + 2px);
  }

  @media (max-width: 1024px) {
    .table-group-header {
      margin-left: none;
    }
  }
</style>
