<template>
  <div class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center">
    <div class="bg-gray-900 text-white p-6 rounded-lg shadow-lg w-96">
      <h2 class="text-2xl font-bold text-center text-indigo-400">
        {{ isLogin ? "Welcome to TaskMaster" : "Create Account" }}
      </h2>
      <p class="text-sm text-gray-400 text-center mt-1">
        {{ isLogin ? "Use demo credentials to start organizing" : "Register to manage your tasks" }}
      </p>

      <!-- Name Input (Signup only) -->
      <div v-if="!isLogin" class="mt-4">
        <label class="text-sm text-gray-400">Name</label>
        <input
          type="text"
          v-model="name"
          class="w-full p-2 mt-1 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
          placeholder="Enter your name"
        >
      </div>

      <!-- Email Input -->
      <div class="mt-4">
        <label class="text-sm text-gray-400">Email</label>
        <input
          type="email"
          v-model="email"
          class="w-full p-2 mt-1 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
          :placeholder="isLogin ? 'public@mail.com' : 'Enter your email'"
        >
      </div>

      <!-- Password Input -->
      <div class="mt-3">
        <label class="text-sm text-gray-400">Password</label>
        <input
          type="password"
          v-model="password"
          class="w-full p-2 mt-1 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
          :placeholder="isLogin ? 'public' : 'Create password'"
        >
      </div>

      <!-- Confirm Password (Signup only) -->
      <div v-if="!isLogin" class="mt-3">
        <label class="text-sm text-gray-400">Confirm Password</label>
        <input
          type="password"
          v-model="confirmPassword"
          class="w-full p-2 mt-1 bg-gray-800 border border-gray-700 rounded focus:outline-none focus:border-indigo-400"
          placeholder="Confirm password"
        >
      </div>

      <!-- Error Message -->
      <p v-if="errorMessage" class="text-red-400 text-sm mt-2 text-center">{{ errorMessage }}</p>

      <!-- Action Button -->
      <button
        @click="handleAuth"
        class="w-full bg-indigo-500 hover:bg-indigo-400 text-white py-2 rounded-lg mt-4"
      >
        {{ isLogin ? "Login" : "Sign Up" }}
      </button>

      <!-- Toggle between Login/Signup -->
      <p class="text-center text-sm text-gray-400 mt-2">
        {{ isLogin ? "Don't have an account?" : "Already have an account?" }}
        <button
          @click="toggleAuthMode"
          class="text-indigo-400 hover:text-indigo-300 focus:outline-none"
        >
          {{ isLogin ? "Sign Up" : "Login" }}
        </button>
      </p>

      <!-- Demo Credentials Hint (Login only) -->
      <p v-if="isLogin" class="text-center text-sm text-gray-400 mt-2">
        Demo credentials:<br>
        Email: <span class="text-indigo-400">public@mail.com</span><br>
        Password: <span class="text-indigo-400">public</span>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const isLogin = ref(true);
const name = ref('');
const email = ref('');
const password = ref('');
const confirmPassword = ref('');
const errorMessage = ref('');

const emit = defineEmits(["login", "signup"]);

// Auto-fill demo credentials on mount when in login mode
onMounted(() => {
  if (isLogin.value) {
    email.value = 'public@mail.com';
    password.value = 'public';
  }
});

const toggleAuthMode = () => {
  isLogin.value = !isLogin.value;
  errorMessage.value = '';
  // Clear fields when toggling
  if (!isLogin.value) {
    name.value = '';
    email.value = '';
    password.value = '';
    confirmPassword.value = '';
  }
};

const handleAuth = () => {
  if (isLogin.value) {
    handleLogin();
  } else {
    handleSignup();
  }
};

const handleLogin = () => {
  if (email.value === 'public@mail.com' && password.value === 'public') {
    emit("login");
  } else {
    errorMessage.value = "Invalid email or password";
  }
};

const handleSignup = () => {
  // Basic validation
  if (!name.value || !email.value || !password.value || !confirmPassword.value) {
    errorMessage.value = "All fields are required!";
    return;
  }
  
  if (password.value !== confirmPassword.value) {
    errorMessage.value = "Passwords do not match!";
    return;
  }

  if (password.value.length < 6) {
    errorMessage.value = "Password must be at least 6 characters";
    return;
  }

  // Emit signup event with user data
  emit("signup", {
    name: name.value,
    email: email.value,
    password: password.value
  });
};
</script>