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
                        <!-- <el-button v-show="curCatId === 0 ? false : true" @click="query()">查询</el-button> -->
                        <el-button type="warning" @click="reset">重置</el-button>
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
                    <el-table-column prop="attrGroupId" label="分组id" header-align="center" align="center"
                        width="70"></el-table-column>
                    <el-table-column prop="attrGroupName" label="组名" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column prop="sort" label="排序" header-align="center" align="center"
                        width="70"></el-table-column>
                    <el-table-column prop="descript" label="描述" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="icon" label="组图标" header-align="center" align="center" #default="scope">
                        <img v-if="scope.row.icon" :src="scope.row.icon" class="icon" />
                    </el-table-column>
                    <el-table-column prop="catelogId" label="所属分类" header-align="center" align="center"
                        #default="scope">
                        <el-cascader v-model="scope.row.catelogId" :options="treeDataRef" :props="casProps" />
                    </el-table-column>
                    <el-table-column label="操作" fixed="right" header-align="center" align="center" width="200">
                        <template v-slot="scope">
                            <el-button v-if="state.hasPermission('product:attrgroup:update')" type="primary" link
                                @click="addOrUpdateHandle(scope.row.attrGroupId)">修改</el-button>
                            <el-button v-if="state.hasPermission('product:attrgroup:delete')" type="primary" link
                                @click="deleteBatch()">删除</el-button>
                            <el-button type="warning" link @click="handleGroupRelation(scope.row)">关联分类</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
                    :total="state.total" layout="total, sizes, prev, pager, next, jumper"
                    @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle">
                </el-pagination>
                <!-- 弹窗, 新增 / 修改 -->
                <add-or-update ref="addOrUpdateRef" :treeData="treeDataRef" :curCatId="curCatId"
                    @refreshDataList="getInfo">确定</add-or-update>
            </div>
        </el-col>
    </el-row>

    <el-dialog title="分组规格" v-model="groupRelationDialogVisible" width="50%">
        <div>
            <!-- <el-button style="margin-bottom: 12px;" type="primary" @click="addOrUpdateRelation()">新增关联</el-button> -->
            <el-button style="margin-bottom: 12px;" type="danger" @click="deleteRelation()">删除所选关联</el-button>
            <el-button style="margin-bottom: 12px;" type="primary" @click="addRelation()">新增关联</el-button>
        </div>
        <el-table @selection-change="relationSelectionChangeHandle" ref="attrAttrGroupTable" v-loading="relationLoading"
            :data="curGroupRelations" border style="width: 90%">
            <el-table-column type="selection" header-align="center" align="center" width="50" />
            <el-table-column prop="id" label="id" header-align="center" align="center"></el-table-column>
            <el-table-column prop="attrGroupName" label="分组" header-align="center" align="center"></el-table-column>
            <el-table-column prop="attrName" label="规格参数" header-align="center" align="center"></el-table-column>
            <el-table-column prop="valueSelect" header-align="center" align="center" width="120" label="可选值">
                <template #default="scope">
                    <el-tooltip placement="top">
                        <template #content>
                            <div style="white-space: pre;">{{ scope.row.valueSelect.split(";").join("\n") }}
                            </div>
                        </template>
                        <el-tag>{{ scope.row.valueSelect.split(";")[0] + "..." }}</el-tag>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
        <template #footer>
            <el-button @click="groupRelationDialogVisible = false">取消</el-button>
            <el-button type="primary" @click="groupRelationSubmitHandle()">确定</el-button>
        </template>
    </el-dialog>

    <el-dialog title="选择属性" v-model="attrDialogVisible" width="50%">
        <el-form :inline="true">
            <el-form-item>
                <el-input v-model="relationQueryKey" placeholder="属性名查询" width="120"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="queryNoneRelation()">查询</el-button>
            </el-form-item>
        </el-form>
        <el-table ref="attrTable" v-loading="attrDialogLoading" :data="attrPage.list" border
            @selection-change="attrSelectionChangeHandle" style="width: 100%">
            <el-table-column type="selection" header-align="center" align="center"></el-table-column>
            <el-table-column prop="attrId" header-align="center" align="center" label="id"></el-table-column>
            <el-table-column prop="attrName" header-align="center" align="center" label="属性名"
                width="100"></el-table-column>
            <el-table-column prop="icon" header-align="center" align="center" label="图标" #default="scope">
                <img v-if="scope.row.icon" :src="scope.row.icon" class="icon" />
            </el-table-column>
            <el-table-column prop="valueSelect" header-align="center" align="center" label="可选值">
                <template #default="scope">
                    <el-tooltip placement="top">
                        <template #content>
                            <div style="white-space: pre;">{{ scope.row.valueSelect.split(";").join("\n") }}
                            </div>
                        </template>
                        <el-tag>{{ scope.row.valueSelect.split(";")[0] + "..." }}</el-tag>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
        <template #footer>
            <el-button @click="attrDialogVisible = false">取消</el-button>
            <el-button type="primary" v-if="attrSelection.length > 0"
                @click="attrRelationSubmitHandle()">确认新增</el-button>
        </template>
        <el-pagination :current-page="attrPage.page" :page-sizes="[10, 20, 50, 100]" :page-size="attrPage.limit"
            :total="attrPage.total" layout="total, sizes, prev, pager, next, jumper"
            @size-change="attrPageSizeChangeHandle" @current-change="attrPageCurrentChangeHandle"> </el-pagination>
        <!-- 弹窗, 新增 / 修改 -->
    </el-dialog>
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

