<template>
  <div>
      <!--面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
      <!--  卡片视图  -->
    <el-card class="box-card">
      <!--  添加分类按钮   -->
      <el-row class="Bottom">
        <el-col>
        <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格内容-->
      <tree-table :data="cateList" :columns="columns"
                  :selection-type="false" :expand-type="false"
                  show-index border>
        <!-- 是否有效 -->
         <template slot="isOk" scope="scope"><!--v-slot="scope"/ <pre>{{scope}}</pre>-->
           <!-- <template slot="$expand" scope="scope">
        {{ `My name is ${scope.row.name},
           this rowIndex is ${scope.rowIndex}.`
         }}
      </template>
      <template slot="likes" scope="scope">
        {{ scope.row.likes.join(',') }}
      </template>-->
           <i style="color:lightgreen" class="el-icon-success" v-if="scope.row.cat_deleted === false"></i>
           <i style="color:red" class="el-icon-error" v-else></i>
         </template>
        <!-- 排序模板 -->
        <template slot="order" scope="scope"> <!--{{scope.row}} scope="scope"-->
          <el-tag size="small" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="small" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="small" v-else>三级</el-tag>
        </template>
        <!-- 操作模板 -->
        <template slot="opt">
          <el-row>
          <el-button type="primary" icon="el-icon-edit" size="small"></el-button>
          <el-button @click="dele" type="danger" icon="el-icon-delete" size="small"></el-button>
          </el-row>
        </template>
      </tree-table>
      <!--   分页   -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[2, 3, 5, 10]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <el-dialog title="编辑商品" :visible.sync="shangpindialogVisible"
      width="30%">
      <span>这是一段信息</span>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
  </span>
    </el-dialog>
    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible"
      width="30%" @close="addCateDialogClosed">
      <!-- 添加分类的表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name">
          </el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!--:options="parentCateList" 数据源-->
          <el-cascader v-model="selectedKeys" :options="parentCateList"
            :props="cascaderProps" @change="parentCateChanged" clearable>
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click=addCate>确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 查询条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示，当前这一列使用的模板名称
          template: 'isOk'
        },
        {
          label: '排序',
          // 将当前列定义为模板列
          type: 'template',
          // 当前这一列使用的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 将当前列定义为模板列
          type: 'template',
          // 当前这一列使用的模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 将要添加的分类的名称
        // 父级分类的Id
        cat_pid: 0,
        // 分类的等级，默认要添加的是1级分类
        cat_level: 0
      },
      // 添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [{ required: true, message: '请输入分类的名称', trigger: 'blur' }]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: 'true',
        value: 'cat_id', //  指定选项的--值--为选项对象的某个属性值
        label: 'cat_name', //  指定选项--标签--为选项对象的某个属性值
        children: 'children' // 指定选项的--子选项--为选项对象的某个属性值
      },
      // 选中的父级分类的Id数组
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.querInfo })
      if (res.meta.status !== 200) { return this.$message.error('获取商品列表数据失败！') }
      // console.log(res.data)
      this.cateList = res.data.result// status: 200
      // 为总数据条数赋值
      this.total = res.data.total
    },
    // 点击删除按钮显示提示框
    dele () {
      this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
    },
    // 监听pagesize 改变的事件
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog () {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      // 再展示出对话框
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) { return this.$message.error('获取父级分类数据失败！') }
      console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChanged () {
      console.log(this.selectedKeys)
      // 如果 selectedKeys 数组中的 length 大于0，证明选中的父级分类
      // 反之，就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮，添加新的分类
    addCate () {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    //  监听对话框的关闭事件，重置表单数据
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>
<style lang="less" scoped>
  .Bottom {
    margin-bottom: 15px
  }
  .el-cascader {
    width: 100%;
  }
</style>
