

<script lang="ts">
import React, { useCallback, useMemo } from 'react';
import {
  defaultRangeExtractor,
  Range,
  useVirtualizer,
} from '@tanstack/svelte-virtual';
import { Table, TableBody, TableBodyCell, TableBodyRow, TableHead, TableHeadCell, Checkbox, TableSearch } from 'flowbite-svelte';
import { MRT_TableHead } from '../head/MRT_TableHead';
import { Memo_MRT_TableBody, MRT_TableBody } from '../body/MRT_TableBody';
import { MRT_TableFooter } from '../footer/MRT_TableFooter';
import { parseCSSVarId } from '../column.utils';
import type { MRT_TableInstance, MRT_Virtualizer } from '..';

export let table

const {
    getFlatHeaders,
    getState,
    options: {
      columns,
      columnVirtualizerInstanceRef,
      columnVirtualizerProps,
      enableColumnResizing,
      enableColumnVirtualization,
      enablePinning,
      enableTableFooter,
      enableTableHead,
      layoutMode,
      memoMode,
      mantineTableProps,
    },
    refs: { tableContainerRef },
  } = table;

  interface Props {
  table: MRT_TableInstance;
}

const {
    columnSizing,
    columnSizingInfo,
    columnPinning,
    columnVisibility,
    density,
  } = getState();



  const tableProps =
    mantineTableProps instanceof Function
      ? mantineTableProps({ table })
      : mantineTableProps;

  const vProps =
    columnVirtualizerProps instanceof Function
      ? columnVirtualizerProps({ table })
      : columnVirtualizerProps;

  const columnSizeVars = useMemo(() => {
    const headers = getFlatHeaders();
    const colSizes: { [key: string]: number } = {};
    for (let i = 0; i < headers.length; i++) {
      const header = headers[i];
      const colSize = header.getSize();
      colSizes[`--header-${parseCSSVarId(header.id)}-size`] = colSize;
      colSizes[`--col-${parseCSSVarId(header.column.id)}-size`] = colSize;
    }
    return colSizes;
  }, [columns, columnSizing, columnSizingInfo, columnVisibility]);

  //get first 16 column widths and average them
  const averageColumnWidth = useMemo(() => {
    if (!enableColumnVirtualization) return 0;
    const columnsWidths =
      table
        .getRowModel()
        .rows[0]?.getCenterVisibleCells()
        ?.slice(0, 16)
        ?.map((cell) => cell.column.getSize() * 1.2) ?? [];
    return columnsWidths.reduce((a, b) => a + b, 0) / columnsWidths.length;
  }, [table.getRowModel().rows, columnPinning, columnVisibility]);

  const [leftPinnedIndexes, rightPinnedIndexes] = useMemo(
    () =>
      enableColumnVirtualization && enablePinning
        ? [
            table.getLeftLeafColumns().map((c) => c.getPinnedIndex()),
            table
              .getRightLeafColumns()
              .map(
                (c) =>
                  table.getVisibleLeafColumns().length - c.getPinnedIndex() - 1,
              ),
          ]
        : [[], []],
    [columnPinning, enableColumnVirtualization, enablePinning],
  );

  const columnVirtualizer:
    | MRT_Virtualizer<HTMLDivElement, HTMLTableCellElement>
    | undefined = enableColumnVirtualization
    ? useVirtualizer({
        count: table.getVisibleLeafColumns().length,
        estimateSize: () => averageColumnWidth,
        getScrollElement: () => tableContainerRef.current,
        horizontal: true,
        overscan: 3,
        rangeExtractor: useCallback(
          (range: Range) => [
            ...new Set([
              ...leftPinnedIndexes,
              ...defaultRangeExtractor(range),
              ...rightPinnedIndexes,
            ]),
          ],
          [leftPinnedIndexes, rightPinnedIndexes],
        ),
        ...vProps,
      })
    : undefined;

  if (columnVirtualizerInstanceRef && columnVirtualizer) {
    columnVirtualizerInstanceRef.current = columnVirtualizer;
  }

  const virtualColumns = columnVirtualizer
    ? columnVirtualizer.getVirtualItems()
    : undefined;

  let virtualPaddingLeft: number | undefined;
  let virtualPaddingRight: number | undefined;

  if (columnVirtualizer && virtualColumns?.length) {
    virtualPaddingLeft = virtualColumns[leftPinnedIndexes.length]?.start ?? 0;
    virtualPaddingRight =
      columnVirtualizer.getTotalSize() -
      (virtualColumns[virtualColumns.length - 1 - rightPinnedIndexes.length]
        ?.end ?? 0);
  }

  const props = {
    enableHover: tableProps?.highlightOnHover,
    table,
    virtualColumns,
    virtualPaddingLeft,
    virtualPaddingRight,
  };

</script>



    <Table
      highlightOnHover
      horizontalSpacing={density}
      verticalSpacing={density}
      {...tableProps}
      sx={(theme) => ({
        display: layoutMode === 'grid' ? 'grid' : 'table',
        tableLayout:
          layoutMode !== 'grid' && enableColumnResizing ? 'fixed' : undefined,
        '& tr:first-of-type td': {
          borderTop: `1px solid ${
            theme.colors.gray[theme.colorScheme === 'dark' ? 8 : 3]
          }`,
        },
        '& tr:last-of-type td': {
          borderBottom: `1px solid ${
            theme.colors.gray[theme.colorScheme === 'dark' ? 8 : 3]
          }`,
        },
        ...(tableProps?.sx instanceof Function
          ? tableProps.sx(theme)
          : (tableProps?.sx)),
      })}
      style={{ ...columnSizeVars, ...tableProps?.style }}
    >
      {enableTableHead && <MRT_TableHead {...props} />}
      {memoMode === 'table-body' || columnSizingInfo.isResizingColumn ? (
        <Memo_MRT_TableBody columnVirtualizer={columnVirtualizer} {...props} />
      ) : (
        <MRT_TableBody columnVirtualizer={columnVirtualizer} {...props} />
      )}
      {enableTableFooter && <MRT_TableFooter {...props} />}
    </Table>

