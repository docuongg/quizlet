<script setup>
const client = useSupabaseClient()
const email = ref('')
const password = ref('')
const errorMessage = ref('')
const successMessage = ref('')

const signUp = async () => {
  errorMessage.value = ''
  successMessage.value = ''
  const { error } = await client.auth.signUp({
    email: email.value,
    password: password.value,
  })
  if (error) {
    errorMessage.value = error.message
  } else {
    successMessage.value = 'Sign-up successful! Please check your email to confirm your account.'
  }
}
</script>

<template>
  <div class="flex min-h-screen flex-col items-center justify-center bg-gray-50">
    <div class="bg-white p-8 rounded-lg shadow-lg w-96">
      <h2 class="text-2xl font-semibold text-center">Create an account</h2>
      <p class="text-gray-500 text-center mb-4">Sign up to start creating and taking quizzes</p>

      <button class="w-full py-2 border rounded flex items-center cursor-pointer justify-center text-gray-600">
        <Icon name="flat-color-icons:google" class="h-5 w-5 mr-2" />
        Sign up with Google
      </button>

      <div class="flex items-center my-4">
        <hr class="flex-grow border-gray-300" />
        <span class="mx-2 text-gray-500 text-sm">Or continue with</span>
        <hr class="flex-grow border-gray-300" />
      </div>

      <div v-if="errorMessage" class="text-red-500 text-center mb-4">{{ errorMessage }}</div>
      <div v-if="successMessage" class="text-green-500 text-center mb-4">{{ successMessage }}</div>

      <form @submit.prevent="signUp" class="space-y-4">
        <input v-model="email" type="email" placeholder="Email address" class="w-full p-3 border rounded" required>
        <input v-model="password" type="password" placeholder="Password" class="w-full p-3 border rounded" required>
        <button type="submit" class="w-full bg-blue-600 text-white py-3 rounded">Sign up</button>
      </form>

      <p class="text-center text-gray-500 mt-4">
        Already have an account?
        <NuxtLink to="/login" class="text-blue-600">Login</NuxtLink>
      </p>
    </div>
  </div>
</template>
