<template>
  <div>
    <div :id="id" :class="className" :style="{height: height, width: width}"></div>
    <button @click="randomizeData">随机生成数据</button>
  </div>
</template>

<script>
import echarts from 'echarts'

export default {
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    id: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '400px'
    }
  },
  data() {
    return {
      chart: null,
      queryTime1: 0,
      queryTime2: 0,
      queryTime3: 0
    }
  },
  mounted() {
    this.initChart()
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose()
    }
  },
  methods: {
    initChart() {
      this.chart = echarts.init(document.getElementById(this.id))
      this.updateChart()
    },
    randomizeData() {
      // 随机生成3个查询时间数据
      this.queryTime1 = Math.floor(Math.random() * 5000) + 500
      this.queryTime2 = Math.floor(Math.random() * 5000) + 500
      this.queryTime3 = Math.floor(Math.random() * 5000) + 500

      this.updateChart() // 更新图表
    },
    updateChart() {
      const xData = ['Query 1', 'Query 2', 'Query 3']
      const yData = [this.queryTime1, this.queryTime2, this.queryTime3]

      this.chart.setOption({
        title: {
          text: '查询时间统计',
          left: 'center',
          textStyle: {
            color: '#333',
            fontSize: 18
          }
        },
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: xData,
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        },
        yAxis: {
          type: 'value',
          axisLine: {
            lineStyle: {
              color: '#333'
            }
          }
        },
        series: [{
          data: yData,
          type: 'line',
          smooth: true,
          lineStyle: {
            color: '#ff7f50'
          },
          symbol: 'circle',
          itemStyle: {
            color: '#ff7f50'
          }
        }]
      })
    }
  }
}
</script>

<style scoped>
.chart {
  width: 100%;
  height: 400px;
}
</style>
