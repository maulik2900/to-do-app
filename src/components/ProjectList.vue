<script setup>
import { computed, ref } from "vue";
import { TrashIcon, PencilIcon } from "@heroicons/vue/24/outline";

const props = defineProps({
  projects: {
    type: Array,
    required: true,
  },
  selectedProject: {
    type: Object,
    default: null,
  },
});

const emit = defineEmits(["select", "delete", "update"]);
const editingProject = ref(null);
const showDeleteConfirm = ref(null);

const sortedProjects = computed(() => {
  return [...props.projects].sort((a, b) => a.name.localeCompare(b.name));
});

const startEdit = (project, event) => {
  event.stopPropagation();
  editingProject.value = {
    id: project.id,
    name: project.name,
  };
};

const saveEdit = (project) => {
  if (editingProject.value && editingProject.value.name.trim()) {
    emit("update", project.id, editingProject.value.name);
    editingProject.value = null;
  }
};

const discardEdit = () => {
  editingProject.value = null;
};

const confirmDelete = (project, event) => {
  event.stopPropagation();
  showDeleteConfirm.value = project.id;
};

const handleDelete = (projectId) => {
  emit("delete", projectId);
  showDeleteConfirm.value = null;
};
</script>

<template>
  <div class="space-y-2">
    <div v-for="project in sortedProjects" :key="project.id"
      class="flex items-center justify-between p-3 rounded-lg cursor-pointer transition-colors duration-200" :class="selectedProject?.id === project.id
        ? 'bg-primary'
        : 'bg-white hover:bg-gray-50'
        " @click="emit('select', project)">
      <div class="flex items-center gap-2 flex-1">
        <div v-if="editingProject?.id === project.id" class="flex-1 relative z-10 bg-white shadow-lg p-2 rounded">
          <div class="flex gap-1">
            <input v-model="editingProject.name" type="text" class="input flex" @click.stop />
            <button @click.stop="saveEdit(project)" class="btn btn-primary">
              Save
            </button>
            <button @click.stop="discardEdit" class="btn bg-gray-200 hover:bg-gray-300">
              Discard
            </button>
          </div>
        </div>

        <span v-else class="font-medium text-gray-700">{{ project.name }}</span>
      </div>

      <div class="flex gap-2">
        <button v-if="!editingProject || editingProject.id !== project.id" @click="startEdit(project, $event)"
          class="text-gray-400 hover:text-gray-600">
          <PencilIcon class="h-5 w-5" />
        </button>
        <button @click="confirmDelete(project, $event)" class="text-gray-400 hover:text-red-500">
          <TrashIcon class="h-5 w-5" />
        </button>
      </div>
    </div>

    <div v-if="projects.length === 0" class="text-center py-4 text-gray-500">
      No projects yet
    </div>

    <!-- Delete Confirmation Modal -->
    <div v-if="showDeleteConfirm" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white rounded-lg p-6 w-full max-w-md">
        <h3 class="text-lg font-semibold mb-4">Delete Project {{
          projects.find((p) => p.id === showDeleteConfirm)?.name
        }}</h3>
        <p class="mb-4">
          Are you sure you want to delete project <b> {{
            projects.find((p) => p.id === showDeleteConfirm)?.name
            }}</b> ? <br> This action cannot be undone.
        </p>
        <div class="flex justify-end gap-2">
          <button @click="showDeleteConfirm = null" class="btn bg-gray-200 hover:bg-gray-300">
            Cancel
          </button>
          <button @click="handleDelete(showDeleteConfirm)" class="btn bg-red-500 hover:bg-red-600 text-white">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
