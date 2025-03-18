<script setup>
const client = useSupabaseClient()
const user = useSupabaseUser()
const router = useRouter()
const quizzes = ref([])
const errorMessage = ref('')

const fetchQuizzes = async () => {
  try {
    const { data, error } = await client
      .from('quizzes')
      .select(`
        *,
        flashcards (id)
      `)
      .order('created_at', { ascending: false })

    if (error) throw error
    quizzes.value = data
  } catch (error) {
    errorMessage.value = error.message
  }
}

onMounted(() => {
  fetchQuizzes()
})

const playQuiz = (quizId) => {
  router.push(`/quizzes/${quizId}/play`)
}
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold">Explore Quizzes</h1>
      <NuxtLink
        to="/quizzes/new"
        class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
      >
        Create New Quizz
      </NuxtLink>
    </div>

    <div v-if="errorMessage" class="text-red-500 mb-4">{{ errorMessage }}</div>

    <div v-if="quizzes.length === 0" class="text-center py-12 text-gray-500">
      No quizzes found. Create your first quizz!
    </div>

    <div class="grid grid-cols-4 gap-4">
      <div
        v-for="quizz in quizzes"
        :key="quizz.id"
        class="bg-white rounded-lg border border-gray-200 overflow-hidden hover:shadow-lg transition-shadow"
      >
        <div class="p-6">
          <h2 class="text-xl font-bold mb-2">{{ quizz.title }}</h2>
          <p class="text-gray-600 mb-4 line-clamp-2">{{ quizz.description || "Explore now!" }}</p>

          <div class="flex items-center justify-between mb-4">
            <div class="flex items-center space-x-4">
              <div class="text-sm text-gray-500">
                <span class="font-medium">{{ quizz.flashcards.length }}</span> cards
              </div>
              <div class="text-sm text-gray-500">
                {{ new Date(quizz.created_at).toLocaleDateString() }}
              </div>
            </div>
          </div>

          <div class="flex space-x-3">
            <NuxtLink
              :to="`/quizzes/${quizz.id}`"
              class="flex-1 bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700 text-center"
            >
              Play
            </NuxtLink>
            <NuxtLink
              :to="`/quizzes/${quizz.id}/swipe-mode`"
              class="flex-1 border border-blue-600 text-blue-600 px-4 py-2 rounded hover:bg-blue-50 text-center"
            >
              Practice
            </NuxtLink>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
