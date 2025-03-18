<script setup lang="ts">
const supabase = useSupabaseClient()
const route = useRoute()

interface FlashCard {
  original_text: string
  translated_text: string
  memorized: boolean | null
}

const isLoading = ref(true)
const cards = ref<FlashCard[]>([])
const currentIndex = ref(0)
const startX = ref(0)
const currentX = ref(0)
const dragX = ref(0)
const isDragging = ref(false)
const isFlipped = ref(false)
const dragStartTime = ref(0)
const allCardsCount = ref(0)
const currentCard = computed(() => cards.value[currentIndex.value])

// Animation states
const showLike = ref(false)
const showNope = ref(false)

// Get flashcards from API
const fetchCards = async (): Promise<void> =>  {
  try {
    isLoading.value = true
    const { data, error } = await supabase
      .from('flashcards')
      .select('original_text, translated_text')
      .eq('quizz_id', route.params.id)

    if (error) throw error
    if (!Array.isArray(data)) return

    cards.value = data.map((card): FlashCard => ({
      ...(card as Omit<FlashCard, 'memorized'>),
      memorized: null
    }))
    allCardsCount.value = cards.value.length
  } catch (error) {
    console.error('Error:', error)
  } finally {
    isLoading.value = false
  }
}

// Card styles for animation
const cardStyle = computed(() => {
  if (!isDragging.value && dragX.value === 0) return {}

  const rotate = dragX.value * 0.1
  const scale = Math.max(0.8, 1 - Math.abs(dragX.value) / 1000)

  return {
    transform: `translateX(${dragX.value}px) rotate(${rotate}deg) scale(${scale})`,
    transition: isDragging.value ? 'none' : 'transform 0.5s ease-out'
  }
})

const unMemorizedCards = computed<FlashCard[]>(() => cards.value.filter(card => card.memorized === false))
const memorizedCards = computed<FlashCard[]>(() => cards.value.filter(card => card.memorized === true))

// Touch handlers
const handleTouchStart = (e: TouchEvent | MouseEvent) => {
  isDragging.value = true
  startX.value = 'touches' in e ? e.touches[0].clientX : e.clientX
  currentX.value = dragX.value
  dragStartTime.value = Date.now()
}

const handleTouchMove = (e: TouchEvent | MouseEvent) => {
  if (!isDragging.value) return

  const x = 'touches' in e ? e.touches[0].clientX : e.clientX
  dragX.value = currentX.value + (x - startX.value)

  // Show like/nope based on drag direction
  showLike.value = dragX.value > 50
  showNope.value = dragX.value < -50
}

const handleTouchEnd = () => {
  isDragging.value = false
  const threshold = window.innerWidth * 0.15

  if (Math.abs(dragX.value) > threshold) {
    // Swipe out
    const direction = dragX.value > 0 ? 1 : -1
    cards.value[currentIndex.value].memorized = direction === 1
    dragX.value = direction * window.innerWidth * 1.5

    if (currentIndex.value === cards.value.length - 1) {
      cards.value = unMemorizedCards.value
      currentIndex.value = 0
    } else {
      currentIndex.value++
    }

    dragX.value = 0
    showLike.value = false
    showNope.value = false

    setTimeout(() => {
      if (currentIndex.value < cards.value.length - 1) {
        isFlipped.value = false
      }
    }, 250)
  } else {
    // Reset position
    dragX.value = 0
    showLike.value = false
    showNope.value = false
  }
}

// Handle card flip
const handleCardClick = () => {
  // Only flip if it was a quick tap/click (not a drag)
  const touchDuration = Date.now() - dragStartTime.value
  if (touchDuration < 200 && Math.abs(dragX.value) < 10) {
    isFlipped.value = !isFlipped.value
  }
}

