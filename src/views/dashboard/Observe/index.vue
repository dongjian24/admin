<template>
  <div>
    <el-row :gutter="10">
      <el-col :span="12">
        <MySearch></MySearch>
      </el-col>
      <el-col :span="12">
        <MyCategory></MyCategory>
      </el-col>
    </el-row>
  </div>
</template>
  
  <script>
import MyCategory from "./Category";
import MySearch from "./Search";
export default {
  name: "MyObserve",
  components: {
    MyCategory,
    MySearch,
  },
};
</script>
  
  