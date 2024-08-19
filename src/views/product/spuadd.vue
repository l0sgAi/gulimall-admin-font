<template>
  <div>
    <el-row>
      <el-col :span="24">
        <el-steps :active="step" finish-status="success">
          <el-step title="基本信息"></el-step>
          <el-step title="规格参数"></el-step>
          <el-step title="销售属性"></el-step>
          <el-step title="SKU信息"></el-step>
          <el-step title="保存完成"></el-step>
        </el-steps>
      </el-col>
      <el-col :span="24" v-show="step == 0">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-form ref="spuBaseForm" :model="spu" label-width="120px" :rules="spuBaseInfoRules">
            <el-form-item label="商品名称" prop="spuName">
              <el-input v-model="spu.spuName"></el-input>
            </el-form-item>
            <el-form-item label="商品描述" prop="spuDescription">
              <el-input v-model="spu.spuDescription"></el-input>
            </el-form-item>
            <el-form-item label="选择分类" prop="catalogId">
              <category-cascader @update:catelogPath="catelogChangeHandle"></category-cascader>
            </el-form-item>
            <el-form-item label="选择品牌" prop="brandId">
              <brand-select @changeBrand="brandChangeHandle"></brand-select>
            </el-form-item>
            <el-form-item label="商品重量(Kg)" prop="weight">
              <el-input-number v-model.number="spu.weight" :min="0" :precision="3" :step="0.1"></el-input-number>
            </el-form-item>
            <el-form-item label="设置积分" prop="bounds">
              <label style="margin-right:15px">金币</label>
              <el-input-number style="width:130px" placeholder="金币" v-model="spu.bounds.buyBounds" :min="0"
                controls-position="right"></el-input-number>
              <label style="margin-left:30px;margin-right:15px">成长值</label>
              <el-input-number style="width:130px" placeholder="成长值" v-model="spu.bounds.growBounds" :min="0"
                controls-position="right">
                <template slot="prepend">成长值</template>
              </el-input-number>
            </el-form-item>
            <el-form-item label="商品介绍" prop="decript">
              <multi-upload @changeLogos="decriptChangeHandle"></multi-upload>
            </el-form-item>
            <el-form-item label="商品图集" prop="images">
              <multi-upload @changeLogos="imagesChangeHandle" :limit="10"></multi-upload>
            </el-form-item>
            <el-form-item>
              <el-button type="success" @click="collectSpuBaseInfo()">下一步：设置基本参数</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
      <el-col :span="24" v-show="step == 1">
        <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-tabs tab-position="left" style="width:98%">
            <el-tab-pane :label="group.attrGroupName" v-for="(group, gidx) in dataResp.attrGroups"
              :key="group.attrGroupId">
              <!-- 遍历属性,每个tab-pane对应一个表单，每个属性是一个表单项  spu.baseAttrs[0] = [{attrId:xx,val:}]-->
              <el-form ref="form" :model="spu">
                <el-form-item :label="attr.attrName" v-for="(attr, aidx) in group.attrs" :key="attr.attrId"
                  v-show="attr.attrType != 0" style="width: 800px;">
                  <el-input v-model="dataResp.baseAttrs[gidx][aidx].attrId" type="hidden" v-show="false"></el-input>
                  <el-switch v-model="dataResp.baseAttrs[gidx][aidx].showDesc" class="ml-2" inline-prompt
                    style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949;" active-text="快速展示"
                    inactive-text="不快速展示" :active-value="1" :inactive-value="0" />
                  <el-select v-model="dataResp.baseAttrs[gidx][aidx].attrValues" :multiple="attr.valueType == 1"
                    filterable allow-create default-first-option placeholder="请选择或输入值">
                    <el-option v-for="(val, vidx) in attr.valueSelect.split(';')" :key="vidx" :label="val"
                      :value="val"></el-option>
                  </el-select>
                </el-form-item>
              </el-form>
            </el-tab-pane>
          </el-tabs>
          <div style="margin:auto">
            <el-button type="primary" @click="step = 0">上一步</el-button>
            <el-button type="success" @click="generateSaleAttrs">下一步：设置销售属性</el-button>
          </div>
        </el-card>
      </el-col>
      <el-col :span="24" v-show="step == 2">
        <!-- <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>选择销售属性</span>
              <el-form ref="saleform" :model="spu">
                <el-form-item :label="attr.attrName" v-for="(attr, aidx) in dataResp.saleAttrs" :key="attr.attrId">
                  <el-input v-model="dataResp.tempSaleAttrs[aidx].attrId" type="hidden" v-show="false"></el-input>
                  <el-checkbox-group v-model="dataResp.tempSaleAttrs[aidx].attrValues">
                    <el-checkbox v-if="dataResp.saleAttrs[aidx].valueSelect != ''" :label="val"
                      v-for="val in dataResp.saleAttrs[aidx].valueSelect.split(';')" :key="val"></el-checkbox>
                    <div style="margin-left:20px;display:inline">
                      <el-button v-show="!inputVisible[aidx].view" class="button-new-tag" size="mini"
                        @click="showInput(aidx)">+自定义</el-button>
                      <el-input v-show="inputVisible[aidx].view" v-model="inputValue[aidx].val"
                        :ref="'saveTagInput' + aidx" size="mini" style="width:150px"
                        @keyup.enter.native="handleInputConfirm(aidx)" @blur="handleInputConfirm(aidx)"></el-input>
                    </div>
                  </el-checkbox-group>
                </el-form-item>
              </el-form>
            </div>
            <el-button type="primary" @click="step = 1">上一步</el-button>
            <el-button type="success" @click="generateSkus">下一步：设置SKU信息</el-button>
          </el-card>
        </el-card> -->
      </el-col>
      <el-col :span="24" v-show="step == 3">
        <!-- <el-card class="box-card" style="width:80%;margin:20px auto">
          <el-table :data="spu.skus" style="width: 100%">
            <el-table-column label="属性组合">
              <el-table-column :label="item.attrName" v-for="(item, index) in dataResp.tableAttrColumn"
                :key="item.attrId">
                <template slot-scope="scope">
                  <span style="margin-left: 10px">{{ scope.row.attr[index].attrValue }}</span>
                </template>
              </el-table-column>
            </el-table-column>
            <el-table-column label="商品名称" prop="skuName">
              <template slot-scope="scope">
                <el-input v-model="scope.row.skuName"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="标题" prop="skuTitle">
              <template slot-scope="scope">
                <el-input v-model="scope.row.skuTitle"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="副标题" prop="skuSubtitle">
              <template slot-scope="scope">
                <el-input v-model="scope.row.skuSubtitle"></el-input>
              </template>
            </el-table-column>
            <el-table-column label="价格" prop="price">
              <template slot-scope="scope">
                <el-input v-model="scope.row.price"></el-input>
              </template>
            </el-table-column>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-row>
                  <el-col :span="24">
                    <label style="display:block;float:left">选择图集 或</label>
                    <multi-upload style="float:left;margin-left:10px;" :showFile="false" :listType="'text'"
                      v-model="uploadImages"></multi-upload>
                  </el-col>
                  <el-col :span="24">
                    <el-divider></el-divider>
                  </el-col>
                  <el-col :span="24">
                    <el-card style="width:170px;float:left;margin-left:15px;margin-top:15px;"
                      :body-style="{ padding: '0px' }" v-for="(img, index) in spu.images" :key="index">
                      <img :src="img" style="width:160px;height:120px" />
                      <div style="padding: 14px;">
                        <el-row>
                          <el-col :span="12">
                            <el-checkbox v-model="scope.row.images[index].imgUrl" :true-label="img"
                              false-label></el-checkbox>
                          </el-col>
                          <el-col :span="12">
                            <el-tag v-if="scope.row.images[index].defaultImg == 1">
                              <input type="radio" checked :name="scope.row.skuName"
                                @change="checkDefaultImg(scope.row, index, img)" />设为默认
                            </el-tag>
                            <el-tag v-else>
                              <input type="radio" :name="scope.row.skuName"
                                @change="checkDefaultImg(scope.row, index, img)" />设为默认
                            </el-tag>
                          </el-col>
                        </el-row>
                      </div>
                    </el-card>
                  </el-col>
                </el-row> -->
        <!-- 折扣，满减，会员价 -->
        <!-- <el-form :model="scope.row">
                  <el-row>
                    <el-col :span="24">
                      <el-form-item label="设置折扣">
                        <label>满</label>
                        <el-input-number style="width:160px" :min="0" controls-position="right"
                          v-model="scope.row.fullCount"></el-input-number>
                        <label>件</label>

                        <label style="margin-left:15px;">打</label>
                        <el-input-number style="width:160px" v-model="scope.row.discount" :precision="2" :max="1"
                          :min="0" :step="0.01" controls-position="right"></el-input-number>
                        <label>折</label>
                        <el-checkbox v-model="scope.row.countStatus" :true-label="1"
                          :false-label="0">可叠加优惠</el-checkbox>
                      </el-form-item>
                    </el-col>
                    <el-col :span="24">
                      <el-form-item label="设置满减">
                        <label>满</label>
                        <el-input-number style="width:160px" v-model="scope.row.fullPrice" :step="100" :min="0"
                          controls-position="right"></el-input-number>
                        <label>元</label>
                        <label style="margin-left:15px;">减</label>
                        <el-input-number style="width:160px" v-model="scope.row.reducePrice" :step="10" :min="0"
                          controls-position="right"></el-input-number>
                        <label>元</label>
                        <el-checkbox v-model="scope.row.priceStatus" :true-label="1"
                          :false-label="0">可叠加优惠</el-checkbox>
                      </el-form-item>
                    </el-col>

                    <el-col :span="24">
                      <el-form-item label="设置会员价" v-if="scope.row.memberPrice.length > 0">
                        <br /> -->
        <!--   @change="handlePriceChange(scope,mpidx,$event)" -->
        <!-- <el-form-item v-for="(mp, mpidx) in scope.row.memberPrice" :key="mp.id">
                          {{ mp.name }}
                          <el-input-number style="width:160px" v-model="scope.row.memberPrice[mpidx].price"
                            :precision="2" :min="0" controls-position="right"></el-input-number>
                        </el-form-item>
                      </el-form-item>
                    </el-col>
                  </el-row>
                </el-form>
              </template>
            </el-table-column>
          </el-table>
          <el-button type="primary" @click="step = 2">上一步</el-button>
          <el-button type="success" @click="submitSkus">下一步：保存商品信息</el-button>
        </el-card> -->
      </el-col>
      <el-col :span="24" v-show="step == 4">
        <!-- <el-card class="box-card" style="width:80%;margin:20px auto">
          <h1>保存成功</h1>
          <el-button type="primary" @click="addAgian">继续添加</el-button>
        </el-card> -->
      </el-col>
    </el-row>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue'
