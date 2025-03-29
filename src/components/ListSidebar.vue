<template>
    <div class="w-64 h-full flex flex-col bg-gray-900 text-white border-r border-gray-700">
      <!-- Header with Close Button -->
      <div class="p-4 border-b border-gray-700 flex justify-between items-center">
        <h1 class="text-xl font-bold text-indigo-400">TaskMaster</h1>
        <button 
          @click="$emit('close-sidebar')"
          class="md:hidden text-gray-400 hover:text-white"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>
  
      <!-- Lists Section -->
      <div class="flex-1 overflow-y-auto p-4">
        <div class="flex items-center justify-between mb-4">
          <h2 class="text-sm font-semibold text-gray-400">My Lists</h2>
          <button 
            @click="openCreateModal"
            class="px-2 py-1 text-xs bg-indigo-600 hover:bg-indigo-500 rounded-md transition-colors"
          >
            + New List
          </button>
        </div>
  
        <ul class="space-y-2">
          <li 
            v-for="list in lists" 
            :key="list.id"
            class="group relative flex items-center justify-between p-2 rounded-md hover:bg-gray-800 transition-colors cursor-pointer"
            :class="{ 'bg-gray-800': activeListId === list.id }"
          >
            <div 
              class="flex items-center flex-1 space-x-2"
              @click="setActiveList(list.id)"
            >
              <span 
                class="w-3 h-3 rounded-full shrink-0"
                :style="{ backgroundColor: list.color }"
              ></span>
              <span class="text-sm truncate sidebar-text">{{ list.title }}</span>
            </div>
            
            <div class="flex space-x-2 opacity-0 group-hover:opacity-100 transition-opacity">
              <button
                @click.stop="openEditModal(list)"
                class="p-1 text-gray-400 hover:text-indigo-400 rounded hover:bg-gray-700 transition-colors"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"/>
                </svg>
              </button>
              <button
                @click.stop="deleteList(list.id)"
                class="p-1 text-gray-400 hover:text-red-400 rounded hover:bg-gray-700 transition-colors"
              >
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                </svg>
              </button>
            </div>
          </li>
        </ul>
      </div>
  
      <!-- Profile Section -->
      <div class="p-4 border-t border-gray-700">
        <div class="flex items-center justify-between">
          <div class="flex items-center space-x-2">
            <img :src="user.avatar" class="w-8 h-8 rounded-full">
            <span class="text-sm">{{ user.name }}</span>
          </div>
          <button 
            @click="logout"
            class="text-red-400 hover:text-red-300 text-sm transition-colors"
          >
            Logout
          </button>
        </div>
      </div>
  
      <!-- List Editor Modal -->
      <Teleport to="body">
        <div v-if="showModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
          <div class="w-full max-w-md bg-gray-800 rounded-lg p-6">
            <div class="flex justify-between items-center mb-4">
              <h3 class="text-xl text-indigo-400 font-semibold">
                {{ isEditing ? 'Edit List' : 'Create New List' }}
              </h3>
              <button 
                @click="closeModal"
                class="text-gray-400 hover:text-gray-200 transition-colors"
              >
                &times;
              </button>
            </div>
  
            <div class="space-y-4">
              <div>
                <label class="block text-sm font-medium text-gray-300 mb-1">List Name</label>
                <input
                  v-model="currentList.title"
                  type="text"
                  class="w-full px-3 py-2 bg-gray-700 rounded-md border border-gray-600 focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
                  placeholder="Enter list name"
                >
              </div>
  
              <div>
                <label class="block text-sm font-medium text-gray-300 mb-2">Color</label>
                <div class="flex flex-wrap gap-2">
                  <button
                    v-for="color in colors"
                    :key="color"
                    @click="currentList.color = color"
                    class="w-8 h-8 rounded-full border-2 transition-transform transform hover:scale-110"
                    :class="[
                      currentList.color === color ? 'border-white' : 'border-transparent',
                      color === '#6366F1' ? 'ring-2 ring-indigo-500' : ''
                    ]"
                    :style="{ backgroundColor: color }"
                  ></button>
                </div>
              </div>
  
              <button
                @click="handleSave"
                class="w-full py-2 px-4 bg-indigo-600 hover:bg-indigo-500 text-white rounded-md transition-colors"
              >
                {{ isEditing ? 'Save Changes' : 'Create List' }}
              </button>
            </div>
          </div>
        </div>
      </Teleport>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const props = defineProps({
    lists: Array,
    activeListId: [Number, null]
  });
  
  const emit = defineEmits([
    'set-active-list',
    'add-list',
    'edit-list',
    'delete-list',
    'logout',
    'close-sidebar'
  ]);
  
  const user = ref({
    name: "Guest User",
    avatar: "https://randomuser.me/api/portraits/lego/1.jpg"
  });
  
  const showModal = ref(false);
  const isEditing = ref(false);
  const currentList = ref(createNewListObject());
  
  const colors = [
    "#3B82F6", // Blue
    "#22C55E", // Green
    "#F59E0B", // Orange
    "#EF4444", // Red
    "#A855F7", // Purple
    "#EC4899"  // Pink
  ];
  
  function createNewListObject() {
    return {
      id: null,
      title: '',
      color: '#6366F1'
    };
  }
  
  function setActiveList(id) {
    emit('set-active-list', id);
  }
  
  function openCreateModal() {
    isEditing.value = false;
    currentList.value = createNewListObject();
    showModal.value = true;
  }
  
  function openEditModal(list) {
    isEditing.value = true;
    currentList.value = { ...list };
    showModal.value = true;
  }
  
  function handleSave() {
    if (!currentList.value.title.trim()) return;
  
    const listData = { 
      ...currentList.value,
      tasks: isEditing.value ? props.lists.find(l => l.id === currentList.value.id).tasks : []
    };
  
    if (isEditing.value) {
      emit('edit-list', listData);
    } else {
      emit('add-list', { ...listData, id: Date.now() });
    }
    
    closeModal();
  }
  
  function deleteList(listId) {
    if (confirm('Are you sure you want to delete this list? All tasks will be permanently removed!')) {
      emit('delete-list', listId);
    }
  }
  
  function closeModal() {
    showModal.value = false;
    isEditing.value = false;
    currentList.value = createNewListObject();
  }
  
  function logout() {
    emit('logout');
  }
  </script>
  
  <style scoped>
  .sidebar-text {
    font-size: 0.875rem; /* text-sm */
  }
  
  @media (max-width: 768px) {
    .sidebar-text {
      font-size: 1rem; /* text-base for better mobile readability */
    }
  }
  
  /* Custom scrollbar */
  ::-webkit-scrollbar {
    width: 6px;
  }
  
  ::-webkit-scrollbar-track {
    background: #1a202c;
  }
  
  ::-webkit-scrollbar-thumb {
    background: #4a5568;
    border-radius: 3px;
  }
  
  ::-webkit-scrollbar-thumb:hover {
    background: #718096;
  }
  </style>