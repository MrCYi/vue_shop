<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End面包屑导航 -->

    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 警告条区域 -->
      <el-alert
        show-icon
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
      >
      </el-alert>
      <!-- End警告条区域 -->

      <!-- 级联选择器区域 -->
      <el-row class="params-cascader">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            clearable
            v-model="selectCate"
            :options="cateList"
            :props="{
              expandTrigger: 'hover',
              value: 'cat_id',
              label: 'cat_name',
              children: 'children',
            }"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- End级联选择器区域 -->

      <!-- 标签页区域 -->
      <el-tabs v-model="activeTabName" @tab-click="handleTabClick">
        <!-- 动态参数区域 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数区域 -->
          <el-row>
            <el-col>
              <el-button
                @click="dialogVisible = true"
                :disabled="isDisabled"
                type="primary"
                size="mini"
                >添加参数</el-button
              >
            </el-col>
          </el-row>
          <!-- End添加参数区域 -->

          <!-- 动态参数表格区域 -->
          <el-table :data="manyList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="handleClose(index, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  @click="showEditDialog(scope.row.attr_id)"
                  size="mini"
                  icon="el-icon-edit"
                  type="primary"
                  >编辑</el-button
                >
                <el-button
                  @click="delParams(scope.row.attr_id)"
                  size="mini"
                  icon="el-icon-delete"
                  type="danger"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
          <!-- End动态表格区域 -->
        </el-tab-pane>
        <!-- End动态参数区域 -->

        <!-- 静态属性区域 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性区域 -->
          <el-row>
            <el-col>
              <el-button
                @click="dialogVisible = true"
                :disabled="isDisabled"
                type="primary"
                size="mini"
                >添加属性</el-button
              >
            </el-col>
          </el-row>
          <!-- End添加属性区域 -->

          <!-- 静态属性表格区域 -->
          <el-table :data="onlyList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, index) in scope.row.attr_vals"
                  :key="index"
                  closable
                  @close="handleClose(index, scope.row)"
                >
                  {{ item }}
                </el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  @click="showEditDialog(scope.row.attr_id)"
                  size="mini"
                  icon="el-icon-edit"
                  type="primary"
                  >编辑</el-button
                >
                <el-button
                  @click="delParams(scope.row.attr_id)"
                  size="mini"
                  icon="el-icon-delete"
                  type="danger"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
          <!-- End静态属性区域 -->
        </el-tab-pane>
        <!-- End静态属性区域 -->
      </el-tabs>
      <!-- End标签页区域 -->
    </el-card>
    <!-- End内容主体 -->

    <!-- 对话框区域 -->
    <el-dialog
      @close="dialogClose"
      :title="'添加' + dialogTitle"
      :visible.sync="dialogVisible"
      width="50%"
    >
      <!-- 表单区域 -->
      <el-form
        :rules="paramsFormRules"
        ref="paramsFormRef"
        :model="paramsForm"
        label-width="80px"
      >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="paramsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- End表单区域 -->

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParamsClick">确 定</el-button>
      </span>
      <!-- End底部区域 -->
    </el-dialog>
    <!-- End对话框区域 -->

    <!-- 编辑的对话框区域 -->
    <el-dialog
      @close="editDialogClose"
      :title="'添加' + dialogTitle"
      :visible.sync="editDialog"
      width="50%"
    >
      <!-- 编辑的表单区域 -->
      <el-form
        :rules="paramsFormRules"
        ref="editFormRef"
        :model="editParamsForm"
        label-width="80px"
      >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- End编辑的表单区域 -->

      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialog = false">取 消</el-button>
        <el-button type="primary" @click="editDialogClick">确 定</el-button>
      </span>
      <!-- End底部区域 -->
    </el-dialog>
    <!-- End编辑的对话框区域 -->
  </div>
</template>

<script>
export default {
  name: "Params",
  created() {
    this.getCateList();
  },
  data() {
    return {
      //获取商品分类列表
      cateList: [],
      //双向绑定选择商品分类
      selectCate: [],
      //记录当前选中标签页
      activeTabName: "many",
      //动态参数列表
      manyList: [],
      //静态属性列表
      onlyList: [],
      //控制对话框的显示与隐藏
      dialogVisible: false,
      //控制编辑的对话框的显示与隐藏
      editDialog: false,
      //添加动态参数或静态属性的列表
      paramsForm: {
        attr_name: "",
      },
      //添加动态参数或静态属性的列表校验规则
      paramsFormRules: {
        attr_name: [
          { required: true, message: "请输入参数名称", trigger: "blur" },
        ],
      },
      //编辑的参数列表
      editParamsForm: {},
    };
  },
  computed: {
    isDisabled() {
      if (this.selectCate.length !== 3) {
        return true;
      } else {
        return false;
      }
    },
    cateId() {
      if (this.selectCate.length === 3) {
        return this.selectCate[2];
      } else {
        return null;
      }
    },
    sel() {
      if (this.activeTabName === "many") {
        return "many";
      } else {
        return "only";
      }
    },
    dialogTitle() {
      if (this.activeTabName === "many") {
        return "动态参数";
      } else {
        return "静态属性";
      }
    },
  },
  methods: {
    //获取商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败");
      }
      this.cateList = res.data;
    },
    //监听级联选择器的改变
    handleChange() {
      if (this.selectCate.length !== 3) {
        this.selectCate = [];
        this.manyList = [];
        this.onlyList = [];
        return;
      }
      this.getParams();
    },
    //监听标签页的改变
    handleTabClick() {
      this.getParams();
    },
    //根据分类id获取动态参数和静态属性
    async getParams() {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: {
            sel: this.sel,
          },
        }
      );
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
        item.inputVisible = false;
        item.inputValue = "";
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败");
      }
      if (this.activeTabName === "many") {
        this.manyList = res.data;
      } else {
        this.onlyList = res.data;
      }
    },
    //关闭对话框时，表单内容重置
    dialogClose() {
      this.$refs.paramsFormRef.resetFields();
    },
    //添加参数/属性的提交
    addParamsClick() {
      this.$refs.paramsFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.paramsForm.attr_name,
            attr_sel: this.activeTabName,
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加失败");
        }
        this.$message.success("添加成功");
        this.getParams();
        this.dialogVisible = false;
      });
    },
    //打开编辑的对话框，并根据id查询参数
    async showEditDialog(attrId) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrId}`,
        {
          attr_sel: this.activeTabName,
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败");
      }
      this.editParamsForm = res.data;
      this.editDialog = true;
    },
    //关闭编辑对话框，并重置表单
    editDialogClose() {
      this.$refs.editFormRef.resetFields();
    },
    //修改参数/属性的提交
    editDialogClick() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`,
          {
            attr_name: this.editParamsForm.attr_name,
            attr_sel: this.activeTabName,
            attr_vals: this.editParamsForm.attr_vals,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("更新失败");
        }
        this.$message.success("更新成功");
        this.getParams();
        this.editDialog = false;
      });
    },
    //删除属性
    async delParams(attrId) {
      const result = await this.$confirm(
        "此操作将永久删除该参数, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        return this.$message.info("已取消删除该参数");
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除该参数失败");
      }
      this.$message.success("删除该参数成功");
      this.getParams();
    },
    //文本框失去焦点或按下Enter时触发
    handleInputConfirm(row) {
      if (!row.inputValue.trim().length) {
        row.inputValue = "";
        row.inputVisible = false;
        return;
      }
      row.attr_vals.push(row.inputValue.trim());
      row.inputValue = "";
      row.inputVisible = false;
      this.setAttr(row);
    },
    //提交编辑的参数
    async setAttr(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: this.activeTabName,
          attr_vals: row.attr_vals.join(" "),
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("修改失败");
      }
      this.$message.success("修改成功");
    },
    //点击按钮，显示文本框
    showInput(row) {
      row.inputVisible = true;
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    //删除参数
    handleClose(index, row) {
      row.attr_vals.splice(index, 1);
      this.setAttr(row);
    },
  },
};
</script>

<style lang="less" scoped>
.params-cascader {
  margin: 15px;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
