<template>
  <div class="charts" ref="charts"></div>
</template>
  
  <script>
import * as echarts from "echarts";
export default {
  name: "progressChart",
  mounted() {
    let charts = echarts.init(this.$refs.charts);
    charts.setOption({
      xAxis: {
        show: false,
        min: 0,
        max: 100,
      },
      yAxis: {
        show: false,
        //均分
        type: "category",
      },
      series: [
        {
          type: "bar",
          data: [88],
          barWidth: 12,
          color: "green",
          showBackground: true,
          backgroundStyle: {
            color: "#eee",
          },
          label:{
            show:true,
            formatter:'|',
            position:'right'
          }
        },
      ],
      grid: {
        top: 0,
        bottom: 0,
        left: 0,
        right: 0,
      },
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
  

  