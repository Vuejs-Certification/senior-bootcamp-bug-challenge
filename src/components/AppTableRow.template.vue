<script setup>
defineProps({
  row: {
    type: Object,
    required: true,
  },
  selectedRows: {
    type: Array,
    required: true,
  },
});

const emit = defineEmits(["toggle-select"]);
</script>
<template>
  <tr :class="{ 'bg-blue-50': selectedRows.includes(row) }">
    <td class="px-4 py-2 border-b">
      <input
        type="checkbox"
        :checked="selectedRows.includes(row)"
        @change="emit('toggle-select')"
        class="w-5 h-5 text-blue-600 form-checkbox"
      />
      {{ console.log("Rendered row with id:" + row.id) }}
    </td>
    <td v-for="(value, key) in row" :key="key" class="px-4 py-2 border-b">
      {{ value }}
    </td>
  </tr>
</template>
