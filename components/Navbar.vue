<script setup>
const client = useSupabaseClient()
const user = useSupabaseUser()
const route = useRoute()
const router = useRouter()
const showDropdown = ref(false)
const userProfile = ref(null)

// Add function to fetch profile
const fetchUserProfile = async () => {
  if (!user.value) return

  try {
    const { data, error } = await client
      .from('profiles')
      .select('avatar_url, full_name', 'email')
      .eq('id', user.value.id)
      .single()

    if (error) throw error
    userProfile.value = data
  } catch (error) {
    console.error('Error fetching profile:', error)
  }
}

const isActive = (routePath) => {
  return routePath === route.path
}
// Watch for user changes
watch(user, () => {
  if (user.value) {
    fetchUserProfile()
  }
})

onMounted(() => {
  if (user.value) {
    fetchUserProfile()
  }
})

// Close dropdown when clicking outside
onMounted(() => {
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.user-menu')) {
      showDropdown.value = false
    }
  })
})
</script>

<template>
  <header class="bg-white shadow-sm">
    <div class="container mx-auto flex justify-between items-center p-4">
      <NuxtLink href="/" class="text-2xl font-bold text-blue-600">Quizlet</NuxtLink>
      <nav class="flex items-center">
        <ul class="flex space-x-6 items-center">
          <li>
            <NuxtLink
              href="/"
              class="text-gray-700 hover:text-blue-500 py-2"
              :class="{ 'text-blue-600 border-b-2 border-blue-600': isActive('/') }"
            >
              Home
            </NuxtLink>
          </li>
          <li>
            <NuxtLink
              href="/quizzes"
              class="text-gray-700 hover:text-blue-500 py-2"
              :class="{ 'text-blue-600 border-b-2 border-blue-600': isActive('/quizzes') }"
            >
              Quizzes
            </NuxtLink>
          </li>
          <!-- User Avatar & Dropdown -->
          <li v-if="user" class="relative user-menu">
            <button
              @click="showDropdown = !showDropdown"
              class="flex items-center space-x-2 focus:outline-none"
            >
              <img
                :src="userProfile?.avatar_url || 'https://ui-avatars.com/api/?name=' + (userProfile?.full_name || userProfile?.email)"
                :alt="userProfile?.full_name || userProfile?.email"
                class="w-8 h-8 rounded-full object-cover border"
              >
            </button>

            <!-- Dropdown Menu -->
            <div
              v-show="showDropdown"
              class="absolute right-0 mt-2 w-48 bg-white rounded-md shadow-lg py-1 z-50"
            >
              <NuxtLink
                to="/profile"
                class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
                @click="showDropdown = false"
              >
                Profile Settings
              </NuxtLink>
              <button
                @click="signOut"
                class="block cursor-pointer w-full text-left px-4 py-2 text-sm text-gray-700 hover:bg-gray-100"
              >
                Sign Out
              </button>
            </div>
          </li>
        </ul>
      </nav>
    </div>
  </header>
</template>

<style scoped>
.user-menu {
  isolation: isolate;
}
</style>
