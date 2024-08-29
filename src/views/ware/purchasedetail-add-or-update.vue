<template>
  <el-dialog :title="!dataForm.id ? '新增' : '修改'" :close-on-click-modal="false" v-model="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataFormRef" @keyup.enter.native="dataFormSubmit()"
      label-width="120px">
      <el-form-item label="采购商品id" prop="skuId">
        <el-input v-model="dataForm.skuId" placeholder="采购商品id"></el-input>
      </el-form-item>
      <el-form-item label="采购数量" prop="skuNum">
        <el-input v-model="dataForm.skuNum" placeholder="采购数量"></el-input>
      </el-form-item>
      <el-form-item label="仓库" prop="wareId">
        <el-select v-model="dataForm.wareId" placeholder="请选择仓库" clearable>
          <el-option :label="w.name" :value="w.id" v-for="w in wareList" :key="w.id"></el-option>
        </el-select>
      </el-form-item>
      <!-- [0新建，1已分配，2正在采购，3已完成，4采购失败] -->
      <el-form-item label="状态" prop="status">
        <el-select v-model="dataForm.status" placeholder="请选择状态" clearable>
          <el-option label="新建" :value="0"></el-option>
          <el-option label="已分配" :value="1"></el-option>
          <el-option label="正在采购" :value="2"></el-option>
          <el-option label="已完成" :value="3"></el-option>
          <el-option label="采购失败" :value="4"></el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { ElMessage, ElMessageBox } from "element-plus"
import baseService from "@/service/baseService"

const visible = ref(false)

const wareList = ref<any>([])

const dataForm = ref<any>({
  id: '',
  purchaseId: '',
  skuId: '',
  skuNum: '',
  skuPrice: '',
  wareId: '',
  status: ''
})

const emit = defineEmits(["refreshDataList"])

const dataFormRef = ref()

const dataRule = {
  skuId: [
    { required: true, message: "采购商品id不能为空", trigger: "blur" }
  ],
  skuNum: [
    { required: true, message: "采购数量不能为空", trigger: "blur" }
  ],
  wareId: [{ required: true, message: "仓库id不能为空", trigger: "blur" }]
};

const getWares = () => {
  baseService.get("ware/wmswareinfo/page").then((res: { data: { list: any; }; }) => {
    wareList.value = res.data.list
  }).catch((err) => {
    ElMessage.error('获取数据失败')
    console.log("getWares_err", err)
  })
}

const init = (id: number) => {
  visible.value = true

  dataForm.value.id = ''
  dataForm.value.purchaseId = ''
  dataForm.value.skuId = ''
  dataForm.value.skuNum = ''
  dataForm.value.skuPrice = ''
  dataForm.value.wareId = ''
  dataForm.value.status = ''
  if (id) {
    getInfo(id)
  }
}

const dataFormSubmit = () => {
  dataFormRef.value.validate((valid: any) => {
    if (!valid) {
      ElMessage.warning('请正确填必要信息')
      return false
    }
    (!dataForm.value.id ? baseService.post : baseService.put)("ware/wmspurchasedetail", dataForm.value).then(() => {
      ElMessage.success({
        message: '成功',
        duration: 500,
        onClose: () => {
          visible.value = false
          emit("refreshDataList")
        }
      })
    })
  })
}

// 获取信息
const getInfo = (id: number) => {
  baseService.get("ware/wmspurchasedetail/" + id).then((res) => {
    Object.assign(dataForm.value, res.data)
  })
}

onMounted(() => {
  getWares()
})

defineExpose({
  init
})
</script>
