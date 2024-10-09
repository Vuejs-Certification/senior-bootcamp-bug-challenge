<script setup>
defineProps({
  row: {
    type: Object,
    required: true,
  },
  isSelected: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(["toggle-select"]);
</script>
<template>
  <tr :class="{ 'bg-blue-50': isSelected }">
    <td class="px-4 py-2 border-b">
      <input
        type="checkbox"
        :checked="isSelected"
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
