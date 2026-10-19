<script setup>
import { ref, computed, watch } from 'vue'

const STORAGE_KEY = 'vue-task-manager-tasks'

function loadTasks() {
  try {
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEY))
    return Array.isArray(saved) ? saved : []
  } catch {
    return []
  }
}

const tasks = ref(loadTasks())
const newTask = ref('')
const filter = ref('All')
const filters = ['All', 'Active', 'Completed']

const filteredTasks = computed(() => {
  if (filter.value === 'Active') {
    return tasks.value.filter(task => !task.completed)
  }
  if (filter.value === 'Completed') {
    return tasks.value.filter(task => task.completed)
  }
  return tasks.value
})

const activeCount = computed(() => {
  return tasks.value.filter(task => !task.completed).length
})

const completedCount = computed(() => {
  return tasks.value.filter(tasks => tasks.completed).length
})

function addTask() {
  const title = newTask.value.trim()
  if (!title) return

  tasks.value.push({ id: crypto.randomUUID(), title, completed: false })
  newTask.value = ''
}

function removeTask(id) {
  tasks.value = tasks.value.filter(task => task.id !== id)
}

function clearCompleted() {
  tasks.value = tasks.value.filter(task => !task.completed)
}

watch(tasks, (updatedTasks) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(updatedTasks))
}, { deep: true })
</script>

<template>
  <main>
    <h1>Task Manager</h1>

    <form @submit.prevent="addTask">
      <label for="new-task">Новая задача</label>
      <input id="new-task" v-model="newTask" placeholder="Что нужно сделать?" />
      <button type="submit">Добавить</button>
    </form>

    <div class="filters" role="group" aria-label="Фильтр задач">
      <button
        v-for="option in filters"
        :key="option"
        type="button"
        :aria-pressed="filter === option"
        :disabled="filter === option"
        @click="filter = option"
      >
        {{ option }}
      </button>
    </div>

    <p>Активных задач: {{ activeCount }}</p>
    <p>Выполнено задач: {{ completedCount }}</p>
    <p v-if="filteredTasks.length === 0">Нет задач для этого фильтра.</p>
    <ul v-else>
      <li v-for="task in filteredTasks" :key="task.id">
        <label>
          <input v-model="task.completed" type="checkbox" />
          {{ task.title }}
        </label>
        <button type="button" @click="removeTask(task.id)">Удалить</button>
      </li>
    </ul>

    <button type="button" @click="clearCompleted" :disabled="completedCount === 0"> Удалить выполненные задачи</button>
  </main>
</template>

<style scoped>
main {
  max-width: 600px;
  margin: 24px auto;
  padding: 0 16px;
}

form,
.filters,
li {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filters {
  margin-top: 16px;
}

ul {
  padding: 0;
  list-style: none;
}

li {
  margin-bottom: 8px;
}
</style>
