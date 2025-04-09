<script setup>
import { ref, onMounted } from 'vue'
import ProjectList from './components/ProjectList.vue'
import TodoList from './components/TodoList.vue'

const projects = ref([])
const selectedProject = ref(null)
const showNewProjectModal = ref(false)
const newProjectName = ref('')

onMounted(() => {
  const savedProjects = localStorage.getItem('projects')
  if (savedProjects) {
    projects.value = JSON.parse(savedProjects)
  }
})

const saveToLocalStorage = () => {
  localStorage.setItem('projects', JSON.stringify(projects.value))
}

const createProject = () => {
  if (newProjectName.value.trim()) {
    const project = {
      id: Date.now(),
      name: newProjectName.value,
      todos: []
    }
    projects.value.push(project)
    newProjectName.value = ''
    showNewProjectModal.value = false
    saveToLocalStorage()
  }
}

const deleteProject = (projectId) => {
  projects.value = projects.value.filter(p => p.id !== projectId)
  if (selectedProject.value?.id === projectId) {
    selectedProject.value = null
  }
  saveToLocalStorage()
}

const updateProject = (projectId, newName) => {
  const project = projects.value.find(p => p.id === projectId)
  if (project) {
    project.name = newName
    saveToLocalStorage()
  }
}

const updateTodos = (projectId, todos) => {
  const project = projects.value.find(p => p.id === projectId)
  if (project) {
    project.todos = todos
    saveToLocalStorage()
  }
}
</script>

<template>
  <div class="min-h-screen bg-gray-50">
    <header class="bg-primary shadow-sm">
      <div class="max-w-7xl mx-auto px-4 py-4">
        <h1 class="text-2xl font-bold text-gray-800 text-center">Project Management System</h1>
      </div>
    </header>

    <main class="max-w-7xl mx-auto px-4 py-8">
      <div class="flex flex-col md:flex-row md:space-x-12">
        <!-- Projects Sidebar -->
        <div class="w-full md:w-1/4 mb-6 md:mb-0">
          <div class="sticky top-4">
            <div class="flex justify-between items-center mb-4">
              <h2 class="text-xl font-semibold text-gray-700">Projects</h2>
              <button @click="showNewProjectModal = true" class="btn btn-primary">
                + Add Project
              </button>
            </div>

            <div class="bg-white rounded-lg shadow p-4 overflow-visible">
              <ProjectList :projects="projects" :selectedProject="selectedProject" @select="selectedProject = $event"
                @delete="deleteProject" @update="updateProject" />
            </div>
          </div>
        </div>

        <!-- Todo List -->
        <div class="w-full md:w-3/4">
          <TodoList v-if="selectedProject" :project="selectedProject"
            @update:todos="updateTodos(selectedProject.id, $event)" />
          <div v-else class="text-center py-12 bg-white rounded-lg shadow">
            <p class="text-gray-500">Select a project to view tasks</p>
          </div>
        </div>
      </div>
    </main>

    <!-- New Project Modal -->
    <div v-if="showNewProjectModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-semibold mb-4">Create New Project</h3>
        <input v-model="newProjectName" type="text" placeholder="Project name" class="input w-full mb-4"
          @keyup.enter="createProject">
        <div class="flex justify-end gap-2">
          <button @click="showNewProjectModal = false" class="btn bg-gray-200 hover:bg-gray-300">
            Cancel
          </button>
          <button @click="createProject" class="btn btn-primary">
            Create
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
