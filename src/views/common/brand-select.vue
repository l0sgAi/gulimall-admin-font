<template>
  <div>
    <el-select placeholder="请选择" v-model="brandId" filterable clearable>
      <el-option v-for="item in brands" :key="item.brandId" :label="item.brandName" :value="item.brandId"></el-option>
    </el-select>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { ElSelect, ElOption } from 'element-plus'

interface Brand {
  brandName: string;
  brandId: number;
}

const catId = ref(0);
const brands = ref<Brand[]>([
  {
    brandName: "a",
    brandId: 1
  }
]);
const brandId = ref<string>("");
let subscribe: string | null = null;

const getCatBrands = () => {
  // TODO: 编写并调用获取当前分类的品牌的接口
}

const handleCatPathChange = (msg: string, val: number[]) => {
  catId.value = val[val.length - 1];
  getCatBrands();
};

onMounted(() => {
  subscribe = PubSub.subscribe('catPath', handleCatPathChange);
});

onBeforeUnmount(() => {
  if (subscribe) {
    PubSub.unsubscribe(subscribe);
  }
});

</script>