<script setup>
const client = useSupabaseClient()
const user = useSupabaseUser()
const router = useRouter()

const title = ref('')
const description = ref('')
const flashcards = ref([
  { original_text: '', translated_text: '' }
])
const errorMessage = ref('')
const successMessage = ref('')

const addFlashcard = () => {
  flashcards.value.push({ original_text: '', translated_text: '' })
}

const removeFlashcard = (index) => {
  flashcards.value.splice(index, 1)
}

const createQuizz = async () => {
  try {
    const { data: quizz, error: quizzError } = await client
      .from('quizzes')
      .insert([
        {
          title: title.value,
          description: description.value,
          user_id: user.value.id
        }
      ])
      .select()
      .single()

    if (quizzError) throw quizzError

    const { error: flashcardsError } = await client
      .from('flashcards')
      .insert(
        flashcards.value.map(card => ({
          original_text: card.original_text,
          translated_text: card.translated_text,
          user_id: user.value.id,
          quizz_id: quizz.id
        }))
      )

    if (flashcardsError) throw flashcardsError

    successMessage.value = 'Quizz created successfully!'
    router.push('/quizzes/' + quizz.id)
  } catch (error) {
    errorMessage.value = error.message
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto my-8 p-6 bg-white rounded-lg shadow-md">
    <h2 class="text-2xl font-bold mb-4">Create New Quizz</h2>

    <div v-if="errorMessage" class="text-red-500 mb-4">{{ errorMessage }}</div>
    <div v-if="successMessage" class="text-green-500 mb-4">{{ successMessage }}</div>

    <form @submit.prevent="createQuizz" class="space-y-6">
      <div class="space-y-4">
        <div>
          <label class="block text-gray-700 mb-2">Quizz Title</label>
          <input
            v-model="title"
            type="text"
            class="w-full p-3 border border-gray-200 rounded"
            required
          >
        </div>

        <div>
          <label class="block text-gray-700 mb-2">Description</label>
          <textarea
            v-model="description"
            class="w-full p-3 border border-gray-200 rounded"
            rows="3"
          ></textarea>
        </div>
      </div>

      <div class="space-y-4 mb-4">
        <h3 class="text-xl font-semibold">Flashcards</h3>

        <div v-for="(card, index) in flashcards" :key="index" class="p-4 border border-gray-200 rounded">
          <div class="flex justify-between items-center mb-2">
            <h4 class="font-medium">Card {{ index + 1 }}</h4>
            <button
              type="button"
              @click="removeFlashcard(index)"
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
                class="w-full p-3 border border-gray-200 rounded"
                required
              >
            </div>
            <div>
              <label class="block text-gray-700 mb-2">Translated Text</label>
              <input
                v-model="card.translated_text"
                type="text"
                class="w-full p-3 border border-gray-200 rounded"
                required
              >
            </div>
          </div>
        </div>

        <button
          type="button"
          @click="addFlashcard"
          class="w-full border border-gray-200 border border-gray-200-blue-600 text-blue-600 py-2 px-4 rounded hover:bg-blue-50"
        >
          Add Another Card
        </button>
      </div>

      <button
        type="submit"
        class="w-full bg-blue-600 text-white py-3 px-4 rounded hover:bg-blue-700"
      >
        Create Quizz
      </button>
    </form>
  </div>
</template>
