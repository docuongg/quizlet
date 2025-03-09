<script setup>
const client = useSupabaseClient()
const user = useSupabaseUser()
const route = useRoute()
const router = useRouter()

const title = ref('')
const description = ref('')
const flashcards = ref([])
const errorMessage = ref('')
const successMessage = ref('')
const isLoading = ref(true)

const fetchQuizData = async () => {
  try {
    const { data: quiz, error: quizError } = await client
      .from('quizzes')
      .select('*')
      .eq('id', route.params.id)
      .single()

    if (quizError) throw quizError

    if (quiz.user_id !== user.value?.id) {
      router.push('/quizzes')
      return
    }

    title.value = quiz.title
    description.value = quiz.description

    const { data: cards, error: cardsError } = await client
      .from('flashcards')
      .select('*')
      .eq('quizz_id', route.params.id)

    if (cardsError) throw cardsError
    flashcards.value = cards

    isLoading.value = false
  } catch (error) {
    errorMessage.value = error.message
    isLoading.value = false
  }
}

const addFlashcard = () => {
  flashcards.value.push({
    original_text: '',
    translated_text: '',
    quizz_id: route.params.id,
    user_id: user.value.id
  })
}

const removeFlashcard = async (index, cardId) => {
  if (cardId) {
    const { error } = await client
      .from('flashcards')
      .delete()
      .eq('id', cardId)

    if (error) {
      errorMessage.value = error.message
      return
    }
  }
  flashcards.value.splice(index, 1)
}

const saveChanges = async () => {
  try {
    const { error: quizError } = await client
      .from('quizzes')
      .update({
        title: title.value,
        description: description.value
      })
      .eq('id', route.params.id)

    if (quizError) throw quizError

    for (const card of flashcards.value) {
      if (card.id) {
        const { error } = await client
          .from('flashcards')
          .update({
            original_text: card.original_text,
            translated_text: card.translated_text
          })
          .eq('id', card.id)

        if (error) throw error
      } else {
        const { error } = await client
          .from('flashcards')
          .insert({
            quizz_id: route.params.id,
            user_id: user.value.id,
            original_text: card.original_text,
            translated_text: card.translated_text
          })

        if (error) throw error
      }
    }

    successMessage.value = 'Quiz updated successfully!'
    setTimeout(() => {
      router.push('/quizzes')
    }, 1500)
  } catch (error) {
    errorMessage.value = error.message
  }
}

onMounted(() => {
  fetchQuizData()
})
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <div v-if="errorMessage" class="text-red-500 mb-4">{{ errorMessage }}</div>
    <div v-if="successMessage" class="text-green-500 mb-4">{{ successMessage }}</div>

    <div v-if="isLoading" class="text-center py-12">
      Loading quiz data...
    </div>

    <div v-else class="max-w-4xl mx-auto">
      <div class="flex justify-between items-center mb-6">
        <h1 class="text-2xl font-bold">Edit Quiz</h1>
        <NuxtLink
          to="/quizzes"
          class="text-blue-600 hover:text-blue-700"
        >
          Back to Quizzes
        </NuxtLink>
      </div>

      <form @submit.prevent="saveChanges" class="space-y-6">
        <div class="space-y-4">
          <div>
            <label class="block text-gray-700 mb-2">Quiz Title</label>
            <input
              v-model="title"
              type="text"
              class="w-full p-3 border rounded"
              required
            >
          </div>

          <div>
            <label class="block text-gray-700 mb-2">Description</label>
            <textarea
              v-model="description"
              class="w-full p-3 border rounded"
              rows="3"
            ></textarea>
          </div>
        </div>

        <div class="space-y-4">
          <div class="flex justify-between items-center">
            <h2 class="text-xl font-semibold">Flashcards</h2>
            <button
              type="button"
              @click="addFlashcard"
              class="text-blue-600 hover:text-blue-700"
            >
              + Add Card
            </button>
          </div>

          <div v-for="(card, index) in flashcards" :key="index"
               class="p-4 border rounded bg-white">
            <div class="flex justify-between items-center mb-2">
              <h3 class="font-medium">Card {{ index + 1 }}</h3>
              <button
                type="button"
                @click="removeFlashcard(index, card.id)"
                class="text-red-500 hover:text-red-700"
                :disabled="flashcards.length === 1"
              >
                Remove
              </button>
            </div>

            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-gray-700 mb-2">Original Text</label>
                <input
                  v-model="card.original_text"
                  type="text"
                  class="w-full p-3 border rounded"
                  required
                >
              </div>
              <div>
                <label class="block text-gray-700 mb-2">Translated Text</label>
                <input
                  v-model="card.translated_text"
                  type="text"
                  class="w-full p-3 border rounded"
                  required
                >
              </div>
            </div>
          </div>
        </div>

        <div class="flex justify-end space-x-4">
          <NuxtLink
            to="/quizzes"
            class="px-6 py-2 border rounded hover:bg-gray-50"
          >
            Cancel
          </NuxtLink>
          <button
            type="submit"
            class="px-6 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Save Changes
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
