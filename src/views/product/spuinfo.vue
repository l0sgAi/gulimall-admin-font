<template>
  <div class="mod-product__spuinfo">
    <el-form :inline="true" :model="dataForm">
      <el-form-item label="分类">
        <CategoryCascader @update:catelogPath="catelogChangeHandle"></CategoryCascader>
      </el-form-item>
      <el-form-item label="品牌">
        <BrandSelect @changeBrand="brandChangeHandle"></BrandSelect>
      </el-form-item>
      <el-form-item label="状态">
        <el-select style="width:160px" v-model="dataForm.status" clearable>
          <el-option label="新建" :value="0"></el-option>
          <el-option label="上架" :value="1"></el-option>
          <el-option label="下架" :value="2"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.key" placeholder="关键字查询"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('product:spuinfo:save')" type="primary" @click="query()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('product:spuinfo:save')" type="warning" @click="reset()">重置</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" label="商品id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="spuName" label="商品名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="spuDescription" label="商品描述" header-align="center" align="center"></el-table-column>
      <el-table-column prop="catalogName" label="所属分类" header-align="center" align="center">
      </el-table-column>
      <el-table-column prop="brandName" label="品牌" header-align="center" align="center">
      </el-table-column>
      <el-table-column prop="weight" label="重量" header-align="center" align="center"></el-table-column>
      <el-table-column prop="publishStatus" label="上架状态" header-align="center" align="center" #default="scope">
        <el-tag :type="scope.row.publishStatus == 1 ? 'success' :
      scope.row.publishStatus == 0 ? 'warning' : 'danger'">{{
      scope.row.publishStatus == 1 ? '上架' :
        scope.row.publishStatus == 0 ? '新建' : '下架'
    }}</el-tag>
      </el-table-column>
      <el-table-column prop="createTime" label="创建时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="updateTime" label="最后修改时间" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('product:spuinfo:update')" type="primary" link
            @click="grounding(scope.row.id)">上架</el-button>
          <el-button v-if="state.hasPermission('product:spuinfo:delete')" type="primary" link
            @click="showAttr(scope.row.id)">规格</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <!-- <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update> -->
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue"
import CategoryCascader from "../common/category-cascader.vue"
import BrandSelect from "../common/brand-select.vue"
import baseService from "@/service/baseService"
import { ElMessage } from "element-plus"

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/product/spuinfo/page",
  getDataListIsPage: true,
  exportURL: "/product/spuinfo/export",
  deleteURL: "/product/spuinfo"
});

const dataForm = reactive({
  status: "",
  key: "",
  brandId: 0,
  catelogId: 0
})

const state = reactive({ ...useView(view), ...toRefs(view) })

const query = () => {
  // console.log("dataForm_query", dataForm)
  baseService.get(`product/spuinfo/page`, dataForm).then((res: { data: { list: any; total: number } }) => {
    ElMessage.success("查询成功!")
    state.dataList = res.data.list
    state.total = res.data.total
  }).catch((err: any) => {
    ElMessage.error("查询失败!")
  })
}

const catelogChangeHandle = (catId: number[]) => {
  dataForm.catelogId = catId[catId.length - 1]
  // console.log("dataForm.catelogId", dataForm)
}

const brandChangeHandle = (brandId: number) => {
  console.log("brandId", brandId)
  dataForm.brandId = brandId
  // console.log("dataForm.brandId", dataForm)
}

const reset = () => {
  dataForm.catelogId = 0
  dataForm.brandId = 0
  dataForm.status = ""
  dataForm.key = ""
  query()
}

//TODO: 上下架和规格管理
const grounding = (id: number) => {

}

const showAttr = (id: number) => {

}

// const addOrUpdateRef = ref();
// const addOrUpdateHandle = (id?: number) => {
//   addOrUpdateRef.value.init(id);
// };
</script>
