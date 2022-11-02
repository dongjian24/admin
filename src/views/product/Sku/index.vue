<template>
  <div>
    <el-table :data="skuList" border>
      <el-table-column
        type="index"
        align="center"
        width="80px"
        label="序号"
      ></el-table-column>
      <el-table-column label="名称" prop="skuName"></el-table-column>
      <el-table-column label="描述" prop="skuDesc"></el-table-column>
      <el-table-column label="默认图片" width="100px">
        <template slot-scope="{ row }">
          <img
            :src="row.skuDefaultImg"
            style="width: 80px; height: 80px"
            alt=""
          />
        </template>
      </el-table-column>
      <el-table-column
        label="重量(KG)"
        prop="weight"
        width="80px"
      ></el-table-column>
      <el-table-column
        label="价格(元)"
        prop="price"
        width="80px"
      ></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{ row }">
          <hint-button
            v-if="row.isSale == 0"
            type="success"
            icon="el-icon-top"
            size="mini"
            title="上架"
            @click="onSale(row)"
          ></hint-button>
          <hint-button
            v-else
            type="info"
            icon="el-icon-bottom"
            size="mini"
            title="下架"
            @click="cancelSale(row)"
          ></hint-button>
          <hint-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            title="修改sku"
            @click="edit"
          ></hint-button>
          <hint-button
            type="info"
            icon="el-icon-info"
            size="mini"
            title="查看详情"
            @click="info(row)"
          ></hint-button>
          <el-popconfirm title="这是一段内容确定删除吗？">
            <hint-button
              type="danger"
              slot="reference"
              icon="el-icon-delete"
              size="mini"
              title="删除sku"
            ></hint-button>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      style="text-align: center; margin-top: 20px"
      :total="total"
      :current-page="page"
      :page-sizes="[3, 5, 10]"
      :page-size="limit"
      @size-change="handleSizeChange"
      @current-change="getSkuList"
      layout="prev,pager,next,jumper,->,sizes,total"
    ></el-pagination>
    <el-drawer
      :visible.sync="drawer"
      :show-close="false"
      size="40%"
      :direction="direction"
    >
      <el-row>
        <el-col :span="5">名称</el-col>
        <el-col :span="16">{{ skuInfo.skuName }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">描述</el-col>
        <el-col :span="16">{{ skuInfo.skuDesc }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">价格</el-col>
        <el-col :span="16">{{ skuInfo.price }}</el-col>
      </el-row>
      <el-row>
        <el-col :span="5">平台属性</el-col>
        <el-col :span="16">
          <template>
            <el-tag
              type="success"
              style="margin-left: 10px"
              v-for="item in skuInfo.skuAttrValueList"
              :key="item.id"
              >{{ item.attrName }}</el-tag
            >
          </template>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="5">商品图片</el-col>
        <el-col :span="16">
          <el-carousel height="150px">
            <el-carousel-item
              v-for="item in skuInfo.skuImageList"
              :key="item.id"
            >
              <img :src="item.imgUrl" alt="" />
            </el-carousel-item>
          </el-carousel>
        </el-col>
      </el-row>
    </el-drawer>
  </div>
</template>
  
  <script>
export default {
  name: "MySku",
  data() {
    return {
      skuList: [],
      total: 0,
      limit: 10,
      page: 1,
      drawer: false,
      direction: "rtl",
      skuInfo: {},
    };
  },
  mounted() {
    this.getSkuList();
  },
  methods: {
    handleSizeChange(val) {
      this.limit = val;
      this.getSkuList();
    },
    async getSkuList(pages = 1) {
      this.page = pages;
      const { page, limit } = this;
      let result = await this.$API.sku.reqSkuList(page, limit);
      // console.log(result);
      if (result.code == 200) {
        this.total = result.data.total;
        this.skuList = result.data.records;
      }
    },
    async onSale(row) {
      let result = await this.$API.sku.reqOnSale(row.id);
      if (result.code == 200) {
        this.$message({
          type: "success",
          message: "上架成功",
        });
        row.isSale = 1;
      }
    },
    async cancelSale(row) {
      let result = await this.$API.sku.reqCancelSale(row.id);
      if (result.code == 200) {
        this.$message({
          type: "success",
          message: "下架成功",
        });
        row.isSale = 0;
      }
    },
    edit() {
      this.$message({
        type: "info",
        message: "正在开发中.....",
      });
    },
    async info(row) {
      this.drawer = true;
      let result = await this.$API.sku.reqGetSku(row.id);
      // console.log(result)
      if (result.code == 200) {
        this.skuInfo = result.data;
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.el-row .el-col-5 {
  font-size: 20px;
  text-align: right;
  font-weight: bold;
  margin-bottom: 20px;
}
.el-col-16 {
  margin: 3px 10px;
}
.el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
}

.el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
}
::v-deep .el-carousel__button {
  width: 15px;
  height: 15px;
  background: burlywood;
  border-radius: 50%;
}
</style>
<style>

</style>
  