import CategoryCascader from "../common/category-cascader.vue"
import BrandSelect from "../common/brand-select.vue"
import MultiUpload from "@/upload/multiUpload.vue"
import PubSub from 'pubsub-js'
import { ElMessage } from 'element-plus'
import baseService from "@/service/baseService"

const catPathSub = ref('')
const brandIdSub = ref('')

// 定义组件
const CategoryCascaderComponent = ref(null);
const BrandSelectComponent = ref(null);
const MultiUploadComponent = ref(null);

// 定义响应式数据
const uploadDialogVisible = ref(false);
const uploadImages = ref([]);
const step = ref(0);

const spu = reactive({
  spuName: "",
  spuDescription: "",
  catalogId: 0,
  brandId: 0,
  weight: "",
  publishStatus: 0,
  decript: <any>[],
  images: <any>[],
  bounds: {
    buyBounds: 0,
    growBounds: 0
  },
  baseAttrs: [],
  skus: []
})
const spuBaseForm = ref();

const spuBaseInfoRules = reactive({
  spuName: [
    { required: true, message: "请输入商品名字", trigger: "blur" }
  ],
  spuDescription: [
    { required: false, message: "请编写一个简单描述", trigger: "blur" }
  ],
  catalogId: [
    { required: true, message: "请选择一个分类", trigger: "blur" }
  ],
  brandId: [
    { required: true, message: "请选择一个品牌", trigger: "blur" }
  ],
  decript: [
    { required: false, message: "请上传商品详情图集", trigger: "blur" }
  ],
  images: [
    { required: false, message: "请上传商品图片集", trigger: "blur" }
  ],
  weight: [
    {
      type: "number",
      required: true,
      message: "请填写正确的重量值",
      trigger: "blur"
    }
  ]
})

