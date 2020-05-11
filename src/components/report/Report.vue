<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!--2.创建容器-->
      <div id="main" style="width: 600px;height:400px;"></div>
    </el-card>
  </div>
</template>
<script>
// 1. 导入ECharts 数据可视化
// 在 webpack 中使用 ECharts https://echarts.apache.org/zh/tutorial.html#5%20%E5%88%86%E9%92%9F%E4%B8%8A%E6%89%8B%20ECharts
import echarts from 'echarts'
import _ from 'lodash'
export default {
  data () {
    return {
      // 需要合并的数据 利用深拷贝
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created () {
  },
  //  执行mounted代表 页面上的元素，已经被渲染完毕了（mounted 生命周期函数等DOM加载完成后在执行 数据可视化表需要等DOM完成）
  async mounted () {
    // 3.基于准备好的dom，初始化echarts实例
    var myChart = echarts.init(document.getElementById('main'))
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) { return this.$message.error('获取折线图数据失败!') }
    // 4.准备数据和配置项
    // var option = {
    //   title: {
    //     text: 'ECharts 入门示例'
    //   },
    //   tooltip: {},
    //   legend: {
    //     data: ['销量']
    //   },
    //   xAxis: {
    //     data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
    //   },
    //   yAxis: {},
    //   series: [{
    //     name: '销量',
    //     type: 'bar',
    //     data: [5, 20, 36, 10, 10, 20]
    //   }]
    // }
    // 5.展示数据
    const result = _.merge(res.data, this.options)
    // 展示数据
    myChart.setOption(result)
  },
  methods: {}
}
</script>
<style lang="less" scoped>
</style>
