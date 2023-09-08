<template>
  <el-upload
    class="avatar-uploader"
    :action="apiUploadImg"
    :before-upload="beforeUpload"
    :show-file-list="false"
    accept=".png, .jpg, .jpeg"
  >
    <img :src="props.avatar" class="avatar" alt="头像" />
  </el-upload>
  <Modal
    v-model:visible="showCrop"
    width="50%"
    mobile-width="100%"
    title="裁剪图片(移动圆圈可裁剪图片最佳位置)"
    @submit="handleCropSubmit"
    @cancel="handleCropCancel"
  >
    <cropper ref="cropperRef" class="cropper" :src="tempUrl" :stencil-component="CircleStencil" />
  </Modal>
</template>

<script setup lang="ts">
import { uploadImage } from '@/api/file'
import Modal from '@/components/Modal/index.vue'
import { currentEnvConfig } from '@/config'
import { UPLOAD_IMG_TYPES } from '@/constant/common'
import * as url from '@/utils/url'
import type { UploadRawFile } from 'element-plus'
import { ElMessage } from 'element-plus'
import { ref } from 'vue'
import { CircleStencil, Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'

const emit = defineEmits(['updateAvatar'])
const props = defineProps({
  avatar: String
})

const baseURL = currentEnvConfig.uploadBaseURL
const apiUploadImg = url.join(baseURL, '/chato/api/file/upload/file')

const cropperRef = ref(null)
const tempUrl = ref('')
const showCrop = ref(false)
const originalFileType = ref('')

function getMimeType(fileType: string): string {
  return `image/.${fileType}`
}

const beforeUpload = (rawFile: UploadRawFile) => {
  const fileType = rawFile.name.substring(rawFile.name.lastIndexOf('.')).toLowerCase()
  originalFileType.value = fileType
  if (!UPLOAD_IMG_TYPES.includes(fileType)) {
    ElMessage.error('不支持{fileType}格式')
    return false
  }
  const isLt2M = rawFile.size / 1024 / 1024 < 2
  if (!isLt2M) {
    ElMessage.error('上传图片大小不能超过 2MB!')
    return false
  }
  tempUrl.value = URL.createObjectURL(rawFile)
  showCrop.value = true
  return true
}

const handleCropSubmit = () => {
  if (cropperRef.value) {
    const ImgAfterCrop = cropperRef.value.getResult().canvas
    const mimeType = getMimeType(originalFileType.value)

    const form = new FormData()
    ImgAfterCrop.toBlob(async (blob) => {
      form.append('file', blob)
      const res = await uploadImage(form)
      emit('updateAvatar', res.data.data.url)
    }, mimeType)

    URL.revokeObjectURL(tempUrl.value)
    showCrop.value = false
  }
}

const handleCropCancel = () => {
  URL.revokeObjectURL(tempUrl.value)
  showCrop.value = false
}
</script>

<style scoped lang="scss">
.avatar-uploader .avatar {
  overflow: hidden;
}

.avatar-uploader {
  :deep(.el-upload) {
    height: 48.5px;
    width: 48.5px;
    box-sizing: border-box;
    border: 1px solid #c6c7cb;
    border-radius: 50%;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
}
.preview {
  border: dashed 1px rgba(255, 255, 255, 0.25);
}

.avatar-uploader {
  :deep(.el-upload:hover:after) {
    content: '';
    position: absolute;
    color: #fff;
    z-index: 9999;
    justify-content: center;
    align-items: center;
    object-fit: contain;
    background-color: rgba(0, 0, 0, 0.5);
    background-image: url('@/assets/icons/picture.svg');
    background-size: 30px;
    background-repeat: no-repeat;
    background-position: center;
  }
}
</style>
