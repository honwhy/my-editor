<template>
  <div class="input-wrapper">
    <div class="input-container">
      <div
        class="editable-div"
        contenteditable="true"
        placeholder="请输入内容..."
        @blur="handleBlur"
        @input="handleInput"
        @paste="handlePaste"
        :innerHTML="modelValue"
        :class="{'hide-word-limit': !showWordLimit}"
        :data-length="textLength"
        :data-maxlength="maxlength"
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
  maxlength: number;
  showWordLimit: boolean;
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
const textLength = ref(0); // 添加文本长度计数

// 计算纯文本长度的函数
const calculateTextLength = (html: string): number => {
  const tempDiv = document.createElement('div');
  tempDiv.innerHTML = html;
  let text = tempDiv.textContent || '';
  const tags = tempDiv.querySelectorAll('.tag');
  return text.length + tags.length * 10;
};
// 添加 selectionchange 事件监听
onMounted(() => {
  document.addEventListener('selectionchange', handleSelectionChange);
  textLength.value = calculateTextLength(props.modelValue);
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
// 处理输入事件
const handleInput = (event: Event) => {
  const target = event.target as HTMLDivElement;
  const newLength = calculateTextLength(target.innerHTML);
  
  // if (props.maxlength > 0 && newLength > props.maxlength) {
  //   if (lastRange.value) {
  //     const sel = window.getSelection();
  //     if (sel) {
  //       event.preventDefault();
  //       target.innerHTML = props.modelValue;
  //       setTimeout(() => {
  //         sel.removeAllRanges();
  //         sel.addRange(lastRange.value!);
  //       }, 0);
  //       return;
  //     }
  //   }
  // }
  
  textLength.value = newLength;
  // emit('update:modelValue', target.innerHTML);
};
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
  const currentLength = textLength.value;
  const maxLength = props.maxlength;
  // 检查粘贴后是否会超出字符限制 tag 表示的长度设定为10
  if (maxLength > 0 && currentLength + 10 > maxLength) {
    return;
  }
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

  // 更新 textLength 和 modelValue
  textLength.value = calculateTextLength(div.innerHTML);
  showTagDropdown.value = false;
};

// 在onEmojiSelect方法最后添加更新modelValue的代码
// 修改onEmojiSelect方法末尾
const onEmojiSelect = (emoji: any) => {
  const div = document.querySelector('.editable-div') as HTMLDivElement;
  if (!div) return;
  const currentLength = textLength.value;
  const maxLength = props.maxlength;
  // 检查粘贴后是否会超出字符限制
  if (maxLength > 0 && currentLength + emoji.native.length > maxLength) {
    return;
  }
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

  // 更新 textLength 和 modelValue
  textLength.value = calculateTextLength(div.innerHTML);
  // 更新 modelValue
  // emit('update:modelValue', div.innerHTML);
  showEmojiPicker.value = false;
};
const handlePaste = (event: ClipboardEvent) => {
  // 阻止默认粘贴行为
  event.preventDefault();
  const div = document.querySelector('.editable-div') as HTMLDivElement;
  if (!div) return;
  // 获取纯文本内容
  const text = event.clipboardData?.getData('text/plain') || '';
  const currentLength = textLength.value;
  const maxLength = props.maxlength;
  // 检查粘贴后是否会超出字符限制
  if (maxLength > 0 && currentLength + text.length > maxLength) {
    // 如果会超出，只粘贴能容纳的部分
    const allowedText = text.substring(0, maxLength - currentLength);
    if (allowedText.length === 0) return; // 如果已经达到最大长度，不粘贴

    // 将允许的文本插入到当前光标位置
    if (lastRange.value) {
      const sel = window.getSelection();
      if (sel) {
        sel.removeAllRanges();
        sel.addRange(lastRange.value);
        
        // 替换选中内容或插入允许的文本
        const range = sel.getRangeAt(0);
        range.deleteContents();
        const textNode = document.createTextNode(allowedText);
        range.insertNode(textNode);
        
        // 重新设置选区范围到插入的文本后面
        const newRange = document.createRange();
        newRange.setStartAfter(textNode);
        newRange.setEndAfter(textNode);
        sel.removeAllRanges();
        sel.addRange(newRange);
        lastRange.value = newRange.cloneRange(); // 更新保存的选区
        
        // 更新 textLength 和 modelValue
        textLength.value = calculateTextLength(div.innerHTML);
        // emit('update:modelValue', div.innerHTML);
      }
    }
  } else {
    // 将纯文本插入到当前光标位置
    if (lastRange.value) {
      const sel = window.getSelection();
      if (sel) {
        sel.removeAllRanges();
        sel.addRange(lastRange.value);
        
        // 替换选中内容或插入纯文本
        const range = sel.getRangeAt(0);
        range.deleteContents();
        const textNode = document.createTextNode(text);
        range.insertNode(textNode);
        
        // 重新设置选区范围到插入的文本后面
        const newRange = document.createRange();
        newRange.setStartAfter(textNode);
        newRange.setEndAfter(textNode);
        sel.removeAllRanges();
        sel.addRange(newRange);
        lastRange.value = newRange.cloneRange(); // 更新保存的选区
        
        // 更新 textLength 和 modelValue
        textLength.value = calculateTextLength(div.innerHTML);
        // emit('update:modelValue', div.innerHTML);
      }
    }
  }
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
  position: relative;
}

.editable-div:empty::before {
  content: attr(placeholder);
  color: #a8abb2;
  position: absolute;
  pointer-events: none;
}

.editable-div:focus {
  border-color: #409eff;
}
.editable-div::after {
  content: attr(data-length) '/' attr(data-maxlength);
  display: attr(data-show-limit);
  text-align: right;
  font-size: 12px;
  color: #a8abb2;
  position: absolute;
  bottom: 0;
  right: 8px;
}
.hide-word-limit::after {
  display: none;
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