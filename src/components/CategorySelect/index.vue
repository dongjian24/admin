<template>
  <div>
    <!-- inline 行内表单,一行可以放多个表单元素 -->
    <el-form :inline="true" class="demo-form-inline" :model="cForm">
      <el-form-item label="一级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category1Id"
          @change="handle1"
          :disabled="isShow"
        >
          <el-option
            :label="c1.name"
            v-for="c1 in formInline"
            :key="c1.id"
            :value="c1.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="二级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category2Id"
          @change="handle2"
          :disabled="isShow"
        >
          <el-option
            :label="c2.name"
            v-for="c2 in formInline2"
            :key="c2.id"
            :value="c2.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="三级分类">
        <el-select
          placeholder="请选择"
          v-model="cForm.category3Id"
          @change="handle3"
          :disabled="isShow"
        >
          <el-option
            :label="c3.name"
            v-for="c3 in formInline3"
            :key="c3.id"
            :value="c3.id"
          ></el-option>
        </el-select>
      </el-form-item>
    </el-form>
  </div>
</template>
  
  <script>
export default {
  name: "CategorySelect",
  props:['isShow'],
  data() {
    return {
      //一级分类的数据
      formInline: [],
      formInline2: [],
      formInline3: [],
      //收集相应的一级二级三级分类的id
      cForm: {
        category1Id: "",
        category2Id: "",
        category3Id: "",
      },
    };
  },
  mounted() {
    this.getCategory1List();
  },
  methods: {
    async getCategory1List() {
      let result = await this.$API.attr.reqCategory1List();
      //   console.log(result);
      if (result.code == 200) {
        this.formInline = result.data;
      }
    },
    //一级分类的select事件回调
    async handle1() {
      this.formInline2 = [];
      this.formInline3 = [];
      this.cForm.category2Id = "";
      this.cForm.category3Id = "";
    //   console.log(this.category2Id);
      //解构出一级分类的id
      const { category1Id } = this.cForm;
      this.$emit("getCategoryId", {categoryId:category1Id,level:1});
      let result = await this.$API.attr.reqCategory2List(category1Id);
      //   console.log(result);
      if (result.code == 200) {
        this.formInline2 = result.data;
      }
    },
    async handle2() {
      this.formInline3 = [];
      this.cForm.category3Id = "";
      //解构出一级分类的id
      const { category2Id } = this.cForm;
      this.$emit("getCategoryId", {categoryId:category2Id,level:2});
      let result = await this.$API.attr.reqCategory3List(category2Id);
      //   console.log(result);
      if (result.code == 200) {
        this.formInline3 = result.data;
      }
    },
    handle3() {
      const { category3Id } = this.cForm;
      this.$emit("getCategoryId", {categoryId:category3Id,level:3});
    },
  },
};
</script>
  