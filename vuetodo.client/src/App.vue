<script setup>
import { ref, watch, onMounted, onBeforeUnmount, computed } from "vue";
import TaskChart from "./components/TaskChart.vue";

const tasks = ref([]);
const newTask = ref("");
const editedTask = ref(null);
const dragItemIndex = ref(null);
const itemsPerPage = ref(5); // Кількість завдань на сторінці
const currentPage = ref(1); // Поточна сторінка

// Завантажуємо завдання з localStorage при завантаженні сторінки
onMounted(() => {
  loadTasksFromLocalStorage();
  window.addEventListener("storage", syncTasksAcrossTabs);
});

onBeforeUnmount(() => {
  window.removeEventListener("storage", syncTasksAcrossTabs);
});

function loadTasksFromLocalStorage() {
  const savedTasks = JSON.parse(localStorage.getItem("tasks"));
  if (savedTasks) {
    tasks.value = savedTasks;
  }
}

function syncTasksAcrossTabs(event) {
  if (event.key === "tasks") {
    tasks.value = JSON.parse(event.newValue) || [];
  }
}

watch(
  tasks,
  (newTasks) => {
    localStorage.setItem("tasks", JSON.stringify(newTasks));
  },
  { deep: true }
);

function addTask() {
  if (newTask.value.trim()) {
    tasks.value.push({ text: newTask.value, completed: false });
    newTask.value = "";
  }
}

function deleteTask(index) {
  tasks.value.splice(index, 1);
}

function toggleTaskCompletion(index) {
  tasks.value[index].completed = !tasks.value[index].completed;
}

function editTask(index) {
  editedTask.value = { ...tasks.value[index], index };
}

function saveEditedTask() {
  if (editedTask.value.text.trim()) {
    tasks.value[editedTask.value.index].text = editedTask.value.text;
    editedTask.value = null;
  }
}
const filteredTasks = computed(() => {
  if (newTask.value.trim() === "") return [];
  return tasks.value.filter((task) =>
    task.text.toLowerCase().includes(newTask.value.toLowerCase())
  );
});
function handleDragStart(index) {
  dragItemIndex.value = index;
}

function handleDragOver(event) {
  event.preventDefault();
}

function handleDrop(index) {
  if (dragItemIndex.value !== null) {
    const draggedItem = tasks.value[dragItemIndex.value];
    tasks.value.splice(dragItemIndex.value, 1);
    tasks.value.splice(index, 0, draggedItem);
    dragItemIndex.value = null;
  }
}

// Обчислення кількості сторінок
const totalPages = computed(() =>
  Math.ceil(tasks.value.length / itemsPerPage.value)
);

// Переходить на попередню сторінку
function prevPage() {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
}

// Переходить на наступну сторінку
function nextPage() {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
}
function selectTask(task) {
  newTask.value = task.text;
  filteredTasks.value = []; // Закриваємо дропдаун після вибору
}

// Відображення завдань на поточній сторінці
const paginatedTasks = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return tasks.value.slice(start, end);
});
const completedTasks = computed(
  () => tasks.value.filter((task) => task.completed).length
);
const pendingTasks = computed(() => tasks.value.length - completedTasks.value);
</script>

<template>
  <div class="todo-app">
    <header>
      <img
        alt="Vue logo"
        class="logo"
        src="./assets/logo.svg"
        width="125"
        height="125"
      />
      <div class="wrapper"></div>
      <TaskChart
        :completedTasks="completedTasks"
        :pendingTasks="pendingTasks"
      />
    </header>
    <h1>To-Do List</h1>

    <div class="inputbuttonWrapper">
      <input
        type="text"
        v-model="newTask"
        placeholder="Введіть нове завдання"
        @keyup.enter="addTask"
      />
      <button @click="addTask">Додати завдання</button>
      <ul v-if="filteredTasks.length > 0" class="dropdown">
        <li
          v-for="(task, index) in filteredTasks"
          :key="index"
          @click="selectTask(task)"
        >
          {{ task.text }}
        </li>
      </ul>
    </div>

    <!-- Список завдань з пагінацією -->
    <ul>
      <li
        v-for="(task, index) in paginatedTasks"
        :key="index"
        :class="{ completed: task.completed }"
        draggable="true"
        @dragstart="handleDragStart(index)"
        @dragover="handleDragOver"
        @drop="handleDrop(index)"
      >
        <div class="DiWrapper" v-if="editedTask && editedTask.index === index">
          <input v-model="editedTask.text" @keyup.enter="saveEditedTask" />
          <button @click="saveEditedTask">Зберегти</button>
        </div>
        <div class="DiWrapper" v-else>
          <input
            type="checkbox"
            :checked="task.completed"
            @change="toggleTaskCompletion(index)"
          />
          <span @dblclick="editTask(index)">{{ task.text }}</span>
          <button @click="deleteTask(index)">Видалити</button>
        </div>
      </li>
    </ul>

    <!-- Кнопки пагінації -->
    <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 1">Попередня</button>
      <span>Сторінка {{ currentPage }} з {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">
        Наступна
      </button>
    </div>
  </div>
</template>

<style scoped>
.todo-app {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: Arial, sans-serif;
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  width: auto;
  margin: 0 auto;
}
.DiWrapper {
  display: flex;
  width: 100%;
}
header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

h1 {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  margin-bottom: 20px;
}

input[type="text"] {
  padding: 10px;
  width: 100%;

  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  box-sizing: border-box;
}

button {
  padding: 10px 15px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  color: #fff;
  background-color: #42b983;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #3aa374;
}

ul {
  list-style-type: none;
  padding: 0;
  width: 100%;
}

.todo-app ul > li {
  width: 500px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #dddddd;
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s ease;
  color: black;
}

li:hover {
  transform: translateY(-2px);
}

li.completed span {
  text-decoration: line-through;
  color: #aaa;
}

input[type="checkbox"] {
  margin-right: 10px;
  accent-color: #42b983;
  cursor: pointer;
}

span {
  font-size: 16px;
  flex: 1;
  text-align: center;
}

span:hover {
  cursor: pointer;
  color: #42b983;
}

button {
  background-color: #d9534f;
}

button:hover {
  background-color: #c9302c;
}

input[type="text"]:focus,
button:focus {
  outline: none;
  border-color: #42b983;
}
.inputbuttonWrapper {
  display: flex;
  gap: 40px;
  margin-bottom: 40px;
  position: relative;
}
.pagination {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
}
span {
  color: black;
}
.dropdown {
  position: absolute;
  top: 56px;
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100%;
  max-height: 450px;
  overflow-y: auto;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
  z-index: 10;
}

.dropdown li {
  padding: 10px;
  cursor: pointer;
  color: black;
  width: 100% !important;
}

.dropdown li:hover {
  background-color: #f0f0f0;
}
</style>
