<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="等级名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="等级名称"></el-input>
      </el-form-item>
      <el-form-item label="所需成长值" prop="growthPoint">
        <el-input-number v-model="dataForm.growthPoint" :min="0"></el-input-number>
      </el-form-item>
      <el-form-item label="默认等级" prop="defaultStatus">
        <el-switch v-model="dataForm.defaultStatus" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="是" inactive-text="否"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="免运费标准" prop="freeFreightPoint">
        <el-input-number :min="0" v-model="dataForm.freeFreightPoint"></el-input-number>
      </el-form-item>
      <el-form-item label="评价成长值" prop="commentGrowthPoint">
        <el-input-number :min="0" v-model="dataForm.commentGrowthPoint"></el-input-number>
      </el-form-item>
      <el-form-item label="免邮特权" prop="priviledgeFreeFreight">
        <el-switch v-model="dataForm.priviledgeFreeFreight" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="有" inactive-text="无"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="会员价格特权" prop="priviledgeMemberPrice">
        <el-switch v-model="dataForm.priviledgeMemberPrice" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="有" inactive-text="无"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="生日特权" prop="priviledgeBirthday">
        <el-switch v-model="dataForm.priviledgeBirthday" class="ml-2" inline-prompt
          style="--el-switch-on-color: #13ce66; --el-switch-off-color: #ff4949" active-text="有" inactive-text="无"
          :active-value="1" :inactive-value="0" />
      </el-form-item>
      <el-form-item label="备注" prop="note">
        <el-input v-model="dataForm.note" placeholder="备注"></el-input>
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
  id: '', name: '', growthPoint: '', defaultStatus: '', freeFreightPoint: '', commentGrowthPoint: '', priviledgeFreeFreight: '', priviledgeMemberPrice: '', priviledgeBirthday: '', note: ''
});

const rules = ref({
  name: [
    { required: true, message: '等级名称不能为空', trigger: 'blur' }
  ],
  growthPoint: [
    { required: true, message: '等级需要的成长值不能为空', trigger: 'blur' }
  ],
  defaultStatus: [
    { required: true, message: '是否为默认等级[0->不是；1->是]不能为空', trigger: 'blur' }
  ],
  freeFreightPoint: [
    { required: true, message: '免运费标准不能为空', trigger: 'blur' }
  ],
  commentGrowthPoint: [
    { required: true, message: '每次评价获取的成长值不能为空', trigger: 'blur' }
  ],
  priviledgeFreeFreight: [
    { required: true, message: '是否有免邮特权不能为空', trigger: 'blur' }
  ],
  priviledgeMemberPrice: [
    { required: true, message: '是否有会员价格特权不能为空', trigger: 'blur' }
  ],
  priviledgeBirthday: [
    { required: true, message: '是否有生日特权不能为空', trigger: 'blur' }
  ],
  note: [
    { required: true, message: '备注不能为空', trigger: 'blur' }
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
  baseService.get("/member/memberlevel/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/member/memberlevel", dataForm).then((res) => {
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
