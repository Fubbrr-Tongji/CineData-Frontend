<template>
  <div class="container">
    <!-- 左侧输入区域 -->
    <div class="form-container">
      <el-form :model="filters" label-width="100px" ref="form">
        <el-form-item label="开始时间">
          <el-date-picker v-model="filters.startDate" type="daterange" range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期" :disabled-date="disabledDate"></el-date-picker>
        </el-form-item>

        <el-form-item label="电影名">
          <el-input v-model="filters.movieName" placeholder="请输入电影名"></el-input>
        </el-form-item>

        <el-form-item label="导演">
          <el-input v-model="filters.director" placeholder="请输入导演名"></el-input>
        </el-form-item>

        <el-form-item label="演员">
          <el-input v-model="filters.actor" placeholder="请输入演员名"></el-input>
        </el-form-item>

        <!-- 电影风格下拉框 -->
        <el-form-item label="电影风格">
          <el-select v-model="filters.genre" placeholder="请选择电影风格" :input-width="'100%'">
            <el-option label="请选择" value=""></el-option>
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
          ></el-input-number>
        </el-form-item>

        <!-- 选择积极评价 -->
        <el-form-item label="筛选积极评价">
          <el-radio-group v-model="filters.isPositiveReview" class="radio-group">
            <el-radio :label="true" class="radio-option">是</el-radio>
            <el-radio :label="false" class="radio-option">否</el-radio>
          </el-radio-group>
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
          MySQL 查询耗时： {{ queryTime1 }} 
        </div>
        <div class="total-count">
          Hive 查询耗时： {{ queryTime2 }} 
        </div>
        <div class="total-count">
          Neo4j 查询耗时： {{ queryTime3 }} 
        </div>
        <!-- 运行时间对比按钮 -->
        <el-button @click="showComparisonDialog" class="form-button">运行时间对比</el-button>
        <!-- 弹出框 -->
    <el-dialog
      :visible.sync="dialogVisible"
      title="运行时间对比"
      width="500px"
      @close="handleClose"
    >
      <div>
        <!-- 你可以在这里放置对比的内容 -->
        <p>这是运行时间对比内容</p>
        <el-button type="primary" @click="handleOk">确认</el-button>
        <el-button @click="handleClose">取消</el-button>
      </div>
    </el-dialog>
      </el-form>
    </div>
    <!-- 右侧表格区域 -->
    <div class="table-container">
      <el-table :data="movieList" style="width: 100%">
        <el-table-column prop="movieName" label="电影名" :width="400"></el-table-column>
        <el-table-column prop="releaseDate" label="上映时间" :min-width="150"></el-table-column>
        <el-table-column prop="genre" label="电影风格" :min-width="150"></el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 控制弹出框显示/隐藏
      dialogVisible: false,
      filters: {
        startDate: [null, null],
        endDate: null,
        movieName: '',
        director: '',
        actor: '',
        genre: '', // 默认为空，表示没有选择
        rating: 0,
        isPositiveReview: false, // 新增字段，选择积极评价
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
      queryTime1: null, // 第一个数据库的查询时间
      queryTime2: null, // 第二个数据库的查询时间
      queryTime3: null, // 第三个数据库的查询时间
    };
  },
  methods: {
    // 显示弹出框
    showComparisonDialog() {
      this.dialogVisible = true;
    },
    // 关闭弹出框
    handleClose() {
      this.dialogVisible = false;
    },
    // 确认按钮点击处理
    handleOk() {
      // 在这里处理确认逻辑
      console.log("确认对比");
      this.dialogVisible = false; // 关闭弹出框
    },
    // 禁用今天之后的日期
    disabledDate(date) {
      return date.getTime() > Date.now();  // 返回 true 表示该日期不可选
    },
    // 处理多个数据库的接口请求
    async searchMovies() {
      // 第一个数据库查询
      const startTime1 = Date.now(); // 记录第一个数据库查询开始时间
      let requests1 = this.generateRequests(1); // 生成第一个数据库的请求
      const results1 = await Promise.all(requests1); // 执行所有请求
      this.movieList = results1.flat(); // 合并所有结果
      this.totalCount = this.movieList.length; // 更新筛选到的电影数量
      const endTime1 = Date.now(); // 记录第一个数据库查询结束时间
      this.queryTime1 = endTime1 - startTime1; // 计算第一个数据库查询耗时

      // 第二个数据库查询（只记录查询时间，不合并数据）
      const startTime2 = Date.now(); // 记录第二个数据库查询开始时间
      let requests2 = this.generateRequests(2); // 生成第二个数据库的请求
      await Promise.all(requests2); // 执行所有请求（不需要结果）
      const endTime2 = Date.now(); // 记录第二个数据库查询结束时间
      this.queryTime2 = endTime2 - startTime2; // 计算第二个数据库查询耗时

      // 第三个数据库查询（只记录查询时间，不合并数据）
      const startTime3 = Date.now(); // 记录第三个数据库查询开始时间
      let requests3 = this.generateRequests(3); // 生成第三个数据库的请求
      await Promise.all(requests3); // 执行所有请求（不需要结果）
      const endTime3 = Date.now(); // 记录第三个数据库查询结束时间
      this.queryTime3 = endTime3 - startTime3; // 计算第三个数据库查询耗时
    },

    // 根据不同的数据库生成请求
    generateRequests(databaseId) {
      let requests = [];
      switch (databaseId) {
        case 1:
          // 第一个数据库的查询接口
          if (this.filters.startDate.length > 0 || this.filters.endDate) {
            requests.push(this.fetchFromDB1ByDate());
          }
          if (this.filters.movieName) {
            requests.push(this.fetchFromDB1ByMovieName());
          }
          if (this.filters.director) {
            requests.push(this.fetchFromDB1ByDirector());
          }
          if (this.filters.actor) {
            requests.push(this.fetchFromDB1ByActor());
          }
          if (this.filters.genre) {
            requests.push(this.fetchFromDB1ByGenre());
          }
          if (this.filters.rating > 0) {
            requests.push(this.fetchFromDB1ByRating());
          }
          if (this.filters.isPositiveReview !== undefined) {
            requests.push(this.fetchFromDB1ByPositiveReview());
          }
          break;

        case 2:
          // 第二个数据库的查询接口
          if (this.filters.startDate.length > 0 || this.filters.endDate) {
            requests.push(this.fetchFromDB2ByDate());
          }
          if (this.filters.movieName) {
            requests.push(this.fetchFromDB2ByMovieName());
          }
          if (this.filters.director) {
            requests.push(this.fetchFromDB2ByDirector());
          }
          if (this.filters.actor) {
            requests.push(this.fetchFromDB2ByActor());
          }
          if (this.filters.genre) {
            requests.push(this.fetchFromDB2ByGenre());
          }
          if (this.filters.rating > 0) {
            requests.push(this.fetchFromDB2ByRating());
          }
          if (this.filters.isPositiveReview !== undefined) {
            requests.push(this.fetchFromDB2ByPositiveReview());
          }
          break;

        case 3:
          // 第三个数据库的查询接口
          if (this.filters.startDate.length > 0 || this.filters.endDate) {
            requests.push(this.fetchFromDB3ByDate());
          }
          if (this.filters.movieName) {
            requests.push(this.fetchFromDB3ByMovieName());
          }
          if (this.filters.director) {
            requests.push(this.fetchFromDB3ByDirector());
          }
          if (this.filters.actor) {
            requests.push(this.fetchFromDB3ByActor());
          }
          if (this.filters.genre) {
            requests.push(this.fetchFromDB3ByGenre());
          }
          if (this.filters.rating > 0) {
            requests.push(this.fetchFromDB3ByRating());
          }
          if (this.filters.isPositiveReview !== undefined) {
            requests.push(this.fetchFromDB3ByPositiveReview());
          }
          break;

        default:
          break;
      }
      return requests;
    },

    // 第一个数据库的查询接口方法
    fetchFromDB1ByDate() {
      const params = {
        startDate: this.filters.startDate,
        endDate: this.filters.endDate,
      };
      return this.$axios.get('/api/db1/movies/date', { params }).then(res => res.data);
    },
    fetchFromDB1ByMovieName() {
      const params = {
        movieName: this.filters.movieName,
      };
      return this.$axios.get('/api/db1/movies/movieName', { params }).then(res => res.data);
    },
    fetchFromDB1ByDirector() {
      const params = {
        director: this.filters.director,
      };
      return this.$axios.get('/api/db1/movies/director', { params }).then(res => res.data);
    },
    fetchFromDB1ByActor() {
      const params = {
        actor: this.filters.actor,
      };
      return this.$axios.get('/api/db1/movies/actor', { params }).then(res => res.data);
    },
    fetchFromDB1ByGenre() {
      const params = {
        genre: this.filters.genre,
      };
      return this.$axios.get('/api/db1/movies/genre', { params }).then(res => res.data);
    },
    fetchFromDB1ByRating() {
      const params = {
        rating: this.filters.rating,
      };
      return this.$axios.get('/api/db1/movies/rating', { params }).then(res => res.data);
    },
    fetchFromDB1ByPositiveReview() {
      const params = {
        isPositiveReview: this.filters.isPositiveReview,
      };
      return this.$axios.get('/api/db1/movies/positiveReview', { params }).then(res => res.data);
    },

    // 第二个数据库的查询接口方法
    fetchFromDB2ByDate() {
      const params = {
        startDate: this.filters.startDate,
        endDate: this.filters.endDate,
      };
      return this.$axios.get('/api/db2/movies/date', { params }).then(res => res.data);
    },
    fetchFromDB2ByMovieName() {
      const params = {
        movieName: this.filters.movieName,
      };
      return this.$axios.get('/api/db2/movies/movieName', { params }).then(res => res.data);
    },
    fetchFromDB2ByDirector() {
      const params = {
        director: this.filters.director,
      };
      return this.$axios.get('/api/db2/movies/director', { params }).then(res => res.data);
    },
    fetchFromDB2ByActor() {
      const params = {
        actor: this.filters.actor,
      };
      return this.$axios.get('/api/db2/movies/actor', { params }).then(res => res.data);
    },
    fetchFromDB2ByGenre() {
      const params = {
        genre: this.filters.genre,
      };
      return this.$axios.get('/api/db2/movies/genre', { params }).then(res => res.data);
    },
    fetchFromDB2ByRating() {
      const params = {
        rating: this.filters.rating,
      };
      return this.$axios.get('/api/db2/movies/rating', { params }).then(res => res.data);
    },
    fetchFromDB2ByPositiveReview() {
      const params = {
        isPositiveReview: this.filters.isPositiveReview,
      };
      return this.$axios.get('/api/db2/movies/positiveReview', { params }).then(res => res.data);
    },

    // 第三个数据库的查询接口方法
    fetchFromDB3ByDate() {
      const params = {
        startDate: this.filters.startDate,
        endDate: this.filters.endDate,
      };
      return this.$axios.get('/api/db3/movies/date', { params }).then(res => res.data);
    },
    fetchFromDB3ByMovieName() {
      const params = {
        movieName: this.filters.movieName,
      };
      return this.$axios.get('/api/db3/movies/movieName', { params }).then(res => res.data);
    },
    fetchFromDB3ByDirector() {
      const params = {
        director: this.filters.director,
      };
      return this.$axios.get('/api/db3/movies/director', { params }).then(res => res.data);
    },
    fetchFromDB3ByActor() {
      const params = {
        actor: this.filters.actor,
      };
      return this.$axios.get('/api/db3/movies/actor', { params }).then(res => res.data);
    },
    fetchFromDB3ByGenre() {
      const params = {
        genre: this.filters.genre,
      };
      return this.$axios.get('/api/db3/movies/genre', { params }).then(res => res.data);
    },
    fetchFromDB3ByRating() {
      const params = {
        rating: this.filters.rating,
      };
      return this.$axios.get('/api/db3/movies/rating', { params }).then(res => res.data);
    },
    fetchFromDB3ByPositiveReview() {
      const params = {
        isPositiveReview: this.filters.isPositiveReview,
      };
      return this.$axios.get('/api/db3/movies/positiveReview', { params }).then(res => res.data);
    },

    // 重置表单方法
    resetForm() {
      // 重置filters中的数据
      this.filters = {
        startDate: [],
        endDate: null,
        movieName: '',
        director: '',
        actor: '',
        genre: '', // 重置为默认值
        rating: 0,
        isPositiveReview: false, // 重置为默认值
      };
      // 清空表格数据
      this.movieList = [];
      this.totalCount = null; // 重置筛选数量

      // 如果需要重置表单样式，可以调用this.$refs.form.resetFields()
      this.$nextTick(() => {
        this.$refs.form.clearValidate(); // 清除所有表单验证状态
      });
    },
  },
};
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

/* 让el-radio之间有间距 */
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
</style>