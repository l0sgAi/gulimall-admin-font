<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm">
      <el-form-item label="分类">
        <category-cascader @update:catelogPath="catelogChangeHandle"></category-cascader>
      </el-form-item>
      <el-form-item label="品牌">
        <brand-select @changeBrand="brandChangeHandle" style="width:160px;margin-right: 32px;"></brand-select>
      </el-form-item>
      <el-form-item label="价格">
        <el-input-number style="width:160px" v-model="dataForm.min" :min="0"></el-input-number>-
        <el-input-number style="width:160px" v-model="dataForm.max" :min="0"></el-input-number>
      </el-form-item>
      <el-form-item label="检索">
        <el-input style="width:160px" v-model="dataForm.key" placeholder="请输入关键字"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="searchSkuInfo">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="warning" @click="reset()">重置</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle"
      style="width: 100% " @expand-change="getSkuDetails">
      <el-table-column type="expand">
        <template #default="scope">
          商品标题：{{ scope.row.skuTitle }}
          <br />
          商品副标题：{{ scope.row.skuSubtitle }}
          <br />
          商品描述：{{ scope.row.skuDesc }}
          <br />
          分类ID：{{ scope.row.catalogId }}
          <br />
          SpuID：{{ scope.row.spuId }}
          <br />
          品牌ID：{{ scope.row.brandId }}
          <br />
        </template>
      </el-table-column>
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="skuId" header-align="center" align="center" label="skuId"></el-table-column>
      <el-table-column prop="skuName" header-align="center" align="center" label="名称"></el-table-column>
      <el-table-column prop="skuDefaultImg" header-align="center" align="center" label="默认图片">
        <template #default="scope">
          <img :src="scope.row.skuDefaultImg" style="width:80px;height:80px;" />
        </template>
      </el-table-column>
      <el-table-column prop="price" header-align="center" align="center" label="价格"></el-table-column>
      <el-table-column prop="saleCount" header-align="center" align="center" label="销量"></el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template #default="scope">
          <el-button size="small" type="primary" @click="previewHandle(scope.row.skuId)" link>预览</el-button>
          <el-button size="small" type="warning" @click="commentHandle(scope.row.skuId)" link>评论</el-button>
          <el-dropdown @command="handleCommand" type="primary">
            <el-button type="primary">
              更多<el-icon class="el-icon--right"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item :command="['uploadImages', scope.row]">上传图片</el-dropdown-item>
                <el-dropdown-item :command="['seckillSettings', scope.row]">参与秒杀</el-dropdown-item>
                <el-dropdown-item :command="['reductionSettings', scope.row]">满减设置</el-dropdown-item>
                <el-dropdown-item :command="['discountSettings', scope.row]">折扣设置</el-dropdown-item>
                <el-dropdown-item :command="['memberPriceSettings', scope.row]">会员价格</el-dropdown-item>
                <el-dropdown-item :command="['stockSettings', scope.row]">库存管理</el-dropdown-item>
                <el-dropdown-item :command="['couponSettings', scope.row]">优惠劵</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"></el-pagination>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'
import CategoryCascader from "../common/category-cascader.vue"
import BrandSelect from "../common/brand-select.vue"
import PubSub from 'pubsub-js'
import baseService from "@/service/baseService"
import { ElMessage, ElMessageBox } from "element-plus"

const catPathSub = ref<any>([])
const brandIdSub = ref<any>([])
const dataForm = reactive({
  key: "",
  brandId: 0,
  catelogId: 0,
  min: 0,
  max: 0,
})
const dataList = ref([])
const pageIndex = ref(1)
const pageSize = ref(10)
const totalPage = ref(0)
const dataListLoading = ref(false)
const dataListSelections = ref<any>([])
const addOrUpdateVisible = ref(false)
const catelogPath = ref([])
const event = ref('')

const getSkuDetails = (row: any, expand: any) => {
  console.log("展开某行...", row, expand)
}

const handleCommand = (command: any) => {
  // 处理指令和行数据的逻辑
  console.log('Command:', command)
}

const searchSkuInfo = () => {
  getDataList()
  ElMessage.success("查询成功！")
}

const getDataList = () => {
  dataListLoading.value = true
  baseService.get("product/skuinfo/page", dataForm).then((res) => {
    dataList.value = res.data.list
    totalPage.value = res.data.total
    console.log("dataList.value: ", dataList.value)
    dataListLoading.value = false
  }).catch(() => {
    ElMessage.error("获取数据失败！")
    dataListLoading.value = false
  });
}

const sizeChangeHandle = (val: number) => {
  pageSize.value = val
  pageIndex.value = 1
  getDataList()
}

const currentChangeHandle = (val: number) => {
  pageIndex.value = val
  getDataList()
}

const selectionChangeHandle = (val: number) => {
  dataListSelections.value = val
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
  dataForm.max = 0
  dataForm.min = 0
  dataForm.key = ""
  getDataList()
}

const previewHandle = (_skuId: number) => {

}

const commentHandle = (_skuId: number) => {

}

//TODO: 预览、评论、更多...

onMounted(() => {
  // catPathSub.value = PubSub.subscribe("catPath", (msg, val) => {
  //   dataForm.catelogId = val[val.length - 1]
  // })
  // brandIdSub.value = PubSub.subscribe("brandId", (msg, val) => {
  //   dataForm.brandId = val
  // })
  getDataList()
})

</script>
