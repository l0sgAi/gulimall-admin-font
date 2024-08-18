<template>
  <div style="width: 300px;">
    <el-select placeholder="请选择" v-model="brandId" filterable clearable>
      <el-option v-for="item in brands" :key="item.brandId" :label="item.name" :value="item.brandId"></el-option>
    </el-select>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import { ElSelect, ElOption } from 'element-plus'
import PubSub from 'pubsub-js'
import baseService from "@/service/baseService"
import { ElMessage } from 'element-plus'

interface IBrand {
  brandId: number;
  name: string;
  logo: string;
}

const emit = defineEmits(['changeBrand'])

const brands = ref<IBrand[]>([{ brandId: 0, name: '暂无数据', logo: '' }])
const catId = ref(0)

const brandId = ref<number>(0);
let subscribe: string | null = null

const getCatBrands = () => {
  baseService.get(`/product/brand/getCategoryBrand/${catId.value}`).then((res) => {
    // ElMessage.success("查询成功!")
    brands.value = res.data
    console.log("查询成功", res.data)
  }).catch(err => {
    ElMessage.error("查询分类对应品牌失败!")
    console.log("getCatBrands_err", err)
  })

}

const emitInput = (val: number) => {
  emit('changeBrand', val)
}

const handleCatPathChange = (_msg: string, val: number[]) => {
  catId.value = val[val.length - 1]
  console.log("handleCatPathChange", catId.value)
  getCatBrands()
};

onMounted(() => {
  subscribe = PubSub.subscribe('catPath', handleCatPathChange)
  //使用PubSub传值：handleCatPathChange为结束数据的方法
})

onBeforeUnmount(() => {
  if (subscribe) {
    PubSub.unsubscribe(subscribe);
  }
})

watch(() => catId.value, (newValue) => {
  emitInput(newValue)
})
</script>