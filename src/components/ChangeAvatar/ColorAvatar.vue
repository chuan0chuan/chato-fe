<template>
  <div class="container">
    <div class="avatar" :style="{ backgroundColor: selectedColor }"></div>
    <div
      v-for="color in colors"
      :key="color"
      :style="{ backgroundColor: color }"
      class="img-box"
      @click="selectColor(color)"
      :class="{ selected: selectedColor === color }"
    ></div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const colors = [
  '#FF0000',
  '#00FF00',
  '#0000FF',
  '#FFFF00',
  '#00FFFF',
  '#FF00FF',
  '#C0C0C0',
  '#808080',
  '#800000',
  '#808000',
  '#008000',
  '#008080'
]
const selectedColor = ref(null)

const selectColor = (color) => {
  selectedColor.value = color
}
</script>

<style>
.container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.img-box {
  width: 50px;
  height: 50px;
  margin: 5px;
  border-radius: 50%; /* 圆形样式 */
  transition: border 0.3s; /* 平滑的边框过渡效果 */
}
.img-box.selected {
  border: 2px solid purple;
  box-sizing: border-box; /* 以确保元素大小不受边框的影响 */
}
</style>
