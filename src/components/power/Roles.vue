<template>
  <div>
    <!-- 导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End导航区域 -->
    <!-- 内容主体部分 -->
    <el-card class="box-card">
      <!-- 添加角色区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialog = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- End添加角色区域 -->
      <!-- 表格区域 -->
      <el-table :data="rolesList" stripe border>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="[
                'border-bottom',
                index1 === 0 ? 'border-top' : '',
                'vcenter',
              ]"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <el-col :span="5">
                <el-tag>{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row
                  :class="[index2 === 0 ? '' : 'border-top', 'vcenter']"
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag type="success">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      closable
                      @close="tagClose(scope.row, item3.id)"
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              @click="editRoleDialogShow(scope.row.id)"
              type="primary"
              size="mini"
              icon="el-icon-edit"
              >编辑</el-button
            >
            <el-button
              @click="delClick(scope.row.id)"
              type="danger"
              size="mini"
              icon="el-icon-delete"
              >删除</el-button
            >
            <el-button
              @click="setPowerDialogClick(scope.row)"
              type="warning"
              size="mini"
              icon="el-icon-setting"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- End表格区域 -->
    </el-card>
    <!-- End内容主体部分 -->
    <!-- 添加角色对话框区域 -->
    <el-dialog
      @close="resetAddRoleForm"
      title="添加角色"
      :visible.sync="addRoleDialog"
      width="50%"
    >
      <!-- 添加角色表单区域 -->
      <el-form
        :rules="addRoleFormRules"
        ref="addRoleFormRef"
        :model="addRoleForm"
        label-width="80px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- End添加角色表单区域 -->
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialog = false">取 消</el-button>
        <el-button type="primary" @click="addRoleDialogClick">确 定</el-button>
      </span>
      <!-- End底部区域 -->
    </el-dialog>
    <!-- End添加角色对话框区域 -->
    <!-- 编辑角色信息对话框部分 -->
    <el-dialog
      @close="resetEditRoleForm"
      title="编辑角色"
      :visible.sync="editRoleDialog"
      width="50%"
    >
      <!-- 编辑角色表单区域 -->
      <el-form
        :rules="editRoleFormRules"
        ref="editRoleFormRef"
        :model="editRoleForm"
        label-width="80px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- End编辑角色表单区域 -->
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialog = false">取 消</el-button>
        <el-button type="primary" @click="editRoleDialogClick">确 定</el-button>
      </span>
      <!-- End底部区域 -->
    </el-dialog>
    <!-- End编辑角色信息对话框部分 -->
    <!-- 分配角色权限对话框部分 -->
    <el-dialog
      @close="setPowerClose"
      title="分配角色权限"
      :visible.sync="setPowerDialog"
      width="50%"
    >
      <el-tree
        node-key="id"
        :default-checked-keys="checkList"
        default-expand-all
        show-checkbox
        :data="powerList"
        :props="powerListProps"
        ref="setPowerRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setPowerDialog = false">取 消</el-button>
        <el-button type="primary" @click="setPowerClick">确 定</el-button>
      </span>
    </el-dialog>
    <!-- End分配角色权限对话框部分 -->
  </div>
</template>

<script>
export default {
  name: "Roles",
  created() {
    this.getRolesList();
  },
  data() {
    return {
      rolesList: [],
      addRoleDialog: false, //控制添加角色对话框的显示与隐藏
      editRoleDialog: false, //控制编辑角色对话框的显示与隐藏
      setPowerDialog: false,
      addRoleForm: {
        roleName: "",
        roleDesc: "",
      },
      addRoleFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
        ],
      },
      editRoleForm: {},
      editRoleFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
        ],
      },
      powerList: [],
      powerListProps: {
        children: "children",
        label: "authName",
      },
      checkList: [],
      setPowerUser: null,
    };
  },
  methods: {
    //获取角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolesList = res.data;
    },
    //关闭添加角色对话框，并重置
    resetAddRoleForm() {
      this.$refs.addRoleFormRef.resetFields();
      this.addRoleDialog = false;
    },
    //添加角色
    addRoleDialogClick() {
      this.$refs.addRoleFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post("roles", this.addRoleForm);
        if (res.meta.status !== 201) {
          return this.$message.error("添加角色失败");
        }
        this.getRolesList();
        this.addRoleDialog = false;
      });
    },
    //打开编辑角色对话框，并根据id查询角色信息
    async editRoleDialogShow(id) {
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询角色信息失败");
      }
      this.editRoleForm = res.data;
      this.editRoleDialog = true;
    },
    //关闭编辑角色对话框，并重置
    resetEditRoleForm() {
      this.$refs.editRoleFormRef.resetFields();
      this.editRoleDialog = false;
    },
    //提交编辑信息
    editRoleDialogClick() {
      this.$refs.editRoleFormRef.validate(async (valid) => {
        if (!valid) return;
        console.log(this.editRoleForm.roleId);
        const { data: res } = await this.$http.put(
          "roles/" + this.editRoleForm.roleId,
          this.editRoleForm
        );
        if (res.meta.status !== 200) {
          console.log(res);
          return this.$message.error("修改角色信息失败");
        }
        this.$message.success("修改角色信息成功");
        this.getRolesList();
        this.editRoleDialog = false;
      });
    },
    //删除角色
    async delClick(id) {
      const result = await this.$confirm(
        "此操作将永久删除该角色, 是否继续?",
        "删除",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        return this.$message.info("已取消删除操作");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$messgae.error("删除角色失败");
      }
      this.$message.success("删除角色成功");
      this.getRolesList();
    },
    // 删除角色的指定权限
    async tagClose(role, rightId) {
      const result = await this.$confirm(
        "此操作将永久删除权限, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        return this.$message.info("已取消删除该权限");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除该权限失败");
      }
      role.children = res.data;
    },
    //打开分配权限对话框，并获取权限列表
    async setPowerDialogClick(role) {
      const { data: res } = await this.$http.get("rights/tree");
      if (res.meta.status !== 200) {
        return this.$message.error("获取权限列表失败");
      }
      this.powerList = res.data;
      this.setPowerUser = role.id;
      this.getRolePower(role, this.checkList);
      this.setPowerDialog = true;
    },
    //判断当前节点是否为3级节点
    getRolePower(node, arr) {
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach((item) => this.getRolePower(item, arr));
    },
    //清空存储3级节点id的数组
    setPowerClose() {
      this.checkList = [];
    },
    //角色权限提交
    async setPowerClick() {
      let arr = [
        ...this.$refs.setPowerRef.getHalfCheckedKeys(),
        ...this.$refs.setPowerRef.getCheckedKeys(),
      ];
      let arrStr = arr.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.setPowerUser}/rights`,
        {
          rids: arrStr,
        }
      );
      if (res.meta.status !== 200) {
        console.log(res);
        return this.$message.error("分配权限失败");
      }
      this.$message.success("分配权限成功");
      this.getRolesList();
      this.setPowerDialog = false;
    },
  },
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.border-bottom {
  border-bottom: 1px solid #eee;
}
.border-top {
  border-top: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
