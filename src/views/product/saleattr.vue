<template>
    <el-row>
        <el-col :span="4">
            <a class="head">商品分类列表</a>
            <Category @categoryData="getNodeHandle" @categoryTree="getData"></Category>
        </el-col>
        <el-col :span="20">

        </el-col>
    </el-row>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { reactive, ref, toRefs, onMounted, watch } from "vue";
import AddOrUpdate from "./baseattr-add-or-update.vue";
import Category from "../common/category.vue"
import baseService from "@/service/baseService"
import { ElMessage, ElMessageBox } from "element-plus"
import { ElTable } from 'element-plus'

onMounted(() => {
    getData
})

let attrtype = ref(1)

interface Tree {
    [x: string]: any
    id: number
    label: string
    children?: Tree[]
}

const attrTable = ref<InstanceType<typeof ElTable>>()

const treeDataRef = ref<Tree[]>([])
const groupDataRef = ref([])

let curNode = ref<string>('无')
let curCatId = ref<number>(0)
let queryKey = ref<string>('')

const view = reactive({
    deleteIsBatch: true,
    getDataListURL: "/product/attr/page",
    getDataListIsPage: true,
    exportURL: "/product/attr/export",
    deleteURL: "/product/attr"
});

const casProps = reactive({
    value: 'catId',
    label: 'name',
    children: 'children',
    emitPath: false,
    disabled: 'catId',
})

const groupProps = reactive({
    value: 'attrGroupId',
    label: 'attrGroupName',
    emitPath: false,
    disabled: 'attrGroupId',
})

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
    addOrUpdateRef.value.init(id)
    // console.log("打开增改", curCatId.value)
}

const getNodeHandle = (node: any) => {
    curNode.value = node.name
    curCatId.value = node.catId
    if (node.catLevel === 3) {
        baseService.get(`product/attr/page/${node.catId}`).then((res) => {
            ElMessage.success("获取数据成功!")
            // console.log("catId", curCatId.value)
            state.dataList = res.data.list
            state.total = res.data.total
        }).catch(err => {
            ElMessage.error("获取数据失败!")
        })
    }
}

const getData = (treeData: any) => {
    treeDataRef.value = treeData
    getGroupData()
}

const query = () => {
    baseService.get(`product/attr/page/${curCatId.value}?key=${queryKey.value}`).then((res) => {
        ElMessage.success("查询成功!")
        state.dataList = res.data.list
        state.total = res.data.total
    }).catch(err => {
        ElMessage.error("查询失败!")
    })
}

const getGroupData = () => {
    baseService.get("product/attrgroup/page").then((res) => {
        groupDataRef.value = res.data.list
    }).catch(err => {
        ElMessage.error("数据获取失败!")
        console.log("error", err)
    })
}

const getInfo = () => {
    baseService.get(`product/attr/page/${curCatId.value}?key=${queryKey.value}`).then((res) => {
        // ElMessage.success("获取数据成功!")
        state.dataList = res.data.list
        state.total = res.data.total
    }).catch(err => {
        ElMessage.error("获取数据失败!")
    })
    getGroupData()
}

const deleteBatch = () => {
    let checkedAttr = attrTable.value!.getSelectionRows()
    if (checkedAttr.length === 0) {
        ElMessage.warning("请在左侧选择栏选择删除的表单项")
        return
    }
    let ids = checkedAttr.map((attr: { attrId: number }) => attr.attrId)
    ElMessageBox.confirm(`此操作将删除${ids.length === 1 ? ('商品规格参数【' + checkedAttr[0].attrName + '】') : (ids.length + '个商品规格参数')}，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        baseService.delete("/product/attr", ids).then(() => {
            getInfo()
            ElMessage.success("删除规格参数成功！")
            // console.log("***成功删除后state.dataList", state.dataList)
        }).catch(() => {
            getInfo()
            ElMessage.error("删除规格参数失败！")
        })
    }).catch(() => {
        ElMessage.info('已取消删除')
    })
}

const reset = () => {
    curNode.value = '无'
    curCatId.value = 0
    queryKey.value = ''
    getInfo()
    ElMessage.info("重置页面数据")
}

</script>

<style>
.head {
    font-size: 21px;
    font-family: "微软雅黑";
    font-weight: bold;
}

.icon {
    width: 80px;
    height: 80px;
    border-radius: 5px;
}
</style>