<template>
  <div class="charts" ref="charts"></div>
</template>
  
  <script>
import * as echarts from "echarts";
export default {
  name: "barChart",
  mounted() {
    let charts = echarts.init(this.$refs.charts);
    charts.setOption({
      xAxis: {
        show: false,
        type:'category'
      },
      yAxis: {
        show: false,
      },
      series: [
        {
          type: "bar",
          data: [10, 7, 33, 12, 48, 9, 29, 10, 44, 33, 22, 8],
        },
      ],
      grid: {
        top: 0,
        bottom: 0,
        left: 0,
        right: 0,
      },
      tooltip:{
        
      }
    });
  },
};
</script>
<style scoped>
.charts {
  width: 100%;
  height: 100%;
}
</style>
  

  