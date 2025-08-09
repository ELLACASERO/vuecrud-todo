<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue'

// ─── State ──────────────────────────────────────────────
const todos = ref([])
const name = ref('')
const input_content = ref('')
const searchQuery = ref('')

// ─── Lifecycle ──────────────────────────────────────────
onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})

// ─── Watchers ───────────────────────────────────────────
watch(name, newVal => {
  localStorage.setItem('name', newVal)
})

watch(
  todos,
  newVal => {
    localStorage.setItem('todos', JSON.stringify(newVal))
  },
  { deep: true }
)

// ─── Computed ───────────────────────────────────────────
const todos_asc = computed(() => {
  return todos.value.sort((a, b) => a.createdAt - b.createdAt)
})

const filteredTodos = computed(() => {
  return todos_asc.value.filter(todo =>
    todo.content.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

// ─── Methods ────────────────────────────────────────────
const addTodo = () => {
  if (input_content.value.trim() === '') return

  todos.value.push({
    content: input_content.value,
    done: false,
    createdAt: Date.now(),
    editable: false,
  })

  input_content.value = ''
}

const toggleDone = todo => {
  todo.done = !todo.done
}

const editTodo = todo => {
  todo.editable = !todo.editable

  if (todo.editable) {
    nextTick(() => {
      const inputEl = document.querySelector(`input[data-id="${todo.createdAt}"]`)
      if (inputEl) inputEl.focus()
    })
  }
}

const deleteTodo = todo => {
  todos.value = todos.value.filter(t => t !== todo)
}
</script>

<template>
  <main class="app">
    <!-- Greeting -->
    <section class="greeting">
      <h2 class="title">
        Hello,
        <input type="text" v-model="name" placeholder="Your Name" />
      </h2>
    </section>

    <!-- Create Todo -->
    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <form @submit.prevent="addTodo">
        <input
          type="text"
          placeholder="What’s your task?"
          v-model="input_content"
        />
        <input type="submit" value="Add Todo" />
      </form>
    </section>

    <!-- Todo List -->
    <section class="todo-list">
      <h3>TODO LIST</h3>

      <!-- Search Bar -->
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Search todos..."
        class="search-input"
      />

      <div class="list">
        <div
          v-for="todo in filteredTodos"
          :key="todo.createdAt"
          class="todo-item"
          :class="{ done: todo.done }"
        >
          <label>
            <input
              type="checkbox"
              v-model="todo.done"
              @change="toggleDone(todo)"
            />
          </label>

          <div class="todo-content">
            <input
              type="text"
              v-model="todo.content"
              :readonly="!todo.editable || todo.done"
              :data-id="todo.createdAt"
              :class="{ editable: todo.editable }"
            />
            <span v-if="todo.done" class="completed-label">✅ Completed</span>
          </div>

          <div class="actions">
            <button class="edit" @click="editTodo(todo)">
              {{ todo.editable ? 'Save' : 'Edit' }}
            </button>
            <button class="delete" @click="deleteTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.app {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  font-family: sans-serif;
}

/* Greeting Input */
.title input {
  font-size: 1.2rem;
  padding: 5px;
  margin-left: 10px;
}

/* Create Todo Form */
.create-todo input[type="text"] {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
}

.create-todo input[type="submit"] {
  padding: 10px 20px;
}

/* Search Input */
.search-input {
  width: 100%;
  padding: 8px;
  margin: 10px 0 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* Todo List */
.todo-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.todo-content {
  flex-grow: 1;
  margin-left: 10px;
}

.todo-content input {
  width: 100%;
  padding: 5px;
}

.todo-content input.editable {
  border: 1px solid #aaa;
  background: #fff;
}

/* Checkbox */
.todo-item input[type="checkbox"] {
  width: 18px;
  height: 18px;
  accent-color: #007bff;
  margin-right: 8px;
  cursor: pointer;
}

/* Mark as done */
.todo-item.done input[type="text"] {
  text-decoration: line-through;
  color: gray;
}

.completed-label {
  color: green;
  font-weight: bold;
  margin-left: 10px;
  font-size: 0.9rem;
}

/* Action Buttons */
.actions button {
  margin-left: 5px;
  padding: 5px 10px;
}
</style>
