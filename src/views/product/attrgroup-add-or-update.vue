<template>
  <el-dialog v-model="visible" :title="!dataForm.attrGroupId ? '新增' : '修改'" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="组名" prop="attrGroupName">
        <el-input v-model="dataForm.attrGroupName" placeholder="组名"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input-number v-model="dataForm.sort" :precision="0" :step="1" :min="0" :max="10000000" placeholder="排序" />
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" :rows="5" type="textarea" placeholder="组描述"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <SingleUpload :logo="dataForm.icon" @changeLogo="changeLogoHandle"></SingleUpload>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <el-cascader style="width: 75%;" placeholder="搜索分类" v-model="dataForm.catelogId" :options="props.treeData"
          @change="handleChange" filterable :props="casProps">
          <template #default="{ node, data }">
            <span>{{ data.name }}</span>
            <span v-if="!node.isLeaf"> ({{ data.children.length }}) </span>
          </template>
        </el-cascader>
        <!-- <el-input v-model="dataForm.catelogId" placeholder="所属分类"></el-input> -->
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">确定</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { ElMessage } from "element-plus";
import SingleUpload from "@/upload/singleUpload.vue"

interface Tree {
  [x: string]: any
  id: number
  label: string
  children?: Tree[]
}

const emit = defineEmits(["refreshDataList"]);

const props = defineProps({
  treeData: { type: Array },
  curCatId: { type: Number },
})

const casProps = reactive({
  value: 'catId',
  label: 'name',
  children: 'children',
  emitPath: false,
})

// let treeDataRef = ref(props.treeData)

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  attrGroupId: '', attrGroupName: '', sort: 0, descript: '', icon: '', catelogId: props.curCatId
})

const rules = ref({
  attrGroupName: [
    { required: true, message: '必填项不能为空', trigger: 'blur' },
    { min: 1, max: 20, message: '分组名过长!', trigger: 'blur' }
  ],
  sort: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  descript: [

  ],
  icon: [
  ],
  catelogId: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ]
});

const init = (attrGroupId?: number) => {
  visible.value = true
  dataForm.attrGroupId = ""
  dataForm.attrGroupName = ""
  dataForm.sort = 0
  dataForm.descript = ''
  dataForm.icon = ''
  dataForm.catelogId = props.curCatId

  // 重置表单数据
  // if (dataFormRef.value) {
  //   dataFormRef.value.resetFields();
  // }

  if (attrGroupId) {
    getInfo(attrGroupId);
  }
};

// 获取信息
const getInfo = (attrGroupId: number) => {
  baseService.get("/product/attrgroup/" + attrGroupId).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.attrGroupId ? baseService.post : baseService.put)("/product/attrgroup", dataForm).then(() => {
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

defineExpose({
  init
});

const changeLogoHandle = (logo: string) => {
  dataForm.icon = logo
}

const handleChange = (value: any) => {
  console.log("值改变!", value)
}

</script>
