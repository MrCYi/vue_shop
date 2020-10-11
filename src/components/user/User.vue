<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 搜索和添加 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- End搜索和添加 -->
      <!-- 用户数据表 -->
      <el-table :data="userList" :stripe="true" :border="true">
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态" prop="mg_state">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChange(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button
              @click="editClick(scope.row.id)"
              type="primary"
              size="mini"
              icon="el-icon-edit"
            ></el-button>
            <el-button
              @click="delClick(scope.row.id)"
              type="danger"
              size="mini"
              icon="el-icon-delete"
            ></el-button>
            <el-tooltip
              :enterable="false"
              effect="dark"
              content="分配角色"
              placement="top"
            >
              <el-button
                @click="showSetRoleDialog(scope.row)"
                type="warning"
                size="mini"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- End用户数据表 -->
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- End分页 -->
    </el-card>
    <!-- End内容主体 -->
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addVisible"
      width="50%"
      @close="addVisibleClose"
    >
      <!-- 添加用户表单区域 -->
      <el-form
        :model="addForm"
        :rules="addRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- End添加用户表单区域 -->
      <!-- 底部内容 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
      <!-- End底部内容 -->
    </el-dialog>
    <!-- End添加用户对话框 -->
    <!-- 修改信息对话框 -->
    <el-dialog
      @close="editVisibleClose"
      title="修改用户信息"
      :visible.sync="editVisible"
      width="50%"
    >
      <!-- 表单区域 -->
      <el-form
        ref="editFormRef"
        :model="editMessage"
        :rules="editRule"
        label-width="70px"
      >
        <el-form-item label="用户名">
          <el-input v-model="editMessage.username" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editMessage.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editMessage.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- End修改信息对话框 -->
    <!-- 分配角色对话框 -->
    <el-dialog
      @close="setPowerClose"
      title="分配角色"
      :visible.sync="setPowerDialog"
      width="50%"
    >
      <div>
        <p>当前的用户: {{ this.userInfo.username }}</p>
        <p>当前的角色: {{ this.userInfo.role_name }}</p>
        <p>
          分配新角色:
          <el-select v-model="roleId" placeholder="请选择">
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setPowerDialog = false">取 消</el-button>
        <el-button type="primary" @click="setPowerClick">确 定</el-button>
      </span>
    </el-dialog>
    <!-- End分配角色对话框 -->
  </div>
</template>

<script>
export default {
  name: "User",
  created() {
    this.getUserList();
  },
  data() {
    // 验证邮箱规则
    let checkEmail = (rule, value, cb) => {
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])/;
      if (regEmail.test(value)) {
        return cb();
      }
      cb(new Error("请输入合法的邮箱"));
    };

    //验证手机规则
    let checkMoblie = (rule, value, cb) => {
      const regMoblie = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/;
      if (regMoblie.test(value)) {
        return cb();
      }
      cb(new Error("请输入正确的手机"));
    };

    return {
      queryInfo: {
        query: "",
        pagenum: 1, //当前页数
        pagesize: 2, //显示每页多少条数据
      },
      userList: [],
      total: 0, //用户数据的总条数
      addVisible: false, //控制添加用户对话框的显示隐藏
      editVisible: false, //控制修改信息对话框的显示隐藏
      setPowerDialog: false, //控制分配角色对话框的显示隐藏
      //添加用户表单信息
      addForm: {
        username: "",
        password: "",
        email: "",
        mobile: "",
      },
      // 添加用户表单验证规则
      addRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "长度在 3 到 10 个字符",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          {
            min: 6,
            max: 15,
            message: "长度在 6 到 15 个字符",
            trigger: "blur",
          },
        ],
        email: [
          { required: true, message: "请输入邮箱", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
        mobile: [
          { required: true, message: "请输入手机", trigger: "blur" },
          { validator: checkMoblie, trigger: "blur" },
        ],
      },
      //编辑用户表单信息
      editMessage: {},
      //编辑用户表单信息验证规则
      editRule: {
        email: [
          { required: true, message: "请输入邮箱", trigger: "blur" },
          { validator: checkEmail, trigger: "blur" },
        ],
        mobile: [
          { required: true, message: "请输入手机", trigger: "blur" },
          { validator: checkMoblie, trigger: "blur" },
        ],
      },
      userInfo: "",
      rolesList: "",
      roleId: "",
    };
  },
  methods: {
    //获取用户信息
    async getUserList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo,
      });
      if (res.meta.status != 200) return this.$message.error(this.meta.msg);
      this.userList = res.data.users;
      this.total = res.data.total;
    },
    //监听switch状态改变
    async userStateChange(userInfo) {
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      );
      if (res.meta.status != 200) {
        userInfo.mg_state = !userInfo.mg_state;
        return this.$message.error(res.meta.msg);
      }
      this.$message.success(res.meta.msg);
    },
    //监听当前页面条数改变
    handleSizeChange(val) {
      this.queryInfo.pagesize = val;
      this.getUserList();
    },
    //监听页码值改变的事件
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val;
      this.getUserList();
    },
    //添加用户表对话框关闭，表单重置
    addVisibleClose() {
      this.$refs.addFormRef.resetFields();
    },
    //添加用户表单提交
    addUser() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.post("users", this.addForm);
        if (res.meta.status !== 201) {
          return this.$message.error("添加用户失败");
        }
        this.$message.success("添加用户成功");
        this.addVisible = false;
        this.getUserList();
      });
    },
    //修改用户信息表单打开
    async editClick(value) {
      const { data: res } = await this.$http.get("users/" + value);
      if (res.meta.status !== 200) {
        return this.$message.error("查询用户信息失败");
      }
      this.editMessage = res.data;
      console.log(res.data);
      this.editVisible = true;
    },
    //修改用户表对话框关闭，表单重置
    editVisibleClose() {
      this.$refs.editFormRef.resetFields();
      this.editVisible = false;
    },
    //修改用户表单提交
    editUser() {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "users/" + this.editMessage.id,
          {
            email: this.editMessage.email,
            mobile: this.editMessage.mobile,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("更新用户信息失败");
        }
        this.getUserList();
        this.editVisible = false;
        this.$message.success("更新用户信息成功");
      });
    },
    //删除用户
    async delClick(id) {
      const result = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "删除",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((error) => error);
      if (result !== "confirm") {
        this.$message.info("已取消删除该用户");
      } else {
        const { data: res } = await this.$http.delete("users/" + id);
        if (res.meta.status !== 200) {
          this.$message.error("删除该用户使用");
        } else {
          this.$message.success("删除用户成功");
          this.getUserList();
        }
      }
    },
    //打开分配角色对话框
    async showSetRoleDialog(row) {
      this.userInfo = row;
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolesList = res.data;
      console.log(this.rolesList);
      this.setPowerDialog = true;
    },
    //分配用户角色
    async setPowerClick() {
      if (!this.roleId) {
        return this.$message.error("请选择要设置角色");
      }
      const { data: res } = await this.$http.put(
        `users/${this.userInfo.id}/role`,
        {
          rid: this.roleId,
        }
      );
      if (res.meta.status !== 200) {
        console.log(res);
        return this.$message.error("设置角色失败");
      }
      this.$message.success(res.meta.msg);
      this.getUserList();
      this.setPowerDialog = false;
    },
    setPowerClose() {
      this.rolesList = "";
      this.roleId = "";
    },
  },
};
</script>

<style lang="less" scoped></style>
