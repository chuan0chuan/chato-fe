<template>
  <el-upload
    class="avatar-uploader flex justify-center items-center"
    :action="apiUploadImg"
    :before-upload="beforeUpload"
    :show-file-list="false"
    accept=".png, .jpg, .jpeg"
  >
    <template v-if="!props.isLogo">
      <img v-if="isImageUrl(imgURL)" :src="imgURL" class="avatar" alt="头像" />
      <div
        v-else
        :style="{
          backgroundColor: props.colorAvatar
        }"
        class="colorAvatar"
      >
        {{ firstCharacter }}
      </div>
    </template>
    <img v-else :src="props.avatar" class="avatar" alt="头像" />
  </el-upload>
  <Modal
    v-model:visible="showCrop"
    width="50%"
    :footer="true"
    mobile-width="100%"
    title="裁剪图片(移动圆圈可裁剪图片最佳位置)"
    @submit="handleCropSubmit"
    @cancel="handleCropCancel"
  >
    <cropper
      v-if="!props.isLogo"
      ref="cropperRef"
      class="h-[400px]"
      :src="tempUrl"
      :stencil-component="CircleStencil"
    />
    <cropper v-else ref="cropperRef" class="h-[400px]" :src="tempUrl" />
  </Modal>
</template>

<script setup lang="ts">
import { uploadImage } from '@/api/file'
// import DefaultAvatar from '@/assets/img/avatar.png'
import Modal from '@/components/Modal/index.vue'
import { currentEnvConfig } from '@/config'
import { UPLOAD_IMG_TYPES } from '@/constant/common'
import { isImageUrl } from '@/utils/reg'
import * as url from '@/utils/url'
import type { UploadRawFile } from 'element-plus'
import { ElMessage } from 'element-plus'
import { computed, ref, watch, watchEffect } from 'vue'
import { CircleStencil, Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'

const emit = defineEmits(['updateAvatar', 'clearColor'])
const props = defineProps({
  logoAavatar: String,
  avatar: String,
  colorAvatar: {
    type: String,
    default: null
  },
  firstCharacter: String,
  isLogo: {
    type: Boolean,
    default: false
  }
})

const internalAvatar = computed(() => {
  return props.avatar
})

const baseURL = currentEnvConfig.uploadBaseURL
const apiUploadImg = url.join(baseURL, '/chato/api/file/upload/file')

const cropperRef = ref(null)
const tempUrl = ref('')
const showCrop = ref(false)
const originalFileType = ref('')
const imgURL = ref('')

function getMimeType(fileType: string): string {
  return `image/.${fileType}`
}

watchEffect(() => {
  console.log('COLOR!!!!!!!!!!!!' + props.colorAvatar)
})

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
      console.log(res)
      emit('updateAvatar', res.data.data.url)
    }, mimeType)
    showCrop.value = false
    URL.revokeObjectURL(tempUrl.value)
  }
}

const handleCropCancel = () => {
  URL.revokeObjectURL(tempUrl.value)
  showCrop.value = false
}

watch(
  () => props.colorAvatar,
  (v) => {
    //not URL, is color
    imgURL.value = v
  }
)

watch(
  () => props.avatar,
  (v) => {
    imgURL.value = v
  }
)
</script>

<style scoped lang="scss">
.avatar-uploader .avatar {
  overflow: hidden;
}

.avatar-uploader {
  :deep(.el-upload) {
    height: 64px;
    width: 64px;
    box-sizing: border-box;
    border: 1px solid #c6c7cb;
    border-radius: 50%;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    color: #fff;
  }
}
.preview {
  border: dashed 1px rgba(255, 255, 255, 0.25);
}
.colorAvatar {
  width: 64px;
  height: 64px;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center; /* vertical centering */
  justify-content: center; /* horizontal centering */
  font-size: 20px;
}
.avatar-uploader {
  :deep(.el-upload:hover:after) {
    display: inline-block;
    content: '';
    width: 100%;
    height: 100%;
    position: absolute;
    z-index: 9999;
    justify-content: center;
    align-items: center;
    object-fit: contain;
    // opacity: 0.5;
    color: #fff;
    background-color: rgba(0, 0, 0, 0.5);
    background-image: url('@/assets/icons/picture.svg');
    background-size: 30px;
    background-repeat: no-repeat;
    background-position: center;
  }
}
</style>
