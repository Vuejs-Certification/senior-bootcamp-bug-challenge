<script setup>
import { ref } from "vue";
import AppTable from "./components/AppTable.vue";

const headers = ["Id", "Name", "Position", "Department"];

const employees = ref([
  {
    id: 1,
    name: "John Doe",
    position: "Software Engineer",
    department: "Engineering",
  },
  {
    id: 2,
    name: "Jane Smith",
    position: "Product Manager",
    department: "Product",
  },
  {
    id: 3,
    name: "Bob Johnson",
    position: "UX Designer",
    department: "Design",
  },
  {
    id: 4,
    name: "Alice Brown",
    position: "Data Analyst",
    department: "Analytics",
  },
  {
    id: 5,
    name: "Charlie Wilson",
    position: "Marketing Specialist",
    department: "Marketing",
  },
]);

const selectedEmployees = ref([]);
</script>
<template>
  <div class="container p-4 mx-auto">
    <h1 class="mb-4 text-2xl font-bold">Employee Table</h1>
    <AppTable
      :headers="headers"
      :rows="employees"
      v-model="selectedEmployees"
    />
    <div class="mt-4">
      <h2 class="mb-2 text-xl font-semibold">Selected Employees:</h2>
      <ul class="list-disc list-inside">
        <li v-for="employee in selectedEmployees" :key="employee.id">
          {{ employee.name }} ({{ employee.position }})
        </li>
      </ul>
    </div>
  </div>
</template>
