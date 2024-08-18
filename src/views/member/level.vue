<template>
  <div class="mod-member__memberlevel">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-input v-model="queryKey" placeholder="等级名查询"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="query()">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('member:memberlevel:save')" type="primary"
          @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('member:memberlevel:delete')" type="danger"
          @click="state.deleteHandle()">删除</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="id" header-align="center" align="center" label="id"></el-table-column>
      <el-table-column prop="name" header-align="center" align="center" label="等级名称"></el-table-column>
      <el-table-column prop="growthPoint" header-align="center" align="center" label="所需成长值"></el-table-column>
      <el-table-column prop="defaultStatus" header-align="center" align="center" label="默认等级">
        <template #default="scope">
          <el-tag type="primary" v-if="scope.row.defaultStatus == 1">是</el-tag>
          <el-tag type="danger" v-else>否</el-tag>
        </template>
      </el-table-column>
      <el-table-column prop="freeFreightPoint" header-align="center" align="center" label="免运费标准"></el-table-column>
      <el-table-column prop="commentGrowthPoint" header-align="center" align="center"
        label="每次评价获取的成长值"></el-table-column>
      <el-table-column label="特权">
        <el-table-column prop="priviledgeFreeFreight" header-align="center" align="center" label="免邮特权">
          <template #default="scope">
            <el-tag type="primary" v-if="scope.row.priviledgeFreeFreight == 1">有</el-tag>
            <el-tag type="danger" v-else>无</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="priviledgeMemberPrice" header-align="center" align="center" label="会员价格特权">
          <template #default="scope">
            <el-tag type="primary" v-if="scope.row.priviledgeMemberPrice == 1">有</el-tag>
            <el-tag type="danger" v-else>无</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="priviledgeBirthday" header-align="center" align="center" label="生日特权">
          <template #default="scope">
            <el-tag type="primary" v-if="scope.row.priviledgeBirthday == 1">有</el-tag>
            <el-tag type="danger" v-else>无</el-tag>
          </template>
        </el-table-column>
      </el-table-column>
      <el-table-column prop="note" header-align="center" align="center" label="备注"></el-table-column>
      <el-table-column label="操作" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('member:memberlevel:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="state.hasPermission('member:memberlevel:delete')" type="primary" link
            @click="state.deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update ref="addOrUpdateRef" @refreshDataList="state.getDataList">确定</add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView"
import { reactive, ref, toRefs } from "vue";
import AddOrUpdate from "./memberlevel-add-or-update.vue"
import baseService from "@/service/baseService"
import { ElMessage } from "element-plus"

const view = reactive({
  deleteIsBatch: true,
  getDataListURL: "/member/memberlevel/page",
  getDataListIsPage: true,
  exportURL: "/member/memberlevel/export",
  deleteURL: "/member/memberlevel"
})

const queryKey = ref('')

const state = reactive({ ...useView(view), ...toRefs(view) });

const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addOrUpdateRef.value.init(id);
}

const query = () => {
  baseService.get(`/member/memberlevel/pageQuery?key=${queryKey.value}`).then((res) => {
    ElMessage.success("查询成功!")
    state.dataList = res.data.list
    state.total = res.data.total
  }).catch(err => {
    ElMessage.error("查询失败!")
    console.log("query_err", err)
  })
}
</script>
