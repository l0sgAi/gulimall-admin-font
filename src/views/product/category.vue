<template>
    <el-tree style="max-width: 600px" :data="data" :props="defaultProps" @node-click="handleNodeClick" />
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus"

onMounted(() => {
    getInfo();
});

interface Tree {
    label: string
    children?: Tree[]
}

let data = ref<Tree[]>([])
const getInfo = () => {
    baseService.get("/product/category/page/tree").then((res) => {
        console.log("成功获取Tree data: ", res.data)
        data.value = res.data;
    });
};

const handleNodeClick = (data: Tree) => {
    console.log(data)
}

const defaultProps = {
    children: 'children',
    label: 'name',
}
</script>