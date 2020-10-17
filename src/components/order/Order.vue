<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End面包屑导航栏 -->

    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 搜索栏区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- End搜索栏区域 -->

      <!-- 订单表格区域 -->
      <el-table border stripe :data="orderList" style="width: 100%">
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="order_number" label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price" label="订单价格"> </el-table-column>
        <el-table-column label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.order_pay === '1'"
              >已付款</el-tag
            >
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"> </el-table-column>
        <el-table-column label="下单时间">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              @click="showAddressDialog"
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <el-button
              @click="showProgressDialog(scope.row.order_id)"
              type="success"
              icon="el-icon-location"
              size="mini"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- End订单表格区域 -->

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- End分页区域 -->
    </el-card>
    <!-- End内容主体 -->

    <!-- 修改地址对话框 -->
    <el-dialog
      @close="addressDialogClose"
      title="修改地址"
      :visible.sync="addressDialog"
      width="50%"
    >
      <el-form
        :rules="addressFormRules"
        ref="addressFormRef"
        :model="addressForm"
        label-width="100px"
      >
        <el-form-item label="省市区/县：" prop="address1">
          <el-cascader
            v-model="addressForm.address1"
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址：" prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialog = false">取 消</el-button>
        <el-button type="primary" @click="addressDialog = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- End修改地址对话框 -->

    <!-- 物流信息对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="progressDialog"
      width="50%"
      :before-close="handleClose"
    >
      <el-timeline :reverse="reverse">
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
    <!-- End物流信息对话框 -->
  </div>
</template>

<script>
import cityData from "./citydata.js";

export default {
  name: "Order",
  created() {
    this.getOrderList();
  },
  data() {
    return {
      //请求数据
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      //订单数据
      orderList: [],
      //总条数
      total: 0,
      //控制修改地址对话框显示隐藏
      addressDialog: false,
      //修改地址的表单
      addressForm: {
        address1: "",
        address2: "",
      },
      //修改表单的验证规则
      addressFormRules: {
        address1: [
          { required: true, message: "请输入省市区/县", trigger: "blur" },
        ],
        address2: [
          { required: true, message: "请输入详细地址", trigger: "blur" },
        ],
      },
      cityData,
      //控制物流信息对话框显示隐藏
      progressDialog: false,
      //物流信息
      progressInfo: [],
    };
  },
  methods: {
    //获取订单数据
    async getOrderList() {
      const { data: res } = await this.$http.get("orders", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取订单数据失败");
      }
      this.orderList = res.data.goods;
      this.total = res.data.total;
      console.log(this.total);
    },
    //每页总条数发生改变时触发
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getOrderList();
    },
    //当前页码发生改变时触发
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getOrderList();
    },
    //修改对话框
    showAddressDialog() {
      this.addressDialog = true;
    },
    //关闭修改地址对话框时，表单重置
    addressDialogClose() {
      this.$refs.addressFormRef.resetFields();
    },
    //
    async showProgressDialog(id) {
      const { data: res } = await this.$http.get("/kuaidi/1106975712662");
      if (res.meta.status !== 200) {
        return this.$message.error("获取物流信息失败");
      }
      this.progressInfo = res.data;
      this.progressDialog = true;
    },
  },
  filters: {
    dateFormat: function(data) {
      const dt = new Date(data);
      const y = dt.getFullYear();
      const m = (dt.getMonth() + 1 + "").padStart(2, "0");
      const d = (dt.getDate() + "").padStart(2, "0");
      const hh = (dt.getHours() + "").padStart(2, "0");
      const mm = (dt.getMinutes() + "").padStart(2, "0");
      const ss = (dt.getSeconds() + "").padStart(2, "0");
      return `${y}-${m}-${d} ${hh}:${mm}:${ss}`;
    },
  },
};
</script>

<style lang="less" scoped></style>
