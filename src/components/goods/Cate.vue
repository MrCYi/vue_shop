<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End面包屑导航 -->
    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 添加分类按钮 -->
      <el-row>
        <el-col>
          <el-button @click="showAddCateDialog" type="primary"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- End添加分类按钮 -->
      <!-- 分类表格区域 -->
      <el-table
        :data="cateList"
        style="width: 100%;margin-bottom: 20px;"
        row-key="cat_id"
        border
        :tree-props="{ children: 'children' }"
      >
        <el-table-column prop="cat_name" label="分类名称"> </el-table-column>
        <el-table-column label="是否有效">
          <template slot-scope="scope">
            <i
              class="el-icon-success"
              style="color:lightgreen"
              v-if="!scope.row.cat_deleted"
            ></i>
            <i style="color:red" class="el-icon-error" v-else></i>
          </template>
        </el-table-column>
        <el-table-column label="排序">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag v-else-if="scope.row.cat_level === 1" type="success"
              >二级</el-tag
            >
            <el-tag v-else type="warning">三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              @click="showEditDialog(scope.row.cat_id)"
              type="primary"
              icon="el-icon-edit"
              >编辑</el-button
            >
            <el-button
              @click="showDelBox(scope.row.cat_id)"
              type="danger"
              icon="el-icon-delete"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- End分类表格区域 -->
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
    <!-- 添加分类对话框区域 -->
    <el-dialog
      @close="addCateDialogClose"
      title="添加分类"
      :visible.sync="addCateDialog"
      width="50%"
    >
      <el-form
        :rules="addCateFormRules"
        ref="addCateFormRef"
        :model="addCateForm"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            change-on-select
            clearable
            v-model="addCateList"
            :options="parentCateList"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children',
            }"
            @change="handleChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialog = false">取 消</el-button>
        <el-button type="primary" @click="addCateDialogClick">确 定</el-button>
      </span>
    </el-dialog>
    <!-- End添加分类对话框区域 -->
    <!-- 编辑对话框区域 -->
    <el-dialog
      @close="editDialogClose"
      title="提示"
      :visible.sync="editDialog"
      width="50%"
    >
      <el-form
        :rules="addCateFormRules"
        ref="editDialogRef"
        :model="editCate"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCate.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialog = false">取 消</el-button>
        <el-button type="primary" @click="editDialogClick">确 定</el-button>
      </span>
    </el-dialog>
    <!-- End编辑对话框区域 -->
  </div>
</template>

<script>
export default {
  name: "Cate",
  created() {
    this.getCateList();
  },
  data() {
    return {
      //查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      //商品分类的数据列表
      cateList: [],
      //商品分类的总条数
      total: null,
      //控制添加分类对话框的显示与隐藏
      addCateDialog: false,
      //控制编辑对话框的显示与隐藏
      editDialog: false,
      //添加分类的表单提交数据
      addCateForm: {
        cat_name: "",
        cat_pid: 0,
        cat_level: 0,
      },
      //添加分类的表单的验证规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请添加分类名称", trigger: "blur" },
        ],
      },
      //父级分类列表
      parentCateList: [],
      //选中父级分类的id列表
      addCateList: [],
      //编辑分类
      editCate: {
        cat_name: "",
        cat_id: "",
      },
    };
  },
  methods: {
    //获取商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败");
      }
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    //监听 pageSize 改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    //监听 currentPage 改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    //打开添加分类对话框
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateDialog = true;
    },
    //添加分类对话框关闭时，重置表单
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields();
      this.addCateForm.cat_level = 0;
      this.addCateForm.cat_pid = 0;
      this.addCateList = [];
    },
    //获取父级分类列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: {
          type: 2,
        },
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类失败");
      }
      this.parentCateList = res.data;
    },
    //在添加分类中，选中了父级分类时触发
    handleChange() {
      if (this.addCateList) {
        this.addCateForm.cat_pid = this.addCateList[
          this.addCateList.length - 1
        ];
        this.addCateForm.cat_level = this.addCateList.length;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    //添加分类的提交
    addCateDialogClick() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          "categories",
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加分类失败");
        }
        this.$message.success("添加分类成功");
        this.getCateList();
        this.addCateDialog = false;
      });
    },
    //编辑对话框显示，并根据id查询分类
    async showEditDialog(id) {
      const { data: res } = await this.$http.get("categories/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询分类失败");
      }
      this.editCate.cat_name = res.data.cat_name;
      this.editCate.cat_id = id;
      this.editDialog = true;
    },
    //关闭编辑对话框时，重置表单
    editDialogClose() {
      this.$refs.editDialogRef.resetFields();
    },
    //编辑的提交
    editDialogClick() {
      this.$refs.editDialogRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "categories/" + this.editCate.cat_id,
          { cat_name: this.editCate.cat_name }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("编辑失败");
        }
        this.$message.success("编辑成功");
        this.getCateList();
        this.editDialog = false;
      });
    },
    //删除的弹框
    async showDelBox(id) {
      const result = await this.$confirm(
        "此操作将永久删除该分类, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        return this.$message.info("已取消删除该分类");
      }
      const { data: res } = await this.$http.delete("categories/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除该分类失败");
      }
      this.$message.success("删除该分类成功");
      this.getCateList();
    },
  },
};
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
