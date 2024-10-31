<template>
  <div class="todo-app">
    <h1>Мій To-Do Список</h1>

    <input
      v-model="newTask"
      @keyup.enter="addTask"
      placeholder="Додати нове завдання..."
    />

    <ul class="todo-list">
      <ToDoItem
        v-for="(task, index) in tasks"
        :key="task.id"
        :task="task"
        :index="index"
        @remove="removeTask"
        @update="updateTask"
      />
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from "vue";
import ToDoItem from "./ToDoItem.vue";

const newTask = ref("");
const tasks = ref([]);

onMounted(() => {
  const savedTasks = JSON.parse(localStorage.getItem("tasks"));
  if (savedTasks) {
    tasks.value = savedTasks;
  }
});

watch(
  tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
  },
  { deep: true }
);

const addTask = () => {
  if (newTask.value.trim() !== "") {
    tasks.value.push({
      id: Date.now(),
      text: newTask.value,
      done: false,
    });
    newTask.value = "";
  }
};

const removeTask = (id) => {
  tasks.value = tasks.value.filter((task) => task.id !== id);
};

const updateTask = (updatedTask) => {
  const index = tasks.value.findIndex((task) => task.id === updatedTask.id);
  if (index !== -1) {
    tasks.value[index] = updatedTask;
  }
};
</script>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 1rem;
}

input {
  width: 100%;
  padding: 8px;
  margin-bottom: 1rem;
}

.todo-list {
  list-style-type: none;
  padding: 0;
}
</style>