const dataResp = reactive({
  attrGroups: <any>[],
  baseAttrs: <any>[],
  saleAttrs: <any>[],
  tempSaleAttrs: [],
  tableAttrColumn: [],
  memberLevels: [],
  steped: [false, false, false, false, false]
})

const inputVisible = reactive([]);
const inputValue = reactive([]);

// 方法
const addAgian = () => {
  step.value = 0;
  resetSpuData();
};

const resetSpuData = () => {
  Object.assign(spu, {
    spuName: "",
    spuDescription: "",
    catalogId: 0,
    brandId: "",
    weight: "",
    publishStatus: 0,
    decript: [],
    images: [],
    bounds: {
      buyBounds: 0,
      growBounds: 0
    },
    baseAttrs: [],
    skus: []
  })
}

const handlePriceChange = (scope: any, mpidx: any, e: any) => {

}

const getMemberLevels = () => {
  baseService.get("/member/memberlevel/page").then((res) => {
    dataResp.memberLevels = res.data.list;
  }).catch(err => {
    ElMessage.error("获取会员等级数据失败！")
    console.log("getMemberLevels_error", err)
  })
}

const showInput = (idx: any) => {

}

const checkDefaultImg = (row: any, index: any, img: any) => {

}

const handleInputConfirm = (idx: any) => {

}

