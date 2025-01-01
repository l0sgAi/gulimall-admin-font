<template>
  <div>
    <el-row :gutter="20">
      <el-col :span="16">
        <el-card class="box-card">
          <el-tabs tab-position="left" style="width:98%">
            <el-tab-pane :label="group.attrGroupName" v-for="(group, gidx) in dataResp.attrGroups"
              :key="group.attrGroupId">
              <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{attrId:xx,val:}]-->
              <el-form ref="form" :model="dataResp">
                <el-form-item :label="attr.attrName" v-for="(attr, aidx) in group.attrs" :key="attr.attrId">
                  <el-input v-model="dataResp.baseAttrs[gidx][aidx].attrId" type="hidden" v-show="false"></el-input>
                  <el-select v-model="dataResp.baseAttrs[gidx][aidx].attrValues" :multiple="attr.valueType == 1"
                    filterable allow-create default-first-option placeholder="请选择或输入值">
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
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import { useRoute } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import baseService from "@/service/baseService"

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

const route = useRoute()
const spuId: any = ref('')
const catalogId: any = ref('')
const dataResp = reactive<DataResp>({
  attrGroups: [],
  baseAttrs: []
})

let spuAttrsMap = reactive<SpuAttrsMap>({})

const clearData = () => {
  dataResp.attrGroups = []
  dataResp.baseAttrs = []
  spuAttrsMap = {}
}

const getSpuBaseAttrs = () => {
  // 获取基本规格参数
  baseService.get(`/product/attr/base/listforspu/${spuId.value}`).then((res: { data: { data: any } }) => {
    res.data.data.forEach((item: any) => {
      spuAttrsMap["" + item.attrId] = item
    })
    ElMessage.success("查询成功!")
  }).catch((err: any) => {
    ElMessage.error("查询失败!")
  })
}

const getQueryParams = () => {
  spuId.value = route.query.spuId
  catalogId.value = route.query.catalogId
  console.log("----", spuId.value, catalogId.value)
}

const showBaseAttrs = () => {
  baseService.get(`/product/attrgroup/${catalogId.value}/withattr`).then((data: any) => {
    data.data.forEach((item: any) => {
      const attrArray: AttrItem[] = []
      item.attrs.forEach((attr: any) => {
        let v = ''
        if (spuAttrsMap["" + attr.attrId]) {
          v = spuAttrsMap["" + attr.attrId].attrValue.split(' ')
          if (v.length === 1) {
            v = v[0] + ''
          }
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
    dataResp.attrGroups = data.data
    ElMessage.success("修改成功!")
  }).catch((err: any) => {
    ElMessage.error("修改失败!")
  })
}

const submitSpuAttrs = () => {
  console.log("·····", dataResp.baseAttrs)
  const submitData: any[] = []
  dataResp.baseAttrs.forEach((item: AttrItem[]) => {
    item.forEach((attr: AttrItem) => {
      let val = ''
      if (Array.isArray(attr.attrValues)) {
        val = attr.attrValues.join(' ')
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

  ElMessageBox.confirm("修改商品规格信息, 是否继续?", "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
    baseService.post(`/product/spuinfo/update/${spuId.value}`, submitData).then(() => {
      ElMessage.success("修改成功!")
    }).catch((err: any) => {
      ElMessage.error("修改失败!")
    })
  }).catch(() => {
    ElMessage.info('已取消删除')
  })
}

// 初始化操作
clearData()
getQueryParams()
if (spuId.value && catalogId.value) {
  showBaseAttrs()
  getSpuBaseAttrs()
}
</script>

<style scoped>
</style>