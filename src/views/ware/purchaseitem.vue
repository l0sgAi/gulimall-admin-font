<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item label="仓库">
        <el-select style="width:120px;" v-model="dataForm.wareId" placeholder="请选择仓库" clearable>
          <el-option :label="w.name" :value="w.id" v-for="w in wareList" :key="w.id"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="状态">
        <el-select style="width:120px;" v-model="dataForm.status" placeholder="请选择状态" clearable>
          <el-option label="新建" :value="0"></el-option>
          <el-option label="已分配" :value="1"></el-option>
          <el-option label="正在采购" :value="2"></el-option>
          <el-option label="已完成" :value="3"></el-option>
          <el-option label="采购失败" :value="4"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="关键字">
        <el-input style="width:120px;" v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="query()">查询</el-button>
        <el-button type="warning" @click="reset()">重置</el-button>
        <el-button style="margin-right: 12px;" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <!-- <el-dropdown @command="handleBatchCommand" :disabled="dataListSelections.length <= 0">
          <el-button type="danger">
            批量操作
            <i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="delete">批量删除</el-dropdown-item>
            <el-dropdown-item command="merge">合并整单</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown> -->
        <el-dropdown @command="handleBatchCommand" type="primary">
          <el-button type="danger">
            批量操作
            <el-icon class="el-icon--right"><arrow-down /></el-icon>
          </el-button>
          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item command="delete">批量删除</el-dropdown-item>
              <el-dropdown-item command="merge">合并整单</el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
      </el-form-item>
    </el-form>
    <el-table ref="table" :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="id"></el-table-column>
      <el-table-column prop="purchaseId" header-align="center" align="center" label="采购单id"></el-table-column>
      <el-table-column prop="skuId" header-align="center" align="center" label="采购商品id"></el-table-column>
      <el-table-column prop="skuNum" header-align="center" align="center" label="采购数量"></el-table-column>
      <el-table-column prop="skuPrice" header-align="center" align="center" label="采购金额"></el-table-column>
      <el-table-column prop="wareId" header-align="center" align="center" label="仓库id"></el-table-column>
      <el-table-column prop="status" header-align="center" align="center" label="状态">
        <template #default="scope">
          <el-tag v-if="scope.row.status == 0">新建</el-tag>
          <el-tag type="info" v-if="scope.row.status == 1">已分配</el-tag>
          <el-tag type="wanring" v-if="scope.row.status == 2">正在采购</el-tag>
          <el-tag type="success" v-if="scope.row.status == 3">已完成</el-tag>
          <el-tag type="danger" v-if="scope.row.status == 4">采购失败</el-tag>
        </template>
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template #default="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle()">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper"></el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="getDataList"></add-or-update>

    <el-dialog title="合并到整单" v-model="mergedialogVisible">
      <!-- id  assignee_id  assignee_name  phone   priority status -->
      <el-select v-model="purchaseId" placeholder="请选择" clearable filterable>
        <el-option v-for="item in purchasetableData" :key="item.id" :label="item.id" :value="item.id">
          <span style="float: left">{{ item.id }}</span>
          <span style="float: right; color: #8492a6; font-size: 13px">{{ item.assigneeName }}：{{ item.phone }}</span>
        </el-option>
      </el-select>
      <span slot="footer" class="dialog-footer">
        <el-button @click="mergedialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="mergeItem()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onActivated, reactive } from 'vue';
import AddOrUpdate from './purchasedetail-add-or-update.vue'
import baseService from "@/service/baseService";
import { ElMessage, ElMessageBox, ElTable } from "element-plus"

const dataForm = reactive<any>({
  key: '',
  status: '',
  wareId: ''
})

const table = ref<InstanceType<typeof ElTable>>()
const wareList = ref<any>([])
const dataList = ref([])
const pageIndex = ref(1)
const pageSize = ref(10)
const totalPage = ref(0)
const dataListLoading = ref(false)
const dataListSelections = ref<any>([])
// const addOrUpdateVisible = ref(false);
const mergedialogVisible = ref(false)
const purchaseId = ref('')
const purchasetableData = ref<any>([])
const addOrUpdateRef = ref()

