<template>
  <div class="mod-product__brand">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-button v-if="state.hasPermission('product:brand:save')" type="primary"
          @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('product:brand:delete')" type="danger" @click="deleteBatch">删除</el-button>
      </el-form-item>
    </el-form>
    <el-table ref="brandTable" v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="brandId" label="品牌id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="name" label="品牌名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="logo" label="品牌logo地址" header-align="center" align="center"></el-table-column>
      <el-table-column prop="descript" label="介绍" header-align="center" align="center"></el-table-column>
      <el-table-column prop="showStatus" label="显示状态" header-align="center" align="center" #default="scope">
        <el-tag>{{ scope.row.showStatus === 1 ? "显示" : "不显示" }}</el-tag>
      </el-table-column>
      <el-table-column prop="firstLetter" label="检索首字母" header-align="center" align="center"></el-table-column>
      <el-table-column prop="sort" label="排序" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('product:brand:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.brandId)">修改</el-button>
          <el-button v-if="state.hasPermission('product:brand:delete')" type="danger" link
            @click="state.deleteHandle(scope.row.brandId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue"
import AddOrUpdate from "./brand-add-or-update.vue"
import baseService from "@/service/baseService"
import { ElTable } from 'element-plus'
import { ElMessage, ElMessageBox } from "element-plus"

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/product/brand/page",
  getDataListIsPage: true,
  exportURL: "/product/brand/export",
  deleteURL: "/product/brand/delete"
});

const state = reactive({ ...useView(view), ...toRefs(view) })

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
}

const brandTable = ref<InstanceType<typeof ElTable>>()

const getInfo = () => {
  // state.dataListLoading = true
  baseService.get("/product/brand/page").then((res) => {
    console.log("成功获取Brand data: ", res.data)
    state.dataList = res.data.list
  }).catch(() => {
    ElMessage.error("获取数据失败！")
  });
};


const deleteBatch = () => {
  let checkedBrnads = brandTable.value!.getSelectionRows()
  let ids = checkedBrnads.map((brand: { brandId: number }) => brand.brandId)
  ElMessageBox.confirm(`此操作将删除${ids.length}个品牌数据，确定删除？`, '删除确认', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  }).then(() => {
    baseService.delete("/product/brand/delete", ids).then(() => {
      getInfo()
      ElMessage.success("删除品牌数据成功！")
      console.log("***成功删除后state.dataList", state.dataList)
    }).catch(() => {
      getInfo()
      ElMessage.error("删除品牌数据失败！")
    })
  }).catch(() => {
    ElMessage.info('已取消删除')
  })
  // state.deleteHandle(ids)
}
</script>
