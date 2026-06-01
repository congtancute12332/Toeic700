<script setup>
defineProps({
  currentTest: {
    type: Object,
    required: true
  },
  currentPart: {
    type: Object,
    required: true
  }
})
</script>

<template>
  <div class="part5-container">
    <iframe
      :src="currentPart.url"
      :title="`${currentTest.name} Part ${currentPart.part}`"
      frameborder="0"
      allowfullscreen
      class="part5-iframe"
    ></iframe>
  </div>
</template>
