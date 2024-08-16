<template>
    <div>
        <el-upload class="avatar-uploader" action="http://gulimall-losgai.oss-cn-nanjing.aliyuncs.com" :data="dataObj"
            list-type="picture-card" :multiple="true" :show-file-list="true" :file-list="fileList"
            :before-upload="beforeUpload" :on-remove="handleRemove" :on-success="handleUploadSuccess"
            :on-preview="handlePictureCardPreview">
            <el-icon>
                <Plus />
            </el-icon>
        </el-upload>
        <el-dialog v-model="dialogVisible">
            <img w-full :src="dialogImageUrl" alt="Preview Image" />
        </el-dialog>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { ElMessage } from 'element-plus'
import baseService from "@/service/baseService"
import { getUuid } from '@/utils/utils'

const props = defineProps({
    logos: {
        type: Array,
        default: () => [],
    }
})

const dialogImageUrl = ref('')

const dialogVisible = ref(false)

const emit = defineEmits(['changeLogos'])

const fileList = ref(props.logos.map((logo: string) => ({ name: logo, url: logo })))

const dataObj = ref({
    policy: '',
    signature: '',
    key: '',
    OSSAccessKeyId: '',
    dir: '',
    host: '',
});

const emitInput = (val: string[]) => {
    emit('changeLogos', val)
}

const handleRemove = (file: any, updatedFileList: any) => {
    fileList.value = updatedFileList;
    emitInput(fileList.value.map(file => file.url))
    // console.log("文件被移除", fileList.value)
}

const beforeUpload = async (file: any) => {
    try {
        const response = await baseService.get('/thirdparty/oss/policy');
        dataObj.value = {
            policy: response.data.policy,
            signature: response.data.signature,
            OSSAccessKeyId: response.data.accessid,
            key: response.data.dir + getUuid() + '_${filename}',
            dir: response.data.dir,
            host: response.data.host,
        };
        return true;
    } catch (err) {
        ElMessage.error('上传图片失败!');
        return false;
    }
}

const handleUploadSuccess = (response: any, file: any) => {
    ElMessage.success('上传图片成功!')
    const newUrl = dataObj.value.host + '/' + dataObj.value.key.replace("${filename}", file.name)
    fileList.value.push({ name: file.name, url: newUrl })
    emitInput(fileList.value.map(file => file.url))
}

const handlePictureCardPreview: UploadProps['onPreview'] = (uploadFile) => {
    dialogImageUrl.value = uploadFile.url!
    dialogVisible.value = true
}

watch(() => props.logos, (newLogos) => {
    fileList.value = newLogos.map(logo => ({ name: logo, url: logo }))
}, { deep: true })

// watch(() => fileList, (newLogos) => {
//     console.log("文件列表 变化", fileList.value)
// }, { deep: true })
</script>

<style></style>
