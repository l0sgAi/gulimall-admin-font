<template>
  <div class="mod-ware__wmspurchase">
    <el-form :inline="true" :model="dataForm" @keyup.enter="getDataList()">
      <el-form-item label="状态">
        <el-select style="width:120px;" v-model="dataForm.status" placeholder="请选择状态" clearable>
          <el-option label="新建" :value="0"></el-option>
          <el-option label="已分配" :value="1"></el-option>
          <el-option label="已领取" :value="2"></el-option>
          <el-option label="已完成" :value="3"></el-option>
          <el-option label="有异常" :value="4"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="关键字">
        <el-input style="width:120px;" v-model="dataForm.key" placeholder="参数名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="warning" @click="reset()">重置</el-button>
        <el-button @click="query()">查询</el-button>
        <el-button v-if="state.hasPermission('ware:wmspurchase:save')" type="primary"
          @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="state.hasPermission('ware:wmspurchase:delete')" type="danger"
          @click="deleteHandle()">批量删除</el-button>
      </el-form-item>
    </el-form>
    <el-table ref="table" v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" label="采购单id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="assigneeId" label="采购人id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="assigneeName" label="采购人名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="phone" label="联系方式" header-align="center" align="center"></el-table-column>
      <el-table-column prop="priority" label="优先级" header-align="center" align="center"></el-table-column>
      <el-table-column prop="status" label="状态" header-align="center" align="center">
        <template #default="scope">
          <el-tag v-if="scope.row.status == 0">新建</el-tag>
          <el-tag type="info" v-if="scope.row.status == 1">已分配</el-tag>
          <el-tag type="wanring" v-if="scope.row.status == 2">已领取</el-tag>
          <el-tag type="success" v-if="scope.row.status == 3">已完成</el-tag>
          <el-tag type="danger" v-if="scope.row.status == 4">有异常</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="wareId" label="仓库id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="amount" label="总金额" header-align="center" align="center"></el-table-column>
      <el-table-column prop="createTime" label="创建日期" header-align="center" align="center"></el-table-column>
      <el-table-column prop="updateTime" label="更新日期" header-align="center" align="center"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="scope.row.status == 0 || scope.row.status == 1" @click="opendrawer(scope.row)" type="warning"
            link>分配</el-button>
          <el-button v-if="state.hasPermission('ware:wmspurchase:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('ware:wmspurchase:delete')" type="primary" link
            @click="deleteHandle()">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="getDataList()">确定</add-or-update>

    <el-dialog title="分配采购人员" v-model="caigoudialogVisible" width="30%">
      <el-select v-model="userId" filterable placeholder="请选择">
        <el-option v-for="item in userList" :key="item.id" :label="item.username" :value="item.id"></el-option>
      </el-select>
      <span slot="footer" class="dialog-footer">
        <el-button @click="caigoudialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="assignUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue"
import AddOrUpdate from "./wmspurchase-add-or-update.vue"
import baseService from "@/service/baseService"
import { ElMessage, ElMessageBox, ElTable } from "element-plus"
const userId = ref<any>('')
const dataListSelections = ref([])
const table = ref<InstanceType<typeof ElTable>>()
const userList = ref<any[]>([])
const currentRow = ref<any>()
const caigoudialogVisible = ref(false)
const dataForm = reactive({
  key: "",
  status: ""
})

const purchaseDataForm = reactive({
  key: "",
  status: ""
})

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/ware/wmspurchase/page",
  getDataListIsPage: true,
  exportURL: "/ware/wmspurchase/export",
  deleteURL: "/ware/wmspurchase"
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
}

const getDataList = () => {
  state.dataListLoading = true
  // console.log("dataForm: ", dataForm)
  baseService.get("ware/wmspurchase/page", dataForm).then((res: { data: { list: any; total: number; }; }) => {
    state.dataList = res.data.list
    state.total = res.data.total
    // console.log("dataList.value: ", dataList.value)
    state.dataListLoading = false
  }).catch(() => {
    ElMessage.error("获取数据失败！")
    state.dataListLoading = false
  })
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

const query = () => {
  getDataList()
  ElMessage.success("查询成功")
}

const reset = () => {
  dataForm.key = ''
  dataForm.status = ''
  getDataList()
  ElMessage.success("重置成功")
}

const getUserList = () => {
  baseService.get("sys/user/page").then((res) => {
    userList.value = res.data.list
  })
}

const opendrawer = (row: any) => {
  getUserList()
  currentRow.value = row
  caigoudialogVisible.value = true
}

const assignUser = () => {
  let user = <any>{}
  userList.value.forEach(item => {
    if (item.id == userId.value) {
      user = item
    }
  })

  caigoudialogVisible.value = false
  baseService.put("ware/wmspurchase", {
    id: currentRow.value.id || undefined,
    assigneeId: user.id,
    assigneeName: user.username,
    phone: user.mobile,
    status: 1
  }).then(() => {
    ElMessage.success("分配成功")
    userId.value = ''
    getDataList()
  }).catch((err) => {
    ElMessage.error("分配失败")
    console.log("assignUser__err", err)
  })

}
</script>
