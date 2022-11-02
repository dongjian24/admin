<template>
  <el-card class="box-card">
    <div slot="header" class="clearfix">
      <el-tabs v-model="activeName" class="tab" @tab-click="handleClick">
        <el-tab-pane label="销售额" name="first"></el-tab-pane>
        <el-tab-pane label="访问量" name="second"></el-tab-pane>
      </el-tabs>
      <div class="block">
        <span class="demonstration" @click="setDay">今日</span>
        <span class="demonstration" @click="setWeek">本周</span>
        <span class="demonstration" @click="setMonth">本月</span>
        <span class="demonstration" @click="setYear">本年</span>
        <el-date-picker
          class="date"
          v-model="value1"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          size="mini"
          value-format="yyyy-MM-dd"
        >
        </el-date-picker>
      </div>
    </div>
    <el-row>
      <el-col :span="18">
        <div class="charts" ref="charts"></div>
      </el-col>
      <el-col :span="6" class="rank">
        <h3>门店{{ title }}排名</h3>
        <ul>
          <li>
            <span class="index">1</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span class="index">2</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span class="index">3</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span>4</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span>5</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span>6</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
          <li>
            <span>7</span>
            <span>kfc</span>
            <span class="value">234</span>
          </li>
        </ul>
      </el-col>
    </el-row>
  </el-card>
</template>
  
  <script>
import * as echarts from "echarts";
import * as dayjs from "dayjs";
import { mapState } from "vuex";
export default {
  name: "MySale",
  data() {
    return {
      activeName: "first",
      value1: "",
      charts: null,
      date: [1, 2],
    };
  },
  methods: {
    handleClick(tab, event) {
      //   console.log(tab, event);
    },
    setDay() {
      this.value1 = [dayjs().format(), dayjs().format()];
      // console.log(Date())
    },
    setWeek() {
      this.value1 = [dayjs().day(1).format(), dayjs().day(7).format()];
    },
    setMonth() {
      this.value1 = [
        dayjs().startOf("month").format(),
        dayjs().endOf("month").format(),
      ];
    },
    setYear() {
      this.value1 = [
        dayjs().startOf("year").format(),
        dayjs().endOf("year").format(),
      ];
    },
  },
  mounted() {
    this.charts = echarts.init(this.$refs.charts);
    this.charts.setOption({
      title: {
        text: this.title + "趋势",
      },
      tooltip: {
        trigger: "axis",
        axisPointer: {
          type: "shadow",
        },
      },
      grid: {
        left: "3%",
        right: "4%",
        bottom: "3%",
        containLabel: true,
      },
      xAxis: [
        {
          type: "category",
          data: [],
          axisTick: {
            alignWithLabel: true,
          },
        },
      ],
      yAxis: [
        {
          type: "value",
        },
      ],
      series: [
        {
          name: "Direct",
          type: "bar",
          barWidth: "60%",
          data: [],
        },
      ],
    });
  },
  computed: {
    title() {
      return this.activeName === "first" ? "销售额" : "访问量";
    },
    ...mapState({
      listData: (state) => state.home.list,
    }),
  },
  watch: {
    title() {
      this.charts.setOption({
        title: {
          text: this.title,
        },
        xAxis: {
          data:
            this.title == "销售额"
              ? this.listData.orderFullYearAxis
              : this.listData.userFullYearAxis,
        },
        series: [
          {
            data:
              this.title == "销售额"
                ? this.listData.orderFullYear
                : this.listData.userFullYear,
          },
        ],
      });
    },
    listData() {
      this.charts.setOption({
        title: {
          text: this.title + "趋势",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: [
          {
            type: "category",
            data: this.listData.orderFullYearAxis,
            axisTick: {
              alignWithLabel: true,
            },
          },
        ],
        yAxis: [
          {
            type: "value",
          },
        ],
        series: [
          {
            name: "Direct",
            type: "bar",
            barWidth: "60%",
            data: this.listData.orderFullYear,
          },
        ],
      });
    },
  },
};
</script>
<style lang="scss" scoped>
.box-card {
  margin-top: 10px;
  position: relative;
}
::v-deep .el-card__header {
  border-bottom: none;
}
.tab {
  width: 100%;
}
.clearfix {
  display: flex;
  justify-content: space-between;
}
.block {
  position: absolute;
  right: 0;
}
.block span {
  margin-right: 20px;
}
.date {
  width: 280px !important;
  margin: 0 20px;
}
.charts {
  width: 100%;
  height: 300px;
}
.rank h3 {
  /* padding-top: 0px; */
  margin-top: 0px;
}
ul {
  list-style: none;
  width: 100%;
  height: 300px;
  padding: 0px;
}
li {
  height: 13%;
  width: 100%;
}
.index {
  width: 20px;
  height: 20px;
  background: black;
  color: white;
  border-radius: 50%;
  float: left;
  text-align: center;
  line-height: 20px;
  margin-right: 20px;
}
.value {
  float: right;
}
</style>
  