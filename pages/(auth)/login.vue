<script setup>
const client = useSupabaseClient()
const router = useRouter()
const email = ref('')
const password = ref('')
const errorMessage = ref('')
const isLoading = ref(false)

const login = async () => {
  errorMessage.value = ''
  const { error } = await client.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  })
  if (error) {
    errorMessage.value = error.message
  } else {
    router.push('/')
  }
}

const signInWithGoogle = async () => {
  try {
    isLoading.value = true
    errorMessage.value = ''
    const { error } = await client.auth.signInWithOAuth({
      provider: 'google',
      options: {
        redirectTo: `${window.location.origin}/confirm`,
        queryParams: {
          access_type: 'offline',
          prompt: 'consent',
        }
      }
    })

    if (error) throw error

  } catch (error) {
    errorMessage.value = error.message
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="flex min-h-screen flex-col items-center justify-center bg-gray-50">
    <div class="bg-white p-8 rounded-lg shadow-lg w-96">
      <h2 class="text-2xl font-semibold text-center">Login to your account</h2>
      <p class="text-gray-500 text-center mb-4">Enter your email and password to login</p>

      <button
        @click="signInWithGoogle"
        :disabled="isLoading"
        class="w-full py-2 border cursor-pointer rounded flex items-center justify-center text-gray-600 hover:bg-gray-50 transition-colors"
      >
        <Icon name="flat-color-icons:google" class="h-5 w-5 mr-2" />
        {{ isLoading ? 'Connecting...' : 'Continue with Google' }}
      </button>

      <div class="flex items-center my-4">
        <hr class="flex-grow border-gray-300" />
        <span class="mx-2 text-gray-500 text-sm">Or continue with</span>
        <hr class="flex-grow border-gray-300" />
      </div>

      <div v-if="errorMessage" class="text-red-500 text-center mb-4">{{ errorMessage }}</div>

      <form @submit.prevent="login" class="space-y-4">
        <input v-model="email" type="email" placeholder="Email address" class="w-full p-3 border rounded" required>
        <input v-model="password" type="password" placeholder="Password" class="w-full p-3 border rounded" required>
        <button type="submit" class="w-full cursor-pointer bg-blue-600 text-white py-3 rounded">Login</button>
      </form>

      <p class="text-center text-gray-500 mt-4">
        Don't have an account?
        <NuxtLink to="/sign-up" class="text-blue-600">Sign up</NuxtLink>
      </p>
    </div>
  </div>
</template>
