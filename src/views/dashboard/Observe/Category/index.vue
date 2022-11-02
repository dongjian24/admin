<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <div class="card-header">
        <span>销售额</span>
        <el-radio-group v-model="radio" size="small">
          <el-radio-button label="上海"></el-radio-button>
          <el-radio-button label="北京"></el-radio-button>
          <el-radio-button label="广州"></el-radio-button>
          <el-radio-button label="深圳"></el-radio-button>
        </el-radio-group>
      </div>
    </div>
    <div class="cont">
      <div class="charts" ref="charts"></div>
    </div>
  </el-card>
</template>
  
  <script>
import * as echarts from "echarts";
export default {
  name: "MyCategory",
  data() {
    return {
      radio: "上海",
    };
  },
  mounted() {
    let charts = echarts.init(this.$refs.charts);
    charts.setOption({
      title: {
        text: "we",
        subtext: "qw",
        left: "center",
        top: "center",
      },
      tooltip: {
        trigger: "item",
      },
      series: [
        {
          name: "Access From",
          type: "pie",
          radius: ["40%", "70%"],
          avoidLabelOverlap: false,
          itemStyle: {
            borderRadius: 10,
            borderColor: "#fff",
            borderWidth: 2,
          },
          label: {
            show: true,
            position: "outside",
          },
          labelLine: {
            show: true,
          },
          data: [
            { value: 1048, name: "Search Engine" },
            { value: 735, name: "Direct" },
            { value: 580, name: "Email" },
            { value: 484, name: "Union Ads" },
            { value: 300, name: "Video Ads" },
          ],
        },
      ],
    });
    charts.on("mouseover", (params) => {
      const { name, value } = params.data;
      charts.setOption({
        title: {
          text: name,
          subtext: value,
        },
      });
    });
  },
};
</script>
<style scoped>
.box-card {
  margin-top: 10px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.charts {
  width: 100%;
  height: 300px;
}
</style>
  
  