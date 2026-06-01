<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  currentTest: {
    type: Object,
    required: true
  },
  currentPart: {
    type: Object,
    required: true
  }
})

const currentImageIndex = ref(0)

const testImages = computed(() => {
  const images = []
  const testFolder = `test-${props.currentTest.id}`
  const imageCount = 4

  for (let i = 1; i <= imageCount; i++) {
    images.push({
      id: i,
      src: `/imgs/${testFolder}/image-${i}.png`
    })
  }

  return images
})

const currentImage = computed(() => {
  return testImages.value[currentImageIndex.value] || null
})

const previousImage = () => {
  if (currentImageIndex.value > 0) {
    currentImageIndex.value--
  }
}

const nextImage = () => {
  if (currentImageIndex.value < testImages.value.length - 1) {
    currentImageIndex.value++
  }
}

// Xử lý vuốt màn hình (Swipe gestures)
let touchStartX = 0
let touchEndX = 0

const onTouchStart = (e) => {
  touchStartX = e.changedTouches[0].screenX
}

const onTouchEnd = (e) => {
  touchEndX = e.changedTouches[0].screenX
  handleSwipe()
}

const handleSwipe = () => {
  const minSwipeDistance = 50 // Khoảng cách tối thiểu để xác nhận vuốt
  const diff = touchEndX - touchStartX

  if (Math.abs(diff) >= minSwipeDistance) {
    if (diff > 0) {
      // Vuốt sang phải -> Ảnh trước đó
      previousImage()
    } else {
      // Vuốt sang trái -> Ảnh tiếp theo
      nextImage()
    }
  }
}

// Reset image index khi chuyển test
watch(() => props.currentTest.id, () => {
  currentImageIndex.value = 0
}, { immediate: true })
</script>

<template>
  <div class="part6-container">
    <!-- Images Panel -->
    <div class="images-panel">
      <!-- Image Display -->
      <div 
        class="image-display"
        @touchstart="onTouchStart"
        @touchend="onTouchEnd"
      >
        <img
          v-if="currentImage"
          :src="currentImage.src"
          :alt="`${currentTest.name} Image ${currentImage.id}`"
          @error="$event.target.src = '/imgs/placeholder.png'"
          class="image"
        />
      </div>

      <!-- Image Controls -->
      <div class="image-controls">
        <button
          @click="previousImage"
          :disabled="currentImageIndex === 0"
          class="control-btn"
        >
          ❮ Trước
        </button>

        <div class="slider-wrapper">
          <input
            type="range"
            v-model.number="currentImageIndex"
            :min="0"
            :max="testImages.length - 1"
            class="image-slider"
          />
          <span class="image-counter">{{ currentImageIndex + 1 }} / {{ testImages.length }}</span>
        </div>

        <button
          @click="nextImage"
          :disabled="currentImageIndex === testImages.length - 1"
          class="control-btn"
        >
          Tiếp ❯
        </button>
      </div>
    </div>

    <!-- iframe Panel -->
    <div class="iframe-panel">
      <iframe
        :src="currentPart.url"
        :title="`${currentTest.name} Part ${currentPart.part}`"
        frameborder="0"
        allowfullscreen
        class="part6-iframe"
      ></iframe>
    </div>
  </div>
</template>
