<template>
    <div>
        <a class="head">商品分类管理</a>
        <el-tree style="max-width: 600px;margin-top: 12px;" :data="treeData" show-checkbox node-key="catId"
            default-expand-all :expand-on-click-node="false" :props="defaultProps" @node-click="handleNodeClick">
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
import { onMounted, reactive, ref } from "vue";
import baseService from "@/service/baseService";
import type Node from 'element-plus/es/components/tree/src/model/node'
import { ElMessage } from "element-plus"

onMounted(() => {
    getInfo();
});

interface Tree {
    id: number
    label: string
    children?: Tree[]
}

let treeData = ref<Tree[]>([])
const getInfo = () => {
    baseService.get("/product/category/page/tree").then((res) => {
        console.log("成功获取Tree data: ", res.data)
        treeData.value = res.data;
    });
};

const handleNodeClick = (data: Tree) => {
    console.log(data)
}

const defaultProps = {
    children: 'children',
    label: 'name',
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
    // const parent = node.parent
    // const children: Tree[] = parent.data.children || parent.data
    // const index = children.findIndex((d) => d.id === data.id)
    // children.splice(index, 1)
    // treeData.value = [...treeData.value]
    console.log("remove", data.label)
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