<template>
  <div class="charts" ref="charts"></div>
</template>
  
  <script>
import * as echarts from "echarts";
export default {
  name: "LineChart",
  mounted() {
    let lineChart = echarts.init(this.$refs.charts);
    lineChart.setOption({
      xAxis: {
        show: false,
        type: "category",
      },
      yAxis: {
        show: false,
      },
      series: [
        {
          type: "line",
          data: [10, 7, 33, 12, 48, 9, 29, 10, 44],
          grid: {
            left: 0,
            righy: 0,
            top: 0,
            bottom: 0,
          },
          itemStyle: {
            opacity: 0,
          },
          lineStyle: {
            color: "purple",
          },
          areaStyle: {
            color: {
              type: "linear",
              x: 0,
              y: 0,
              x2: 0,
              y2: 1,
              colorStops: [
                {
                  offset: 0,
                  color: "purple", // 0% 处的颜色
                },
                {
                  offset: 1,
                  color: "white", // 100% 处的颜色
                },
              ],
              global: false,
            },
          },
          smooth:true
        },
      ],
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
  

  