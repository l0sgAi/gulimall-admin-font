<template>
    <div>
        <a class="head">商品分类管理</a>
        <el-switch v-model="draggable" style="margin-right: 25px;" active-text="开启拖拽" inactive-text="关闭拖拽" />
        <el-button color="rgb(4, 170, 170)" class="tools-div" type="success" size="small" @click="addFirst">
            新增一级节点
        </el-button>
        <el-button color="rgb(0, 145, 25)" v-show="draggable" class="tools-div" type="success" size="small"
            @click="saveUpdateBatch">保存本次更改
        </el-button>
        <el-button class="tools-div" type="warning" size="small" @click="reset">重置
        </el-button>
        <el-button class="tools-div" type="danger" size="small" @click="deleteBatch">批量删除
        </el-button>
        <br />
        <el-input v-model="filterText" style="width: 240px" placeholder="输入分类名以过滤" />
        <br />
        <el-tree v-loading="loading" style="max-width: 600px;margin-top: 12px;" :data="treeData" show-checkbox
            node-key="catId" :default-expanded-keys="expanded" :expand-on-click-node="false" :props="defaultProps"
            @node-click="handleNodeClick" :allow-drop="allowDrop" :draggable="draggable" @node-drop="handleDrop"
            ref="menuTree" :filter-node-method="filterNode">
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
                        <el-button type="warning" style="margin-left: 8px;" @click="update(data)" link>
                            修改节点
                        </el-button>
                    </span>
                </span>
            </template>
        </el-tree>

        <el-dialog v-model="visible" title='新增分类' width="40%" close>
            <el-form :model="dataForm" ref="dataFormRef" label-width="120px" :rules="rules">
                <el-form-item label="分类名称" prop="name">
                    <el-input v-model="dataForm.name" />
                </el-form-item>
                <el-form-item label="图标" prop="icon">
                    <SingleUpload :logo="dataForm.icon" @changeLogo="changeLogoHandle"></SingleUpload>
                </el-form-item>
                <el-form-item label="计量单位" prop="productUnit">
                    <el-input v-model="dataForm.productUnit" />
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="doAppend()">新增</el-button>
                    <el-button @click="visible = false">取消</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>

        <el-dialog v-model="visibleUpdate" title='修改分类' width="40%">
            <el-form :model="dataForm" ref="dataFormRef" label-width="120px" :rules="rules">
                <el-form-item label="分类名称" prop="name">
                    <el-input v-model="dataForm.name" />
                </el-form-item>
                <el-form-item label="图标" prop="icon">
                    <SingleUpload :logo="dataForm.icon" @changeLogo="changeLogoHandle"></SingleUpload>
                </el-form-item>
                <el-form-item label="计量单位" prop="productUnit">
                    <el-input v-model="dataForm.productUnit" />
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="doUpdate()">修改</el-button>
                    <el-button @click="visibleUpdate = false">取消</el-button>
                </el-form-item>
            </el-form>
        </el-dialog>
    </div>
</template>

<script lang="ts" setup>
import { onMounted, reactive, ref, watch } from "vue"
import baseService from "@/service/baseService"
import type Node from 'element-plus/es/components/tree/src/model/node'
import { ElMessage, ElMessageBox, ElTree } from "element-plus"
import SingleUpload from "@/upload/singleUpload.vue"

const emit = defineEmits(["refreshDataList"]);

onMounted(() => {
    getInfo()
});

const preUpdate = ref(false) //标志之前是否进行更新

const draggable = ref(false)

const rules = ref({
    name: [
        { required: true, message: "请输入分类名称", trigger: "blur" },
        { min: 1, max: 30, message: '名称长度在 1 到 30 个字符之间', trigger: 'blur' }
    ],
    icon: [
        { required: false, message: "请输入分类图标url", trigger: "blur" },
        { min: 0, max: 255, message: '图标url过长!', trigger: 'blur' }
    ],
    productUnit: [
        { required: false, message: "请输入计量单位名称", trigger: "blur" },
        { min: 0, max: 16, message: '单位名过长!', trigger: 'blur' }
    ]
});

