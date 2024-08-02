<template>
    <div>
        <el-upload action="http://gulimall.oss-cn-shanghai.aliyuncs.com" :data="dataObj" list-type="picture-card"
            :file-list="fileList" :before-upload="beforeUpload" :on-remove="handleRemove"
            :on-success="handleUploadSuccess" :on-preview="handlePreview" :limit="maxCount" :on-exceed="handleExceed">
            <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :model-value="dialogVisible" @update:model-value="dialogVisible = $event">
            <img width="100%" :src="dialogImageUrl" alt />
        </el-dialog>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue';
import { policy } from "./policy";
import { getUUID } from '@/utils';
import { ElMessage } from 'element-plus';

const props = defineProps({
    value: Array,
    maxCount: {
        type: Number,
        default: 30
    }
});

const dataObj = ref({
    policy: "",
    signature: "",
    key: "",
    ossaccessKeyId: "",
    dir: "",
    host: "",
    uuid: ""
});
const dialogVisible = ref(false);
const dialogImageUrl = ref(null);

const fileList = computed(() => {
    let fileList = [];
    for (let i = 0; i < props.value.length; i++) {
        fileList.push({ url: props.value[i] });
    }
    return fileList;
});

const emit = defineEmits(['input']);

const emitInput = (fileList) => {
    let value = fileList.map(file => file.url);
    emit('input', value);
};

const handleRemove = (file, fileList) => {
    emitInput(fileList);
};

const handlePreview = (file) => {
    dialogVisible.value = true;
    dialogImageUrl.value = file.url;
};

const beforeUpload = (file) => {
    return new Promise((resolve, reject) => {
        policy().then(response => {
            dataObj.value.policy = response.data.policy;
            dataObj.value.signature = response.data.signature;
            dataObj.value.ossaccessKeyId = response.data.accessid;
            dataObj.value.key = response.data.dir + "/" + getUUID() + "_${filename}";
            dataObj.value.dir = response.data.dir;
            dataObj.value.host = response.data.host;
            resolve(true);
        }).catch(err => {
            reject(false);
        });
    });
};

const handleUploadSuccess = (res, file) => {
    fileList.value.push({
        name: file.name,
        url: dataObj.value.host + "/" + dataObj.value.key.replace("${filename}", file.name)
    });
    emitInput(fileList.value);
};

const handleExceed = (files, fileList) => {
    ElMessage({
        message: `最多只能上传${props.maxCount}张图片`,
        type: 'warning',
        duration: 1000
    });
};

watch(() => props.value, (newValue) => {
    if (!newValue.length) {
        fileList.value = [];
    }
}, { deep: true });
</script>

<style></style>