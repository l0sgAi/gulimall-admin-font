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
      <el-table-column prop="catalogId" label="分类id" header-align="center" align="center">
      </el-table-column>
      <el-table-column prop="catalogName" label="分类名称" header-align="center" align="center">
      </el-table-column>
      <el-table-column prop="brandId" label="品牌id" header-align="center" align="center">
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
            @click="showAttr(scope.row)">规格</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <!-- <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update> -->
    <el-dialog v-loading="loading" title="规格维护" v-model="isShow" width="70%">
      <el-row :gutter="30">
        <el-col :span="23">
          <el-card class="box-card">
            <el-tabs tab-position="left" style="width:100%">
              <el-tab-pane :label="group.attrGroupName" v-for="(group, gidx) in dataResp.attrGroups"
                :key="group.attrGroupId">
                <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{attrId:xx,val:}]-->
                <el-form ref="form" :model="dataResp">
                  <el-form-item :label="attr.attrName" v-for="(attr, aidx) in group.attrs" :key="attr.attrId">
                    <el-input v-model="dataResp.baseAttrs[gidx][aidx].attrId" type="hidden" v-show="false"></el-input>
                    <el-select v-model="dataResp.baseAttrs[gidx][aidx].attrValues" :multiple="true" filterable
                      allow-create default-first-option placeholder="请选择或输入值">
                      <el-option v-for="(val, vidx) in attr.valueSelect.split(' ')" :key="vidx" :label="val"
                        :value="val"></el-option>
                    </el-select>
                    <el-checkbox v-model="dataResp.baseAttrs[gidx][aidx].showDesc" :true-label="1"
                      :false-label="0">快速展示</el-checkbox>
                  </el-form-item>
                </el-form>
              </el-tab-pane>
            </el-tabs>
            <div style="margin:auto">
              <el-button type="success" style="float:right" @click="submitSpuAttrs">确认修改</el-button>
            </div>
          </el-card>
        </el-col>
      </el-row>
      <template #footer>
        <el-button @click="isShow = false">返回</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue"
import CategoryCascader from "../common/category-cascader.vue"
import BrandSelect from "../common/brand-select.vue"
import baseService from "@/service/baseService"
import { ElMessage, ElMessageBox } from "element-plus"
import { useRouter } from 'vue-router';

const router = useRouter();
const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/product/spuinfo/page",
  getDataListIsPage: true,
  exportURL: "/product/spuinfo/export",
  deleteURL: "/product/spuinfo"
});

interface SpuItem {
  id: number
  spuName: string
  spuDescription: string
  catalogId: number
  catalogName: string
  brandId: number
  brandName: string
  weight: number
  publishStatus: number
}

interface DataResp {
  attrGroups: any[]
  baseAttrs: any[]
}

interface SpuAttrsMap {
  [key: string]: any
}

interface AttrItem {
  attrId: number
  attrName: string
  attrValues: string | string[]
  showDesc: boolean
}

const spuId: any = ref('')
const catalogId: any = ref('')
const dataResp = reactive<DataResp>({
  attrGroups: [],
  baseAttrs: []
})

let spuAttrsMap = reactive<SpuAttrsMap>({})

let loading = ref(false) // 控制弹框是否加载
let isShow = ref(false) // 控制弹窗是否显示

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

const grounding = (id: number) => { // 上下架管理
  baseService.post(`product/spuinfo/${id}/up`, dataForm).then((res: { data: { list: any; total: number } }) => {
    reset()
    ElMessage.success("操作成功!")
  }).catch((err: any) => {
    ElMessage.error("操作失败!")
  })
}

const showAttr = (row: SpuItem) => { // 规格管理
  console.log(row)
  spuId.value = row.id
  catalogId.value = row.catalogId
  // 初始化操作
  clearData()

  if (spuId.value && catalogId.value) {
    showBaseAttrs()
    getSpuBaseAttrs()
  }
  isShow.value = true
}

// const addOrUpdateRef = ref();
// const addOrUpdateHandle = (id?: number) => {
//   addOrUpdateRef.value.init(id);
// };


const clearData = () => {
  dataResp.attrGroups = []
  dataResp.baseAttrs = []
  spuAttrsMap = {}
}

const getSpuBaseAttrs = () => {
  // 获取基本规格参数
  baseService.get(`/product/attr/base/list/${spuId.value}`).then((res) => {
    res.data.forEach((item: any) => {
      spuAttrsMap["" + item.attrId] = item
    })
    console.log("***spuAttrsMap", spuAttrsMap)
    ElMessage.success("查询成功!")
  }).catch((err: any) => {
    ElMessage.error("查询失败!")
  })
}

// const getQueryParams = () => {
//   spuId.value = route.query.spuId
//   catalogId.value = route.query.catalogId
//   console.log("----", spuId.value, catalogId.value)
// }

const showBaseAttrs = () => { // 获取属性分组下的所有规格
  baseService.get(`/product/attrgroup/withattr/${catalogId.value}`).then((res) => {
    res.data.forEach((item: any) => {
      const attrArray: AttrItem[] = []
      item.attrs.forEach((attr: any) => {
        let v = '' // 属性值，多选值用;隔开
        if (spuAttrsMap["" + attr.attrId]) {
          v = spuAttrsMap["" + attr.attrId].attrValue.split(';')
          console.log("***v", v)
          // if (v.length === 1) {
          //   v = v[0] + ''
          // }
        }
        attrArray.push({
          attrId: attr.attrId,
          attrName: attr.attrName,
          attrValues: v,
          showDesc: spuAttrsMap["" + attr.attrId] ? spuAttrsMap["" + attr.attrId].quickShow : attr.showDesc
        })
      })
      dataResp.baseAttrs.push(attrArray)
    })
    dataResp.attrGroups = res.data
  }).catch(() => {
    ElMessage.error("请求参数失败!")
  })
}

const submitSpuAttrs = () => {
  console.log("***dataResp", dataResp.baseAttrs)
  const submitData: any[] = []
  dataResp.baseAttrs.forEach((item: AttrItem[]) => {
    item.forEach((attr: AttrItem) => {
      let val = ''
      if (Array.isArray(attr.attrValues)) {
        val = attr.attrValues.join(';')
      } else {
        val = attr.attrValues
      }

      if (val !== '') {
        submitData.push({
          attrId: attr.attrId,
          attrName: attr.attrName,
          attrValue: val,
          quickShow: attr.showDesc
        })
      }
    })
  })

  console.log("***submitData", submitData)

  ElMessageBox.confirm("修改商品规格信息, 是否继续?", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
    baseService.post(`/product/attr/update/${spuId.value}`, submitData).then(() => {
      ElMessage.success("修改成功!")
    }).catch(() => {
      ElMessage.error("修改失败!")
    })
  }).catch(() => {
    ElMessage.info('已取消')
  })
}

</script>
