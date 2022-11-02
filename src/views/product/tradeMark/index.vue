<template>
  <div>
    <el-button
      type="primary"
      @click="showDialog"
      icon="el-icon-plus"
      style="margin: 10px 0px"
      >添加</el-button
    >
    <el-table border :data="list">
      <el-table-column
        label="序号"
        width="80px"
        align="center"
        type="index"
      ></el-table-column>
      <el-table-column prop="tmName" label="品牌名称"></el-table-column>
      <el-table-column prop="logoUrl" label="品牌logo">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" style="width: 100px; height: 100px" alt="" />
        </template>
      </el-table-column>
      <el-table-column prop="prop" label="操作">
        <template slot-scope="{ row }">
          <el-button
            type="warning"
            @click="updateTradeMark(row)"
            icon="el-icon-edit"
            size="mini"
            >修改</el-button
          >
          <el-button
            type="danger"
            @click="deleteTradeMark(row)"
            icon="el-icon-delete"
            size="mini"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      style="margin-top: 20px; text-align: center"
      :total="total"
      :current-page="page"
      :page-size="limit"
      :page-sizes="[3, 5, 10]"
      @current-change="getPageList"
      @size-change="handleSizeChange"
      layout="prev,pager,next,jumper,->,sizes,total"
    ></el-pagination>

    <el-dialog
      :title="this.tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <!-- form表单 :model属性 把表单的数据收集到那个对象身上 表单验证也需要这个属性-->
      <el-form style="width: 80%" :rules="rules" ref="ruleForm" :model="tmForm">
        <el-form-item label="品牌名称" prop="tmName" label-width="100px">
          <el-input autocomplete="off" v-model="tmForm.tmName"></el-input>
        </el-form-item>
        <el-form-item label="品牌logo" prop="logoUrl" label-width="100px">
          <!-- 收集数据 不能使用v-model 不是表单元素 
          action 上传图片地址
          :on-success 可以检测到图片上传成功
          :before-upload 可以在图片上传之前执行一次
          -->
          <el-upload
            class="avatar-uploader"
            action="/api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="tmForm.logoUrl" :src="tmForm.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div slot="tip" class="el-upload__tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>
  
  <script>
export default {
  name: "tradeMark",
  data() {
    return {
      page: 1,
      limit: 3,
      total: 0,
      list: [],
      dialogFormVisible: false,
      tmForm: {
        tmName: "",
        logoUrl: "",
      },
      //表单验证规则
      rules: {
        //品牌名称的验证规则
        //required 必须要校验字段 message 提示信息 trigger 用户行为设置
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "blur" },
          {
            min: 2,
            max: 10,
            message: "长度在 2 到 10 个字符",
            trigger: "change",
          },
        ],
        logoUrl: [{ required: true, message: "请选择品牌图片" }],
      },
    };
  },
  mounted() {
    //获取列表的方法
    this.getPageList();
  },
  methods: {
    //获取品牌列表的数据
    async getPageList(pager = 1) {
      this.page = pager;
      const { page, limit } = this;
      let result = await this.$API.trademark.reqTradeMarkList(page, limit);
      // console.log(result)
      if (result.code == 200) {
        this.total = result.data.total;
        this.list = result.data.records;
      }
    },
    handleSizeChange(limit) {
      this.limit = limit;
      this.getPageList();
    },
    showDialog() {
      (this.dialogFormVisible = true),
        (this.tmForm = { tmName: "", logoUrl: "" });
    },
    handleAvatarSuccess(res, file) {
      //res 上传成功之后返回的数据
      //file 上传成功之后返回的数据
      console.log(file.raw);
      this.tmForm.logoUrl = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },
    addOrUpdateTradeMark() {
      this.$refs.ruleForm.validate(async (valid) => {
        if (valid) {
          this.dialogFormVisible = false;
          let result = await this.$API.trademark.reqAddOrUpdateTradeMark(
            this.tmForm
          );
          if (result.code == 200) {
            this.$message({
              type: "success",
              message: this.tmForm.id ? "修改品牌成功" : "添加品牌成功",
            });
            this.getPageList(this.tmForm.id ? this.page : 1);
          }
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    updateTradeMark(row) {
      this.dialogFormVisible = true;
      console.log(row);
      this.tmForm = { ...row };
    },
    deleteTradeMark(row) {
      this.$confirm(`are u sure delete ${row.tmName}`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          let result = await this.$API.trademark.reqDeleteTradeMark(row.id);
          console.log(result);
          if (result.code == 200) {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            this.getPageList(this.list.length>1?this.page:this.page-1);
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>
<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
  