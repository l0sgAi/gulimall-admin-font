<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="sku_id" prop="skuId">
        <el-input-number style="width:130px" placeholder="sku_id" v-model="dataForm.skuId" :min="0" :precision="0"
          :step="1" controls-position="right"></el-input-number>
      </el-form-item>
      <el-form-item label="仓库" prop="wareId">
        <el-select v-model="dataForm.wareId" placeholder="请选择仓库" clearable>
          <el-option :label="w.name" :value="w.id" v-for="w in wareList" :key="w.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="库存数" prop="stock">
        <el-input v-model="dataForm.stock" placeholder="库存数"></el-input>
      </el-form-item>
      <el-form-item label="sku_name" prop="skuName">
        <el-input v-model="dataForm.skuName" placeholder="sku_name"></el-input>
      </el-form-item>
      <el-form-item label="锁定库存" prop="stockLocked">
        <el-switch v-model="dataForm.stockLocked" class="ml-2" inline-prompt
          style="--el-switch-on-color: #ff4949; --el-switch-off-color: #13ce66" active-text="锁定" inactive-text="不锁定"
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
import { reactive, ref, onMounted } from "vue"
import baseService from "@/service/baseService"
import { ElMessage } from "element-plus"

const emit = defineEmits(["refreshDataList"])
const wareList = ref<any>([]);
const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: '', skuId: '', wareId: '', stock: '', skuName: '', stockLocked: ''
});

const rules = ref({
  skuId: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  wareId: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  stock: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  skuName: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ],
  stockLocked: [
    { required: true, message: '必填项不能为空', trigger: 'blur' }
  ]
});

const getWares = () => {
  baseService.get("ware/wmswareinfo/page").then((res: { data: { list: any; }; }) => {
    wareList.value = res.data.list
  });
};

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
  baseService.get("/ware/wmswaresku/" + id).then((res: { data: any; }) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/ware/wmswaresku", dataForm).then((res: any) => {
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

onMounted(() => {
  getWares();
});
</script>
