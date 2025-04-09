<script setup>
import { ref, watch, computed } from 'vue'
import { TrashIcon, PencilIcon } from '@heroicons/vue/24/outline'

const props = defineProps({
  project: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['update:todos'])

const todos = ref(props.project.todos)
const newTodoText = ref('')
const editingTodo = ref(null)
const sortBy = ref('priority') // 'priority' or 'name'
const newPriority = ref('low')

const priorities = {
  high: 3,
  medium: 2,
  low: 1
}

watch(() => props.project, (newProject) => {
  todos.value = newProject.todos
}, { deep: true })

const sortedTodos = computed(() => {
  return [...todos.value].sort((a, b) => {
    if (a.completed !== b.completed) {
      return a.completed ? 1 : -1
    }

    if (sortBy.value === 'priority') {
      const priorityA = priorities[a.priority] || 0
      const priorityB = priorities[b.priority] || 0
      return priorityB - priorityA
    } else {
      return a.text.localeCompare(b.text)
    }
  })
})

const addTodo = () => {
  if (newTodoText.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodoText.value,
      completed: false,
      priority: newPriority.value
    })
    newTodoText.value = ''
    newPriority.value = 'low'
    emit('update:todos', todos.value)
  }
}

const deleteTodo = (todoId) => {
  todos.value = todos.value.filter(t => t.id !== todoId)
  emit('update:todos', todos.value)
}

const toggleTodo = (todo) => {
  todo.completed = !todo.completed
  emit('update:todos', todos.value)
}

const startEdit = (todo) => {
  editingTodo.value = {
    id: todo.id,
    text: todo.text,
    priority: todo.priority
  }
}

const saveEdit = () => {
  if (editingTodo.value) {
    const todo = todos.value.find(t => t.id === editingTodo.value.id)
    if (todo) {
      todo.text = editingTodo.value.text
      todo.priority = editingTodo.value.priority
      emit('update:todos', todos.value)
    }
    editingTodo.value = null
  }
}

const discardEdit = () => {
  editingTodo.value = null
}

const getPriorityColor = (priority) => {
  const colors = {
    high: 'text-red-500',
    medium: 'text-yellow-500',
    low: 'text-blue-500'
  }
  return colors[priority]
}
</script>

<template>
  <div class="bg-white rounded-lg shadow p-6">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-semibold text-gray-700">
        {{ project.name }} - Tasks
      </h2>
      <select v-model="sortBy" class="input">
        <option value="priority">Sort by Priority</option>
        <option value="name">Sort by Name</option>
      </select>
    </div>

    <!-- Add Todo Form -->
    <div class="flex gap-2 mb-6">
      <input v-model="newTodoText" type="text" placeholder="Add a new task" class="input flex-1" @keyup.enter="addTodo">
      <select v-model="newPriority" class="input w-32">
        <option value="high">High</option>
        <option value="medium">Medium</option>
        <option value="low">Low</option>
      </select>
      <button @click="addTodo" class="btn btn-primary">
        Add New Task
      </button>
    </div>

    <!-- Todo List -->
    <div class="space-y-3">
      <div v-for="todo in sortedTodos" :key="todo.id"
        class="flex items-center gap-3 p-3 rounded-lg border border-gray-200">
        <input type="checkbox" :checked="todo.completed" @change="toggleTodo(todo)"
          class="h-5 w-5 rounded border-gray-300 text-primary focus:ring-primary">

        <div v-if="editingTodo?.id === todo.id" class="flex-1 flex gap-2">
          <input v-model="editingTodo.text" type="text" class="input flex-1">
          <select v-model="editingTodo.priority" class="input w-32">
            <option value="high">High</option>
            <option value="medium">Medium</option>
            <option value="low">Low</option>
          </select>
          <div class="flex gap-2">
            <button @click="saveEdit" class="btn btn-primary">
              Save
            </button>
            <button @click="discardEdit" class="btn bg-gray-200 hover:bg-gray-300">
              Discard
            </button>
          </div>
        </div>
        <div v-else class="flex-1 flex items-center gap-2">
          <span :class="{ 'line-through text-gray-400': todo.completed }">
            {{ todo.text }}
          </span>
          <span :class="[
            'text-xs font-medium px-2 py-1 rounded',
            getPriorityColor(todo.priority)
          ]">
            {{ todo.priority }}
          </span>
        </div>

        <div class="flex gap-2">
          <button v-if="!editingTodo || editingTodo.id !== todo.id" @click="startEdit(todo)"
            class="text-gray-400 hover:text-gray-600">
            <PencilIcon class="h-5 w-5" />
          </button>
          <button @click="deleteTodo(todo.id)" class="text-gray-400 hover:text-red-500">
            <TrashIcon class="h-5 w-5" />
          </button>
        </div>
      </div>

      <div v-if="todos.length === 0" class="text-center py-4 text-gray-500">
        No tasks in {{ project.name }}
      </div>
    </div>
  </div>
</template>