<template>
  <div class="container">
    <!-- 右侧表格区域 -->
    <div class="table-container">
      <el-table :data="movieList" style="width: 100%">
        <el-table-column prop="actorName" label="演员" :width="500"></el-table-column>
        <el-table-column prop="directorName" label="导演" :width="500"></el-table-column>
        <el-table-column prop="cooperationCount" label="合作次数" :width="300"></el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      movieList: [], // 用于存储查询的演员和导演合作数据
    };
  },
  methods: {
    async fetchMovies() {
      this.movieList = [];    
      let requests1 = this.fetchMoviesFromMySQL();
      const results1 = await requests1;
      this.movieList = this.movieList.concat(results1);

      let requests2 = this.fetchMoviesFromHive();
      const results2 = await requests2;
      this.movieList = this.movieList.concat(results2);

      let requests3 = this.fetchMoviesFromNeo4j();
      const results3 = await requests3;
      this.movieList = this.movieList.concat(results3);
    },

    fetchMoviesFromMySQL() {
      return this.$axios.get('/api/db1/movies/cooperation1').then(res => res.data);
    },

    fetchMoviesFromHive() {
      return this.$axios.get('/api/db2/movies/cooperation1').then(res => res.data);
    },

    fetchMoviesFromNeo4j() {
      return this.$axios.get('/api/db3/movies/cooperation1').then(res => res.data);
    },
  },
  created() {
    this.fetchMovies();
  }
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
