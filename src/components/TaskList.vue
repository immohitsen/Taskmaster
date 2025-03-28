<template>
    <div class="p-6 bg-gray-900 text-white rounded-lg shadow-lg">
      <!-- List Title -->
      <h1 class="text-2xl font-bold mb-2">{{ listName }}</h1>
  
      <!-- Progress Bar -->
      <div class="mb-6">
        <p class="text-sm text-gray-400">
          {{ completedTasks }} of {{ tasks.length }} tasks completed
        </p>
        <div class="w-full h-2 bg-gray-700 rounded mt-1">
          <div
            class="h-full bg-indigo-500 rounded transition-all"
            :style="{ width: `${progress}%` }"
          ></div>
        </div>
      </div>
  
      <!-- Task List -->
      <div class="mt-4 space-y-3">
        <div
          v-for="task in tasks"
          :key="task.id"
          class="bg-gray-800 p-4 rounded-lg flex items-center justify-between"
        >
          <div class="flex items-center flex-1">
            <input
              type="checkbox"
              :checked="task.completed"
              @change="() => toggleTask(task.id)"
              class="mr-3 w-4 h-4 cursor-pointer"
            />
            <div :class="{ 'opacity-60': task.completed }">
              <h3 class="font-semibold">{{ task.title }}</h3>
              <p v-if="task.description" class="text-sm text-gray-400 mt-1">
                {{ task.description }}
              </p>
              <p v-if="task.dueDate" class="text-xs text-gray-500 flex items-center mt-1">
                <span class="mr-1">📅</span> Due {{ formattedDate(task.dueDate) }}
              </p>
            </div>
          </div>
          <button
            @click="() => deleteTask(task.id)"
            class="text-gray-400 hover:text-red-400 ml-2"
          >
            ✕
          </button>
        </div>
      </div>
  
      <!-- Add Task Button -->
      <button
        @click="showModal = true"
        class="fixed bottom-6 right-6 bg-indigo-600 hover:bg-indigo-500 text-white p-4 rounded-full shadow-lg text-2xl transition-all"
      >
        +
      </button>
  
      <!-- Add Task Modal -->
      <Teleport to="body">
        <div
          v-if="showModal"
          class="fixed inset-0 bg-black bg-opacity-50 backdrop-blur-sm flex justify-center items-center"
        >
          <div class="bg-gray-900 text-white p-6 rounded-lg shadow-lg w-96">
            <div class="flex justify-between items-center mb-4">
              <h2 class="text-xl font-bold text-indigo-400">Add New Task</h2>
              <button
                @click="showModal = false"
                class="text-gray-400 hover:text-gray-200 text-lg"
              >
                &times;
              </button>
            </div>
  
            <form @submit.prevent="addTask">
              <div class="space-y-4">
                <div>
                  <label class="block text-sm text-gray-400 mb-1">Title</label>
                  <input
                    v-model="newTask.title"
                    type="text"
                    required
                    class="w-full p-2 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
                    placeholder="Task title"
                  />
                </div>
  
                <div>
                  <label class="block text-sm text-gray-400 mb-1">Description</label>
                  <textarea
                    v-model="newTask.description"
                    class="w-full p-2 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
                    placeholder="Task details"
                    rows="3"
                  ></textarea>
                </div>
  
                <div>
                  <label class="block text-sm text-gray-400 mb-1">Due Date</label>
                  <input
                    v-model="newTask.dueDate"
                    type="date"
                    class="w-full p-2 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
                  />
                </div>
  
                <button
                  type="submit"
                  class="w-full bg-indigo-500 hover:bg-indigo-400 text-white py-2 rounded-lg mt-4 transition-colors"
                >
                  Add Task
                </button>
              </div>
            </form>
          </div>
        </div>
      </Teleport>
    </div>
  </template>
  
  <script setup>
  import { defineProps, defineEmits, computed, ref } from 'vue';
  
  const props = defineProps({
    listName: {
      type: String,
      default: 'Untitled List'
    },
    tasks: {
      type: Array,
      default: () => []
    }
  });
  
  const emit = defineEmits(['add-task', 'toggle-task', 'delete-task']);
  
  const showModal = ref(false);
  const newTask = ref({
    title: '',
    description: '',
    dueDate: ''
  });
  
  const completedTasks = computed(() => 
    props.tasks.filter(task => task.completed).length
  );
  
  const progress = computed(() =>
    props.tasks.length ? Math.round((completedTasks.value / props.tasks.length) * 100) : 0
  );
  
  const formattedDate = (dateString) => {
    const options = { year: 'numeric', month: 'short', day: 'numeric' };
    return new Date(dateString).toLocaleDateString(undefined, options);
  };
  
  const addTask = () => {
    if (!newTask.value.title.trim()) return;
  
    emit('add-task', {
      ...newTask.value,
      id: Date.now(),
      completed: false,
      dueDate: newTask.value.dueDate || undefined
    });
  
    newTask.value = { title: '', description: '', dueDate: '' };
    showModal.value = false;
  };
  
  const toggleTask = (taskId) => {
    emit('toggle-task', taskId);
  };
  
  const deleteTask = (taskId) => {
    emit('delete-task', taskId);
  };
  </script>