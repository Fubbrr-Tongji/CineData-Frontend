<template>
  <div class="container">
    <!-- 左侧输入区域 -->
    <div class="form-container">
      <el-form ref="form" :model="filters" label-width="100px">
        <el-form-item label="开始时间">
          <el-date-picker v-model="filters.Date" type="daterange" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" :disabled-date="disabledDate" />
        </el-form-item>

        <el-form-item label="电影名">
          <el-input v-model="filters.movieName" placeholder="请输入电影名" />
        </el-form-item>

        <el-form-item label="导演">
          <el-input v-model="filters.director" placeholder="请输入导演名" />
        </el-form-item>

        <el-form-item label="演员">
          <el-input v-model="filters.actor" placeholder="请输入演员名" />
        </el-form-item>

        <!-- 电影风格下拉框 -->
        <el-form-item label="电影风格">
          <el-select v-model="filters.genre" placeholder="请选择电影风格" :input-width="'100%'">
            <el-option label="请选择" value="" />
            <el-option v-for="item in genres" :key="item" :label="item" :value="item" />
          </el-select>
        </el-form-item>

        <el-form-item label="评分">
          <el-input-number
            v-model="filters.rating"
            :min="0"
            :max="5"
            :step="0.1"
            precision="1"
            placeholder="评分"
            @change="handleRatingChange"
          />
        </el-form-item>

        <!-- 选择积极评价 -->
        <el-form-item label="筛选积极评价">
          <el-radio-group v-model="filters.isPositiveReview" class="radio-group">
            <el-radio :label="true" class="radio-option">是</el-radio>
            <el-radio :label="false" class="radio-option">否</el-radio>
          </el-radio-group>
        </el-form-item>

        <div class="button-container">
          <el-button type="primary" class="form-button" @click="searchMovies">查询</el-button>
          <!-- 重置按钮 -->
          <el-button class="form-button" @click="resetForm">重置</el-button>
        </div>

        <!-- 显示筛选到的电影数量 -->
        <div class="total-count">
          共筛选到 {{ totalCount }} 个电影数据
        </div>

        <!-- 显示查询时间 -->
        <div class="total-count">
          MySQL 查询耗时： {{ queryTime1 }} ms
        </div>
        <div class="total-count">
          Hive 查询耗时： {{ queryTime2 }} ms
        </div>
        <div class="total-count">
          Neo4j 查询耗时： {{ queryTime3 }} ms
        </div>

        <!-- 运行时间对比按钮 -->
        <el-button class="form-button" @click="showComparisonDialog">运行时间对比</el-button>
        <!-- 弹出框 -->
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
            <div ref="pieChart" class="chart" style="height: 300px;" />
          </div>

          <div slot="footer">
            <el-button @click="handleClose">关闭</el-button>
          </div>
        </el-dialog>
      </el-form>
    </div>
    <!-- 右侧表格区域 -->

    <div class="table-container">
      <el-table :data="movieList" style="width: 100%">
        <el-table-column prop="movieName" label="电影名" :width="400" />
        <el-table-column prop="releaseDate" label="上映时间" :min-width="150" />
        <el-table-column prop="genre" label="电影风格" :min-width="150" />
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
      filters: {
        Date: [],
        startDate: null,
        endDate: null,
        movieName: '',
        director: '',
        actor: '',
        genre: '', // 默认为空，表示没有选择
        rating: 0.00,
        isPositiveReview: false // 新增字段，选择积极评价
      },
      movieList: [], // 用于存储查询的电影列表
      totalCount: 0, // 用于存储筛选到的电影数量
      genres: [
        'Action', 'Comedy', 'Drama', 'Adventure', 'Horror', 'LGBTQ', 'Romance', 'Arthouse',
        'Suspense', 'Documentary', 'Kids', 'Science Fiction', 'Special Interest', 'International',
        'Animation', 'Sports', 'Fantasy', 'Historical', 'Unscripted', 'Western', 'Military and War',
        'Arts  Entertainment  and Culture', 'Music Videos and Concerts', 'Young Adult Audience',
        'Anime', 'Talk Show and Variety', 'Faith and Spirituality', 'Fitness'
      ], // 下拉框选项
      queryTime1: 0, // 第一个数据库的查询时间
      queryTime2: 0, // 第二个数据库的查询时间
      queryTime3: 0 // 第三个数据库的查询时间
    }
  },
  methods: {
    handleRatingChange() {
      // 确保 rating 被转换为 double 类型
      if (typeof this.filters.rating !== 'number') {
        this.filters.rating = parseFloat(this.filters.rating)
      }
    },
    // 超时函数，500ms后自动返回拒绝的 Promise
    timeoutPromise(ms) {
      return new Promise((_, reject) => setTimeout(() => reject(new Error('请求超时')), ms))
    },

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
    // 禁用今天之后的日期
    disabledDate(date) {
      return date.getTime() > Date.now() // 返回 true 表示该日期不可选
    },
    // 处理多个数据库的接口请求
    async searchMovies() {
      try {
        this.movieList = [] // 清空之前的数据
        this.totalCount = 0

        if (this.filters.Date && this.filters.Date.length === 2) {
          // 将开始时间格式化为 'yyyy-MM-dd' 格式
          this.filters.startDate = this.filters.Date[0].toISOString().split('T')[0]

          // 将结束时间格式化为 'yyyy-MM-dd' 格式
          this.filters.endDate = this.filters.Date[1].toISOString().split('T')[0]
        }

        const startTime1 = Date.now()
        const requests1 = []
        // 根据筛选条件添加请求
        if (this.filters.startDate && this.filters.endDate) {
          requests1.push(this.fetchFromDB1ByDate())
        }
        if (this.filters.movieName) {
          requests1.push(this.fetchFromDB1ByMovieName())
        }
        if (this.filters.director) {
          requests1.push(this.fetchFromDB1ByDirector())
        }
        if (this.filters.actor) {
          requests1.push(this.fetchFromDB1ByActor())
        }
        if (this.filters.genre) {
          requests1.push(this.fetchFromDB1ByGenre())
        }
        if (this.filters.rating > 0) {
          requests1.push(this.fetchFromDB1ByRating())
        }
        if (this.filters.isPositiveReview) {
          requests1.push(this.fetchFromDB1ByPositiveReview())
        }

        // 如果没有任何筛选条件，返回错误提示
        if (requests1.length === 0) {
          this.$message.error('请至少选择一个筛选条件')
          return
        }

        // 为每个请求加上超时机制
        const timeout = 10000 // 设置超时时间为500ms
        const wrappedRequests = requests1.map(request =>
          Promise.race([
            request, // 原始请求
            this.timeoutPromise(timeout) // 超时 Promise
          ])
            .catch(error => {
              console.error('请求失败或超时:', error.message)
              return []
            })
        )

        // 使用 Promise.all 执行所有请求，并捕获可能的错误
        const results1 = await Promise.all(wrappedRequests)

        // 过滤掉空数组
        const nonEmptyResults = results1.filter(result => result && Array.isArray(result) && result.length > 1)

        // 如果有返回数据，取交集
        if (nonEmptyResults.length > 0) {
          // 过滤掉 movieName 为 "*" 或无效数据
          const filteredResults = nonEmptyResults.map(result =>
            result.filter(movie => movie.movieName && movie.movieName !== '*')
          )

          // 获取所有非空请求的交集
          let intersection = filteredResults[0]
          for (let i = 1; i < filteredResults.length; i++) {
            const currentSet = new Set(filteredResults[i].map(movie => movie.movieName))
            intersection = intersection.filter(movie => currentSet.has(movie.movieName))
          }

          // 更新电影列表
          this.movieList = intersection

          // 如果没有交集，给出提示
          if (this.movieList.length === 0) {
            this.$message.warning('未找到符合条件的电影')
          }
        } else {
          this.$message.warning('未找到符合条件的电影')
        }

        const endTime1 = Date.now()
        this.queryTime1 = endTime1 - startTime1

        this.totalCount = this.movieList.length

        const startTime2 = Date.now()
        const requests2 = []
        // 根据筛选条件添加请求
        if (this.filters.startDate && this.filters.endDate) {
          requests2.push(this.fetchFromDB2ByDate())
        }
        if (this.filters.movieName) {
          requests2.push(this.fetchFromDB2ByMovieName())
        }
        if (this.filters.director) {
          requests2.push(this.fetchFromDB2ByDirector())
        }
        if (this.filters.actor) {
          requests2.push(this.fetchFromDB2ByActor())
        }
        if (this.filters.genre) {
          requests2.push(this.fetchFromDB2ByGenre())
        }
        if (this.filters.rating > 0) {
          requests2.push(this.fetchFromDB2ByRating())
        }
        if (this.filters.isPositiveReview) {
          requests2.push(this.fetchFromDB2ByPositiveReview())
        }

        // 如果没有任何筛选条件，返回错误提示
        if (requests2.length === 0) {
          this.$message.error('请至少选择一个筛选条件')
          return
        }
        const wrappedRequests2 = requests2.map(request =>
          Promise.race([
            request, // 原始请求
            this.timeoutPromise(timeout) // 超时 Promise
          ])
            .catch(error => {
              console.error('请求失败或超时:', error.message)
              return []
            })
        )

        // 使用 Promise.all 执行所有请求，并捕获可能的错误
        const results2 = await Promise.all(wrappedRequests2)

        // 过滤掉空数组
        const nonEmptyResults2 = results2.filter(result => result && Array.isArray(result) && result.length > 1)

        // 如果有返回数据，取交集
        if (nonEmptyResults2.length > 0) {
          // 过滤掉 movieName 为 "*" 或无效数据
          const filteredResults2 = nonEmptyResults2.map(result =>
            result.filter(movie => movie.movieName && movie.movieName !== '*')
          )

          // 获取所有非空请求的交集
          let intersection2 = filteredResults2[0]
          for (let i = 1; i < filteredResults2.length; i++) {
            const currentSet = new Set(filteredResults2[i].map(movie => movie.movieName))
            intersection2 = intersection2.filter(movie => currentSet.has(movie.movieName))
          }
        }

        const endTime2 = Date.now()
        this.queryTime2 = endTime2 - startTime2
        // 处理数据库3的请求
        const startTime3 = Date.now()
        const requests3 = []

        if (this.filters.startDate && this.filters.endDate) {
          requests3.push(this.fetchFromDB3ByDate())
        }
        if (this.filters.movieName) {
          requests3.push(this.fetchFromDB3ByMovieName())
        }
        if (this.filters.director) {
          requests3.push(this.fetchFromDB3ByDirector())
        }
        if (this.filters.actor) {
          requests3.push(this.fetchFromDB3ByActor())
        }
        if (this.filters.genre) {
          requests3.push(this.fetchFromDB3ByGenre())
        }
        if (this.filters.rating > 0) {
          requests3.push(this.fetchFromDB3ByRating()) // 修改为数据库3的请求
        }
        if (this.filters.isPositiveReview) {
          requests3.push(this.fetchFromDB3ByPositiveReview())
        }

        if (requests3.length === 0) {
          this.$message.error('请至少选择一个筛选条件')
          return
        }

        const wrappedRequests3 = requests3.map(request =>
          Promise.race([
            request, // 原始请求
            this.timeoutPromise(timeout) // 超时 Promise
          ])
            .catch(error => {
              console.error('请求失败或超时:', error.message)
              return [] // 返回空数组以便后续处理
            })
        )

        const results3 = await Promise.all(wrappedRequests3)

        // 过滤掉空数组
        const nonEmptyResults3 = results3.filter(result => result && Array.isArray(result) && result.length > 1)

        if (nonEmptyResults3.length > 0) {
          const filteredResults3 = nonEmptyResults3.map(result =>
            result.filter(movie => movie.movieName && movie.movieName !== '*')
          )

          let intersection3 = filteredResults3[0]
          for (let i = 1; i < filteredResults3.length; i++) {
            const currentSet = new Set(filteredResults3[i].map(movie => movie.movieName))
            intersection3 = intersection3.filter(movie => currentSet.has(movie.movieName))
          }

          // 这里可以根据需要对数据库3的交集结果做进一步处理
        }

        const endTime3 = Date.now()
        this.queryTime3 = endTime3 - startTime3
      } catch (error) {
        console.error('查询过程中发生错误:', error)
        this.$message.error('查询失败，请稍后重试')
      }
    },
    // 第一个数据库的查询接口方法
    fetchFromDB1ByDate() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/date',
        params: { 'startDate': this.filters.startDate, 'endDate': this.filters.endDate },
        headers: {}
      }
      return axios(config)
        .then(response => {
          if (Array.isArray(response.data)) {
            return response.data
          } else {
            console.error('DB1 返回的数据格式不正确:', response.data)
            return []
          }
        })
        .catch(error => {
          console.log('请求错误:', error)
          return []
        })
    },
    fetchFromDB1ByMovieName() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/movieName',
        params: { 'movieName': this.filters.movieName },
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
    fetchFromDB1ByDirector() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/director',
        params: { 'director': this.filters.director },
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
    fetchFromDB1ByActor() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/actor',
        params: { 'actor': this.filters.actor },
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
    fetchFromDB1ByGenre() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/genre',
        params: { 'genre': this.filters.genre },
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
    fetchFromDB1ByRating() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/rating',
        params: { 'rating': this.filters.rating },
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
    fetchFromDB1ByPositiveReview() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8080/api/db1/movies/positiveReview',
        params: { 'isPositiveReview': this.filters.isPositiveReview },
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
    fetchFromDB2ByDate() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/date',
        params: { 'startDate': this.filters.startDate, 'endDate': this.filters.endDate },
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
    fetchFromDB2ByMovieName() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/movieName',
        params: { 'movieName': this.filters.movieName },
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
    fetchFromDB2ByDirector() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/director',
        params: { 'director': this.filters.director },
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
    fetchFromDB2ByActor() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/actor',
        params: { 'actor': this.filters.actor },
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
    fetchFromDB2ByGenre() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/genre',
        params: { 'genre': this.filters.genre },
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
    fetchFromDB2ByRating() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/rating',
        params: { 'rating': this.filters.rating },
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
    fetchFromDB2ByPositiveReview() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8081/api/db2/movies/positiveReview',
        params: { 'isPositiveReview': this.filters.isPositiveReview },
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
    fetchFromDB3ByDate() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/date',
        params: { 'startDate': this.filters.startDate, 'endDate': this.filters.endDate },
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
    fetchFromDB3ByMovieName() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/movieName',
        params: { 'movieName': this.filters.movieName },
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
    fetchFromDB3ByDirector() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/director',
        params: { 'director': this.filters.director },
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
    fetchFromDB3ByActor() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/actor',
        params: { 'actor': this.filters.actor },
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
    fetchFromDB3ByGenre() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/genre',
        params: { 'genre': this.filters.genre },
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
    fetchFromDB3ByRating() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/rating',
        params: { 'rating': this.filters.rating },
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
    fetchFromDB3ByPositiveReview() {
      var axios = require('axios')
      var config = {
        method: 'get',
        url: 'http://localhost:8082/api/db3/movies/positiveReview',
        params: { 'isPositiveReview': this.filters.isPositiveReview },
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

    // 重置表单方法
    resetForm() {
      // 重置filters中的数据
      this.filters = {
        Date: [],
        startDate: null,
        endDate: null,
        movieName: '',
        director: '',
        actor: '',
        genre: '', // 重置为默认值
        rating: 0,
        isPositiveReview: false // 重置为默认值
      }
      // 清空表格数据
      this.movieList = []
      this.totalCount = null // 重置筛选数量

      // 如果需要重置表单样式，可以调用this.$refs.form.resetFields()
      this.$nextTick(() => {
        this.$refs.form.clearValidate() // 清除所有表单验证状态
      })
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  gap: 50px; /* 控制左右两部分之间的间距 */
  padding: 20px; /* 给整个容器添加内边距，增加空白空间 */
  margin-top: 10px;
  margin-left: 20px;
}


.form-container {
  flex: 1; /* 左侧表单区域占据剩余空间 */
  padding-right: 20px; /* 给表单区域右侧加一些间距，使布局不紧凑 */
}

.table-container {
  flex: 2; /* 右侧表格区域占据更多空间 */
}
.button-container {
  display: flex;
  justify-content: space-between; /* 按钮之间保持间距 */
  gap: 20px; /* 给按钮之间添加间距 */
  margin-bottom: 20px; /* 给按钮区域底部添加间距，避免与表单区域重叠 */
  align-items: center; /* 确保按钮在垂直方向居中对齐 */
}

.form-button {
  flex: 1; /* 设置按钮宽度相等 */
  padding: 10px 0; /* 确保按钮内部间距一致 */
  width: 100%; /* 确保按钮宽度100%占满容器 */
  height: 100%; /* 确保按钮宽度100%占满容器 */
}
.custom-select-width {
  width: 200px; /* 自定义宽度，可以根据需要修改 */
}

.custom-input-width ::part(input) {
  width: 500px; /* 自定义宽度，或设置为固定宽度如 200px */
}

.el-select {
  width: 100%; /* 设置el-select的宽度 */
}

.el-select input {
  width: 100%; /* 设置内部输入框宽度 */
}

.el-form-item {
  margin-bottom: 10px; /* 给每个表单项之间增加一些垂直间距 */
}

/* ��el-radio之间有间距 */
.radio-group .radio-option {
  margin-right: 30px !important; /* 强制应用样式，使"是"和"否"之间有足够间距 */
}

/* 显示筛选到的电影数量 */
.total-count {
  text-align: center;
  background-color: #f5f5f5;
  padding: 10px;
  margin-top: 10px;
  font-size: 16px;
  font-weight: bold;
  color: #333;
  border-radius: 5px;
}

.el-dialog {
  width: 60% !important; /* 控制弹出框的宽度 */
  max-width: 800px;      /* 最大宽度 */
  height: 20% !important; /* 控制高度，注意这里设置为百分比时会根据父容器的高度来计算 */
  max-height: 400px;     /* 最大高度，防止高度过大 */
}

.el-dialog__body {
  overflow-y: auto;      /* 内容超出时显示滚动条 */
}
</style>