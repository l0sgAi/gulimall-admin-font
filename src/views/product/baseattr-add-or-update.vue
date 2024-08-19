<template>
  <el-dialog v-model="visible" :title="!dataForm.attrId ? '新增' : '修改'" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="属性名" prop="attrName">
        <el-input v-model="dataForm.attrName" placeholder="属性名"></el-input>
      </el-form-item>
      <el-form-item label="值类型" prop="valueType">
        <el-switch v-model="dataForm.valueType" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="多选" inactive-text="单选"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item v-if="dataForm.attrType != '0'" label="是否需要检索" prop="searchType">
        <el-switch v-model="dataForm.searchType" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="需要检索" inactive-text="不需要检索"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="属性图标" prop="icon">
        <SingleUpload :logo="dataForm.icon" @changeLogo="changeLogoHandle"></SingleUpload>
      </el-form-item>
      <el-form-item label="可选值列表" prop="valueSelectArray">
        <!-- <el-input v-model="dataForm.valueSelect" placeholder="可选值列表[用逗号分隔]"></el-input> -->
        <el-select v-model="dataForm.valueSelectArray" multiple filterable allow-create
          placeholder="请输入内容,点击确认"></el-select>
      </el-form-item>
      <el-form-item label="属性类型" prop="attrType">
        <el-select v-model="dataForm.attrType" placeholder="请选择">
          <el-option label="规格参数" :value="1"></el-option>
          <el-option label="销售属性" :value="0"></el-option>
          <el-option label="规格参数和销售属性" :value="2"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="启用状态" prop="enable">
        <!-- <el-input v-model="dataForm.enable" placeholder="启用状态[0 - 禁用，1 - 启用]"></el-input> -->
        <el-switch v-model="dataForm.enable" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="启用" inactive-text="禁用"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <el-cascader style="width: 75%;" placeholder="搜索分类" v-model="dataForm.catelogId" :options="props.treeData"
          @change="handleChange" filterable :props="casProps">
          <template #default="{ node, data }">
            <span>{{ data.name }}</span>
            <span v-if="!node.isLeaf"> ({{ data.children.length }}) </span>
          </template>
        </el-cascader>
      </el-form-item>
      <el-form-item v-if="dataForm.attrType != '0'" label="分类分组" prop="groupName">
        <el-cascader style="width: 75%;" placeholder="搜索分组" v-model="dataForm.groupName" :options="catAttrgroupRef"
          @change="handleChange" filterable :props="attrGroupProps">
        </el-cascader>
      </el-form-item>
      <el-form-item v-if="dataForm.attrType != '0'" label="快速展示" prop="showDesc">
        <!-- <el-input v-model="dataForm.showDesc" placeholder="快速展示【是否展示在介绍上；0-否 1-是】，在sku中仍然可以调整"></el-input> -->
        <el-switch v-model="dataForm.showDesc" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="启用" inactive-text="禁用"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref, watch } from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";
import SingleUpload from "@/upload/singleUpload.vue"

interface DataForm {
  attrId: any;
  valueType: number;
  attrName: any;
  searchType: any;
  icon: any;
  valueSelect: any;
  valueSelectArray: any[];
  attrType: any;
  enable: any;
  catelogId: any;
  groupName: any;
  showDesc: any; // 允许 showDesc 是 number 或 string 类型
}

const emit = defineEmits(["refreshDataList"]);

const visible = ref(false)
const dataFormRef = ref()

const props = defineProps({
  treeData: { type: Array },
  curCatId: { type: Number }
})

const casProps = reactive({
  value: 'catId',
  label: 'name',
  children: 'children',
  emitPath: false,
})

const attrGroupProps = reactive({
  value: 'attrGroupId',
  label: 'attrGroupName',
})


const catAttrgroupRef = ref([])

const dataForm = reactive<DataForm>({
  attrId: '',
  attrName: '',
  valueType: 0,
  searchType: 0,
  icon: '',
  valueSelect: '',
  valueSelectArray: [],
  attrType: '',
  enable: 0,
  catelogId: props.curCatId,
  groupName: '',
  showDesc: 0
})

const rules = ref({
  attrName: [
    { required: true, message: '属性名为必填', trigger: 'blur' }
  ],
  searchType: [

  ],
  icon: [

  ],
  valueSelect: [

  ],
  attrType: [
    { required: true, message: '请选择一个属性类型!', trigger: 'blur' }
  ],
  enable: [

  ],
  catelogId: [
    { required: true, message: '请选择至少一个分类!', trigger: 'blur' }
  ],
  showDesc: [

  ]
});

const init = (attrId?: number) => {
  visible.value = true
  dataForm.attrId = ""

  dataForm.attrName = ""
  dataForm.searchType = 0
  dataForm.icon = ''
  dataForm.valueSelect = ''
  dataForm.valueSelectArray = []
  dataForm.attrType = ''
  dataForm.enable = 0
  dataForm.catelogId = props.curCatId
  dataForm.groupName = ''
  dataForm.showDesc = 0

  // 重置表单数据
  // if (dataFormRef.value) {
  //   dataFormRef.value.resetFields();
  // }

  if (attrId) {
    getInfo(attrId);
  }
};

// 获取信息
const getInfo = (attrId: number) => {
  baseService.get("/product/attr/" + attrId).then((res) => {
    Object.assign(dataForm, res.data)
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  if (dataForm.attrType == '0') {
    dataForm.searchType = ''
    dataForm.showDesc = ''
    dataForm.groupName = ''
  }
  if (dataForm.valueSelectArray)
    dataForm.valueSelect = dataForm.valueSelectArray.join(';')
  if (dataForm.groupName)
    dataForm.groupName = dataForm.groupName[0]
  // console.log("dataForm", dataForm)

  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.attrId ? baseService.post : baseService.put)("/product/attr", dataForm).then(() => {
      ElMessage.success({
        message: '成功',
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};

const handleChange = (value: any) => {
  console.log("值改变!", value)
}

defineExpose({
  init
})

const changeLogoHandle = (logo: string) => {
  dataForm.icon = logo
}

const getCategoryAttrGroup = (curCatId: number) => {
  baseService.get(`product/attrgroup/query/attrgroup/${curCatId}`).then((res) => {
    catAttrgroupRef.value = res.data
  }).catch(err => {
    ElMessage.error("数据获取失败!")
    console.log("error", err)
  })
}

watch(() => dataForm.catelogId, (newValue) => {
  getCategoryAttrGroup(Number(newValue))
})

</script>