onMounted(fetchCards)
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-4">
    <div v-if="isLoading" class="text-center py-12">
      Loading cards...
    </div>

    <div v-else>
      <div v-if="cards.length === 0" class="flex items-center justify-center h-full">
        <div class="max-w-md mx-auto">
          <div class="bg-white rounded-xl shadow-lg p-6">
            <div class="text-2xl font-bold text-green-500 text-center">
              Congratulations! You've memorized all the flashcards.
            </div>
            <div class="text-center mt-4">
              <NuxtLink to="/quizzes">Back to quizzes</NuxtLink>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="max-w-md mx-auto">
        <div class="mb-5 flex items-center justify-between gap-4">
          <!-- <NuxtLink to="/quizzes">
            <svg
              class="w-6 h-6 text-gray-500 cursor-pointer"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke="currentColor"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M10 19l-7-7m0 0l7-7m-7 7h18"
              />
            </svg>
          </NuxtLink> -->
          <div class="bg-white text-red-500 font-bold rounded-xl text-center shadow-lg py-6 px-8">
            <span
              class="absolute opacity-0"
              :class="{ 'score-memorized opacity-100': showNope }"
            >+1</span>
            {{ unMemorizedCards.length }}
          </div>
          <div class="bg-white font-bold rounded-xl shadow-lg p-6 flex-grow-1 text-center text-gray-500">
            {{ currentIndex + 1 }} / {{ cards.length }}
          </div>
          <div class="bg-white text-green-500 font-bold rounded-xl text-center shadow-lg py-6 px-8">
            <span
              class="absolute opacity-0"
              :class="{ 'score-memorized opacity-100': showLike }"
            >+1</span>
            {{ memorizedCards.length }}
          </div>
        </div>
        <!-- Card Stack -->
        <div class="relative h-[550px]">
          <!-- Next card preview -->
          <div
            v-if="currentIndex + 1 < cards.length"
            class="card-preview absolute w-full"
          >
            <div class="bg-white rounded-xl shadow-lg p-6 w-full h-[550px]">
              <div class="h-full flex items-center justify-center text-2xl font-bold text-gray-400">
                {{ cards[currentIndex + 1].original_text }}
              </div>
            </div>
          </div>

          <!-- Current card -->
          <div
            v-if="currentCard"
            class="card absolute w-full"
            :style="cardStyle"
            @mousedown="handleTouchStart"
            @mousemove="handleTouchMove"
            @mouseup="handleTouchEnd"
            @click="handleCardClick"
          >
            <!-- Like/Nope stamps -->
            <div
              class="absolute top-8 right-8 transform -rotate-12 transition-opacity duration-200 z-10"
              :class="{ 'opacity-0': !showLike }"
            >
              <div class="text-4xl text-green-500 border-4 border-green-500 rounded-lg px-4 py-1">
                MEMORIZED
              </div>
            </div>
            <div
              class="absolute top-8 left-8 transform rotate-12 transition-opacity duration-200 z-10"
              :class="{ 'opacity-0': !showNope }"
            >
              <div class="text-4xl text-red-500 border-4 border-red-500 rounded-lg px-4 py-1">
                UNMEMORIZED
              </div>
            </div>

            <!-- Card content with flip -->
            <div class="card-container" :class="{ 'is-flipped': isFlipped }">
              <!-- Front side -->
              <div class="card-face card-front">
                <div class="h-full flex items-center justify-center text-2xl font-bold">
                  {{ currentCard.original_text }}
                </div>
              </div>
              <!-- Back side -->
              <div class="card-face card-back">
                <div class="h-full flex items-center justify-center text-2xl font-bold">
                  {{ currentCard.translated_text }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.card {
  transform-origin: center;
  will-change: transform;
  touch-action: none;
  user-select: none;
  cursor: grab;
}

.card:active {
  cursor: grabbing;
}

.card-preview {
  transform: scale(0.95) translateY(10px);
  opacity: 0.8;
  pointer-events: none;
}

/* Flip card styles */
.card-container {
  position: relative;
  width: 100%;
  height: 550px;
  perspective: 1000px;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.card-container.is-flipped {
  transform: rotateY(180deg);
}

.card-face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  background: white;
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

.card-front {
  transform: rotateY(0deg);
}

.card-back {
  transform: rotateY(180deg);
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-5px); }
}

.card:not(:active) {
  animation: float 3s ease-in-out infinite;
}

.score-memorized {
  transform: translateY(-75%);
  transition: ease-in-out 0.5s;
}
</style>
