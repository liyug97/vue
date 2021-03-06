<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!--提示区域-->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      ></el-alert>
      <!--步骤条区域-->
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          :tab-position="tabPosition"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="catelist"
                :props="cateProps"
                props.expandTrigger="hover"
                @change="handleChange"
                clearable
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!--渲染表单的item项-->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <!--复选框组-->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="baseURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">商品内容</el-tab-pane>
        </el-tabs>
      </el-form>
      <!--图片预览-->
      <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
        <img :src="previewPath" alt="" class="img" />
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      activeIndex: "0",
      tabPosition: "left",
      addForm: {
        goods_name: "",
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        //商品所属的分类数组
        goods_cat: [],
        pics: []
      },
      addFormRules: {
        goods_name: [
          {
            required: true,
            message: "请输入商品名称",
            trigger: "blur"
          }
        ],
        goods_price: [
          {
            required: true,
            message: "请输入商品价格",
            trigger: "blur"
          }
        ],
        goods_weight: [
          {
            required: true,
            message: "请输入商品重量",
            trigger: "blur"
          }
        ],
        goods_number: [
          {
            required: true,
            message: "请输入商品数量",
            trigger: "blur"
          }
        ]
      },
      //获取商品分类列表
      catelist: [],
      cateProps: {
        label: "cat_name",
        value: "cat_id",
        children: "children"
      },
      manyTableData: [],
      onlyTableData: [],
      //上传的图片url地址
      baseURL: "http://127.0.0.1:8888/api/private/v1/upload",
      //图片上传组件的headers请求头对象
      headerObj: {
        Authorization: window.sessionStorage.getItem("token")
      },
      //图片路径
      previewPath: "",
      previewVisible: false
    };
  },
  created() {
    this.getCateList();
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2];
      }
      return null;
    }
  },
  methods: {
    //获取所有商品分类的数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类数据失败");
      }
      this.catelist = res.data;
    },
    //级联选择器选中项变化，会触发这个函数
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = [];
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === "0" && this.addForm.goods_cat.length !== 3) {
        this.$message.error("请先选择商品分类");
        return false;
      }
    },
    async tabClicked() {
      //证明访问的是动态参数面板
      if (this.activeIndex === "1") {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: "many" }
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取动态参数失败");
        }
        console.log(res.data);
        res.data.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
        });
        this.manyTableData = res.data;
      } else if (this.activeIndex === "2") {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: "only" }
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("获取静态参数失败");
        }
        console.log(res.data);
        this.onlyTableData = res.data;
      }
    },
    //处理图片预览效果
    handlePreview(file) {
      console.log(file);
      this.previewPath = file.response.data.url;
      this.previewVisible = true;
    },
    //处理移除图片的操作
    handleRemove(file) {
      //console.log(file)
      //1.获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path;
      //从pics数组中，找到这个图片对应的索引值
      const i = this.addForm.pics.findIndex(x => {
        x.pic === filePath;
      });
      //3.调用数组的splice方法把图片信息对象，从pics数组中删除
      this.addForm.pics.splice(i, 1);
      console.log(this.addForm);
    },
    //监听图片上传成功的事件
    handleSuccess(response) {
      console.log(response);
      //1.拼接得到一个图片信息对象
      const picInfo = {
        pic: response.data.tmp_path
      };
      //2.将图片信息对象，push到pics数组中
      this.addForm.pics.push(picInfo);
      console.log(this.addForm);
    }
  }
};
</script>
=<style scoped>
.img {
  width: 100%;
}
</style>
