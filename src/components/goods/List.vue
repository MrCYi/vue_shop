<template>
  <div>
    <!-- 面包屑导航栏 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End面包屑导航栏 -->

    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 搜索栏区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            @clear="getGoodsList"
            clearable
            placeholder="请输入内容"
            v-model="queryInfo.query"
          >
            <el-button
              @click="getGoodsList"
              slot="append"
              icon="el-icon-search"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button @click="addGoodsClick" type="primary">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- End搜索栏区域 -->

      <!-- 商品列表区域 -->
      <el-table border stripe :data="goodsList" style="width: 100%">
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="95px">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="70px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="140px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <el-button
              @click="delGoodsClick(scope.row.goods_id)"
              type="primary"
              icon="el-icon-delete"
              size="mini"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- End商品列表区域 -->

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
  </div>
</template>

<script>
export default {
  name: "List",
  created() {
    this.getGoodsList();
  },
  data() {
    return {
      //请求商品列表的信息
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      //商品列表数据
      goodsList: [],
      //总页数
      total: 0,
    };
  },
  methods: {
    //获取商品列表数据
    async getGoodsList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败");
      }
      console.log(res);
      this.goodsList = res.data.goods;
      this.total = res.data.total;
    },
    //监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getGoodsList();
    },
    //监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getGoodsList();
    },
    //删除商品
    async delGoodsClick(id) {
      const result = await this.$confirm(
        "此操作将永久删除该商品, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        return this.$message.info("已取消删除该商品");
      }
      const { data: res } = await this.$http.delete("goods/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除该商品失败");
      }
      this.$message.success("删除成功");
      this.getGoodsList();
    },
    //点击跳转添加商品页面
    addGoodsClick() {
      this.$router.push("/goods/add");
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
