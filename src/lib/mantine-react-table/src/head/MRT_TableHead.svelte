<script lang="ts">


// import type { MRT_TableInstance, MRT_VirtualItem } from '..';
import { TableHead } from 'flowbite-svelte';
import { MRT_TableHeadRow } from './MRT_TableHeadRow.svelte';

// interface Props {
//   table: MRT_TableInstance;
//   virtualColumns?: MRT_VirtualItem[];
//   virtualPaddingLeft?: number;
//   virtualPaddingRight?: number;
// }

export let table;
export let virtualColumns;
export let virtualPaddingLeft;
export let virtualPaddingRight;
const {
    getHeaderGroups,
    getState,
    options: { enableStickyHeader, layoutMode, mantineTableHeadProps },
  } = table;
  const { isFullScreen } = getState();

  const tableHeadProps =
    mantineTableHeadProps instanceof Function
      ? mantineTableHeadProps({ table })
      : mantineTableHeadProps;

  const stickyHeader = enableStickyHeader || isFullScreen;
</script>

    <TableHead
    
      {...tableHeadProps}


      sx={(theme) => ({
        display: layoutMode === 'grid' ? 'grid' : 'table-row-group',
        opacity: 0.97,
        position: stickyHeader && layoutMode === 'grid' ? 'sticky' : 'relative',
        top: stickyHeader ? 0 : undefined,
        zIndex: stickyHeader ? 2 : undefined,
        ...(tableHeadProps?.sx instanceof Function
          ? tableHeadProps?.sx(theme)
          : (tableHeadProps?.sx)),
      })}
    >
    {#each  getHeaderGroups() as headerGroup}

    <MRT_TableHeadRow
    headerGroup={headerGroup}
    table={table}
    virtualColumns={virtualColumns}
    virtualPaddingLeft={virtualPaddingLeft}
    virtualPaddingRight={virtualPaddingRight}
  />
    {/each}

    </TableHead>

