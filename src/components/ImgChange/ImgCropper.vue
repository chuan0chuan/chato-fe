<template>
  <Modal
    v-model:visible="internalVisible"
    width="50%"
    mobile-width="100%"
    title="裁剪图片(移动圆圈可裁剪图片最佳位置)"
    @cancel="handleCropCancel"
  >
    <cropper ref="cropperRef" class="cropper" :src="avatarUrl" :default-size="defaultSize" />
  </Modal>
</template>
<script setup lang="ts">
import Modal from '@/components/Modal/index.vue'
import { computed, ref } from 'vue'
import { Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'

const props = withDefaults(
  defineProps<{
    avatarUrl: string
    cropVisible: boolean
    fileName: string // 接收传入的文件名
    fileType: string // 接收传入的文件类型
  }>(),
  {
    cropVisible: false
  }
)

const handleCropCancel = () => {
  console.log('Dialog was cancelled!')
}

const handleCropSubmit = () => {
  console.log('Dialog submitted!')
}
const cropperRef = ref(null)

// const saveImg = () => {
//   if (cropperRef.value) {
//     const canvas = cropperRef.value.getResult()
//     canvas.toBlob((blob) => {
//       const file = new File([blob], props.fileName, { type: props.fileType })
//       console.log(file)
//     })
//   }
// }

const saveImg = async () => {
  if (cropperRef.value) {
    const croppedImageUrl = cropperRef.value.getResult() // 获取裁剪后的图像的URL
    try {
      const response = await fetch(croppedImageUrl)
      const blob = await response.blob()

      const file = new File([blob], props.fileName, { type: props.fileType })
      console.log(file)
    } catch (error) {
      console.error('Error fetching the blob from the URL:', error)
    }
  }
}

const defaultSize = ref({
  width: 400,
  height: 400
})

const emit = defineEmits(['update:cropVisible'])

const internalVisible = computed({
  get: () => props.cropVisible,
  set: (v) => emit('update:cropVisible', v)
})
</script>
<style scoped lang="scss"></style>
