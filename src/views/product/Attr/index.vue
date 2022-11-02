<template>
  <div>
    <el-card style="margin: 20px 0px">
      <CategorySelect
        :isShow="!isShow"
        @getCategoryId="getCategoryId"
      ></CategorySelect>
    </el-card>
    <el-card>
      <div v-show="isShow">
        <el-button
          type="primary"
          :disabled="!category3Id"
          @click="addAttr"
          icon="el-icon-plus"
          >添加属性</el-button
        >
        <el-table :data="attrList" border style="width: 100%">
          <el-table-column type="index" align="center" label="序号" width="80">
          </el-table-column>
          <el-table-column prop="attrName" label="属性姓名" width="150">
          </el-table-column>
          <el-table-column prop="attrValueList" label="属性值列表">
            <template slot-scope="{ row }">
              <el-tag
                type="success"
                style="margin: 10px 20px"
                v-for="(attr, index) in row.attrValueList"
                :key="index"
                >{{ attr.valueName }}</el-tag
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template slot-scope="{ row }">
              <el-button
                size="mini"
                @click="editAttr(row)"
                type="warning"
                icon="el-icon-edit"
              ></el-button>
              <el-button
                size="mini"
                type="danger"
                icon="el-icon-delete"
                @click="handleDelete(row)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div v-show="!isShow">
        <el-form :inline="true" label-width="80px" :model="attrInfo">
          <el-form-item label="属性名">
            <el-input
              placeholder="请输入属性名"
              v-model="attrInfo.attrName"
            ></el-input>
          </el-form-item>
        </el-form>
        <el-button
          type="primary"
          :disabled="!attrInfo.attrName"
          icon="el-icon-plus"
          @click="addAttrValue"
          >添加属性值</el-button
        >
        <el-button @click="isShow = true">取消</el-button>
        <el-table
          border
          style="margin: 20px 0px"
          :data="attrInfo.attrValueList"
        >
          <el-table-column
            type="index"
            align="center"
            label="序号"
            width="80"
          ></el-table-column>
          <el-table-column label="属性值名称" width="width">
            <template slot-scope="{ row, $index }">
              <el-input
                placeholder="请输入属性值名称"
                v-model="row.valueName"
                size="mini"
                v-if="row.flag"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
                :ref="$index"
              ></el-input>
              <span
                v-else
                @click="toEdit(row, $index)"
                style="display: block"
                >{{ row.valueName }}</span
              >
            </template>
          </el-table-column>
          <el-table-column label="操作" width="width">
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除${row.valueName}属性值吗`"
                @onConfirm="handleValueDelete($index)"
              >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  slot="reference"
                ></el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>
        <el-button
          type="primary"
          @click="addOrUpdateAttr"
          :disabled="attrInfo.attrValueList.length <= 0"
          >保存</el-button
        >
        <el-button @click="isShow = true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>
  
  <script>
import cloneDeep from "lodash/cloneDeep";
export default {
  name: "MyAttr",
  data() {
    return {
      category1Id: "",
      category2Id: "",
      category3Id: "",
      attrList: [],
      isShow: true,
      attrInfo: {
        attrName: "",
        attrValueList: [],
        categoryId: 0,
        categoryLevel: 3,
      },
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

        this.getAttrList();
      }
    },
    async getAttrList() {
      let result = await this.$API.attr.reqGetAttrList(
        this.category1Id,
        this.category2Id,
        this.category3Id
      );
      if (result.code == 200) {
        this.attrList = result.data;
        // console.log(this.attrList);
      }
    },
    handleEdit(row) {
      console.log(row);
    },
    handleDelete(row) {
      console.log(row);
    },
    addAttrValue() {
      this.attrInfo.attrValueList.push({
        attrId: this.attrInfo.id,
        valueName: "",
        //给每一个属性添加 用户切换查看与编辑模式 每一个属性值可以控制自己的切换
        flag: true,
      });
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus();
      });
    },
    handleValueDelete(index) {
      this.attrInfo.attrValueList.splice(index, 1);
    },
    addAttr() {
      this.isShow = false;
      this.attrInfo = {
        attrName: "",
        attrValueList: [],
        categoryId: this.category3Id,
        categoryLevel: 3,
      };
    },
    editAttr(row) {
      this.isShow = false;
      // console.log(row)
      //由于数据结构当中存在对象里面套数组 数组里面又套对象 因此需要深拷贝
      this.attrInfo = cloneDeep(row);
      //在修改某一个属性 相应的属性值加上flag
      this.attrInfo.attrValueList.forEach((item) => {
        //$set 让添加的新的属性是响应式的
        this.$set(item, "flag", false);
      });
    },
    toLook(row) {
      //新增的属性不能与已有的属性值名称相同
      let recur = this.attrInfo.attrValueList.some((item) => {
        //需要将row从数组判断里面移除
        //判断的时候需要把新增的row去除 判断之前数组是否和现在新增的row里的有相同的属性值
        if (row != item) {
          return row.valueName == item.valueName;
        }
      });
      if (row.valueName.trim() == "") {
        this.$message("不可以不填东西哦");
        return;
      } else if (recur) {
        this.$message("不可以填重复的东西哦");
        return;
      } else {
        row.flag = false;
      }
    },
    toEdit(row, index) {
      row.flag = true;
      //获取input节点 实现自动聚焦
      //点击span切换到input 不能立马拿到input节点 用$nextTick
      this.$nextTick(() => {
        this.$refs[index].focus();
        // console.log(this.$refs)
      });
    },
    async addOrUpdateAttr() {
      //属性值为空并且flag参数不需要给服务器
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(
        (item) => {
          if (item.valueName !== "") {
            delete item.flag;
            return true;
          }
        }
      );
      let result = await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo);
      if (result.code == 200) {
        this.$message({
          message: "保存成功",
          type: "success",
        });
        this.getAttrList();
        this.isShow = true;
      }
    },
  },
};
</script>
  