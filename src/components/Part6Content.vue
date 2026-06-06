<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

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
const isLoadingIframe = ref(true)

// Resize state
const leftPanelWidth = ref(50) // Phần trăm
const isResizing = ref(false)

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

// Image Preloading
watch(() => testImages.value, (images) => {
  images.forEach(img => {
    const preload = new Image()
    preload.src = img.src
  })
}, { immediate: true })

watch(() => props.currentPart.url, () => {
  isLoadingIframe.value = true
})

const onIframeLoad = () => {
  isLoadingIframe.value = false
}

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

// Xử lý Zoom & Pan
const isZoomed = ref(false)
const scale = ref(1)
const panX = ref(0)
const panY = ref(0)

const toggleZoom = () => {
  if (isZoomed.value) {
    isZoomed.value = false
    scale.value = 1
    panX.value = 0
    panY.value = 0
  } else {
    isZoomed.value = true
    scale.value = 2 // Zoom 2x
  }
}

// Xử lý vuốt màn hình (Swipe gestures) & Pan (Kéo)
const isDragging = ref(false)
const transitionName = ref('slide-left')
let startX = 0
let startY = 0
let initialPanX = 0
let initialPanY = 0
let touchStartX = 0
let touchEndX = 0
let lastTapTime = 0

// Kéo thả chuột
const onDragStart = (e) => {
  if (!isZoomed.value) return
  isDragging.value = true
  startX = e.clientX || e.touches[0].clientX
  startY = e.clientY || e.touches[0].clientY
  initialPanX = panX.value
  initialPanY = panY.value
}

const onDragMove = (e) => {
  if (!isDragging.value || !isZoomed.value) return
  const currentX = e.clientX || e.touches[0].clientX
  const currentY = e.clientY || e.touches[0].clientY
  panX.value = initialPanX + (currentX - startX)
  panY.value = initialPanY + (currentY - startY)
}

const onDragEnd = () => {
  isDragging.value = false
}

// Chạm màn hình
const onTouchStart = (e) => {
  const currentTime = new Date().getTime()
  const tapLength = currentTime - lastTapTime
  
  if (tapLength > 0 && tapLength < 300) {
    // Double tap
    e.preventDefault()
    toggleZoom()
    lastTapTime = 0
    return
  }
  lastTapTime = currentTime

  if (isZoomed.value) {
    onDragStart(e)
  } else {
    touchStartX = e.changedTouches[0].screenX
  }
}

const onTouchMove = (e) => {
  if (isZoomed.value) {
    onDragMove(e)
  }
}

const onTouchEnd = (e) => {
  if (isZoomed.value) {
    onDragEnd()
  } else {
    touchEndX = e.changedTouches[0].screenX
    handleSwipe()
  }
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

// Reset image index khi chuyển test hoặc chuyển ảnh
watch(() => props.currentTest.id, () => {
  currentImageIndex.value = 0
  isZoomed.value = false
  scale.value = 1
  panX.value = 0
  panY.value = 0
  transitionName.value = 'slide-left'
}, { immediate: true })

watch(currentImageIndex, (newVal, oldVal) => {
  if (newVal > oldVal) {
    transitionName.value = 'slide-left'
  } else if (newVal < oldVal) {
    transitionName.value = 'slide-right'
  }
  isZoomed.value = false
  scale.value = 1
  panX.value = 0
  panY.value = 0
})

// Resizer handlers
const onResizerMouseDown = (e) => {
  e.preventDefault()
  isResizing.value = true
  document.body.style.userSelect = 'none'
  document.body.style.cursor = 'col-resize'
}

const onMouseMove = (e) => {
  if (!isResizing.value) return
  e.preventDefault()
  
  const container = document.querySelector('.part6-container')
  if (!container) return
  
  const containerRect = container.getBoundingClientRect()
  const newX = e.clientX - containerRect.left
  const newWidth = (newX / containerRect.width) * 100
  
  // Đặt giới hạn: min 20%, max 80%
  if (newWidth >= 20 && newWidth <= 80) {
    leftPanelWidth.value = newWidth
  }
}

const onMouseUp = (e) => {
  e.preventDefault()
  isResizing.value = false
  document.body.style.userSelect = ''
  document.body.style.cursor = ''
}

// Setup event listeners khi component mount
onMounted(() => {
  document.addEventListener('mousemove', onMouseMove, true)
  document.addEventListener('mouseup', onMouseUp, true)
  document.addEventListener('selectstart', (e) => {
    if (isResizing.value) e.preventDefault()
  }, true)
})

// Cleanup event listeners khi component unmount
onUnmounted(() => {
  document.removeEventListener('mousemove', onMouseMove, true)
  document.removeEventListener('mouseup', onMouseUp, true)
})
</script>

<template>
  <div class="part6-container" :class="{ 'is-resizing': isResizing }">
    <!-- Images Panel -->
    <div 
      class="images-panel"
      :style="{ width: `${leftPanelWidth}%` }"
    >
      <!-- Image Display -->
      <div 
        class="image-display"
        :class="{ 'is-zoomed': isZoomed }"
        @mousedown="onDragStart"
        @mousemove="onDragMove"
        @mouseup="onDragEnd"
        @mouseleave="onDragEnd"
        @dblclick="toggleZoom"
        @touchstart="onTouchStart"
        @touchmove.passive="onTouchMove"
        @touchend="onTouchEnd"
      >
        <transition :name="transitionName">
          <div :key="currentImage ? currentImage.id : 'empty'" class="image-wrapper">
            <img
              v-if="currentImage"
              :src="currentImage.src"
              :alt="`${currentTest.name} Image ${currentImage.id}`"
              @error="$event.target.src = '/imgs/placeholder.png'"
              class="image"
              :class="{ 'is-dragging': isDragging }"
              :style="{ transform: `scale(${scale}) translate(${panX / scale}px, ${panY / scale}px)` }"
              draggable="false"
            />
          </div>
        </transition>
        
        <button class="zoom-btn" @click="toggleZoom">
          <svg v-if="!isZoomed" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line><line x1="11" y1="8" x2="11" y2="14"></line><line x1="8" y1="11" x2="14" y2="11"></line></svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line><line x1="8" y1="11" x2="14" y2="11"></line></svg>
        </button>
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

    <!-- Resizer Divider -->
    <div 
      class="resizer"
      @mousedown="onResizerMouseDown"
      :class="{ 'is-resizing': isResizing }"
    ></div>

    <!-- iframe Panel -->
    <div 
      class="iframe-panel"
      :style="{ width: `${100 - leftPanelWidth}%` }"
    >
      <div v-if="isLoadingIframe" class="loading-overlay">
        <div class="spinner"></div>
        <p>Đang tải bài thi, vui lòng đợi...</p>
      </div>
      <iframe
        v-show="!isLoadingIframe"
        :src="currentPart.url"
        :title="`${currentTest.name} Part ${currentPart.part}`"
        frameborder="0"
        allowfullscreen
        class="part6-iframe"
        @load="onIframeLoad"
      ></iframe>
    </div>
  </div>
</template>