const dataForm = reactive({
    catId: "",
    name: "",
    parentCid: 0,
    catLevel: 1,
    showStatus: 1,
    sort: 0,
    icon: "",
    productUnit: "",
    productCount: 0
});

const filterText = ref('')

const filterNode = (value: string, data: Tree) => {
    if (!value) return true
    return data.name.includes(value)
}

const dataFormRef = ref()

const visible = ref(false)
const visibleUpdate = ref(false)

let updateNodes = ref<NodeVo[]>([])

interface Tree {
    [x: string]: any
    // id: number
    // label: string
    // children?: Tree[]
}

const menuTree = ref<InstanceType<typeof ElTree>>()

// 定义 Node 接口
interface NodeVo {
    catId: string | number
    parentCid: number
    catLevel: number
    sort: number
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

const update = (data: Tree) => {
    baseService.get(`/product/category/${data.catId}`).then((res) => {
        console.log("成功获取Node data: ", res.data)
        dataForm.catId = res.data.catId
        dataForm.name = res.data.name
        dataForm.parentCid = res.data.parentCid
        dataForm.catLevel = res.data.catLevel
        dataForm.icon = res.data.icon
        dataForm.productUnit = res.data.productUnit
        console.log("update", res.data)
    }).catch(() => {
        ElMessage.error("获取节点数据失败！")
    });
    visibleUpdate.value = true
}

const append = (data: Tree) => {
    dataForm.name = ""
    dataForm.icon = ""
    dataForm.productUnit = ""
    visible.value = true
    dataForm.parentCid = data.catId
    dataForm.catLevel = data.catLevel + 1
    console.log("append", data)
}

const doUpdate = () => {
    expanded.value = []
    dataFormRef.value.validate((valid: boolean) => {
        if (!valid) {
            ElMessage.error("数据格式错误!")
            visibleUpdate.value = false
            dataForm.name = ""
            dataForm.catId = ""
            return false;
        } else {
            baseService.put("/product/category/update", dataForm).then(() => {
                ElMessage.success("修改分类数据成功！")
                expanded.value = [dataForm.parentCid]
                getInfo()
                console.log("修改分类数据成功！", dataForm)
            }).catch(() => {
                ElMessage.error("修改分类数据失败！")
            });
            visibleUpdate.value = false
            dataForm.name = ""
            dataForm.catId = ""
        }
    })
}
const addFirst = () => {
    dataForm.name = ""
    dataForm.icon = ""
    dataForm.productUnit = ""
    visible.value = true
    dataForm.parentCid = 0
    dataForm.catLevel = 1
}

// 表单提交
const doAppend = () => {
    expanded.value = []
    dataFormRef.value.validate((valid: boolean) => {
        if (!valid) {
            ElMessage.error("数据格式错误!")
            visible.value = false
            dataForm.name = ""
            return false;
        } else {
            baseService.post("/product/category/save", dataForm).then(() => {
                ElMessage.success("添加分类数据成功！")
                expanded.value = [dataForm.parentCid]
                getInfo()
                console.log("提交分类数据成功", dataForm)
            }).catch(() => {
                ElMessage.error("添加分类数据失败！")
            });
            visible.value = false
            dataForm.name = ""
        }
    })
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
    const ids = [data.catId]
    expanded.value = []
    baseService.delete("/product/category/delete", ids).then(() => {
        getInfo()
        console.log("成功删除分类数据: ", ids)
        ElMessage.success("删除分类成功！")
        expanded.value = [node.parent.data.catId]
    }).catch(() => {
        ElMessage.error("删除分类数据失败！")
    });
}

const allowDrop = (draggingNode: Node, dropNode: Node, type: string) => {
    return type === "inner" ? (countNodeLevel(draggingNode) + dropNode.level <= 3) : (countNodeLevel(draggingNode) <= 3)
}

const countNodeLevel = (node: Node) => {
    let level = 1
    let children = node.childNodes
    if (children.length > 0) {
        level++
        for (let i = 0; i < children.length; i++) {
            if (children[i].childNodes.length > 0) {
                level++
                break;
            }
        }
    }

    return level
}

const handleDrop = (draggingNode: Node, dropNode: Node, type: string) => {
    if (preUpdate.value === true) {
        expanded.value = []
        preUpdate.value = false
    }
    let pCid = 0
    let sibilings = null
    let catLevel = 0
    //1.当前节点最新父Id
    //2.当前节点的最新排序
    //3.当前节点的最新层级
    if (type === "inner") {
        pCid = dropNode.data.catId
        sibilings = dropNode.childNodes
        catLevel = dropNode.data.catLevel + 1
        expanded.value.push(dropNode.data.catId)
    } else {
        pCid = dropNode.data.parentCid
        sibilings = dropNode.parent.childNodes
        catLevel = dropNode.data.catLevel
        expanded.value.push(dropNode.parent.data.catId)
    }

    for (let i = 0; i < sibilings.length; i++) { //处理兄弟节点
        if (sibilings[i].data.catId === draggingNode.data.catId) {
            //如果正在遍历拖拽中的节点，更新父节点、排序和层级
            updateNodes.value.push({
                catId: sibilings[i].data.catId,
                sort: i,
                parentCid: pCid,
                catLevel: catLevel,
            })

            updateChildNodeLevel(sibilings[i], catLevel)
        } else {
            //否则只更新排序
            updateNodes.value.push({
                catId: sibilings[i].data.catId,
                sort: i,
                parentCid: sibilings[i].data.parentCid,
                catLevel: sibilings[i].data.catLevel,
            })
        }
    }
}

const updateChildNodeLevel = (node: Node, catLevel: number) => {
    //因为树只有三层，所以子节点有节点的，通过拖拽一定不能改变所在层级
    //所以，最多只需要遍历一层节点
    if (node.childNodes && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
            updateNodes.value.push({
                catId: node.childNodes[i].data.catId,
                sort: i,
                parentCid: node.childNodes[i].data.parentCid,
                catLevel: catLevel + 1,
            })
        }
    }
}

