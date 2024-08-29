<template>
  <div class="mod-ware__wmswareinfo">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="queryKey" placeholder="仓库名查询"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="query()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('ware:wmswareinfo:save')" type="primary"
          @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('ware:wmswareinfo:delete')" type="danger"
          @click="state.deleteHandle()">删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" label="id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="name" label="仓库名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="address" label="仓库地址" header-align="center" align="center"></el-table-column>
      <el-table-column prop="areacode" label="区域编码" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('ware:wmswareinfo:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('ware:wmswareinfo:delete')" type="primary" link
            @click="state.deleteHandle(scope.row.id)">删除</el-button>
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
import AddOrUpdate from "./wmswareinfo-add-or-update.vue"
import baseService from "@/service/baseService"
import { ElMessage } from "element-plus"

const queryKey = ref("")

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/ware/wmswareinfo/page",
  getDataListIsPage: true,
  exportURL: "/ware/wmswareinfo/export",
  deleteURL: "/ware/wmswareinfo"
})

const state = reactive({ ...useView(view), ...toRefs(view) })

const addOrUpdateRef = ref()
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
}

const query = () => {
  baseService.get(`/ware/wmswareinfo/page?key=${queryKey.value}`).then((res: { data: { list: any; total: number; }; }) => {
    // console.log("成功获取wmswareinfo data: ", res.data)
    state.dataList = res.data.list
    state.total = res.data.total
    ElMessage.success(`查询成功`)
    // console.log("成功获取state.dataList: ", state.dataList)
  }).catch(() => {
    ElMessage.error("获取数据失败！")
  });
}
</script>
