
<script lang="ts">
	import type { MRT_Cell, MRT_TableInstance, TData } from "../types";
  

  export let cell:  MRT_Cell<TData>;
  export let showLabel: boolean;
  export let table: MRT_TableInstance<TData>;

  const {
    getState,
    options: { mantineEditTextInputProps },
    refs: { editInputRefs },
    setEditingCell,
    setEditingRow,
  } = table;
  const { column, row } = cell;
  const { columnDef } = column;
  const { editingRow } = getState();

  let value=cell.getValue<string>();

  const mTableBodyCellEditTextInputProps =
    mantineEditTextInputProps instanceof Function
      ? mantineEditTextInputProps({ cell, column, row, table })
      : mantineEditTextInputProps;

  const mcTableBodyCellEditTextInputProps =
    columnDef.mantineEditTextInputProps instanceof Function
      ? columnDef.mantineEditTextInputProps({
          cell,
          column,
          row,
          table,
        })
      : columnDef.mantineEditTextInputProps;

  const textFieldProps = {
    ...mTableBodyCellEditTextInputProps,
    ...mcTableBodyCellEditTextInputProps,
  };

  const saveRow = (newValue: string) => {
    if (editingRow) {
      setEditingRow({
        ...editingRow,
        _valuesCache: { ...editingRow._valuesCache, [column.id]: newValue },
      });
    }
  };

  const handleChange = (event:  Event) => {
    textFieldProps.onChange?.(event);
    value = (event.target  as HTMLInputElement).value;
  };

  const handleBlur = (event: Event) => {
    textFieldProps.onBlur?.(event);
    saveRow(value);
    setEditingCell(null);
  };

  const handleEnterKeyDown = (event: Event) => {
    if ((event as KeyboardEvent).key === 'Enter') {
      editInputRefs.current[column.id]?.blur();
    }
  };

</script>


{#if columnDef.Edit}
{columnDef.Edit?.({ cell, column, row, table })}
{:else}
<input
disabled={
  (columnDef.enableEditing instanceof Function
    ? columnDef.enableEditing(row)
    : columnDef.enableEditing) === false
}
label={showLabel ? column.columnDef.header : undefined}
name={column.id}
on:keydown={handleEnterKeyDown}
placeholder={columnDef.header}
value={value}
variant="default"
{...textFieldProps}
on:click={(e) => {
  e.stopPropagation();
  textFieldProps?.onClick?.(e);
}}
on:blur={handleBlur}
on:change={handleChange}
/>
{/if}


  