const doMerge = (itemsIds: number[]) => {
  //整单合并
  baseService.post("ware/wmspurchase/merge", {
    items: itemsIds,
    purchaseId: purchaseId.value
  }).then(() => {
    getDataList()
    ElMessage.success(`合并成功`)
  }).catch((err) => {
    ElMessage.error('合并失败')
    console.log("mergeItem__err", err)
  })
}

const mergeItem = () => {
  let itemsIds = dataListSelections.value.map((item: { id: number; }) => item.id)

  if (!purchaseId.value) {
    ElMessageBox.confirm(`未选中任何【采购单】，将创建新单，确认创建？`, '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    }).then(() => {
      doMerge(itemsIds)
      mergedialogVisible.value = false
    })
  } else {
    doMerge(itemsIds)
    mergedialogVisible.value = false
  }

}

const getUnreceivedPurchase = () => {
  baseService.get("ware/wmspurchase/pageNoAssign").then((res: { data: { list: any; }; }) => {
    purchasetableData.value = res.data.list
  }).catch((err) => {
    ElMessage.error('获取数据失败')
    console.log("getUnreceivedPurchase__err", err)
  })
}

const handleBatchCommand = (cmd: string) => {
  if (cmd === 'delete') {
    deleteHandle()
  }
  else if (cmd === 'merge') {
    dataListSelections.value = table.value!.getSelectionRows()
    if (dataListSelections.value.length === 0) {
      ElMessage.warning("请选择要合并的采购需求")
      return false
    }
    purchaseId.value = ''
    mergedialogVisible.value = true
    getUnreceivedPurchase()
  }
}

const getWares = () => {
  baseService.get("ware/wmswareinfo/page").then((res: { data: { list: any; }; }) => {
    wareList.value = res.data.list
  }).catch((err) => {
    ElMessage.error('获取数据失败')
    console.log("getWares_err", err)
  })
}

const getDataList = () => {
  dataListLoading.value = true
  // console.log("dataForm: ", dataForm)
  baseService.get("ware/wmspurchasedetail/page", dataForm).then((res: { data: { list: any; total: number; }; }) => {
    dataList.value = res.data.list
    totalPage.value = res.data.total
    // console.log("dataList.value: ", dataList.value)
    dataListLoading.value = false
  }).catch(() => {
    ElMessage.error("获取数据失败！")
    dataListLoading.value = false
  })
}

const query = () => {
  getDataList()
  ElMessage.success("查询成功")
}

const reset = () => {
  dataForm.key = ''
  dataForm.status = ''
  dataForm.wareId = ''
  getDataList()
  ElMessage.success("重置成功")
}

const sizeChangeHandle = (val: number) => {
  pageSize.value = val;
  pageIndex.value = 1;
  getDataList();
};

const currentChangeHandle = (val: number) => {
  pageIndex.value = val;
  getDataList();
};

const selectionChangeHandle = (val: any[]) => {
  dataListSelections.value = val;
};

const addOrUpdateHandle = (id?: string) => {
  addOrUpdateRef.value.init(id)
  // This assumes the component has an `init` method

}

const deleteHandle = () => {
  let checked = table.value!.getSelectionRows()
  if (checked.length === 0) {
    ElMessage.warning("请在左侧选择栏选择删除的表单项")
    return
  }
  let ids = checked.map((item: { id: number }) => item.id)
  ElMessageBox.confirm(`此操作将批量删除数据,确认删除？`, '删除确认', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  }).then(() => {
    baseService.delete("ware/wmspurchasedetail", ids).then(() => {
      getDataList()
      ElMessage.success("删除采购需求成功！")
    }).catch(() => {
      getDataList()
      ElMessage.error("删除采购需求失败！")
    })
  }).catch(() => {
    ElMessage.info('已取消删除')
  })
}

onActivated(() => {
  getDataList()
  getWares()
})
</script>
