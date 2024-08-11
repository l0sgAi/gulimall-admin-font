<template>
    <el-row>
        <el-col :span="6">
            <a class="head">商品分类列表</a>
            <Category @categoryData="getNodeHandle" @categoryTree="getTreeData"></Category>
        </el-col>
        <el-col :span="18">
            <div class="mod-product__attrgroup">
                <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
                    <el-form-item>
                        <!-- <el-input v-show="curCatId === 0 ? false : true" v-model="queryKey"
                            placeholder="组名查询"></el-input> -->
                        <el-input v-model="queryKey" placeholder="组名查询"></el-input>

                    </el-form-item>
                    <el-form-item>
                        <!-- <el-button v-show="curCatId === 0 ? false : true" @click="query()">查询</el-button> -->
                        <el-button @click="query()">查询</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button v-if="state.hasPermission('product:attrgroup:save')" type="primary"
                            @click="addOrUpdateHandle()">新增</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button v-if="state.hasPermission('product:attrgroup:delete')" type="danger"
                            @click="deleteBatch">删除</el-button>
                        <!-- <a class="head">当前分类: {{ curNode }}</a> -->
                    </el-form-item>
                </el-form>
                <el-table ref="attrGroupTable" v-loading="state.dataListLoading" :data="state.dataList" border
                    @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
                    <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
                    <el-table-column prop="attrGroupId" label="分组id" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column prop="attrGroupName" label="组名" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column prop="sort" label="排序" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="descript" label="描述" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="icon" label="组图标" header-align="center" align="center" #default="scope">
                        <img v-if="scope.row.icon" :src="scope.row.icon" class="icon" />
                    </el-table-column>
                    <el-table-column prop="catelogId" label="所属分类" header-align="center" align="center"
                        #default="scope">
                        <el-cascader v-model="scope.row.catelogId" :options="treeDataRef" :props="casProps" />
                    </el-table-column>
                    <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
                        <template v-slot="scope">
                            <el-button v-if="state.hasPermission('product:attrgroup:update')" type="primary" link
                                @click="addOrUpdateHandle(scope.row.attrGroupId)">修改</el-button>
                            <el-button v-if="state.hasPermission('product:attrgroup:delete')" type="primary" link
                                @click="state.deleteHandle(scope.row.attrGroupId)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
                    :total="state.total" layout="total, sizes, prev, pager, next, jumper"
                    @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle">
                </el-pagination>
                <!-- 弹窗, 新增 / 修改 -->
                <add-or-update ref="addOrUpdateRef" :treeData="treeDataRef"
                    @refreshDataList="state.getDataList">确定</add-or-update>
            </div>
        </el-col>
    </el-row>
</template>

<script lang="ts" setup>
import Category from "../common/category.vue"
import useView from "@/hooks/useView";
import { reactive, ref, toRefs, onMounted } from "vue";
import AddOrUpdate from "./attrgroup-add-or-update.vue";
import baseService from "@/service/baseService"
import { ElMessage, ElMessageBox } from "element-plus"
import { ElTable } from 'element-plus'

onMounted(() => {
    getTreeData
})

const casProps = reactive({
    value: 'catId',
    label: 'name',
    children: 'children',
    emitPath: false,
    disabled: 'catId',
})

interface Tree {
    [x: string]: any
    id: number
    label: string
    children?: Tree[]
}

const attrGroupTable = ref<InstanceType<typeof ElTable>>()

const treeDataRef = ref<Tree[]>([])

let curNode = ref<string>('无')
let curCatId = ref<number>(0)
let queryKey = ref<string>('')

const view = reactive({
    deleteIsBatch: true,
    getDataListURL: "/product/attrgroup/page",
    getDataListIsPage: true,
    exportURL: "/product/attrgroup/export",
    deleteURL: "/product/attrgroup"
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
    console.log("id: ", id)
    addOrUpdateRef.value.init(id)
}

const getInfo = () => {
    // state.dataListLoading = true
    baseService.get(`product/attrgroup/page/${curCatId.value}?key=${queryKey.value}`).then((res) => {
        state.dataList = res.data.list
        state.total = res.data.total
        console.log("attrgroup.dataList: ", state.dataList)
    }).catch(() => {
        ElMessage.error("获取数据失败！")
    });
};

const getNodeHandle = (node: any) => {
    if (node.catLevel === 3) {
        baseService.get(`product/attrgroup/page/${node.catId}`).then((res) => {
            curNode.value = node.name
            curCatId.value = node.catId
            ElMessage.success("获取数据成功!")
            state.dataList = res.data.list
            state.total = res.data.total
        }).catch(err => {
            ElMessage.error("获取数据失败!")
        })
    }
}

const getTreeData = (treeData: any) => {
    treeDataRef.value = treeData
}

const query = () => {
    baseService.get(`product/attrgroup/page/${curCatId.value}?key=${queryKey.value}`).then((res) => {
        ElMessage.success("查询成功!")
        state.dataList = res.data.list
        state.total = res.data.total
    }).catch(err => {
        ElMessage.error("查询失败!")
    })
}

const deleteBatch = () => {
    let checkedAttrGroups = attrGroupTable.value!.getSelectionRows()
    let ids = checkedAttrGroups.map((attrGroup: { attrGroupId: number }) => attrGroup.attrGroupId)
    ElMessageBox.confirm(`此操作将删除${ids.length}个分组数据，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        baseService.delete("/product/attrgroup", ids).then(() => {
            getInfo()
            ElMessage.success("删除分组数据成功！")
            console.log("***成功删除后state.dataList", state.dataList)
        }).catch(() => {
            getInfo()
            ElMessage.error("删除分组数据失败！")
        })
    }).catch(() => {
        ElMessage.info('已取消删除')
    })
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