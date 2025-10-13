<script setup lang="ts">
import { RouterView } from 'vue-router'
import { useTheme } from 'vuetify'
import { ref, watch } from 'vue'

const theme = useTheme()

// Optional: track icon state separately
const lightOn = ref(!theme.global.current.value.dark)

// Watch the theme and update lightOn
watch(
  () => theme.global.current.value.dark,
  (isDark) => {
    lightOn.value = !isDark
  }
)

// Toggle function
function toggleTheme() {
  theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'
  lightOn.value = !theme.global.current.value.dark
}
</script>

<template>
  <header>
    <nav class="d-flex justify-space-between pa-4">
      <v-btn
        @click="toggleTheme"
        :icon="lightOn ? 'mdi-lightbulb-on-outline' : 'mdi-lightbulb-outline'"
        variant="plain"
        class="mb-2"
      />
    </nav>
  </header>
  <RouterView />
</template>

<style>
body {
  font-family: 'Roboto', sans-serif;
}
</style>
