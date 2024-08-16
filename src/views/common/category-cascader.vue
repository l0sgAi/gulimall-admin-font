<template>
  <!-- 
使用说明：
1）、引入category-cascader.vue
2）、语法：<category-cascader :catelogPath.sync="catelogPath"></category-cascader>
    解释：
      catelogPath：指定的值是cascader初始化需要显示的值，应该和父组件的catelogPath绑定;
          由于有sync修饰符，所以cascader路径变化以后自动会修改父的catelogPath，这是结合子组件this.$emit("update:catelogPath",v);做的
      -->
  <div>
    <el-cascader filterable clearable placeholder="搜索分类" v-model="paths" :options="categorys" :props="settings">
      <template #default="{ node, data }">
        <span>{{ data.name }}</span>
        <span v-if="!node.isLeaf"> ({{ data.children.length }}) </span>
      </template>
    </el-cascader>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { ElMessage } from 'element-plus';
import baseService from "@/service/baseService"

onMounted(() => {
  getCategorys()
})

interface Tree {
  [key: string]: any
  id: number
  label: string
  children?: Tree[]
}

const props = withDefaults(defineProps<{
  catelogPath: number[];
}>(), {
  catelogPath: () => []
});

const paths = ref<number[]>(props.catelogPath)

const settings = {
  value: 'catId',
  label: 'name',
  children: 'children'
}

const categorys = ref<Tree[]>([]);

const getCategorys = () => {
  baseService.get("/product/category/page/tree").then((res: any) => {
    console.log("成功获取Tree data: ", res.data)
    categorys.value = res.dataForm
  }).catch(() => {
    ElMessage.error("获取分类数据失败！")
  })
}

watch(() => props.catelogPath, (newVal) => {
  paths.value = newVal;
}
)

watch(() => paths.value, (newValue) => {
  emit('update:catelogPath', newValue)
})

const emit = defineEmits(['update:catelogPath']);
</script>