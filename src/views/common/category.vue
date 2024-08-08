<template>
    <el-tree v-loading="loading" style="max-width: 600px;margin-top: 12px;" :data="treeData" node-key="catId"
        :default-expanded-keys="expanded" :expand-on-click-node="false" :props="defaultProps"
        @node-click="handleNodeClick" ref="menuTree">
    </el-tree>
</template>

<script lang="ts" setup>
import { onMounted, ref } from "vue"
import baseService from "@/service/baseService"
import { ElMessage, ElTree } from "element-plus"
import { RefSymbol } from "@vue/reactivity";

const emit = defineEmits(["categoryData", "categoryTree"])

const emitInput = (node: any) => {
    emit('categoryData', node)
}

const emitTree = (tree: any) => {
    emit('categoryTree', tree)
}

onMounted(() => {
    getInfo()
})

interface Tree {
    [x: string]: any
    id: number
    label: string
    children?: Tree[]
}

const defaultProps = {
    children: 'children',
    label: 'name',
}

let expanded = ref<number[]>([])
let treeData = ref<Tree[]>([])
let loading = ref(true) // 控制加载状态的变量

const getInfo = () => {
    loading.value = true
    baseService.get("/product/category/page/tree").then((res) => {
        console.log("成功获取Tree data: ", res.data)
        treeData.value = res.data
        emitTree(res.data)
        loading.value = false // 数据加载完成后，关闭加载状态
    }).catch(() => {
        ElMessage.error("获取分类数据失败！")
        loading.value = false // 如果请求失败，也关闭加载状态
    });
};

const handleNodeClick = (node: any) => {
    emitInput(node)
}
</script>

<style>
.head {
    display: flex;
    font-size: 21px;
    font-family: "微软雅黑";
    font-weight: bold;
}
</style>