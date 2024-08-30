<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="优先级" prop="priority">
        <el-input v-model="dataForm.priority" placeholder="优先级"></el-input>
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
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: '', assigneeId: '', assigneeName: '', phone: '', priority: '', status: 0, wareId: '', amount: '', createTime: '', updateTime: ''
});

const rules = ref({
  assigneeId: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  assigneeName: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  phone: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  priority: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  status: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  wareId: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  amount: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  createTime: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  updateTime: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ]
});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if (id) {
    getInfo(id);
  }
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/ware/wmspurchase/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false
    }
    dataForm.status = 0;
    (!dataForm.id ? baseService.post : baseService.put)("/ware/wmspurchase", dataForm).then((res) => {
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
</script>
