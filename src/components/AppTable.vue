<script setup>
import { computed, ref, watch } from "vue";
import TableRow from "./AppTableRow.vue";

const props = defineProps({
  headers: {
    type: Array,
    required: true,
  },
  rows: {
    type: Array,
    required: true,
  },
  modelValue: {
    type: Array,
    default: () => [],
  },
});

const emit = defineEmits(["update:modelValue"]);

const selectedRows = ref(props.modelValue);

const allRowsSelected = computed(() => {
  return (
    props.rows.length > 0 && selectedRows.value.length === props.rows.length
  );
});

const isRowSelected = (row) => {
  return selectedRows.value.includes(row);
};

const toggleRowSelection = (row) => {
  const index = selectedRows.value.indexOf(row);
  if (index === -1) {
    selectedRows.value.push(row);
  } else {
    selectedRows.value.splice(index, 1);
  }
  emit("update:modelValue", selectedRows.value);
};

const toggleAllRows = () => {
  if (allRowsSelected.value) {
    selectedRows.value = [];
  } else {
    selectedRows.value = [...props.rows];
  }
  emit("update:modelValue", selectedRows.value);
};

watch(
  () => props.modelValue,
  (newValue) => {
    selectedRows.value = newValue;
  }
);
</script>
<template>
  <div class="overflow-x-auto">
    <table class="min-w-full bg-white border border-gray-300">
      <thead>
        <tr class="bg-gray-100">
          <th class="px-4 py-2 border-b">
            <input
              type="checkbox"
              @change="toggleAllRows"
              :checked="allRowsSelected"
              class="w-5 h-5 text-blue-600 form-checkbox"
            />
          </th>
          <th
            v-for="header in headers"
            :key="header"
            class="px-4 py-2 font-semibold text-left text-gray-700 border-b"
          >
            {{ header }}
          </th>
        </tr>
      </thead>
      <tbody>
        <TableRow
          v-for="(row, index) in rows"
          :key="index"
          :row="row"
          :isSelected="modelValue.includes(row)"
          @toggle-select="toggleRowSelection(row)"
        />
      </tbody>
    </table>
  </div>
</template>
