<script setup>
const route = useRoute()
const client = useSupabaseClient()
const user = useSupabaseUser()
const errorMessage = ref('')
const cards = ref([])
const matchedPairs = ref([])
const selectedCards = ref([])
const isLoading = ref(true)
const isShowResult = ref(false)
const isMatched = ref(false)
const isMismatched = ref(false)

const startTime = ref(null)
const bestTimes = ref([])

const fetchQuizzData = async () => {
  try {
    const { data: flashcards, error } = await client
      .from('flashcards')
      .select('*')
      .eq('quizz_id', route.params.id)

    if (error) throw error

    const selectedFlashcards = flashcards
      .sort(() => Math.random() - 0.5)
      .slice(0, 6)

    const cardPairs = selectedFlashcards.flatMap(card => [
      { id: `orig-${card.id}`, text: card.original_text, pairId: card.id, isFlipped: false },
      { id: `trans-${card.id}`, text: card.translated_text, pairId: card.id, isFlipped: false }
    ])

    cards.value = cardPairs.sort(() => Math.random() - 0.5)
    isLoading.value = false
    await fetchBestTimes()
  } catch (error) {
    errorMessage.value = error.message
    isLoading.value = false
  }
}

const selectCard = (card) => {
  if (selectedCards.value.length === 2 ||
    matchedPairs.value.includes(card.pairId) ||
    card.isFlipped) return

  if (selectedCards.value.length === 0 && matchedPairs.value.length === 0) {
    startGame()
  }

  card.isFlipped = true
  selectedCards.value.push(card)

  if (selectedCards.value.length === 2) {
    checkMatch()
  }
}

const checkMatch = () => {
  const [card1, card2] = selectedCards.value

  if (card1.pairId === card2.pairId) {
    isMatched.value = true
    matchedPairs.value.push(card1.pairId)

    if (matchedPairs.value.length === cards.value.length / 2) {
      saveGameResult()
    }

    setTimeout(() => {
      selectedCards.value = []
      isMatched.value = false
    }, 500)
  } else {
    isMismatched.value = true
    setTimeout(() => {
      card1.isFlipped = false
      card2.isFlipped = false
      selectedCards.value = []
      isMismatched.value = false
    }, 500)
  }
}

const isGameComplete = computed(() => {
  return matchedPairs.value.length === cards.value.length / 2
})

const startGame = () => {
  startTime.value = Date.now()
}

const saveGameResult = async () => {
  const completionTime = Math.floor((Date.now() - startTime.value) / 1000)

  try {
    await client.from('flipcard_results').insert({
      user_id: user.value.id,
      quizz_id: route.params.id,
      completion_time: completionTime,
      profile_id: user.value.id,
    })

    await fetchBestTimes()

    setTimeout(() => {
      isShowResult.value = true
    }, 1000)
  } catch (error) {
    errorMessage.value = error.message
  }
}

