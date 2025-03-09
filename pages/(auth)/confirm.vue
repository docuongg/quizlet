<script setup>
const client = useSupabaseClient()
const router = useRouter()

onMounted(async () => {
  try {
    const { error } = await client.auth.getSession()
    if (error) throw error

    router.push('/')
  } catch (error) {
    console.error('Auth error:', error)
    router.push('/login')
  }
})
</script>

<template>
  <div class="flex min-h-screen items-center justify-center">
    <div class="text-center">
      <h2 class="text-xl font-semibold mb-2">Completing sign in...</h2>
      <p class="text-gray-600">Please wait while we confirm your authentication.</p>
    </div>
  </div>
</template>
