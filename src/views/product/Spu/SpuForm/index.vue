<template>
  <div>
    <el-form ref="form" :model="form" label-width="80px">
      <el-form-item label="SPU名称">
        <el-input v-model="form.spuName" placeholder="SPU名称"></el-input>
      </el-form-item>
      <el-form-item label="品牌">
        <el-select v-model="form.tmId" placeholder="请选择品牌">
          <el-option
            :label="item.tmName"
            :value="item.id"
            v-for="item in tradeMarkList"
            :key="item.id"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="SPU描述">
        <el-input
          type="textarea"
          rows="4"
          placeholder="SPU描述"
          v-model="form.description"
        ></el-input>
      </el-form-item>
      <el-form-item label="SPU图片">
        <el-upload
          action="/api/admin/product/fileUpload"
          list-type="picture-card"
          :file-list="spuImageList"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handleAvatarSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="" />
        </el-dialog>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-select
          v-model="attrIdAndName"
          :placeholder="`还有${unseletedValue.length}个未选择`"
        >
          <el-option
            :label="item.name"
            :value="`${item.id};${item.name}`"
            v-for="item in unseletedValue"
            :key="item.id"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          :disabled="!attrIdAndName"
          @click="addSaleAttr"
          icon="el-icon-plus"
          >添加销售属性</el-button
        >
        <el-table border style="width: 100%" :data="form.spuSaleAttrList">
          <el-table-column align="center" type="index" label="序号" width="80">
          </el-table-column>
          <el-table-column prop="saleAttrName" label="属性名">
          </el-table-column>
          <el-table-column prop="description" label="属性值名称列表">
            <template slot-scope="{ row }">
              <el-tag
                :key="tag.id"
                v-for="tag in row.spuSaleAttrValueList"
                closable
                :disable-transitions="false"
                @close="handleClose(row, tag)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <el-input
                class="input-new-tag"
                v-if="row.inputVisible"
                v-model="row.inputValue"
                ref="saveTagInput"
                size="small"
                @blur="handleInputConfirm(row)"
                @keyup.enter.native="$event.target.blur"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="showInput(row)"
                >+ New Tag</el-button
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template slot-scope="{ $index }">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="deleteValue($index)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">保存</el-button>
        <el-button @click="goScene()">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { throwStatement } from "@babel/types";
import { Slider } from "element-ui";
export default {
  name: "SpuForm",
  data() {
    return {
      form: {
        category3Id: "",
        description: "",
        spuImageList: [],
        spuName: "",
        spuSaleAttrList: [],
        tmId: "",
      },
      tradeMarkList: [],
      spuImageList: [],
      baseSaleAttrList: [],
      attrIdAndName: "",
      dialogImageUrl: "",
      dialogVisible: false,
    };
  },
  methods: {
    handleRemove(file, fileList) {
      //   console.log(file, fileList);
      this.spuImageList = fileList;
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleAvatarSuccess(res, file, fileList) {
      this.spuImageList = fileList;
    },
    async onSubmit() {
      this.form.spuImageList = this.spuImageList.map((item) => {
        return {
          imgName: item.name,
          imgUrl: (item.response && item.response.data) || item.url,
        };
      });
      let result = await this.$API.spu.reqAddOrUpdateSpu(this.form);
      //   console.log(result);
      if (result.code == 200) {
        this.$message({
          type: "success",
          message: this.form.id ? "修改成功" : "添加成功",
        });
        this.$emit("changeScene", 1, this.form.id);
      }
      Object.assign(this._data, this.$options.data());
    },
    goScene() {
      this.$emit("changeScene", 1);
      //    清理数据
      //Object.assign 合并对象
      //this._data 可以操作data响应数据
      //this.$options可以获取配置对象 配置对象中的data函数执行 返回的响应式数据为空
      Object.assign(this._data, this.$options.data());
    },
    async initSpuData(spu) {
      //   获取spu信息
      let spuResult = await this.$API.spu.reqSpu(spu.id);
      if (spuResult.code == 200) {
        this.form = spuResult.data;
      }
      //获取品牌信息
      let tradeResult = await this.$API.spu.reqTradeMarkList();
      if (tradeResult.code == 200) {
        this.tradeMarkList = tradeResult.data;
      }
      //获取图片
      let imgResult = await this.$API.spu.reqSpuImageList(spu.id);
      if (imgResult.code == 200) {
        //照片墙需要name url 需要把服务器返回的数据处理
        let imageList = imgResult.data;
        imageList.forEach((item) => {
          item.name = item.imgName;
          item.url = item.imgUrl;
        });
        this.spuImageList = imageList;
      }
      //获取平台销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.baseSaleAttrList = saleResult.data;
      }
    },
    async getSpuData(category3Id) {
      //获取品牌信息
      let tradeResult = await this.$API.spu.reqTradeMarkList();
      if (tradeResult.code == 200) {
        this.tradeMarkList = tradeResult.data;
      }
      //获取平台销售属性
      let saleResult = await this.$API.spu.reqBaseSaleAttrList();
      if (saleResult.code == 200) {
        this.baseSaleAttrList = saleResult.data;
      }
      this.form.category3Id = category3Id;
    },
    handleClose(row, tag) {
      row.spuSaleAttrValueList.splice(row.spuSaleAttrValueList.indexOf(tag), 1);
    },
    showInput(row) {
      this.$set(row, "inputVisible", true);
      this.$set(row, "inputValue", "");
      this.$nextTick(() => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    handleInputConfirm(row) {
      const { baseSaleAttrId, inputValue } = row;
      let newSaleAttrValue = { baseSaleAttrId, saleAttrValueName: inputValue };

      if (inputValue.trim() == "") {
        this.$message("不能为空");
        return;
      }
      let result = row.spuSaleAttrValueList.every(
        (item) => item.saleAttrValueName != inputValue
      );
      if (!result) {
        this.$message("不能填重复的");
        return;
      }
      row.spuSaleAttrValueList.push(newSaleAttrValue);
      row.inputVisible = false;
      row.inputValue = "";
    },
    addSaleAttr() {
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndName.split(";");
      let newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: [],
      };
      this.form.spuSaleAttrList.push(newSaleAttr);
      this.attrIdAndName = "";
    },
    deleteValue(index) {
      this.form.spuSaleAttrList.splice(index, 1);
    },
  },
  computed: {
    unseletedValue() {
      return this.baseSaleAttrList.filter((item) => {
        return this.form.spuSaleAttrList.every((item1) => {
          return item.name != item1.saleAttrName;
        });
      });
    },
  },
};
</script>
<style>
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>