<template>
  <el-row>
    <el-col :span="4">
      <a class="head">商品分类列表</a>
      <Category @categoryData="getNodeHandle" @categoryTree="getData"></Category>
    </el-col>
    <el-col :span="20">
      <div class="mod-product__attr">
        <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
          <el-form-item>
            <el-input v-model="queryKey" placeholder="组名查询"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button @click="query()">查询</el-button>
          </el-form-item>
          <el-form-item>
            <el-button type="warning" @click="reset">重置</el-button>
          </el-form-item>
          <el-form-item>
            <el-button v-if="state.hasPermission('product:attr:save')" type="primary"
              @click="addOrUpdateHandle()">新增</el-button>
          </el-form-item>
          <el-form-item>
            <el-button v-if="state.hasPermission('product:attr:delete')" type="danger"
              @click="deleteBatch">删除</el-button>
          </el-form-item>
        </el-form>
        <el-table ref="attrTable" v-loading="state.dataListLoading" :data="state.dataList" border
          @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
          <el-table-column type="selection" header-align="center" align="center"></el-table-column>
          <el-table-column prop="attrId" header-align="center" align="center" label="id" width="50"></el-table-column>
          <el-table-column prop="attrName" header-align="center" align="center" label="属性名"
            width="100"></el-table-column>
          <el-table-column v-if="attrtype == 1" prop="searchType" header-align="center" align="center" label="可检索"
            width="90" #default="scope">
            <el-switch v-show="scope.row.attrType != '0'" v-model="scope.row.searchType" class="ml-2" inline-prompt
              style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="是" inactive-text="否"
              :active-value="1" :inactive-value="0" disabled />
            <el-tag v-show="scope.row.attrType == '0'" type="info">不支持</el-tag>
          </el-table-column>
          <el-table-column prop="valueType" header-align="center" align="center" label="值类型">
            <template #default="scope">
              <el-tag type="warning" v-if="scope.row.valueType == 0">单选</el-tag>
              <el-tag type="success" v-else>多选</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="icon" header-align="center" align="center" label="图标" width="110" #default="scope">
            <img v-if="scope.row.icon" :src="scope.row.icon" class="icon" />
          </el-table-column>
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
          <el-table-column prop="enable" header-align="center" align="center" label="启用" width="70">
            <template #default="scope">
              <el-switch v-model="scope.row.enable" class="ml-2" inline-prompt
                style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="是" inactive-text="否"
                :active-value="1" :inactive-value="0" disabled />
            </template>
          </el-table-column>
          <el-table-column prop="catelogName" header-align="center" align="center" label="所属分类" #default="scope">
            <el-cascader placeholder="无" v-model="scope.row.catelogId" :options="treeDataRef" :props="casProps">
            </el-cascader>
          </el-table-column>
          <el-table-column prop="groupName" header-align="center" align="center" label="所属分组">
            <template #default="scope">
              <el-tag v-show="scope.row.attrType == '0'" type="info">不支持</el-tag>
              <el-cascader placeholder="无" v-if="scope.row.attrType != '0'" v-model="scope.row.groupName"
                :options="groupDataRef" :props="groupProps"></el-cascader>
            </template>
          </el-table-column>
          <el-table-column prop="showDesc" header-align="center" align="center" label="快速展示" width="90">
            <template #default="scope">
              <el-tag v-show="scope.row.attrType == '0'" type="info">不支持</el-tag>
              <el-switch v-show="scope.row.attrType != '0'" v-model="scope.row.showDesc" class="ml-2" inline-prompt
                style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="是" inactive-text="否"
                :active-value="1" :inactive-value="0" disabled />
            </template>
          </el-table-column>
          <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
            <template v-slot="scope">
              <el-button v-if="state.hasPermission('product:attr:update')" type="primary" link
                @click="addOrUpdateHandle(scope.row.attrId)">修改</el-button>
              <el-button v-if="state.hasPermission('product:attr:delete')" type="primary" link
                @click="deleteBatch()">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination :current-page="page" :page-sizes="[10, 20, 50, 100]" :page-size="limit" :total="state.total"
          layout="total, sizes, prev, pager, next, jumper" @size-change="pageSizeChangeHandle"
          @current-change="pageCurrentChangeHandle"> </el-pagination>
        <!-- 弹窗, 新增 / 修改 -->
        <add-or-update ref="addOrUpdateRef" @refreshDataList="getInfo" :treeData="treeDataRef"
          :curCatId="curCatId">确定</add-or-update>
      </div>
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
//TODO: 分页bug修正

onMounted(() => {
  getData
})

const page = ref(1)
const limit = ref(10)

const attrtype = ref(1)

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

const pageSizeChangeHandle = (value: number) => {
  limit.value = value
}

const pageCurrentChangeHandle = (value: number) => {
  page.value = value
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
