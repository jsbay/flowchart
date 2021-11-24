<template>
  <div class="dashboard-container">
    <div class="dashboard-text">username: {{ username }}</div>
    <div class="dashboard-text">nickname: {{ nickname }}</div>
    <div id="chart-wrap">
      <flow-chart />
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import FlowChart from './components/FlowChart'

export default {
  name: 'Dashboard',
  components: { FlowChart },
  computed: {
    ...mapGetters([
      'username',
      'nickname'
    ])
  },
  methods: {
    onSubmit() {
      this.save()
      console.log(this.chartData)
      return false
    }
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
  height: calc(100vh - 236px);
}
</style>