const fetchBestTimes = async () => {
  try {
    const { data, error } = await client
      .from('flipcard_results')
      .select(`
        completion_time,
        completed_at,
        profiles (
          id,
          email,
          avatar_url,
          full_name
        )
      `)
      .eq('quizz_id', route.params.id)
      .order('completion_time', { ascending: true })
      .limit(5)

    if (error) throw error
    bestTimes.value = data.map(result => ({
      ...result,
      users: {
        email: result.profiles.email,
        avatar_url: result.profiles.avatar_url,
        full_name: result.profiles.full_name
      }
    }))
  } catch (error) {
    errorMessage.value = error.message
  }
}

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`
}

const resetGame = async () => {
  cards.value = []
  matchedPairs.value = []
  selectedCards.value = []
  isMatched.value = false
  isMismatched.value = false
  startTime.value = null
  isShowResult.value = false
  await fetchQuizzData()
}

onMounted(async () => {
  await fetchQuizzData()
})
</script>

<template>
  <div class="container mx-auto px-4 py-8">
    <div v-if="errorMessage" class="text-red-500 mb-4">{{ errorMessage }}</div>

    <div v-if="isLoading" class="text-center py-12">
      Loading cards...
    </div>

    <div v-else>
      <div class="text-center mb-8">
        <h2 class="text-2xl font-bold mb-2">Memory Flashcard Game</h2>
        <p class="text-gray-600">
          Matched: {{ matchedPairs.length }} / {{ cards.length / 2 }} pairs
        </p>
      </div>

      <div v-if="isShowResult" class="text-center mb-8">
        <h3 class="text-xl font-bold text-green-600 mb-4">Congratulations! You've completed the game!</h3>

        <div class="max-w-4xl mx-auto mb-6 bg-white p-4 rounded-lg shadow">
          <h3 class="text-lg font-semibold mb-4">Best Times</h3>
          <div v-if="bestTimes.length > 0">
            <div class="grid grid-cols-12 gap-4 py-2 border-b font-medium text-gray-600">
              <div class="col-span-1">Rank</div>
              <div class="col-span-1">Avatar</div>
              <div class="col-span-3 text-left">Name</div>
              <div class="col-span-2 text-center">Time</div>
              <div class="col-span-5 text-right">Completed At</div>
            </div>

            <div v-for="(result, index) in bestTimes" :key="index"
              class="grid grid-cols-12 gap-4 items-center py-3 border-b last:border-0">
              <div class="col-span-1 text-gray-600 font-medium">
                #{{ index + 1 }}
              </div>

              <div class="col-span-1">
                <div class="w-10 h-10 rounded-full overflow-hidden">
                  <img
                    :src="result.users.avatar_url || 'https://ui-avatars.com/api/?name=' + result.users.email"
                    :alt="result.users.full_name || result.users.email"
                    class="w-full h-full object-cover"
                  >
                </div>
              </div>

              <div class="col-span-3 text-left font-medium">
                {{ result.users.full_name || result.users.email.split('@')[0] }}
              </div>

              <div class="col-span-2 text-center text-blue-600 font-bold">
                {{ formatTime(result.completion_time) }}
              </div>

              <div class="col-span-5 text-right text-gray-500">
                {{ new Date(result.completed_at).toLocaleString() }}
              </div>
            </div>
          </div>
          <div v-else class="text-center text-gray-500 py-2">
            No records yet
          </div>
        </div>

        <div class="space-x-4">
          <button
            @click="resetGame"
            class="inline-block bg-green-600 cursor-pointer text-white px-6 py-2 rounded-lg hover:bg-green-700"
          >
            Play Again
          </button>
          <NuxtLink
            to="/quizzes"
            class="inline-block bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700"
          >
            Back to Quizzes
          </NuxtLink>
        </div>
      </div>

      <div class="grid grid-cols-2 md:grid-cols-6 gap-4">
        <div v-for="card in cards" :key="card.id" @click="selectCard(card)"
          class="aspect-w-4 aspect-h-5"
          :class="{
            'invisible': matchedPairs.includes(card.pairId)
          }">
          <div class="relative w-full h-full transition-transform duration-300 preserve-3d" :class="{
            'rotate-y-180': card.isFlipped,
            'cursor-pointer': !matchedPairs.includes(card.pairId) && !card.isFlipped
          }">
            <!-- Card Back -->
            <div
              class="absolute w-full h-full backface-hidden bg-blue-600 rounded-lg flex items-center justify-center text-white p-4">
              Click to reveal
            </div>

            <!-- Card Front -->
            <div
              class="absolute w-full h-full backface-hidden bg-white border-2 rounded-lg flex items-center justify-center p-4 text-center rotate-y-180"
              :class="{
                'border-green-500': isMatched && selectedCards.includes(card),
                'border-red-500': isMismatched && selectedCards.includes(card),
                'border-blue-600': !selectedCards.includes(card)
              }">
              {{ card.text }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.preserve-3d {
  transform-style: preserve-3d;
  backface-visibility: hidden;
}

.backface-hidden {
  backface-visibility: hidden;
}

.rotate-y-180 {
  transform: rotateY(180deg);
}

.aspect-w-3 {
  position: relative;
  padding-bottom: calc(var(--tw-aspect-h) / var(--tw-aspect-w) * 100%);
}

.aspect-h-4 {
  --tw-aspect-h: 4;
  --tw-aspect-w: 3;
}

.aspect-w-3>* {
  position: absolute;
  height: 100%;
  width: 100%;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.invisible {
  visibility: hidden;
  opacity: 0;
  transition: visibility 0s 0.8s, opacity 0.8s linear;
}

.aspect-w-4 {
  position: relative;
  padding-bottom: calc(var(--tw-aspect-h) / var(--tw-aspect-w) * 100%);
}

.aspect-h-5 {
  --tw-aspect-h: 5;
  --tw-aspect-w: 4;
}

.aspect-w-4>* {
  position: absolute;
  height: 100%;
  width: 100%;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}
</style>
