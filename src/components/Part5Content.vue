<script setup>
import { ref, watch } from 'vue'

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

const isLoading = ref(true)

watch(() => props.currentPart.url, () => {
  isLoading.value = true
})

const onIframeLoad = () => {
  isLoading.value = false
}
</script>

<template>
  <div class="part5-container">
    <div v-if="isLoading" class="loading-overlay">
      <div class="spinner"></div>
      <p>Đang tải bài thi, vui lòng đợi...</p>
    </div>
    <iframe
      v-show="!isLoading"
      :src="currentPart.url"
      :title="`${currentTest.name} Part ${currentPart.part}`"
      frameborder="0"
      allowfullscreen
      class="part5-iframe"
      @load="onIframeLoad"
    ></iframe>
  </div>
</template>
