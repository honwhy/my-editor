<template>
  <div class="input-wrapper">
    <div class="input-container">
      <div
        class="editable-div"
        contenteditable="true"
        @input="handleInput"
        @blur="handleBlur"
        :innerHTML="modelValue"
      ></div>
      <button class="emoji-button" @click="toggleEmojiPicker">😊</button>
    </div>
    <div v-show="showEmojiPicker" class="emoji-picker-container" ref="pickerContainer"></div>
  </div>
</template>

<script setup lang="ts">
import data from '@emoji-mart/data'
import { Picker } from 'emoji-mart'
import { ref, watch } from 'vue'

interface Props {
  modelValue: string;
}

interface Emits {
  (e: 'update:modelValue', value: string): void;
  (e: 'blur', value: string): void;
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();
const showEmojiPicker = ref(false);
const pickerContainer = ref<HTMLDivElement | null>(null);

watch(showEmojiPicker, async (newValue) => {
  if (newValue && pickerContainer.value) {
    const picker = new Picker({
      data,
      onEmojiSelect,
    })
    pickerContainer.value.innerHTML = ''
    pickerContainer.value.appendChild(picker)
  } else if (!newValue && pickerContainer.value) {
    pickerContainer.value.innerHTML = ''
  }
});

const handleInput = (event: Event) => {
  const target = event.target as HTMLDivElement;
  emit('update:modelValue', target.innerHTML);
};

const handleBlur = (event: Event) => {
  const target = event.target as HTMLDivElement;
  emit('blur', target.innerHTML);
};

const toggleEmojiPicker = () => {
  showEmojiPicker.value = !showEmojiPicker.value;
};

const onEmojiSelect = (emoji: any) => {
  const div = document.querySelector('.editable-div') as HTMLDivElement;
  if (div) {
    div.innerHTML += emoji.native;
    emit('update:modelValue', div.innerHTML);
  }
  showEmojiPicker.value = false;
};
</script>

<style scoped>
.input-wrapper {
  width: 100%;
  position: relative;
}

.input-container {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.editable-div {
  flex: 1;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  padding: 8px 12px;
  min-height: 100px;
  max-height: 300px;
  overflow-y: auto;
  outline: none;
  line-height: 1.5;
  font-size: 14px;
}

.editable-div:focus {
  border-color: #409eff;
}

.emoji-button {
  padding: 5px 10px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  background: white;
  cursor: pointer;
  font-size: 16px;
}

.emoji-button:hover {
  background: #f5f7fa;
}

.emoji-picker-container {
  position: absolute;
  right: 0;
  top: 100%;
  margin-top: 8px;
  z-index: 1000;
}
</style>