const collectSpuBaseInfo = () => {
  // console.log("验证表单", spuBaseForm.value)
  spuBaseForm.value.validate((valid: boolean) => {
    if (valid) {
      console.log("spu_第一步", spu)
      showBaseAttrs()
    } else {
      ElMessage.warning("请填写这一步必要的信息")
      // console.log("验证失败")
    }
  })
}

const generateSaleAttrs = () => {

}

const generateSkus = () => {

}

const hasAndReturnSku = (skus: any, descar: any) => {

}

const getShowSaleAttr = () => {

}

const showBaseAttrs = () => {
  baseService.get(`/product/attrgroup/withattr/${spu.catalogId}`).then((res) => {
    //先对表单的baseAttrs进行初始化
    // console.log("res.data", res.data)
    res.data.forEach((item: { attrs: any[] }) => {
      let attrArray = <any>[]
      if (item && item.attrs) {
        item.attrs.forEach((attr: { attrId: any; showDesc: any; attrType: number }) => {
          attrArray.push({
            attrId: attr.attrId,
            attrValues: "",
            showDesc: attr.showDesc
          })
        })
      }
      // console.log("attrArray", attrArray)
      dataResp.baseAttrs.push(attrArray)
    })
    console.log("dataResp", dataResp)
    dataResp.steped[0] = true
    dataResp.attrGroups = res.data
  }).catch(err => {
    ElMessage.error("获取规格数据失败！")
    console.log("getMemberLevels_error", err)
  })

  step.value = 1
}

const submitSkus = () => {

}

const descartes = (list: Array<any>) => {

}

onBeforeUnmount(() => {
  PubSub.unsubscribe(catPathSub.value)
  PubSub.unsubscribe(brandIdSub.value)
})

// 生命周期钩子
onMounted(() => {
  const catPathSub = PubSub.subscribe("catPath", (msg, val) => {
    spu.catalogId = val[val.length - 1];
  });
  const brandIdSub = PubSub.subscribe("brandId", (msg, val) => {
    spu.brandId = val;
  });
  getMemberLevels();
})

const catelogChangeHandle = (catId: number) => {
  spu.catalogId = catId
}

const brandChangeHandle = (brandId: number) => {
  spu.brandId = brandId
}

const decriptChangeHandle = (imgList: string[]) => {
  spu.decript = imgList
}

const imagesChangeHandle = (imgList: string[]) => {
  spu.images = imgList
}
</script>
