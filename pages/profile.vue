<template>
  <div class="container mx-auto px-4 py-8">
    <div class="max-w-2xl mx-auto">
      <h1 class="text-2xl font-bold mb-6">Profile Settings</h1>

      <div v-if="errorMessage" class="bg-red-50 text-red-500 p-4 rounded mb-4">
        {{ errorMessage }}
      </div>

      <div v-if="successMessage" class="bg-green-50 text-green-500 p-4 rounded mb-4">
        {{ successMessage }}
      </div>

      <div v-if="isLoading" class="text-center py-12">
        Loading profile...
      </div>

      <form v-else @submit.prevent="updateProfile" class="space-y-6">
        <div>
          <div class="mb-4">
            <img
              :src="profile.avatar_url || 'https://ui-avatars.com/api/?name=' + profile.full_name"
              :alt="profile.full_name"
              class="w-32 h-32 rounded-lg object-cover border shadow-sm"
            >
          </div>

          <label class="block text-gray-700 mb-2">Profile Picture URL</label>
          <input
            v-model="profile.avatar_url"
            type="url"
            class="w-full p-3 border rounded focus:ring-2 focus:ring-blue-200 focus:border-blue-500"
            placeholder="https://example.com/avatar.jpg"
          >
        </div>

        <div>
          <label class="block text-gray-700 mb-2">Full Name</label>
          <input
            v-model="profile.full_name"
            type="text"
            class="w-full p-3 border rounded focus:ring-2 focus:ring-blue-200 focus:border-blue-500"
            required
          >
        </div>

        <div>
          <label class="block text-gray-700 mb-2">Email Address</label>
          <input
            v-model="profile.email"
            type="email"
            class="w-full p-3 border rounded focus:ring-2 focus:ring-blue-200 focus:border-blue-500"
            required
          >
        </div>

        <div class="flex justify-end">
          <button
            type="submit"
            class="bg-blue-600 text-white px-6 py-2 rounded hover:bg-blue-700 transition-colors"
          >
            Save Changes
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
const client = useSupabaseClient()
const user = useSupabaseUser()

const profile = ref({
  email: '',
  full_name: '',
  avatar_url: ''
})
const errorMessage = ref('')
const successMessage = ref('')
const isLoading = ref(true)

const fetchProfile = async () => {
  try {
    const { data, error } = await client
      .from('profiles')
      .select('*')
      .eq('id', user.value.id)
      .single()

    if (error) throw error

    profile.value = {
      email: data.email || user.value.email,
      full_name: data.full_name || '',
      avatar_url: data.avatar_url || ''
    }
    isLoading.value = false
  } catch (error) {
    errorMessage.value = error.message
    isLoading.value = false
  }
}

const updateProfile = async () => {
  try {
    const { error } = await client
      .from('profiles')
      .upsert({
        id: user.value.id,
        email: profile.value.email,
        full_name: profile.value.full_name,
        avatar_url: profile.value.avatar_url,
        updated_at: new Date()
      })

    if (error) throw error

    successMessage.value = 'Profile updated successfully!'
    setTimeout(() => {
      successMessage.value = ''
    }, 3000)
  } catch (error) {
    errorMessage.value = error.message
  }
}

onMounted(() => {
  if (!user.value) {
    return navigateTo('/login')
  }
  fetchProfile()
})
</script>