const saveUpdateBatch = () => {
    preUpdate.value === false ? preUpdate.value = true : preUpdate.value = false
    baseService.put("/product/category/update/sort", updateNodes.value).then(() => {
        ElMessage.success("修改分类成功！")
        getInfo()
        // console.log("批量修改分类成功！", updateNodes)
        updateNodes.value = [] //重置更新节点列表
    }).catch(() => {
        ElMessage.error("批量修改分类失败！")
        getInfo()
        updateNodes.value = [] //重置更新节点列表
    })
    // console.log("展开列表", expanded.value)
}

const deleteBatch = () => {
    let checkedNodes = menuTree.value!.getCheckedNodes()
    let nodeIds = checkedNodes.map(node => node.catId)
    let parentCids = checkedNodes.map(node => node.parentCid)
    ElMessageBox.confirm(`此操作将删除${checkedNodes.length}个菜单，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        baseService.delete("/product/category/delete", nodeIds).then(() => {
            getInfo()
            console.log("成功删除分类数据: ", checkedNodes)
            ElMessage.success("删除分类成功！")
            expanded.value = parentCids
        }).catch(() => {
            ElMessage.error("删除分类数据失败！")
        })
    }).catch(() => {
        ElMessage.info('已取消删除')
    })
}

const reset = () => {
    getInfo()
    expanded.value = [] //重置展开列表
    updateNodes.value = [] //重置更新节点列表
    filterText.value = '' //重置过滤文本
}

const changeLogoHandle = (logo: string) => {
    dataForm.icon = logo
}

watch(filterText, (val) => {
    // console.log("值改变!", val)
    menuTree.value!.filter(val)
})
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

.tools-div {
    width: 100px;
    height: 40px;
    font-size: 14px;
    margin-bottom: 10px;
    margin-top: 10px;
    padding: 10px;
    border-radius: 3px;
}
</style>