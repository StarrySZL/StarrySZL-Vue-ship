<template>
  <div>
    <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片-->
    <el-card class="box-card">
      <el-row :gutter="20">
      <!--搜索区域-->
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
          <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
      </el-input>
        </el-col>
      <!--添加商品按钮-->
        <el-col :span="4">
          <el-button type="primary" @click="goAddpage">添加商品</el-button>
        </el-col>
      </el-row>
      <!--table表格区域-->
      <el-table :data="goodslist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column width="70px" label="商品价格(元)" prop="goods_price"></el-table-column>
        <el-table-column width="70px" label="商品重量" prop="goods_weight"></el-table-column>
        <el-table-column width="140px" label="创建时间" prop="add_time">
          <template v-slot="scope">
            {{(scope.row.add_time) | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column width="130px" label="操作">
          <template v-slot="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
          <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 4, 8, 12]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total" background><!-- background 背景 -->
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 8
      },
      // 商品列表
      goodslist: [],
      // 总数据条数
      total: 0
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) { return this.$message.error('获取商品列表失败!') }
      this.$message.success('获取商品列表成功！')
      console.log(res.data)
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    async getGoodList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) { return this.$message.error('获取商品列表失败!') }
      // this.$message.success('获取商品列表成功！')
      console.log(res.data)
      this.goodslist = res.data.goods
      this.total = res.data.total
    },
    // 改变分页数量
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodList()
    },
    // 显示第几页
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodList()
    },
    // 删除提示框
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') { return this.$message.info('已取消删除!') }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) { return this.$message.error('删除失败！') }
      this.$message.success('删除成功!')
      this.getGoodList()
    },
    // 点击添加按钮事件
    goAddpage () {
      // 调用路由导航函数跳转到指定页面
      this.$router.push('/goods/add')
    }
  }
}
</script>
<style lang="less" scoped>
</style>
