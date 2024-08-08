<template>
    <div>
        <el-upload class="avatar-uploader" action="http://gulimall-losgai.oss-cn-nanjing.aliyuncs.com" :data="dataObj"
            list-type="picture" :multiple="false" :show-file-list="false" :file-list="fileList"
            :before-upload="beforeUpload" :on-remove="handleRemove" :on-success="handleUploadSuccess">
            <img v-if="imageUrl" :src="imageUrl" class="avatar" />
            <el-icon v-else class="avatar-uploader-icon">
                <Plus />
            </el-icon>
        </el-upload>
        <div style="margin-left: 8px;" class="el-upload__tip">只能上传jpg/png文件，且不超过10MB</div>
    </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { ElMessage } from 'element-plus'
import baseService from "@/service/baseService"
import { getUuid } from '@/utils/utils'

const props = defineProps({
    logo: {
        type: String,
        default: '',
    }
})

const emit = defineEmits(['changeLogo'])

const imageUrl = ref(props.logo)

const imageName = computed(() => {
    if (props.logo != null && props.logo !== '') {
        return props.logo.substr(props.logo.lastIndexOf("/") + 1);
    } else {
        return null;
    }
})

const fileList = ref([{ name: imageName.value, url: imageUrl.value }])

// const showFileList = computed({
//     get: () => props.logo !== null && props.logo !== '' && props.logo !== undefined,
//     set: (newValue) => { }
// })

const dataObj = ref({
    policy: '',
    signature: '',
    key: '',
    OSSAccessKeyId: '',
    dir: '',
    host: '',
});

const emitInput = (val: string) => {
    emit('changeLogo', val)
}

const handleRemove = (file: any, fileList: any) => {
    emitInput('');
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

const handleUploadSuccess = (res: any, file: any) => {
    ElMessage.success('上传图片成功!')
    fileList.value.pop();
    const newUrl = dataObj.value.host + '/' + dataObj.value.key.replace("${filename}", file.name)
    fileList.value.push({ name: file.name, url: newUrl })
    imageUrl.value = fileList.value[0].url
    emitInput(fileList.value[0].url)
}

watch(() => props.logo, (newValue) => {
    fileList.value = [{ name: imageName.value, url: newValue }]
    imageUrl.value = newValue
})
</script>

<style>
.avatar-uploader .el-upload {
    border: 1px dashed var(--el-border-color);
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
    transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
    border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    text-align: center;
}
</style>