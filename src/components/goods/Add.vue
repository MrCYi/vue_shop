<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- End面包屑导航区域 -->

    <!-- 内容主体 -->
    <el-card class="box-card">
      <!-- 警告提示区域 -->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- End警告提示区域 -->

      <!-- 步骤条区域 -->
      <el-steps
        align-center
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品照片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- End步骤条区域 -->

      <!-- Tabs区域 -->
      <el-form
        :rules="addFormRules"
        label-position="top"
        ref="addFormRef"
        :model="addForm"
        label-width="80px"
      >
        <el-tabs
          :before-leave="beforeLeave"
          :tab-position="'left'"
          v-model="activeIndex"
          @tab-click="tabClick"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input type="number" v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input type="number" v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input type="number" v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="{
                  expandTrigger: 'hover',
                  value: 'cat_id',
                  label: 'cat_name',
                  children: 'children',
                }"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item1.attr_name"
              v-for="item1 in manyTabList"
              :key="item1.attr_id"
            >
              <el-checkbox-group v-model="item1.attr_vals">
                <el-checkbox
                  border
                  v-for="(item2, index) in item1.attr_vals"
                  :label="item2"
                  :key="index"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              v-for="item in onlyTabList"
              :label="item.attr_name"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品照片" name="3">
            <el-upload
              :on-success="hanldSuccess"
              :action="actionURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button @click="addClick" type="primary" class="bth-add"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
      <!-- End Tabs区域 -->
    </el-card>
    <!-- End内容主体 -->

    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="pictureDialog" width="50%">
      <img :src="previewPath" alt="" class="preview" />
    </el-dialog>
    <!-- End图片预览对话框 -->
  </div>
</template>

<script>
import _ from "lodash";

export default {
  name: "Add",
  created() {
    this.getCateList();
  },
  data() {
    return {
      activeIndex: "0",
      //添加商品的表单信息
      addForm: {
        goods_name: "",
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: "",
        attrs: [],
      },
      //添加商品表单的验证条件
      addFormRules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_weight: [
          { required: true, message: "请输入商品重量", trigger: "blur" },
        ],
        goods_number: [
          { required: true, message: "请输入商品数量", trigger: "blur" },
        ],
        goods_cat: [
          { required: true, message: "请选择商品分类", trigger: "blur" },
        ],
      },
      //商品分类列表
      cateList: [],
      //动态参数列表
      manyTabList: [],
      //静态参数列表
      onlyTabList: [],
      //图片上传地址
      actionURL: "http://timemeetyou.com:8889/api/private/v1/upload",
      //请求头
      headerObj: {
        Authorization: window.sessionStorage.getItem("token"),
      },
      //控制图片预览对话框显示与隐藏
      pictureDialog: false,
      //图片地址
      previewPath: "",
    };
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2];
      }
      return null;
    },
  },
  methods: {
    //获取商品分类
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败");
      }
      this.cateList = res.data;
      console.log(this.cateList);
    },
    //级联选择器发生改变时触发，判断选择分类是否为3级
    handleChange(value) {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = [];
      }
    },
    //在离开基本信息时，验证基本信息是否全填完
    beforeLeave(newActive, oldActive) {
      if (this.activeIndex === "0" && this.addForm.goods_cat.length != 3) {
        this.$message.error("请填写必填项");
        return false;
      }
      return true;
    },
    //
    async tabClick() {
      if (this.activeIndex === "1") {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: "many",
            },
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取参数列表失败");
        }
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
        });
        this.manyTabList = res.data;
      } else if (this.activeIndex === "2") {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: "only",
            },
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取参数列表失败");
        }
        this.onlyTabList = res.data;
        console.log(this.onlyTabList);
      }
    },
    //图片放大效果
    handlePreview(file) {
      this.previewPath = file.response.data.url;
      this.pictureDialog = true;
    },
    //点击x号移除图片时
    handleRemove(file) {
      const filePath = file.response.data.tmp_path;
      const index = this.addForm.pics.findIndex((x) => filePath === x.pic);
      this.addForm.pics.splice(index, 1);
      console.log(this.addForm);
    },
    //图片上传成功
    hanldSuccess(file) {
      let pict = { pic: file.data.tmp_path };
      this.addForm.pics.push(pict);
    },
    //表单预校验
    addClick() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error("请填写所有的必填项");
        }
        const form = _.cloneDeep(this.addForm);
        form.goods_cat = form.goods_cat.join(",");
        this.manyTabList.forEach((item) => {
          let many = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(" "),
          };
          this.addForm.attrs.push(many);
        });
        this.onlyTabList.forEach((item) => {
          let only = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals,
          };
          this.addForm.attrs.push(only);
        });
        form.attrs = this.addForm.attrs;
        console.log(form);
        form.goods_number = form.goods_number - 0;
        form.goods_price = form.goods_price - 0;
        form.goods_weight = form.goods_weight - 0;
        const { data: res } = await this.$http.post("goods", form);
        if (res.meta.status !== 201) {
          console.log(res);
          return this.$message.error("添加商品失败");
        }
        this.$message.success("添加商品成功");
        this.$router.push("/goods");
      });
    },
  },
};
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 5px 0 0;
}
.preview {
  width: 100%;
}
.bth-add {
  margin-top: 15px;
}
</style>
