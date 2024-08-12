<template>
  <div class="mod-product__brand">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="queryKey" placeholder="品牌名查询" width="500"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="query()">查询</el-button>
      </el-form-item>
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
      <el-table-column prop="logo" label="品牌logo" header-align="center" align="center" #default="scope">
        <img v-if="scope.row.logo" :src="scope.row.logo" class="logo" />
      </el-table-column>
      <el-table-column prop="descript" label="介绍" header-align="center" align="center"></el-table-column>
      <el-table-column prop="showStatus" label="显示状态" header-align="center" align="center" #default="scope">
        <el-tag :type="scope.row.showStatus === 1 ? 'success' : 'danger'">{{ scope.row.showStatus === 1 ? "显示" : "不显示"
          }}</el-tag>
      </el-table-column>
      <el-table-column prop="firstLetter" label="检索首字母" header-align="center" align="center"></el-table-column>
      <el-table-column prop="sort" label="排序" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="200">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('product:brand:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.brandId)">修改</el-button>
          <el-button v-if="state.hasPermission('product:brand:delete')" type="danger" link
            @click="state.deleteHandle(scope.row.brandId)">删除</el-button>
          <el-button type="warning" link @click="handleCategoryRelation(scope.row)">关联分类</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="getInfo">确定</add-or-update>

    <el-dialog title="关联分类" v-model="cateRelationDialogVisible" width="50%">
      <div>
        <el-button style="margin-bottom: 12px;" type="primary" @click="addOrUpdateRelation()">新增关联</el-button>
      </div>
      <el-table ref="brandCategoryTable" v-loading="relationLoading" :data="curCatRelations" border style="width: 90%">
        <el-table-column prop="brandName" label="品牌名" header-align="center" align="center"></el-table-column>
        <el-table-column prop="catelogName" label="分类名" header-align="center" align="center"></el-table-column>
        <el-table-column label="操作" fixed="right" header-align="center" align="center" width="200">
          <template v-slot="scope">
            <el-button type="primary" link @click="addOrUpdateRelation(scope.row.id)">修改</el-button>
            <el-button type="danger" link @click="deleteRelation(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <template #footer>
        <el-button @click="cateRelationDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
      </template>
    </el-dialog>

    <el-dialog v-loading="loading" title="关联新增或修改" v-model="relationAddOrUpdate" width="50%">
      <el-form :model="dataForm" :rules="rules" label-width="120px">
        <el-form-item label="关联分类" prop="catelogName" required>
          <el-cascader placeholder="搜索分类" filterable v-model="dataFormRef.catelogId" :options="treeDataRef"
            :props="casProps">
            <template #default="{ node, data }">
              <span>{{ data.name }}</span>
              <span v-if="!node.isLeaf">
                ({{ data.children.length }})
              </span>
            </template>
          </el-cascader>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="relationAddOrUpdate = false">取消</el-button>
        <el-button type="primary" @click="relationAddOrUpdateHandle()">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue"
import AddOrUpdate from "./brand-add-or-update.vue"
import baseService from "@/service/baseService"
import { ElTable } from 'element-plus'
import { ElMessage, ElMessageBox } from "element-plus"

interface Tree {
  [x: string]: any
  id: number
  label: string
  children?: Tree[]
}

const casProps = reactive({
  value: 'catId',
  label: 'name',
  children: 'children',
  emitPath: false,
})

const treeDataRef = ref<Tree[]>([])

let cateRelationDialogVisible = ref(false)
let relationAddOrUpdate = ref(false)

let relationLoading = ref(false)
let loading = ref(false)

let curCatRelations = ref([])

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/product/brand/page",
  getDataListIsPage: true,
  exportURL: "/product/brand/export",
  deleteURL: "/product/brand/delete"
});

const rules = ref({
  catelogName: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ]
})

let dataForm = {
  id: '', brandId: '', catelogId: '', brandName: '', catelogName: ''
}

let dataFormRef = ref(dataForm)

let queryKey = ref<string>('')

const state = reactive({ ...useView(view), ...toRefs(view) })

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  getInfo()
  addOrUpdateRef.value.init(id);
}

