<template>
  <div>
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="SPU名称">
        {{ spuName }}
      </el-form-item>
      <el-form-item label="SKU名称">
        <el-input v-model="form.skuName" placeholder="SKU名称"></el-input>
      </el-form-item>
      <el-form-item label="价格(元)">
        <el-input
          type="number"
          v-model="form.price"
          placeholder="价格(元)"
        ></el-input>
      </el-form-item>
      <el-form-item label="重量(千克)">
        <el-input v-model="form.weight" placeholder="重量(千克)"></el-input>
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input
          type="textarea"
          rows="4"
          placeholder="规格描述"
          v-model="form.skuDesc"
        ></el-input>
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form :inline="true">
          <el-form-item
            :label="item.attrName"
            v-for="item in attrInfoList"
            :key="item.id"
          >
            <el-select v-model="item.attrIdAndValueId" placeholder="请选择">
              <el-option
                :label="attrValue.valueName"
                :value="`${item.id}:${attrValue.id}`"
                v-for="attrValue in item.attrValueList"
                :key="attrValue.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form :model="form.spuSaleAttrList">
          <el-form-item
            :label="item.saleAttrName"
            v-for="item in spuSaleAttrList"
            :key="item.id"
          >
            <el-select v-model="item.attrIdAndValueId" placeholder="请选择">
              <el-option
                :label="sale.saleAttrValueName"
                v-for="sale in item.spuSaleAttrValueList"
                :key="sale.id"
                :value="`${item.id}:${sale.id}`"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table
          ref="multipleTable"
          border
          tooltip-effect="dark"
          style="width: 100%"
          :data="spuImageList"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" width="55"> </el-table-column>
          <el-table-column label="图片">
            <template slot-scope="{ row }">
              <el-image
                :src="row.imgUrl"
                style="width: 100px; height: 100px"
              ></el-image>
            </template>
          </el-table-column>
          <el-table-column prop="imgName" label="名称"> </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="{ row }">
              <el-button
                type="primary"
                @click="setNormal(row)"
                v-if="row.isDefault == 0"
                size="mini"
                >设为默认</el-button
              >
              <el-button size="mini" v-else>默认</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">保存</el-button>
        <el-button @click="goScene">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
export default {
  name: "SkuForm",
  data() {
    return {
      form: {
        category3Id: "",
        tmId: "",
        spuId: "",
        skuName: "",
        price: "",
        weight: "",
        skuDefaultImg: "",
        skuAttrValueList: [
          // {
          //   attrId:'',
          //   valueId:''
          // }
        ],
        skuImageList: [],
        skuSaleAttrValueList: [],
        skuDesc: "",
      },
      spuName: "",
      spuSaleAttrList: [],
      spuImageList: [],
      attrInfoList: [],
      multipleSelection: [],
    };
  },
  methods: {
    async onSubmit() {
      this.form.skuImageList = this.multipleSelection.map((item) => {
        return {
          imgName: item.imgName,
          imgUrl: item.imgUrl,
          isDefault: item.isDefault,
          spuImgId: item.id,
        };
      });
      //方法1
      /* let saleArr = [];
      this.spuSaleAttrList.forEach((item) => {
        if (item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(":");
          let attr = { attrId, valueId };
          saleArr.push(attr);
        }
      });
      this.form.skuSaleAttrValueList = saleArr;
      let arr = [];
      this.attrInfoList.forEach((item) => {
        if (item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(":");
          let attr = { attrId, valueId };
          arr.push(attr);
        }
      });
      this.form.skuAttrValueList = arr; */
      // 方法2
      this.form.skuAttrValueList = this.attrInfoList.reduce((prev, item) => {
        if (item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(":");
          prev.push({ attrId, valueId });
        }
        return prev;
      }, []);
      this.form.skuSaleAttrValueList = this.spuSaleAttrList.reduce(
        (prev, item) => {
          if (item.attrIdAndValueId) {
            const [saleAttrId, saleAttrValueId] =
              item.attrIdAndValueId.split(":");
            prev.push({ saleAttrId, saleAttrValueId });
          }
          return prev;
        },
        []
      );
      let result = await this.$API.spu.reqAddSku(this.form);
      // console.log(result)
      if (result.code == 200) {
        this.$message({
          type: "success",
          message: "添加成功",
        });
        this.$emit("changeScene", 1);
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    goScene() {
      this.$emit("changeScene", 1);
      Object.assign(this._data, this.$options.data());
    },
    async getData(row, category1Id, category2Id) {
      this.spuName = row.spuName;
      this.form.category3Id = row.category3Id;
      this.form.tmId = row.tmId;
      this.form.spuId = row.id;
      let Iresult = await this.$API.sku.reqSpuImageList(row.id);
      if (Iresult.code == 200) {
        let list = Iresult.data;
        list.forEach((item) => {
          item.isDefault = 0;
        });
        this.spuImageList = list;
      }
      let Sresult = await this.$API.sku.reqSpuSaleAttrList(row.id);

      if (Sresult.code == 200) {
        this.spuSaleAttrList = Sresult.data;
      }
      let Aresult = await this.$API.sku.reqAttrInfoList(
        category1Id,
        category2Id,
        row.category3Id
      );
      // console.log(Aresult);
      if (Aresult.code == 200) {
        this.attrInfoList = Aresult.data;
      }
    },
    setNormal(row) {
      // console.log(row);
      this.spuImageList.forEach((item) => {
        item.isDefault = 0;
      });
      row.isDefault = 1;
      this.form.skuDefaultImg = row.imgUrl;
    },
  },
};
</script>

