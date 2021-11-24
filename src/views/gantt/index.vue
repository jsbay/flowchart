<!--
 * @FilePath src/views/gantt/index.vue
 * @Created Bay丶<baizhanying@autobio.com.cn> 2021-11-11 11:11:29
 * @Modified Bay丶<baizhanying@autobio.com.cn> 2021-11-11 11:19:03
 * @Description
-->

<template>
  <div class="dashboard-container">
    <div id="chart-wrap">
      <gantt-chart />
    </div>
  </div>
</template>

<script>
import GanttChart from './components/GanttChart'

export default {
  name: 'Gantt',
  components: { GanttChart },
  data() {
    return {

    }
  },
  mounted() {

  },
  methods: {

  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}

#chart-wrap {
  width: 100%;
  height: calc(100vh - 144px);
}
</style>
