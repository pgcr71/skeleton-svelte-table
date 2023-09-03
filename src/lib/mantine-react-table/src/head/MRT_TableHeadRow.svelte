<script lang="ts">
import { MRT_TableHeadCell } from './MRT_TableHeadCell';
import { TableHead } from 'flowbite-svelte';
// import type {
//   MRT_Header,
//   MRT_HeaderGroup,
//   MRT_TableInstance,
//   MRT_VirtualItem,
// } from '..';

export let headerGroup;
export let table;
export let virtualColumns;
export let virtualPaddingLeft;
export let virtualPaddingRight;

// interface Props {
//   headerGroup: MRT_HeaderGroup;
//   table: MRT_TableInstance;
//   virtualColumns?: MRT_VirtualItem[];
//   virtualPaddingLeft?: number;
//   virtualPaddingRight?: number;
// }

const {
    getState,
    options: { enableStickyHeader, layoutMode, mantineTableHeadRowProps },
  } = table;
  const { isFullScreen } = getState();

  const tableRowProps =
    mantineTableHeadRowProps instanceof Function
      ? mantineTableHeadRowProps({ headerGroup, table })
      : mantineTableHeadRowProps;

  const stickyHeader = enableStickyHeader || isFullScreen;
</script>


    <TableHead
      component="tr"
      {...tableRowProps}
      sx={(theme) => ({
        backgroundColor:
          theme.colorScheme === 'dark' ? theme.colors.dark[7] : theme.white,
        boxShadow: `4px 0 8px ${theme.fn.rgba(theme.black, 0.1)}`,
        display: layoutMode === 'grid' ? 'flex' : 'table-row',
        top: stickyHeader ? 0 : undefined,
        ...(tableRowProps?.sx instanceof Function
          ? tableRowProps?.sx(theme)
          : (tableRowProps?.sx)),
        position: stickyHeader ? 'sticky' : undefined,
      })}
    >
    {#if virtualPaddingLeft}
    <th style= "display: 'flex'; width: {virtualPaddingLeft}"  />
    {/if}
  {@const  temp = virtualColumns ?? headerGroup.headers}
    {#each (temp) as headerOrVirtualHeader}
   
    {#if virtualColumns}
    {@const header =  headerGroup.headers[headerOrVirtualHeader.index]}
    <MRT_TableHeadCell header={header} key={header.id} table={table} />
    {:else}
    {@const header = headerOrVirtualHeader}
    <MRT_TableHeadCell header={header} key={header.id} table={table} />
    {/if}
 
    {/each}
      {#if virtualPaddingRight}
      <th style="display: 'flex'; width: {virtualPaddingRight}" />
      {/if}
 
    </TableHead>

