<template>
  <div class="container">
    <!-- 显示筛选到的电影数量 -->
        <!-- 显示查询时间 -->
       
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
        <div class="total-count">
          <div>MySQL 查询耗时： {{ queryTime1 }} ms</div>
          <div>Hive 查询耗时： {{ queryTime2 }} ms</div>
          <div>Neo4j 查询耗时： {{ queryTime3 }} ms</div>
        </div>
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
      dialogVisible: false, // 控制弹出框显示/隐藏
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
      console.log('确认对比')
      this.dialogVisible = false
    },
    async fetchMovies() {
      this.movieList = [] // 清空之前的数据
      const startTime1 = Date.now() // 记录第一个数据库查询开��时间
      const requests1 = this.fetchMoviesFromMySQL()
      const results1 = await requests1.catch(error => {
        console.error('MySQL 请求失败:', error)
        return [] // 如果失败，返回空数组
      })
      this.movieList = this.movieList.concat(results1) // 合并数据
      const endTime1 = Date.now()
      this.queryTime1 = endTime1 - startTime1

      // 第二个数据库查询
      const startTime2 = Date.now()
      const requests2 = this.fetchMoviesFromHive()
      const results2 = await requests2.catch(error => {
        console.error('Hive 请求失败:', error)
        return [] // 如果失败，返回空数组
      })
      this.movieList = this.movieList.concat(results2) // 合并数据
      const endTime2 = Date.now()
      this.queryTime2 = endTime2 - startTime2

      // 第三个数据库查询
      const startTime3 = Date.now()
      const requests3 = this.fetchMoviesFromNeo4j()
      const results3 = await requests3.catch(error => {
        console.error('Neo4j 请求失败:', error)
        return [] // 如果失败，返回空数组
      })
      this.movieList = this.movieList.concat(results3) // 合并数据
      const endTime3 = Date.now()
      this.queryTime3 = endTime3 - startTime3
    },

    fetchMoviesFromMySQL() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/cooperationaa',
        params: {},
        headers: {}
      }
      return axios(config)
        .then(response => {
          if (Array.isArray(response.data)) {
            return response.data
          } else {
            console.error('返回的数据格式不正确')
            return []
          }
        })
        .catch(error => {
          console.log(error)
          return []
        })
    },

    fetchMoviesFromHive() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/cooperationaa',
        params: {},
        headers: {}
      }
      return axios(config)
        .then(response => {
          if (Array.isArray(response.data)) {
            return response.data
          } else {
            console.error('返回的数据格式不正确')
            return []
          }
        })
        .catch(error => {
          console.log(error)
          return []
        })
    },

    fetchMoviesFromNeo4j() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/cooperationaa',
        params: {},
        headers: {}
      }
      return axios(config)
        .then(response => {
          if (Array.isArray(response.data)) {
            return response.data
          } else {
            console.error('返回的数据格式不正确')
            return []
          }
        })
        .catch(error => {
          console.log(error)
          return []
        })
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
  padding-left: 20px;
  padding-right: 20px;
}

.form-button {
  flex: 1;
  padding: 10px 0;
  width: 100%;
  height: 100%;
}
</style>
