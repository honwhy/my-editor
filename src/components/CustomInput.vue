<template>
  <div class="input-wrapper">
    <div class="input-container">
      <div
        class="editable-div"
        contenteditable="true"
        @blur="handleBlur"
        :innerHTML="modelValue"
      ></div>
      <div class="buttons">
        <button class="emoji-button" @click="toggleEmojiPicker">😊</button>
        <button class="tag-button" @click="toggleTagDropdown">插入Tag</button>
        <div v-show="showTagDropdown" class="tag-dropdown">
          <div class="tag-item" @click="insertTag('连接1')">连接1</div>
          <div class="tag-item" @click="insertTag('连接2')">连接2</div>
          <div class="tag-item" @click="insertTag('连接3')">连接3</div>
        </div>
      </div>
    </div>
    <div v-show="showEmojiPicker" class="emoji-picker-container" ref="pickerContainer"></div>
  </div>
</template>

<script setup lang="ts">
import data from '@emoji-mart/data'
import { Picker } from 'emoji-mart'
import { ref, watch, onMounted, onUnmounted } from 'vue'
const showTagDropdown = ref(false);

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
const emitUpdate = () => {
  const target = document.querySelector('.editable-div');
  if (!target) return;
  emit('update:modelValue', target.innerHTML);
};

const handleBlur = (event: Event) => {
  const target = event.target as HTMLDivElement;
  emit('update:modelValue', target.innerHTML);
};

const toggleEmojiPicker = () => {
  showEmojiPicker.value = !showEmojiPicker.value;
  if (showEmojiPicker.value) {
    emitUpdate()
  }
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
const toggleTagDropdown = () => {
  showTagDropdown.value = !showTagDropdown.value;
  // 关闭emoji选择器
  if (showTagDropdown.value) {
    showEmojiPicker.value = false;
  }
  if (showTagDropdown.value) {
    emitUpdate()
  }
};

const insertTag = (tagName: string) => {
  const div = document.querySelector('.editable-div') as HTMLDivElement;
  if (!div) return;

  div.focus(); // 先获取焦点

  // 创建tag元素
  const tagElement = document.createElement('hr');
  tagElement.className = 'tag';
  tagElement.setAttribute('data-value', tagName);
  tagElement.contentEditable = 'false';
  // 如果有之前保存的选区，恢复它
  if (lastRange.value) {
    const sel = window.getSelection();
    if (sel) {
      sel.removeAllRanges();
      sel.addRange(lastRange.value);
      
      // 替换选中内容或插入tag
      const range = sel.getRangeAt(0);
      range.deleteContents();
      range.insertNode(tagElement);
    
      
      // 重新设置选区范围到插入的tag后面
      const newRange = document.createRange();
      newRange.setStartAfter(tagElement);
      newRange.setEndAfter(tagElement);
      sel.removeAllRanges();
      sel.addRange(newRange);
      lastRange.value = newRange.cloneRange(); // 更新保存的选区
    }
  } else {
    // 如果没有选区，就追加到末尾
    div.appendChild(tagElement);
    
    // 设置光标到末尾
    const range = document.createRange();
    const sel = window.getSelection();
    range.setStartAfter(tagElement);
    range.setEndAfter(tagElement);
    sel?.removeAllRanges();
    sel?.addRange(range);
    lastRange.value = range.cloneRange(); // 更新保存的选区
  }
  showTagDropdown.value = false;
};

// 在onEmojiSelect方法最后添加更新modelValue的代码
// 修改onEmojiSelect方法末尾
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
  // emit('update:modelValue', div.innerHTML);
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

.buttons {
  display: flex;
  flex-direction: column;
  gap: 8px;
  position: relative;
}

.emoji-button,
.tag-button {
  padding: 5px 10px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  background: white;
  cursor: pointer;
  font-size: 14px;
  width: 100%;
}

.emoji-button:hover,
.tag-button:hover {
  background: #f5f7fa;
}

.tag-dropdown {
  position: absolute;
  right: 0;
  top: 100%;
  margin-top: 4px;
  background: white;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  z-index: 1000;
  min-width: 100px;
}

.tag-item {
  padding: 8px 12px;
  cursor: pointer;
}

.tag-item:hover {
  background: #f5f7fa;
}

:deep(.tag) {
  display: inline-block;
  height: 22px;
  width: auto;
  min-width: 60px;
  background-color: #ecf5ff;
  border: 1px solid #d9ecff;
  border-radius: 4px;
  margin: 0 4px;
  position: relative;
  vertical-align: middle;
}

:deep(.tag::before) {
  content: attr(data-value);
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #409eff;
  font-size: 12px;
  white-space: nowrap;
}

/* 移除 hr 默认样式 */
:deep(.tag:not([type="text"])) {
  border: none;
  margin: 0;
  padding: 0;
  overflow: visible;
}
</style>