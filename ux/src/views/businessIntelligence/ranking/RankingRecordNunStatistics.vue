<template>
  <div
    v-loading="loading"
    class="main-container">
    <filtrate-handle-view
      :show-user-select="false"
      class="filtrate-bar"
      module-type="ranking"
      @load="loading=true"
      @change="getDataList"/>
    <div class="content">
      <div class="content-title">跟进次数排行（按拜访时间统计）</div>
      <div
        v-empty="list.length === 0"
        class="axis-content"
        xs-empty-text="暂无排行">
        <div id="axismain"/>
      </div>
      <div class="table-content">
        <el-table
          :data="list"
          height="400"
          stripe
          border
          highlight-current-row>
          <el-table-column
            align="center"
            header-align="center"
            show-overflow-tooltip
            label="公司总排名">
            <template slot-scope="scope">
              {{ scope.$index + 1 }}
            </template>
          </el-table-column>
          <el-table-column
            v-for="(item, index) in fieldList"
            :key="index"
            :prop="item.field"
            :label="item.name"
            align="center"
            header-align="center"
            show-overflow-tooltip/>
        </el-table>
      </div>
    </div>
  </div>
</template>

<script>
import rankingMixins from '../mixins/ranking'
import echarts from 'echarts'
import { biRankingRecordNunAPI } from '@/api/businessIntelligence/ranking'

export default {
  /** 跟进次数排行 */
  name: 'RankingRecordNunStatistics',
  mixins: [rankingMixins],
  data() {
    return {}
  },
  computed: {},
  mounted() {
    this.fieldList = [
      { field: 'user_name', name: '员工' },
      { field: 'structure_name', name: '部门' },
      { field: 'count', name: '跟进次数（次）' }
    ]
    this.initAxis()
  },
  methods: {
    getDataList(params) {
      this.loading = true
      biRankingRecordNunAPI(params)
        .then(res => {
          this.loading = false
          this.list = res.data || []

          const showData = []
          const yAxis = []

          const rankingIndex = res.data.length > 10 ? 10 : res.data.length
          for (let index = 0; index < rankingIndex; index++) {
            const element = res.data[index]
            showData.splice(0, 0, parseFloat(element.count))
            yAxis.splice(0, 0, element.user_name)
          }
          this.axisOption.yAxis[0].data = yAxis
          this.axisOption.series[0].data = showData
          this.axisChart.setOption(this.axisOption, true)
        })
        .catch(() => {
          this.loading = false
        })
    },
    /** 柱状图 */
    initAxis() {
      this.axisChart = echarts.init(document.getElementById('axismain'))
      this.axisOption.tooltip.formatter = '{b} : {c}次'
      this.axisOption.xAxis[0].name = '（次）'
      this.axisChart.setOption(this.axisOption, true)
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import '../styles/detail.scss';
</style>
