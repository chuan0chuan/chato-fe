<template>
  <Modal v-model:visible="internalVisible" :footer="false" width="40%" mobile-width="100%">
    <!-- <div class="avatar" :style="{ backgroundColor: selectedColor }"></div> -->
    <UploadToCrop
      :avatar="settingForm.domain.avatar"
      @updateAvatar="(v) => onAvatarUpdate(v)"
      :colorAvatar="selectedColor"
      @clearColor="selectedColor = null"
    ></UploadToCrop>
    <div class="main-container">
      <p>头像预览</p>
      <p class="!text-align:left">自定义头像颜色</p>
      <div class="colors-container">
        <div
          v-for="color in colors"
          :key="color"
          :style="{ backgroundColor: color }"
          class="color-box"
          @click="selectColor(color)"
        ></div>
      </div>
    </div>
  </Modal>
</template>

<script setup lang="ts">
import Modal from '@/components/Modal/index.vue'
import UploadToCrop from '@/components/UploadToCrop/index.vue'
import { ref } from 'vue'

const internalVisible = ref(true)
internalVisible.value = true
const colors = [
  '#4462C0',
  '#7992D9',
  '#56B9BB',
  '#63AD4C',
  '#7C5CF5',
  '#CF83D8',
  '#D9675B',
  '#E7A951',
  '#F2DA68',
  '#C0CEF3',
  '#666666',
  '#333333'
]

const selectedColor = ref(colors[0]) // Default to the first color

const selectColor = (color) => {
  selectedColor.value = color
}
</script>

<style>
.main-container {
  display: flex;
  flex-direction: column;
  align-self: stretch;
}

.avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  margin-bottom: 20px;
  align-items: center;
}

.colors-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.color-box {
  width: 50px;
  height: 50px;
  margin: 5px;
  border-radius: 50%;
  cursor: pointer; /* Indicates clickable items */
}
</style>
