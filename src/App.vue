<script setup lang="ts">
import CustomInput from './components/CustomInput.vue'
import { ref } from 'vue'

const content = ref('')

const onBlur = (value: string) => {
  console.log('失去焦点时的值：', value)
}
</script>

<template>
  <div class="input-container">
    <CustomInput v-model="content" @blur="onBlur" />
    <p>内容：{{ content }}</p>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

.input-container {
  margin-top: 2em;
  max-width: 500px;
  margin-left: auto;
  margin-right: auto;
}
</style>
