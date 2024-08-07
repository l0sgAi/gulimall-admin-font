<template>
    <el-row>
        <el-col :span="6">
            <a class="head">商品分类列表</a>
            <Category @categoryData="getNodeHandle"></Category>
        </el-col>
        <el-col :span="18">
            <div class="mod-product__attrgroup">
                <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
                    <el-form-item>
                        <el-button v-if="state.hasPermission('product:attrgroup:save')" type="primary"
                            @click="addOrUpdateHandle()">新增</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button v-if="state.hasPermission('product:attrgroup:delete')" type="danger"
                            @click="state.deleteHandle()">删除</el-button>
                        <a class="head">当前分类: {{ curNode }}</a>
                    </el-form-item>
                </el-form>
                <el-table v-loading="state.dataListLoading" :data="state.dataList" border
                    @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
                    <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
                    <el-table-column prop="attrGroupId" label="分组id" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column prop="attrGroupName" label="组名" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column prop="sort" label="排序" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="descript" label="描述" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="icon" label="组图标" header-align="center" align="center"></el-table-column>
                    <el-table-column prop="catelogId" label="所属分类id" header-align="center"
                        align="center"></el-table-column>
                    <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
                        <template v-slot="scope">
                            <el-button v-if="state.hasPermission('product:attrgroup:update')" type="primary" link
                                @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
                            <el-button v-if="state.hasPermission('product:attrgroup:delete')" type="primary" link
                                @click="state.deleteHandle(scope.row.id)">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
                <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
                    :total="state.total" layout="total, sizes, prev, pager, next, jumper"
                    @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle">
                </el-pagination>
                <!-- 弹窗, 新增 / 修改 -->
                <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
            </div>
        </el-col>
    </el-row>
</template>

<script lang="ts" setup>
import Category from "../common/category.vue"
import useView from "@/hooks/useView";
import { reactive, ref, toRefs } from "vue";
import AddOrUpdate from "./attrgroup-add-or-update.vue";
import baseService from "@/service/baseService"
import { ElMessage } from "element-plus"
let curNode = ref<string>('无')

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
    addOrUpdateRef.value.init(id);
}

const getNodeHandle = (node: any) => {
    if (node.catLevel === 3) {
        baseService.get(`product/attrgroup/page/${node.catId}`).then((res) => {
            curNode.value = node.name
            ElMessage.success("获取数据成功!")
            state.dataList = res.data.list
            state.total = res.data.total
        })
    }
}
</script>

<style>
.head {
    display: flex;
    font-size: 21px;
    font-family: "微软雅黑";
    font-weight: bold;
    margin-left: 64px;
}
</style>