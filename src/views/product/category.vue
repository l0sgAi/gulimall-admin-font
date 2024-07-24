<template>
    <div>
        <a class="head">商品分类管理</a>
        <el-tree v-loading="loading" style="max-width: 600px;margin-top: 12px;" :data="treeData" show-checkbox
            node-key="catId" :default-expanded-keys="expanded" :expand-on-click-node="false" :props="defaultProps"
            @node-click="handleNodeClick">
            <template #default="{ node, data }">
                <span class="custom-tree-node">
                    <span>{{ node.label }}</span>
                    <span>
                        <el-button v-if="node.level <= 2" type="success" @click="append(data)" link> 新增子节点
                        </el-button>
                        <el-button v-if="!data.children || data.children.length === 0" type="danger"
                            style="margin-left: 8px;" @click="remove(node, data)" link>
                            删除节点
                        </el-button>
                    </span>
                </span>
            </template>
        </el-tree>
    </div>
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref } from "vue"
import baseService from "@/service/baseService"
import type Node from 'element-plus/es/components/tree/src/model/node'
import { ElMessage, ElMessageBox } from "element-plus"
onMounted(() => {
    getInfo()
});

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
        loading.value = false // 数据加载完成后，关闭加载状态
    }).catch(() => {
        ElMessage.error("获取分类数据失败！")
        loading.value = false // 如果请求失败，也关闭加载状态
    });
};

const handleNodeClick = (data: Tree) => {
    console.log(data)
}

const append = (data: Tree) => {
    // const newChild = { id: id++, label: 'testtest', children: [] }
    // if (!data.children) {
    //     data.children = []
    // }
    // data.children.push(newChild)
    // treeData.value = [...treeData.value]
    console.log("append", data)
}

const remove = (node: Node, data: Tree) => {
    ElMessageBox.confirm(`此操作将删除【${data.name}】菜单，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        doRemove(node, data)
    }).catch(() => {
        ElMessage.info('已取消删除')
    });
}

const doRemove = (node: Node, data: Tree) => {
    let ids = [data.catId]
    console.log("ids", ids)
    baseService.delete("/product/category/delete", ids).then(() => {
        getInfo()
        console.log("成功删除分类数据: ", ids)
        ElMessage.success("删除分类成功！")
        expanded.value = [node.parent.data.catId]
    }).catch(() => {
        ElMessage.error("删除分类数据失败！")
    });
}
</script>

<style>
.custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
}

.head {
    display: flex;
    font-size: 25px;
    font-family: "微软雅黑";
    font-weight: bold;
}
</style>