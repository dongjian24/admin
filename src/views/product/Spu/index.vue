<template>
  <div>
    <el-card style="margin: 20px 0px">
      <CategorySelect
        @getCategoryId="getCategoryId"
        :isShow="flag != 1"
      ></CategorySelect>
    </el-card>
    <el-card>
      <div v-show="flag == 1">
        <el-button
          type="primary"
          @click="addSpu"
          icon="el-icon-plus"
          :disabled="!category3Id"
          >添加SPU</el-button
        >
        <el-table border style="width: 100%" :data="spuList">
          <el-table-column align="center" type="index" label="序号" width="80">
          </el-table-column>
          <el-table-column prop="spuName" label="spu名称"> </el-table-column>
          <el-table-column prop="description" label="spu描述">
          </el-table-column>
          <el-table-column fixed="right" label="操作">
            <template slot-scope="{ row }">
              <hint-button
                type="success"
                icon="el-icon-plus"
                size="mini"
                title="添加sku"
                @click="addSku(row)"
              ></hint-button>
              <hint-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                title="修改spu"
                @click="updateSpu(row)"
              ></hint-button>
              <hint-button
                type="info"
                icon="el-icon-info"
                size="mini"
                title="查看当前spu全部sku列表"
                @click="getSkuList(row)"
              ></hint-button>

              <el-popconfirm
                title="这是一段内容确定删除吗？"
                @onConfirm="deleteSpu(row)"
              >
                <hint-button
                  type="danger"
                  slot="reference"
                  icon="el-icon-delete"
                  size="mini"
                  title="删除spu"
                ></hint-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="getSpuList"
          :current-page="page"
          style="margin-top: 20px; text-align: center"
          :page-sizes="[3, 5, 10]"
          :page-size="limit"
          layout=" prev, pager, next, sizes,->, jumper,total"
          :total="total"
        >
        </el-pagination>
      </div>
      <SpuForm
        v-show="flag == 2"
        ref="spu"
        @changeScene="changeScene"
      ></SpuForm>
      <SkuForm
        v-show="flag == 3"
        ref="sku"
        @changeScene="changeScene"
      ></SkuForm>
    </el-card>
    <el-dialog
      :title="`${spuName}的sku列表`"
      :visible.sync="dialogTableVisible"
      :before-close="close"
    >
      <el-table :data="skuList" border v-loading="loading">
        <el-table-column property="skuName" label="名称"></el-table-column>
        <el-table-column property="price" label="价格"></el-table-column>
        <el-table-column property="weight" label="重量"></el-table-column>
        <el-table-column label="默认图片">
          <template slot-scope="{ row }">
            <img
              :src="row.skuDefaultImg"
              style="width: 100px; height: 100px"
              alt=""
            />
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>
  
  <script>
import SkuForm from "./SkuForm";
import SpuForm from "./SpuForm";
export default {
  name: "MySpu",
  components: { SkuForm, SpuForm },
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      spuList: [],
      total: 0,
      page: 1,
      limit: 3,
      isShow: true,
      flag: 1,
      dialogTableVisible: false,
      skuList: [],
      spuName: "",
      loading: true,
    };
  },
  methods: {
    getCategoryId({ categoryId, level }) {
      if (level == 1) {
        this.category1Id = categoryId;
        this.category2Id = "";
        this.category3Id = "";
      } else if (level == 2) {
        this.category2Id = categoryId;
        this.category3Id = "";
      } else {
        this.category3Id = categoryId;
        this.getSpuList();
      }
    },
    async getSpuList(val = 1) {
      this.page = val;
      const { page, limit, category3Id } = this;
      let result = await this.$API.spu.reqSpuList(page, limit, category3Id);
      // console.log(result);
      if (result.code == 200) {
        this.total = result.data.total;
        this.spuList = result.data.records;
      }
    },
    async getSkuList(row) {
      this.dialogTableVisible = true;
      this.spuName = row.spuName;
      let result = await this.$API.spu.reqSkuList(row.id);
      // console.log(result);
      if (result.code == 200) {
        this.skuList = result.data;
        this.loading = false;
      }
    },
    close(done) {
      this.skuList = []
      this.loading = true;
      done()
    },
    handleSizeChange(val) {
      this.limit = val;
      this.getSpuList();
    },
    addSpu() {
      this.flag = 2;
      this.$refs.spu.getSpuData(this.category3Id);
    },
    async deleteSpu(row) {
      let result = await this.$API.spu.reqDeleteSpu(row.id);
      // console.log(result);
      if (result.code == 200) {
        this.$message({ type: "success", message: "删除成功" });
        this.getSpuList(this.spuList.length <= 1 ? this.page - 1 : this.page);
      }
    },
    addSku(row) {
      this.flag = 3;
      this.$refs.sku.getData(row, this.category1Id, this.category2Id);
    },
    updateSpu(row) {
      this.flag = 2;
      this.$refs.spu.initSpuData(row);
    },
    toggleSelection(rows) {
      if (rows) {
        rows.forEach((row) => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    changeScene(scene, id) {
      this.flag = scene;
      this.getSpuList(id ? this.page : 1);

      // console.log(scene,id)
      // if (flag == "修改") {
      //   this.getSpuList(this.page);
      // } else {
      //   this.getSpuList();
      // }
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
  width: 140px;
  height: 140px;
  line-height: 140px;
  text-align: center;
}
.avatar {
  width: 140px;
  height: 140px;
  display: block;
}
</style>
  