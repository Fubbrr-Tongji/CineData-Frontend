<template>
  <div class="container">
    <!-- 右侧表格区域 -->
    <el-button class="form-button" @click="showComparisonDialog">运行时间对比</el-button>
    <!-- 弹出框 -->
    <el-dialog
      :visible.sync="dialogVisible"
      title="运行时间对比"
      width="60%"
      @close="handleClose"
    >
      <div class="charts-container">
        <!-- 折线图 -->
        <div ref="lineChart" class="chart" style="height: 300px;" />

        <!-- 饼状图 -->
        <div ref="pieChart" class="chart" style="height: 300px" />
      </div>
      <div slot="footer">
        <el-button @click="handleClose">关闭</el-button>
      </div>
    </el-dialog>
    <div class="table-container">
      <el-table :data="movieList" style="width: 100%">
        <el-table-column prop="actorName1" label="演员1" :width="500" />
        <el-table-column prop="actorName2" label="演员2" :width="500" />
        <el-table-column prop="cooperationCount" label="合作次数" :width="300" />
      </el-table>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
export default {
  data() {
    return {
    // 控制弹出框显示/隐藏
      dialogVisible: false,
      movieList: [], // 用于存储查询的演员合作数据
      queryTime1: 0, // 第一个数据库的查询时间
      queryTime2: 0, // 第二个数据库的查询时间
      queryTime3: 0 // 第三个数据库的查询时间
    }
  },
  created() {
    this.fetchMovies()
  },
  methods: {
  // 显示弹出框
    showComparisonDialog() {
      this.dialogVisible = true
      this.$nextTick(() => {
        this.initCharts()
      })
    },
    // 初始化图表
    initCharts() {
      // 初始化折线图
      this.initLineChart()
      // 初始化饼状图
      this.initPieChart()
    },
    // 折线图初始化
    initLineChart() {
      const lineChart = echarts.init(this.$refs.lineChart)
      const lineOptions = {
        title: {
          text: '折线图运行时间对比'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['运行时间']
        },
        xAxis: {
          type: 'category',
          data: ['MySQL', 'Hive', 'Neo4j']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '运行时间/ms',
            type: 'line',
            data: [this.queryTime1, this.queryTime2, this.queryTime3]
          }
        ]
      }
      lineChart.setOption(lineOptions)
    },
    // 饼状图初始化
    initPieChart() {
      const pieChart = echarts.init(this.$refs.pieChart)
      const pieOptions = {
        title: {
          text: '饼图运行时间对比',
          left: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)'
        },
        legend: {
          orient: 'vertical',
          left: 'left',
          data: ['MySQL', 'Hive', 'Neo4j']
        },
        series: [
          {
            name: '运行时间/ms',
            type: 'pie',
            radius: '50%',
            data: [
              { value: this.queryTime1, name: 'MySQL' },
              { value: this.queryTime2, name: 'Hive' },
              { value: this.queryTime3, name: 'Neo4j' }
            ]
          }
        ]
      }
      pieChart.setOption(pieOptions)
    },
    // 关闭弹出框
    handleClose() {
      this.dialogVisible = false
    },
    // 确认按钮点击处理
    handleOk() {
      // 在这里处理确认逻辑
      console.log('确认对比')
      this.dialogVisible = false // 关闭弹出框
    },
    async fetchMovies() {
      this.movieList = []
      const startTime1 = Date.now() // 记录第一个数据库查询开始时间
      const requests1 = this.fetchMoviesFromMySQL()
      const results1 = await requests1
      this.movieList = this.movieList.concat(results1)
      const endTime1 = Date.now() // 记录第一个数据库查询结束时间
      this.queryTime1 = endTime1 - startTime1 // 计算第一个数据库查询耗时

      // 第二个数据库查询（只记录查询时间，不合并数据）
      const startTime2 = Date.now() // 记录第二个数据库查询开始时间
      const requests2 = this.fetchMoviesFromHive()
      await requests2
      const endTime2 = Date.now() // 记录第二个数据库查询结束时间
      this.queryTime2 = endTime2 - startTime2 // 计算第二个数据库查询耗时

      // 第三个数据库查询（只记录查询时间，不合并数据）
      const startTime3 = Date.now() // 记录第三个数据库查询开始时间
      const requests3 = this.fetchMoviesFromNeo4j()
      await requests3
      const endTime3 = Date.now() // 记录第三个数据库查询结束时间
      this.queryTime3 = endTime3 - startTime3 // 计算第三个数据库查询耗时
    },

    fetchMoviesFromMySQL() {
      return this.$axios.get('/api/db1/movies/cooperationaa').then(res => res.data)
    },

    fetchMoviesFromHive() {
      return this.$axios.get('/api/db2/movies/cooperationaa').then(res => res.data)
    },

    fetchMoviesFromNeo4j() {
      return this.$axios.get('/api/db3/movies/cooperationaa').then(res => res.data)
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  gap: 50px;
  padding: 20px;
  margin-top: 10px;
  margin-left: 20px;
}

.table-container {
  flex: 2;
  padding-left: 20px; /* 左侧间距 */
  padding-right: 20px; /* 右侧间距 */
}

.button-container {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 20px;
  align-items: center;
}

.form-button {
  flex: 1;
  padding: 10px 0;
  width: 100%;
  height: 100%;
}
</style>
