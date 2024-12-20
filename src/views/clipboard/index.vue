<template>
    <div class="container">
      <!-- 左侧输入区域 -->
      <div class="form-container">
        <el-form :model="filters" label-width="100px" ref="form">
    <!-- 电影风格下拉框 -->
    <el-form-item label="电影风格">
          <el-select v-model="filters.genre" placeholder="请选择电影风格" :input-width="'100%'">
            <el-option label="请选择" value=""></el-option>
            <el-option v-for="item in genres" :key="item" :label="item" :value="item" />
          </el-select>
        </el-form-item>
          <div class="button-container">
            <el-button type="primary" @click="searchMovies" class="form-button">查询</el-button>
            <!-- 重置按钮 -->
            <el-button @click="resetForm" class="form-button">重置</el-button>
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
          <el-button @click="showComparisonDialog" class="form-button">运行时间对比</el-button>
        </el-form>
      </div>
  
      <!-- 右侧表格区域 -->
      <div class="table-container">
        <el-table :data="movieList" style="width: 100%">
          <el-table-column prop="actorName1" label="演员1" :width="350"></el-table-column>
          <el-table-column prop="actorName2" label="演员2" :width="350"></el-table-column>
          <el-table-column prop="commetcount" label="评论数量" :width="200"></el-table-column>
        </el-table>
      </div>
  
      <!-- 弹出框 -->
      <el-dialog :visible.sync="dialogVisible" title="运行时间对比" width="60%" @close="handleClose">
        <div class="charts-container">
          <!-- 折线图 -->
          <div ref="lineChart" class="chart" style="height: 300px;"></div>
  
          <!-- 饼状图 -->
          <div ref="pieChart" class="chart" style="height: 300px;"></div>
        </div>
        <div slot="footer">
          <el-button @click="handleClose">关闭</el-button>
        </div>
      </el-dialog>
    </div>
  </template>
  
  <script>
  import * as echarts from 'echarts';
  export default {
    data() {
      return {
        // 控制弹出框显示/隐藏
        dialogVisible: false,
        filters: {
            genre: '',  // 初始化 genre 字段
        },
        genres: [
        'Action', 'Comedy', 'Drama', 'Adventure', 'Horror', 'LGBTQ', 'Romance', 'Arthouse', 
        'Suspense', 'Documentary', 'Kids', 'Science Fiction', 'Special Interest', 'International', 
        'Animation', 'Sports', 'Fantasy', 'Historical', 'Unscripted', 'Western', 'Military and War', 
        'Arts  Entertainment  and Culture', 'Music Videos and Concerts', 'Young Adult Audience', 
        'Anime', 'Talk Show and Variety', 'Faith and Spirituality', 'Fitness'
      ], // 下拉框选项
        movieList: [], // 用于存储查询的电影列表
        totalCount: 0, // 用于存储筛选到的电影数量
        queryTime1: 0, // MySQL查询时间
        queryTime2: 0, // Hive查询时间
        queryTime3: 0, // Neo4j查询时间
      };
    },
    methods: {
      // 显示弹出框
      showComparisonDialog() {
        this.dialogVisible = true;
        this.$nextTick(() => {
          this.initCharts();
        });
      },
      // 初始化图表
      initCharts() {
        // 初始化折线图
        this.initLineChart();
        // 初始化饼状图
        this.initPieChart();
      },
      // 折线图初始化
      initLineChart() {
        const lineChart = echarts.init(this.$refs.lineChart);
        const lineOptions = {
          title: {
            text: '折线图运行时间对比',
          },
          tooltip: {
            trigger: 'axis',
          },
          legend: {
            data: ['运行时间'],
          },
          xAxis: {
            type: 'category',
            data: ['MySQL', 'Hive', 'Neo4j'],
          },
          yAxis: {
            type: 'value',
          },
          series: [
            {
              name: '运行时间/ms',
              type: 'line',
              data: [this.queryTime1, this.queryTime2, this.queryTime3],
            },
          ],
        };
        lineChart.setOption(lineOptions);
      },
      // 饼状图初始化
      initPieChart() {
        const pieChart = echarts.init(this.$refs.pieChart);
        const pieOptions = {
          title: {
            text: '饼图运行时间对比',
            left: 'center',
          },
          tooltip: {
            trigger: 'item',
            formatter: '{a} <br/>{b}: {c} ({d}%)',
          },
          legend: {
            orient: 'vertical',
            left: 'left',
            data: ['MySQL', 'Hive', 'Neo4j'],
          },
          series: [
            {
              name: '运行时间/ms',
              type: 'pie',
              radius: '50%',
              data: [
                { value: this.queryTime1, name: 'MySQL' },
                { value: this.queryTime2, name: 'Hive' },
                { value: this.queryTime3, name: 'Neo4j' },
              ],
            },
          ],
        };
        pieChart.setOption(pieOptions);
      },
      // 关闭弹出框
      handleClose() {
        this.dialogVisible = false;
      },
      // 确认按钮点击处理
      handleOk() {
        this.dialogVisible = false;
      },
      // 处理多个数据库的接口请求
      async searchMovies() {
        // 清空现有电影数据
        this.movieList = [];
        this.totalCount = 0;
  
        const startTime1 = Date.now();
        let requests1 = this.fetchMoviesFromMySQL();
        const results1 = await requests1;
        this.movieList = this.movieList.concat(results1);
        this.totalCount = this.movieList.length;
        const endTime1 = Date.now();
        this.queryTime1 = endTime1 - startTime1; // 保存MySQL查询时间
  
        const startTime2 = Date.now();
        let requests2 = this.fetchMoviesFromHive();
        const results2 = await requests2;
        this.movieList = this.movieList.concat(results2);
        const endTime2 = Date.now();
        this.queryTime2 = endTime2 - startTime2; // 保存Hive查询时间
  
        const startTime3 = Date.now();
        let requests3 = this.fetchMoviesFromNeo4j();
        const results3 = await requests3;
        this.movieList = this.movieList.concat(results3);
        const endTime3 = Date.now();
        this.queryTime3 = endTime3 - startTime3; // 保存Neo4j查询时间
      },
  
      // 获取MySQL数据库的电影数据
      fetchMoviesFromMySQL() {
        const params = {
            genre: this.filters.genre, // 添加风格过滤条件
        };
        return this.$axios.get('/api/db1/movies/movieStyle1', { params }).then((res) => res.data);
      },
  
      // 获取Hive数据库的电影数据
      fetchMoviesFromHive() {
        const params = {
            genre: this.filters.genre, // 添加风格过滤条件
        };
        return this.$axios.get('/api/db2/movies/movieStyle1', { params }).then((res) => res.data);
      },
  
      // 获取Neo4j数据库的电影数据
      fetchMoviesFromNeo4j() {
        const params = {
            genre: this.filters.genre, // 添加风格过滤条件
        };
        return this.$axios.get('/api/db3/movies/movieStyle1', { params }).then((res) => res.data);
      },
  
      // 重置表单方法
      resetForm() {
        this.filters.genre = '';
        this.movieList = [];
        this.totalCount = 0;
        this.queryTime1 = 0;
        this.queryTime2 = 0;
        this.queryTime3 = 0;
      },
    },
  };
  </script>
  
  <style scoped>
  .container {
    display: flex;
    gap: 50px;
    padding: 20px;
    margin-top: 10px;
    margin-left: 20px;
  }
  
  .form-container {
    flex: 4; /* 增加左侧区域的宽度 */
    padding-right: 20px;
  }
  
  .table-container {
    flex: 2;
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
  
  .el-table {
    width: 100%;
  }
  
  .el-table-column {
    padding: 10px;
  }
  
  .el-dialog {
    width: 60% !important;
    max-width: 800px;
    height: 20% !important;
    max-height: 400px;
  }
  </style>
  