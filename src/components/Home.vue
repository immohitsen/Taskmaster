<template>
  <div class="flex bg-gray-800 h-screen relative">
    <!-- Mobile Menu Button -->
    <button 
      v-if="!isSidebarOpen"
      @click="isSidebarOpen = !isSidebarOpen"
      class="md:hidden fixed top-6 left-4 z-50 p-2 text-gray-400 hover:text-white bg-gray-800 rounded-lg shadow-md"
    >
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
      </svg>
    </button>

    <!-- Sidebar -->
    <div 
      class="w-64 h-screen flex flex-col bg-gray-900 text-white border-r border-gray-700 shadow-lg transition-transform duration-300 transform fixed md:relative z-40"
      :class="{
        '-translate-x-full md:translate-x-0': !isSidebarOpen,
        'translate-x-0': isSidebarOpen
      }"
    >
      <ListSidebar
        :lists="lists"
        :activeListId="activeListId"
        @set-active-list="setActiveList"
        @add-list="addList"
        @edit-list="editList"
        @delete-list="deleteList"
        @logout="handleLogout"
        @close-sidebar="isSidebarOpen = false"
      />
    </div>

    <!-- Main Content -->
    <div class="flex-1 p-6  text-white overflow-auto mt-12 md:mt-0">
      <TaskList
        v-if="activeList"
        :key="activeListId"
        :tasks="activeList.tasks"
        :listName="activeList.title"
        @add-task="addTask"
        @toggle-task="toggleTask"
        @delete-task="deleteTask"
      />
      <div v-else class="text-center text-gray-500 text-xl mt-8">
        Select or create a list
      </div>
    </div>

    <!-- Mobile Overlay -->
    <div 
      v-if="isSidebarOpen"
      @click="isSidebarOpen = false"
      class="fixed inset-0 bg-black bg-opacity-50 z-30 md:hidden"
    ></div>

    <!-- Auth Modal -->
    <AuthModal v-if="!isAuthenticated" @login="handleLogin" />
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import ListSidebar from '@/components/ListSidebar.vue'
import TaskList from '@/components/TaskList.vue'
import AuthModal from '@/components/AuthModal.vue'

const isAuthenticated = ref(true)
const isSidebarOpen = ref(false)
const activeListId = ref(null)
const lists = ref([
  { id: 1, title: 'Work', color: '#3b82f6', tasks: [] },
  { id: 2, title: 'Personal', color: '#10b981', tasks: [] },
  { id: 3, title: 'Learning', color: '#A855F7', tasks: [] }
])

const activeList = ref(null)

watch(activeListId, () => {
  activeList.value = lists.value.find(list => list.id === activeListId.value) || null
}, { immediate: true })

const setActiveList = (id) => {
  activeListId.value = id
  isSidebarOpen.value = false // Close sidebar on mobile selection
}

const addList = (newList) => {
  lists.value.push(newList)
  setActiveList(newList.id)
}

const editList = (updatedList) => {
  const index = lists.value.findIndex(l => l.id === updatedList.id)
  if (index !== -1) {
    lists.value[index] = updatedList
    if (activeList.value?.id === updatedList.id) {
      activeList.value = updatedList
    }
  }
}

const deleteList = (listId) => {
  lists.value = lists.value.filter(l => l.id !== listId)
  if (activeListId.value === listId) {
    activeListId.value = null
  }
}

const addTask = (newTask) => {
  if (activeList.value) {
    activeList.value.tasks.push(newTask)
  }
}

const toggleTask = (taskId) => {
  const task = activeList.value?.tasks.find(t => t.id === taskId)
  if (task) task.completed = !task.completed
}

const deleteTask = (taskId) => {
  if (activeList.value) {
    activeList.value.tasks = activeList.value.tasks.filter(t => t.id !== taskId)
  }
}

const handleLogin = () => {
  isAuthenticated.value = true
}

const handleLogout = () => {
  alert('Logged out!')
  isAuthenticated.value = false
}
</script>

<style scoped>
/* Mobile-friendly font sizes */
@media (max-width: 768px) {
  .sidebar-text {
    font-size: 1rem;
  }
}
</style>