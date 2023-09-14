<template>
  <Modal v-model:visible="internalVisible" :footer="false" width="580px" mobile-width="100%">
    <UploadToCrop
      :colorAvatar="selectedColor"
      :avatar="props.avatar"
      :firstCharacter="props.firstCharacter"
      @updateAvatar="handleUpdateAvatar"
    ></UploadToCrop>
    <div class="main-container">
      <p>头像预览</p>
      <p class="!text-align:left">自定义头像颜色</p>
      <div class="colors-container">
        <div
          v-for="color in colors"
          :key="color"
          :style="{
            backgroundColor: color,
            boxShadow: selectedColor === color ? ' 0 0 0 6px #7C5CFC, 0 0 0 4px #fff' : 'none'
          }"
          class="color-box"
          @click="selectColor(color)"
        >
          {{ props.firstCharacter }}
        </div>
      </div>
      <el-button @click="handleCancel">取消</el-button>
      <el-button @click="handleSubmit">更改</el-button>
    </div>
  </Modal>
</template>

<script setup lang="ts">
import { uploadImage } from '@/api/file'
import Modal from '@/components/Modal/index.vue'
import UploadToCrop from '@/components/UploadToCrop/index.vue'
import { computed, ref, watch } from 'vue'
const storedUrl = ref(null)
const props = defineProps({
  firstCharacter: String,
  visible: Boolean,
  avatar: String
})

const emit = defineEmits(['update:visible', 'updateAvatar'])
const internalVisible = computed({
  get: () => props.visible,
  set: (v) => emit('update:visible', v)
})

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
  storedUrl.value = null
  selectedColor.value = color
}

watch(selectedColor, (newValue, oldValue) => {
  console.log('selectedColor has changed!', newValue)
})

const createColorBoxImage = (color, character) => {
  return new Promise(async (resolve, reject) => {
    const canvas = document.createElement('canvas')
    canvas.width = 100 // 设置适当的宽度和高度
    canvas.height = 100

    const ctx = canvas.getContext('2d')

    // 绘制背景
    ctx.fillStyle = color
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    ctx.fillStyle = 'white'
    ctx.font = '38px Source Han Sans'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.fillText(character, canvas.width / 2, canvas.height / 2)

    canvas.toBlob(async (blob) => {
      const form = new FormData()
      form.append('file', blob)
      try {
        const res = await uploadImage(form)
        const imgURL = res.data.data.url
        resolve(imgURL)
      } catch (error) {
        reject(error)
      }
    })
  })
}

const handleUpdateAvatar = (url) => {
  console.log('Received in child:', url)
  storedUrl.value = url
  emit('updateAvatar', url)
}

const handleSubmit = async () => {
  if (storedUrl.value) {
    emit('updateAvatar', storedUrl.value)
  } else {
    const imageUrl = await createColorBoxImage(selectedColor.value, props.firstCharacter)
    emit('updateAvatar', imageUrl)
  }
  emit('update:visible', false)
}

const handleCancel = () => {
  emit('update:visible', false)
}
</script>

<style scoped lang="scss">
.main-container {
  display: flex;
  flex-direction: column;
  align-self: stretch;
}

.avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  align-items: center;
}

.colors-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-bottom: 36px;
  padding: 0 4px;
}

.color-box {
  width: 56px;
  height: 56px;
  margin: 20px 30px 0 0;
  border-radius: 50%;
  cursor: pointer; /* Indicates clickable items */
  border: 2px solid transparent;
  /* Add flex properties for centering */
  display: flex;
  align-items: center; /* vertical centering */
  justify-content: center; /* horizontal centering */
  color: white;
  font-size: 18px;
  font-weight: normal;
  line-height: 35px;
  letter-spacing: 0px;
  &:nth-child(6n) {
    margin-right: 0 !important;
  }
}
</style>
