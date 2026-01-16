<script setup>
import { ref } from 'vue'
import ZeroContainer from './ZeroContainer.vue'

const colors = ['#f5f5f5', '#fef3c7', '#e0f2fe', '#e8f5e9', '#f3e8ff', '#fff1f2', '#ede9fe']
const backgroundColor = ref(colors[0])

const changeBackground = () => {
  const next = colors[Math.floor(Math.random() * colors.length)]
  backgroundColor.value = next
}
</script>

<template>
  <div class="container" :style="{ backgroundColor }" @click="changeBackground">
    <ZeroContainer />
  </div>
</template>

<style scoped>
.container {
  width: 100%;
  height: 100%;
  padding: 1rem;
  display: flex;
  justify-content: center;
  align-items: center;
  transition: background-color 200ms ease;
}
</style>
