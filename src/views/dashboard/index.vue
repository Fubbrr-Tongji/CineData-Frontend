<template>
  <div class="home-page">
    <div class="cine-data-intro">
      <div class="intro-header">
        <h1>欢迎使用 CineData 系统</h1>
        <p>管理和查询电影数据、演员合作关系及更多。</p>
      </div>
      <div class="intro-body">
        <h2>系统介绍</h2>
        <p>
          CineData 是一个强大的电影数据管理平台，旨在为用户提供全面的电影信息，
          包括演员、导演、合作次数、电影风格等。通过数据的可视化展示，用户能够
          轻松分析电影行业中的各种趋势。
        </p>

        <h2>如何开始</h2>
        <p>
          1. 登录 CineData 系统。<br>
          2. 选择你感兴趣的电影或演员数据。<br>
          3. 使用筛选和分析功能来查看详细信息。<br>
          4. 利用可视化图表查看趋势。
        </p>
      </div>
    </div>

    <div class="actions">
      <!-- 可添加其他页面操作或按钮 -->
      <button class="get-started-btn">开始使用</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomePage'
}
</script>

<style scoped>
.home-page {
  padding: 40px;
  background-color: #f9f9f9;
}

.cine-data-intro {
  background-color: #ffffff;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 30px;
  margin-bottom: 40px;
}

.intro-header {
  text-align: center;
  margin-bottom: 30px;
}

.intro-header h1 {
  font-size: 2.5em;
  color: #333;
}

.intro-header p {
  font-size: 1.2em;
  color: #666;
}

.intro-body h2 {
  font-size: 1.8em;
  color: #333;
  margin-top: 30px;
}

.intro-body p,
.intro-body ul {
  font-size: 1.1em;
  color: #555;
}

.intro-body ul {
  list-style: disc;
  margin-left: 20px;
}

.intro-body li {
  margin-bottom: 10px;
}

.actions {
  text-align: center;
}

.get-started-btn {
  padding: 15px 30px;
  font-size: 1.1em;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.get-started-btn:hover {
  background-color: #0056b3;
}
</style>
