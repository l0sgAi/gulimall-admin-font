<template>
  <div class="mod-ware__wmswaresku">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item label="仓库" prop="wareId">
        <el-select v-model="wareId" placeholder="请选择仓库" clearable>
          <el-option :label="w.name" :value="w.id" v-for="w in wareList" :key="w.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="sku_id" prop="skuId">
        <el-input-number style="width:130px" placeholder="输入skuId" v-model="skuId" :min="0" :precision="0" :step="1"
          controls-position="right"></el-input-number>
      </el-form-item>
      <el-form-item>
        <el-button @click="query()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="warning" @click="reset()">重置</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('ware:wmswaresku:save')" type="primary"
          @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('ware:wmswaresku:delete')" type="danger"
          @click="state.deleteHandle()">删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" label="id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="skuId" label="sku_id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="wareId" label="仓库" header-align="center" align="center" #default="scope">
        {{ wareList.find(w => w.id == scope.row.wareId)?.name }}
      </el-table-column>
      <el-table-column prop="stock" label="库存数" header-align="center" align="center"></el-table-column>
      <el-table-column prop="skuName" label="sku_name" header-align="center" align="center"></el-table-column>
      <el-table-column prop="stockLocked" label="锁定库存" header-align="center" align="center" #default="scope">
        <el-switch v-model="scope.row.stockLocked" class="ml-2" inline-prompt
          style="--el-switch-on-color: #ff4949; --el-switch-off-color: #13ce66" active-text="锁定" inactive-text="不锁定"
          :active-value="1" :inactive-value="0" disabled />
      </el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('ware:wmswaresku:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('ware:wmswaresku:delete')" type="primary" link
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
import useView from "@/hooks/useView";
import { reactive, ref, toRefs, onMounted } from "vue";
import AddOrUpdate from "./wmswaresku-add-or-update.vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus"

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/ware/wmswaresku/page",
  getDataListIsPage: true,
  exportURL: "/ware/wmswaresku/export",
  deleteURL: "/ware/wmswaresku"
});
const wareId = ref('')
const skuId = ref('')
const wareList = ref<any>([])

const state = reactive({ ...useView(view), ...toRefs(view) })

const addOrUpdateRef = ref()
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id)
}

const getWares = () => {
  baseService.get("ware/wmswareinfo/page").then((res: { data: { list: any; }; }) => {
    wareList.value = res.data.list
  })
}

const query = () => {
  baseService.get(`/ware/wmswaresku/page?wareId=${wareId.value}&skuId=${skuId.value}`).then((res: { data: { list: any; total: number; }; }) => {
    // console.log("成功获取wmswareinfo data: ", res.data)
    state.dataList = res.data.list
    state.total = res.data.total
    ElMessage.success(`查询成功`)
    // console.log("成功获取state.dataList: ", state.dataList)
  }).catch(() => {
    ElMessage.error("获取数据失败！")
  })
}

const reset = () => {
  wareId.value = ''
  skuId.value = ''
  state.getDataList()
  ElMessage.success('重置成功')
}

onMounted(() => {
  getWares()
})
</script>
