<template>
  <li
    class="todo-item"
    :class="{ done: task.done }"
    draggable="true"
    @dragstart="onDragStart"
    @dragend="onDragEnd"
  >
    <input type="checkbox" v-model="task.done" @change="updateTask" />
    <span
      :contenteditable="isEditing"
      @blur="finishEdit"
      @keypress.enter.prevent="finishEdit"
      >{{ task.text }}</span
    >

    <button @click="isEditing = true">✏️</button>
    <button @click="removeTask">❌</button>
  </li>
</template>

<script setup>
import { ref } from "vue";
import { emit } from "vue";

const props = defineProps(["task"]);
const isEditing = ref(false);

const removeTask = () => {
  emit("remove", props.task.id);
};

const finishEdit = (event) => {
  props.task.text = event.target.textContent;
  isEditing.value = false;
  emit("update", props.task);
};

const updateTask = () => {
  emit("update", props.task);
};

const onDragStart = (event) => {
  event.dataTransfer.setData("text/plain", props.task.id);
};
const onDragEnd = (event) => {
  event.preventDefault();
  emit("drag-end", props.task);
};
</script>

<style scoped>
.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px;
  border: 1px solid #ddd;
  margin-bottom: 8px;
}

.done span {
  text-decoration: line-through;
  color: #888;
}
</style>
