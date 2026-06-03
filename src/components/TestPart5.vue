<script setup>
import { ref } from 'vue'
import Part5Content from './Part5Content.vue'
import Part6Content from './Part6Content.vue'

const selectedTest = ref(1)
const selectedPart = ref(5)
const openDropdown = ref(null)

const tests = [
  {
    id: 1,
    name: 'Test 1',
    parts: [
      { part: 5, url: 'https://wayground.com/embed/quiz/6a0b0a7fc9583d3f7ebff982' },
      { part: 6, url: 'https://wayground.com/embed/quiz/6a0b13835aec1c2d7556191d' }
    ]
  },
  {
    id: 2,
    name: 'Test 2',
    parts: [
      { part: 5, url: 'https://wayground.com/embed/quiz/6a0b197f4f6920d181076435' },
      { part: 6, url: 'https://wayground.com/embed/quiz/6a0c662e037e1c11919931d4' }
    ]
  },
  {
    id: 3,
    name: 'Test 3',
    parts: [
      { part: 5, url: 'https://wayground.com/embed/quiz/6a0c69b40ff9329e1f35cb07' },
      { part: 6, url: 'https://wayground.com/embed/quiz/6a0c92960d9cf616b04fb7dd' }
    ]
  },
  {
    id: 4,
    name: 'Test 4',
    parts: [
      { part: 5, url: 'https://wayground.com/embed/quiz/6a0cb58a0ff9329e1f36347f' },
      { part: 6, url: 'https://wayground.com/embed/quiz/6a1846c55870909baa07a681' }
    ]
  }
]

const currentTest = () => tests.find(test => test.id === selectedTest.value)
const currentPart = () => currentTest()?.parts.find(p => p.part === selectedPart.value)

const selectTest = (testId) => {
  selectedTest.value = testId
  selectedPart.value = 5
  openDropdown.value = null
}

const selectPart = (part) => {
  selectedPart.value = part
  openDropdown.value = null
  mobileMenuOpen.value = false
}

const toggleDropdown = (testId) => {
  openDropdown.value = openDropdown.value === testId ? null : testId
}

const mobileMenuOpen = ref(false)
const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value
  if (!mobileMenuOpen.value) {
    openDropdown.value = null
  }
}

// Light / Dark Mode Toggle
import { onMounted } from 'vue'

const isDarkMode = ref(true)

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  updateTheme()
}

const updateTheme = () => {
  if (isDarkMode.value) {
    document.documentElement.removeAttribute('data-theme')
    localStorage.setItem('theme', 'dark')
  } else {
    document.documentElement.setAttribute('data-theme', 'light')
    localStorage.setItem('theme', 'light')
  }
}

onMounted(() => {
  const savedTheme = localStorage.getItem('theme')
  if (savedTheme) {
    isDarkMode.value = savedTheme === 'dark'
  } else {
    // Check system preference
    isDarkMode.value = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches
  }
  updateTheme()
})
</script>

<template>
  <div class="app-container">
    <!-- Header with menu -->
    <header class="app-header">
      <div class="header-content">
        <div class="header-left">
          <h1 class="header-title">CỐ LÊN :)))</h1>
        </div>
        
        <div class="header-actions">
          <button class="mobile-menu-btn" @click="toggleMobileMenu">
            <svg v-if="!mobileMenuOpen" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </button>
        </div>

        <div class="menu-bar" :class="{ 'is-open': mobileMenuOpen }">
          <div v-for="test in tests" :key="test.id" class="dropdown-wrapper">
            <button
              class="dropdown-trigger"
              :class="{ active: selectedTest === test.id }"
              @click="toggleDropdown(test.id)"
            >
              <span>{{ test.name }}</span>
              <span class="dropdown-arrow" :class="{ open: openDropdown === test.id }">▼</span>
            </button>

            <div v-show="openDropdown === test.id" class="dropdown-menu">
              <button
                v-for="part in test.parts"
                :key="part.part"
                class="dropdown-item"
                :class="{ active: selectedTest === test.id && selectedPart === part.part, disabled: !part.url }"
                @click="selectTest(test.id); selectPart(part.part)"
                :disabled="!part.url"
              >
                Part {{ part.part }}
              </button>
            </div>
          </div>
            <button class="theme-toggle-btn" @click="toggleTheme" :title="isDarkMode ? 'Bật giao diện Nền Sáng' : 'Bật giao diện Nền Tối'">
            <svg v-if="isDarkMode" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>
          </button>
            <button class=" mobile-menu-btn" @click="toggleMobileMenu">
            <svg v-if="!mobileMenuOpen" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </button>
        </div>
      </div>
    </header>

    <!-- Main content -->
    <main class="app-main">
      <div v-if="currentPart()?.url" class="content-wrapper">
        <!-- Breadcrumb/Title -->
        <div class="content-header">
          <h2 class="content-title">{{ currentTest().name }} - Part {{ selectedPart }}</h2>
        </div>

        <!-- Content area -->
        <div class="content-area">
          <Part5Content
            v-if="selectedPart === 5"
            :current-test="currentTest()"
            :current-part="currentPart()"
          />
          <Part6Content
            v-else
            :current-test="currentTest()"
            :current-part="currentPart()"
          />
        </div>
      </div>

      <!-- Empty state -->
      <div v-else class="empty-state">
        <p>Bài test này chưa có dữ liệu</p>
      </div>
    </main>
  </div>
</template>

