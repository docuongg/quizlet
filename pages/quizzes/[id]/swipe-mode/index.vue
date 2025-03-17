<script setup lang="ts">
const supabase = useSupabaseClient()
const route = useRoute()

interface FlashCard {
  original_text: string
  translated_text: string
}

const cards = ref<FlashCard[]>([])
const currentIndex = ref(0)
const startX = ref(0)
const currentX = ref(0)
const dragX = ref(0)
const isDragging = ref(false)
const isFlipped = ref(false)
const dragStartTime = ref(0)

const currentCard = computed(() => cards.value[currentIndex.value])

// Animation states
const showLike = ref(false)
const showNope = ref(false)

// Get flashcards from API
const fetchCards = async () => {
  try {
    const { data, error } = await supabase
      .from('flashcards')
      .select('original_text, translated_text')
      .eq('quizz_id', route.params.id)
    
    if (error) throw error
    cards.value = data
  } catch (error) {
    console.error('Error:', error)
  }
}

// Card styles for animation
const cardStyle = computed(() => {
  if (!isDragging.value && dragX.value === 0) return {}
  
  const rotate = dragX.value * 0.1
  const scale = Math.max(0.85, 1 - Math.abs(dragX.value) / 1000)
  
  return {
    transform: `translateX(${dragX.value}px) rotate(${rotate}deg) scale(${scale})`,
    transition: isDragging.value ? 'none' : 'transform 0.5s ease-out'
  }
})

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
    dragX.value = direction * window.innerWidth * 1.5
    
    // Next card
    setTimeout(() => {
      if (currentIndex.value < cards.value.length - 1) {
        currentIndex.value++
        dragX.value = 0
        showLike.value = false
        showNope.value = false
        isFlipped.value = false
      }
    }, 300)
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
    <div class="max-w-md mx-auto">
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
          @mousedown.prevent="handleTouchStart"
          @mousemove.prevent="handleTouchMove"
          @mouseup.prevent="handleTouchEnd"
          @click="handleCardClick"
        >
          <!-- Like/Nope stamps -->
          <div 
            class="absolute top-8 right-8 transform -rotate-12 transition-opacity duration-200 z-10"
            :class="{ 'opacity-0': !showLike }"
          >
            <div class="text-4xl text-green-500 border-4 border-green-500 rounded-lg px-4 py-1">
              LIKE
            </div>
          </div>
          <div 
            class="absolute top-8 left-8 transform rotate-12 transition-opacity duration-200 z-10"
            :class="{ 'opacity-0': !showNope }"
          >
            <div class="text-4xl text-red-500 border-4 border-red-500 rounded-lg px-4 py-1">
              NOPE
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
      <div class="text-center text-gray-500 mt-4">
        {{ currentIndex + 1 }} / {{ cards.length }}
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
</style>