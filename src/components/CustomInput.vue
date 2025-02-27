<template>
  <div class="input-wrapper">
    <div class="input-container">
      <div
        class="editable-div"
        contenteditable="true"
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
import { ref, watch, onMounted, onUnmounted } from 'vue'

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
const selection = ref<Selection | null>(null);
const lastRange = ref<Range | null>(null);

// 添加 selectionchange 事件监听
onMounted(() => {
  document.addEventListener('selectionchange', handleSelectionChange);
});

onUnmounted(() => {
  document.removeEventListener('selectionchange', handleSelectionChange);
});
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
const handleSelectionChange = () => {
  const sel = window.getSelection();
  if (!sel) return;

  const div = document.querySelector('.editable-div');
  if (!div) return;

  // 检查选区是否在 editable-div 内
  if (sel.rangeCount > 0) {
    const range = sel.getRangeAt(0);
    if (div.contains(range.commonAncestorContainer)) {
      selection.value = sel;
      lastRange.value = range.cloneRange();
    }
  }
};

const onEmojiSelect = (emoji: any) => {
  const div = document.querySelector('.editable-div') as HTMLDivElement;
  if (!div) return;

  div.focus(); // 先获取焦点

  // 如果有之前保存的选区，恢复它
  if (lastRange.value) {
    const sel = window.getSelection();
    if (sel) {
      sel.removeAllRanges();
      sel.addRange(lastRange.value);
      
      // 替换选中内容或插入 emoji
      const range = sel.getRangeAt(0);
      range.deleteContents();
      const textNode = document.createTextNode(emoji.native);
      range.insertNode(textNode);
      
      // 重新设置选区范围到插入的 emoji 后面
      const newRange = document.createRange();
      newRange.setStartAfter(textNode);
      newRange.setEndAfter(textNode);
      sel.removeAllRanges();
      sel.addRange(newRange);
      lastRange.value = newRange.cloneRange(); // 更新保存的选区
    }
  } else {
    // 如果没有选区，就追加到末尾
    const textNode = document.createTextNode(emoji.native);
    div.appendChild(textNode);
    
    // 设置光标到末尾
    const range = document.createRange();
    const sel = window.getSelection();
    range.setStartAfter(textNode);
    range.setEndAfter(textNode);
    sel?.removeAllRanges();
    sel?.addRange(range);
    lastRange.value = range.cloneRange(); // 更新保存的选区
  }

  // 更新 modelValue
  emit('update:modelValue', div.innerHTML);
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