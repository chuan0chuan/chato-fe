<template>
  <el-upload
    class="avatar-uploader"
    :multiple="false"
    :action="apiUploadImg"
    :before-upload="beforeUpload"
    :show-file-list="false"
    accept=".png, .jpg, .jpeg"
  >
    <img :src="avatarUrl" class="avatar" />
    <!-- <img v-else class="avatar" :src="avatarUrlProp" /> -->
    <!-- <el-icon v-else class="avatar-uploader-icon" :size="30"><Picture /></el-icon> -->
  </el-upload>
  <Modal
    v-model:visible="showImgCrop"
    width="50%"
    mobile-width="100%"
    title="裁剪图片(移动圆圈可裁剪图片最佳位置)"
    @cancel="handleCropCancel"
    @submit="handleCropSubmit"
  >
    <cropper ref="cropperRef" class="cropper" :src="tempUrl" :stencil-component="CircleStencil" />
  </Modal>
</template>

<script setup lang="ts">
import Modal from '@/components/Modal/index.vue'
import { currentEnvConfig } from '@/config'
import { UPLOAD_IMG_TYPES } from '@/constant/common'
import * as url from '@/utils/url'
import type { UploadFile, UploadRawFile } from 'element-plus'
import { ElMessage } from 'element-plus'
import { ref } from 'vue'
import { CircleStencil, Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'
const avatarUrlProp = defineProps(['avatarUrlProp'])
const emit = defineEmits(['updateAvatar'])

const baseURL = currentEnvConfig.uploadBaseURL
const apiUploadImg = url.join(baseURL, '/chato/api/file/upload/file')

const avatarUrl = ref('')
const cropperRef = ref(null)
const tempUrl = ref('')
const showImgCrop = ref(false)
const originalFileName = ref('')
const originalFileType = ref('')
const uploadRef = ref(null)
const currentFile = ref<UploadFile | null>(null)

function getMimeType(fileType: string): string {
  return `image/.${fileType}`
}

const beforeUpload = (rawFile: UploadRawFile) => {
  const fileType = rawFile.name.substring(rawFile.name.lastIndexOf('.')).toLowerCase()
  originalFileName.value = rawFile.name
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
  showImgCrop.value = true
  return true
}

const handleCropSubmit = () => {
  if (cropperRef.value) {
    const result = cropperRef.value.getResult().canvas
    const mimeType = getMimeType(originalFileType.value)

    result.toBlob((blob) => {
      const blobUrl = URL.createObjectURL(blob)
      const file: UploadFile = {
        uid: Math.random(),
        url: blobUrl,
        name: originalFileName.value,
        raw: blob,
        status: 'success'
      }

      avatarUrl.value = blobUrl
      emit('updateAvatar', avatarUrl.value)

      currentFile.value = file
      uploadRef.value.submit()
      URL.revokeObjectURL(blobUrl)
    }, mimeType)

    URL.revokeObjectURL(tempUrl.value)
    showImgCrop.value = false
  }
}

const handleCropCancel = () => {
  URL.revokeObjectURL(tempUrl.value)
  showImgCrop.value = false
}
// const handleCropSubmit = () => {
//   if (cropperRef.value) {
//     const result = cropperRef.value.getResult()
//     //avatar after crop
//     console.dir(result)
//     avatarUrl.value = result.canvas.toDataURL()
//     emit('updateAvatar', avatarUrl.value)
//   }
//   showImgCrop.value = false // 关闭裁剪Modal
// }

// const handleCropSubmit = () => {
//   if (cropperRef.value) {
//     const result = cropperRef.value.getResult()
//     console.log(tempUrl.value)
//     console.log(result)
//     // 创建 File 对象
//     // const uid = Date.now()
//     // const file = new File([blob], originalFileName.value, { type: originalFileType.value })
//     const file: UploadFile = {
//       uid: Math.random(),
//       url: result.canvas.toDataURL(),
//       name: originalFileName.value,
//       raw: null as any,
//       status: 'success'
//     }
//     //console.log(file)
//     currentFile.value = file
//     // 手动触发上传
//     uploadRef.value.submit()

//     showImgCrop.value = false
//   }
// }
</script>

<style scoped lang="scss">
.avatar-uploader .avatar {
  height: 55px;
  width: 55px;
  border-radius: 50%;
  border: 1px solid #d9d9d9;
  overflow: hidden;
}

.avatar-uploader {
  :deep(.el-upload) {
    height: 55px;
    width: 55px;
    box-sizing: border-box;
    border: 0.8px solid #c6c7cb;
    border-radius: 100%;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
}
.preview {
  border: dashed 1px rgba(255, 255, 255, 0.25);
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
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
