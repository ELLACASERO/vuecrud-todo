<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue'

const todos = ref([])
const name = ref('')

const input_content = ref('')

const todos_asc = computed(() => todos.value.sort((a, b) => {
  return a.createdAt - b.createdAt
}))

watch(name, (newVal) => {
  localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, {
  deep: true
})

const addTodo = () => {
  if (input_content.value.trim() === '') {
    return
  }

  todos.value.push({
    content: input_content.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime()
  })
  input_content.value = ''
}

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo)
}

const editTodo = (todo) => {
  todo.editable = true
  nextTick(() => {
    const inputEl = document.querySelector(`input[data-id="${todo.createdAt}"]`)
    if (inputEl) inputEl.focus()
  })
}

const saveTodo = (todo) => {
  todo.editable = false
}

const toggleEditSave = (todo) => {
  if (todo.editable) {
    saveTodo(todo)
  } else {
    editTodo(todo)
  }
}

onMounted(() => {
  name.value = localStorage.getItem('name') || ''
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up,
        <input type="text" id="name" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>

      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="e.g. make a video"
          v-model="input_content"
        />
        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="list" id="todo-list">
        <div
          v-for="todo in todos_asc"
          :key="todo.createdAt"
          :class="['todo-item', { done: todo.done }]"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
          </label>

          <div class="todo-content">
            <input
              type="text"
              v-model="todo.content"
              :readonly="!todo.editable"
              :class="{ editable: todo.editable }"
              :data-id="todo.createdAt"
            />
          </div>

          <div class="actions">
            <button class="edit" @click="toggleEditSave(todo)">
  {{ todo.editable ? 'Save' : 'Edit' }}
</button>

            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