const attrSelection = ref<any[]>([])

const curAttrGroupId = ref(0)

interface Page {
    total: number;
    list: any[];
    page: number;
    limit: number;
}

const relationCatId = ref(0)
const relationQueryKey = ref('')

const attrPage = reactive<Page>({
    total: 0,
    list: [],
    page: 1,
    limit: 10
})

const attrDialogVisible = ref(false)

const attrDialogLoading = ref(false)

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
const relationSelection = ref<any[]>([])
const treeDataRef = ref<Tree[]>([])

const curNode = ref<string>('无')
const curCatId = ref<number>(0)
const queryKey = ref<string>('')

const groupRelationDialogVisible = ref(false)
const relationLoading = ref(false)
const curGroupRelations = ref([])

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
    // console.log("id: ", id)
    addOrUpdateRef.value.init(id)
}

const getInfo = () => {
    // state.dataListLoading = true
    baseService.get(`product/attrgroup/page/${curCatId.value}?key=${queryKey.value}`).then((res) => {
        state.dataList = res.data.list
        state.total = res.data.total
        // console.log("attrgroup.dataList: ", state.dataList)
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
    const checkedAttrGroups = attrGroupTable.value!.getSelectionRows()
    // console.log("checkedAttrGroups", checkedAttrGroups)
    if (checkedAttrGroups.length === 0) {
        ElMessage.warning("请在左侧选择栏选择删除的表单项")
        return
    }
    const ids = checkedAttrGroups.map((attrGroup: { attrGroupId: number }) => attrGroup.attrGroupId)
    ElMessageBox.confirm(`此操作将删除${ids.length === 1 ? ('分组数据【' + checkedAttrGroups[0].attrGroupName + '】') : (ids.length + '个分组数据')}，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        baseService.delete("/product/attrgroup", ids).then(() => {
            getInfo()
            ElMessage.success("删除分组数据成功！")
            // console.log("***成功删除后state.dataList", state.dataList)
        }).catch(() => {
            getInfo()
            ElMessage.error("删除分组数据失败！")
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
}

// const deleteOne = (attrGroupId: string) => {
//     state.deleteHandle(attrGroupId)
// }

const handleGroupRelation = (row: any) => { //获取对应行信息
    relationCatId.value = row.catelogId
    curAttrGroupId.value = row.attrGroupId
    getGroupRelation(row.attrGroupId)
}

const getGroupRelation = (attrGroupId: number) => { //刷新分组下已经分配的关联数据
    relationLoading.value = true
    baseService.get(`product/attrattrgrouprelation/getGroupRelation/${attrGroupId}`).then((res) => {
        curGroupRelations.value = res.data
        groupRelationDialogVisible.value = true
        relationLoading.value = false
    }).catch(err => {
        ElMessage.error("数据获取失败!")
        relationLoading.value = false
    })
    relationLoading.value = false
}

const deleteRelation = () => {
    // console.log("checkedAttrGroups", checkedAttrGroups)
    if (relationSelection.value.length === 0) {
        ElMessage.warning("请在左侧选择栏选择删除的表单项")
        return
    }
    const ids = relationSelection.value.map((relation: { id: number }) => relation.id)
    ElMessageBox.confirm(`此操作将删除${ids.length === 1 ?
        ('分组关联数据【' + relationSelection.value[0].attrGroupName + ' : '
            + relationSelection.value[0].attrName + '】') : (ids.length + '个分组关联数据')}，确定删除？`, '删除确认', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
    }).then(() => {
        baseService.delete("/product/attrattrgrouprelation", ids).then(() => {
            getGroupRelation(curAttrGroupId.value)
            ElMessage.success("删除分组关联数据成功！")
            // console.log("***成功删除后state.dataList", state.dataList)
        }).catch(() => {
            getGroupRelation(curAttrGroupId.value)
            ElMessage.error("删除分组关联数据失败！")
        })
    }).catch(() => {
        ElMessage.info('已取消删除')
    })
}

const groupRelationSubmitHandle = () => {
    ElMessage.info("保存更改")
    groupRelationDialogVisible.value = false
}

const relationSelectionChangeHandle = (val: []) => {
    relationSelection.value = val
    // console.log("relationSelectionChangeHandle", relationSelection.value)
}

const attrSelectionChangeHandle = (val: []) => {
    attrSelection.value = val
    // console.log("relationSelectionChangeHandle", relationSelection.value)
}

const getNoneRelationData = () => { //刷新对应分类分组未分配的规格数据
    baseService.get(`product/attr/base/noRelation/page/${relationCatId.value}?key=${relationQueryKey.value}`).then((res) => {
        // ElMessage.success("获取数据成功!")
        attrPage.list = res.data.list
        attrPage.total = res.data.total
    }).catch(err => {
        ElMessage.error("获取数据失败!")
    })
}

const addRelation = () => {
    attrDialogVisible.value = true
    relationQueryKey.value = ''
    getNoneRelationData()
}

const queryNoneRelation = () => {
    baseService.get(`product/attr/base/noRelation/page/${relationCatId.value}?key=${relationQueryKey.value}`).then((res) => {
        ElMessage.success("查询成功!")
        attrPage.list = res.data.list
        attrPage.total = res.data.total
    }).catch(err => {
        ElMessage.error("查询失败!")
    })
}

const attrRelationSubmitHandle = () => {
    console.log("attrSelection.value", attrSelection.value)
    const attrAttrGroupSelection = ref(attrSelection.value.map(attr => ({
        ...attr,
        attrGroupId: curAttrGroupId.value,
        attrSort: 0
    })))
    console.log("attrAttrGroupSelection", attrAttrGroupSelection.value)

    baseService.post(`product/attrattrgrouprelation/saveBatch`, attrAttrGroupSelection.value).then((res) => {
        ElMessage.success("保存分组关联成功!")
        getNoneRelationData() //刷新对应分类分组未分配的规格数据
        getGroupRelation(curAttrGroupId.value) //刷新分组下已经分配的关联数据
    }).catch(err => {
        ElMessage.error("保存分组关联失败!")
        getNoneRelationData()
        getGroupRelation(curAttrGroupId.value)
    })
}

const attrPageSizeChangeHandle = (val: number) => {
    attrPage.limit = val
}

const attrPageCurrentChangeHandle = (val: number) => {
    attrPage.page = val
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