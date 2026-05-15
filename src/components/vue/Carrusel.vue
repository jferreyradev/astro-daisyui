<template>
  <div
    class="relative w-full bg-base-300 rounded-lg overflow-hidden"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
  >
    <!-- Carousel Slides Container -->
    <div class="relative w-full h-96 overflow-hidden">
      <div
        class="flex transition-transform duration-500 ease-in-out h-full"
        :style="{ transform: `translateX(-${currentIndex * 100}%)` }"
      >
        <!-- Individual Slides -->
        <div
          v-for="(slide, index) in slides"
          :key="index"
          class="relative w-full h-full flex-shrink-0"
        >
          <!-- Image -->
          <img
            :src="slide.image"
            :alt="slide.title || `Slide ${index + 1}`"
            class="w-full h-full object-cover"
          />

          <!-- Text Overlay -->
          <div
            v-if="slide.title || slide.description"
            class="absolute inset-0 bg-black/30 flex flex-col justify-end p-6"
          >
            <h3 v-if="slide.title" class="text-white text-2xl font-bold mb-2">
              {{ slide.title }}
            </h3>
            <p v-if="slide.description" class="text-white/90 text-sm">
              {{ slide.description }}
            </p>
          </div>
        </div>
      </div>

      <!-- Navigation Buttons -->
      <div class="absolute inset-0 flex items-center justify-between px-5 pointer-events-none">
        <button
          @click="prevSlide"
          class="btn btn-circle btn-sm bg-white hover:bg-white/80 text-black border-0 pointer-events-auto"
          aria-label="Slide anterior"
        >
          ❮
        </button>
        <button
          @click="nextSlide"
          class="btn btn-circle btn-sm bg-white hover:bg-white/80 text-black border-0 pointer-events-auto"
          aria-label="Siguiente slide"
        >
          ❯
        </button>
      </div>
    </div>

    <!-- Indicators (Dots) -->
    <div class="flex justify-center items-center gap-2 py-4">
      <button
        v-for="(slide, index) in slides"
        :key="`dot-${index}`"
        @click="goToSlide(index)"
        :class="[
          'w-2 h-2 rounded-full transition-all duration-300',
          index === currentIndex
            ? 'bg-primary w-8'
            : 'bg-gray-400 hover:bg-gray-500'
        ]"
        :aria-label="`Ir al slide ${index + 1}`"
        :aria-current="index === currentIndex"
      />
    </div>

    <!-- Autoplay Status Indicator (optional, for debugging) -->
    <div v-if="false" class="absolute top-2 right-2 text-white text-xs">
      {{ isAutoplayActive ? '▶' : '⏸' }}
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'

const props = defineProps({
  slides: Array,
  autoplay: {
    type: Boolean,
    default: true
  },
  interval: {
    type: Number,
    default: 3000
  },
  pauseOnHover: {
    type: Boolean,
    default: true
  },
})

const currentIndex = ref(0)
const isAutoplayActive = ref(false)
let autoplayInterval = null

// Navigate to specific slide
const goToSlide = (index) => {
  currentIndex.value = index
  // Reset autoplay timer when manually navigating
  if (isAutoplayActive.value) {
    stopAutoplay()
    startAutoplay()
  }
}

// Next slide
const nextSlide = () => {
  currentIndex.value = (currentIndex.value + 1) % props.slides.length
  if (isAutoplayActive.value) {
    stopAutoplay()
    startAutoplay()
  }
}

// Previous slide
const prevSlide = () => {
  currentIndex.value =
    (currentIndex.value - 1 + props.slides.length) % props.slides.length
  if (isAutoplayActive.value) {
    stopAutoplay()
    startAutoplay()
  }
}

// Start autoplay
const startAutoplay = () => {
  if (!props.autoplay || props.slides.length === 0) return

  isAutoplayActive.value = true
  autoplayInterval = setInterval(() => {
    currentIndex.value = (currentIndex.value + 1) % props.slides.length
  }, props.interval)
}

// Stop autoplay
const stopAutoplay = () => {
  isAutoplayActive.value = false
  if (autoplayInterval) {
    clearInterval(autoplayInterval)
    autoplayInterval = null
  }
}

// Handle mouse enter - pause autoplay
const handleMouseEnter = () => {
  if (props.pauseOnHover && isAutoplayActive.value) {
    stopAutoplay()
  }
}

// Handle mouse leave - resume autoplay
const handleMouseLeave = () => {
  if (props.pauseOnHover && props.autoplay && !isAutoplayActive.value) {
    startAutoplay()
  }
}

// Initialize autoplay on mount
onMounted(() => {
  if (props.autoplay && props.slides.length > 0) {
    startAutoplay()
  }
})

// Cleanup on unmount
onUnmounted(() => {
  stopAutoplay()
  console.log(props.slides)
})

// Watch slides prop for changes
watch(
  () => props.slides,
  () => {
    if (currentIndex.value >= props.slides.length) {
      currentIndex.value = 0
    }
    if (isAutoplayActive.value) {
      stopAutoplay()
    }
    if (props.autoplay && props.slides.length > 0) {
      startAutoplay()
    }
  }
)
</script>

<style scoped>
/* Smooth scroll behavior for carousel */
.carousel-item {
  scroll-behavior: smooth;
}

/* Optional: Add a subtle shadow for depth */
.relative {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>