const brandTable = ref<InstanceType<typeof ElTable>>()

const getInfo = () => { //替换state.dataList
  // state.dataListLoading = true
  baseService.get("/product/brand/page").then((res) => {
    console.log("成功获取Brand data: ", res.data)
    state.dataList = res.data.list
    state.total = res.data.total
    console.log("成功获取state.dataList: ", state.dataList)
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

const getCategoryRelation = (brandId: number | string) => {
  baseService.get(`product/brand/getCategoryRelation/${brandId}`).then((res) => {
    curCatRelations.value = res.data
  }).catch(err => {
    ElMessage.error("获取品牌分类失败!")
  })
}

const handleCategoryRelation = (row: any) => {
  dataFormRef.value.brandId = row.brandId
  dataFormRef.value.brandName = row.name
  console.log("row", dataFormRef.value, row)
  curCatRelations.value = []
  cateRelationDialogVisible.value = true
  getCategoryRelation(row.brandId)
}

const query = () => {
  baseService.get(`product/brand/page?key=${queryKey.value}`).then((res) => {
    ElMessage.success("查询成功!")
    console.log("查询成功", res.data.list)
    state.dataList = res.data.list
    state.total = res.data.total
  }).catch(err => {
    ElMessage.error("查询失败!")
  })
}

const addOrUpdateRelation = (id?: number) => {
  loading.value = true
  baseService.get("/product/category/page/tree").then((res) => { //获取分类数据
    console.log("成功获取Tree data: ", res.data)
    treeDataRef.value = res.data
    loading.value = false // 数据加载完成后，关闭加载状态
    if (id) { //修改
      baseService.get(`product/categorybrandrelation/${id}`).then((res) => {
        console.log("查询成功", res.data)
        dataFormRef.value = res.data
        relationAddOrUpdate.value = true
      }).catch(err => {
        ElMessage.error("获取品牌信息失败!")
      })
    } else { //新增
      dataFormRef.value.id = ''
      dataFormRef.value.catelogId = ''
      dataFormRef.value.catelogName = ''
      relationAddOrUpdate.value = true
    }
  }).catch(() => {
    ElMessage.error("获取分类数据失败！")
    loading.value = false // 如果请求失败，也关闭加载状态
  })
}

const deleteRelation = (row: any) => {
  ElMessageBox.confirm(`此操作将永久删除该关联，确定删除？`, '删除确认', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  }).then(() => {
    baseService.delete("/product/categorybrandrelation", [row.id]).then(() => {
      getCategoryRelation(row.brandId)
      ElMessage.success("删除关联成功！")
    }).catch(() => {
      ElMessage.error("删除品牌数据失败！")
    })
  }).catch(() => {
    ElMessage.info('已取消删除')
  })
}

const dataFormSubmitHandle = () => {
  cateRelationDialogVisible.value = false
  ElMessage.success("保存更改")
}

const relationAddOrUpdateHandle = () => {
  if (!dataFormRef.value.catelogId) {
    ElMessage.warning("请选择分类!")
    return
  }

  baseService.get(`/product/category/${dataFormRef.value.catelogId}`).then((res) => {
    dataFormRef.value.catelogName = res.data.name
    if (!dataFormRef.value.id) { //新增
      baseService.post("/product/categorybrandrelation", dataFormRef.value).then((res) => {
        relationAddOrUpdate.value = false
        getCategoryRelation(dataFormRef.value.brandId)
        ElMessage.success("保存成功")
      }).catch(err => {
        ElMessage.error("保存失败!")
      })
    } else { //修改
      baseService.put("/product/categorybrandrelation", dataFormRef.value).then((res) => {
        relationAddOrUpdate.value = false
        getCategoryRelation(dataFormRef.value.brandId)
        ElMessage.success("保存成功")
      }).catch(err => {
        ElMessage.error("保存失败!")
      })
    }
  }).catch(err => {
    ElMessage.error("数据获取失败!")
  })
}
</script>

<style>
.logo {
  width: 80px;
  height: 80px;
  border-radius: 5px;
}
</style>
