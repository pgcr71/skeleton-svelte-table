<script lang="ts">

import React from 'react';
import { ActionIcon, Indicator, Tooltip } from '@svelteuidev/core';
import type { MRT_Header, MRT_TableInstance } from '..';

interface Props {
  header: MRT_Header;
  table: MRT_TableInstance;
}

export let table;
export let header;


  const {
    getState,
    options: {
      icons: { IconSortDescending, IconSortAscending, IconArrowsSort },
      localization,
    },
  } = table;
  const { column } = header;
  const { columnDef } = column;
  const { sorting } = getState();

  const sortTooltip = column.getIsSorted()
    ? column.getIsSorted() === 'desc'
      ? localization.sortedByColumnDesc.replace('{column}', columnDef.header)
      : localization.sortedByColumnAsc.replace('{column}', columnDef.header)
    : localization.unsorted;

  const showIndicator = sorting.length >= 2 && column.getSortIndex() !== -1;
</script>

    <Tooltip withinPortal withArrow position="top" label={sortTooltip}>
      <Indicator
        color="transparent"
        disabled={!showIndicator}
        inline
        label={column.getSortIndex() + 1}
        offset={3}
      >
        <ActionIcon
          aria-label={sortTooltip}
          size="xs"
          sx={{
            opacity: column.getIsSorted() ? 1 : 0,
            transform: showIndicator
              ? 'translate(-2px, 2px) scale(0.9)'
              : undefined,
            transition: 'opacity 100ms ease-in-out',
            '&:hover': {
              opacity: 1,
            },
          }}
        >
        {#if column.getIsSorted() === 'desc'}
        <IconSortDescending />
        {:else if column.getIsSorted() === 'asc'}
        <IconSortAscending />
        {:else}
        <IconArrowsSort />
        {/if}
 
        </ActionIcon>
      </Indicator>
    </Tooltip>
