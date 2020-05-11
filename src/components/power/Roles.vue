<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
    <!-- 内容区域 -->
    <el-table :data="rolelist" border stripe>
      <!-- 展开列 expand展开列表 -->
      <el-table-column type="expand">
        <template v-slot="scope">
          <el-row :class="['bdBottom', i1===0 ?'bdTop':'', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
            <!-- 渲染一级权限 -->
            <el-col :span="5">
              <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
              <i class="el-icon-caret-right"></i>
            </el-col>
            <!-- 渲染二级权限和三级权限 -->
            <el-col :span="19">
              <!-- 通过for循环嵌套渲染二级权限 -->
              <el-row  :class="[i2 ===0?'':'bdTop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                <el-col :span="6">
                  <el-tag @close="removeRightById(scope.row, item2.id)" closable type="success">{{item2.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="18">
                  <el-tag @close="removeRightById(scope.row, item3.id)" closable type="warning" v-for="item3 in item2.children" :key="item3.id">
                    {{item3.authName}}
                  </el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          <!--<pre>{{scope.row}}</pre>-->
        </template>
      </el-table-column>
      <!-- 索引列 -->
      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column prop="roleName" label="角色名称"></el-table-column>
      <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
      <el-table-column label="操作" width="295px">
        <template v-slot="scope">
          <el-row>
            <el-button size="mini" icon="el-icon-edit" type="primary" round>编辑</el-button>
            <el-button size="mini" icon="el-icon-delete" type="danger" round>删除</el-button>
            <el-button size="mini" icon="el-icon-setting" type="warning"
                       round @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </el-row>
        </template>
      </el-table-column>
    </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible"
      width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps"
               show-checkbox node-key="id" default-expand-all
               :default-checked-keys="defKeys" ref="treeRef">
      </el-tree>
      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolelist: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点ID值数组
      defKeys: [],
      // 当前即将分配权限的角色ID
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取所有角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) { return this.$message.error('获取角色列表失败') }
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 根据ID删除对应的权限
    async removeRightById (role, rightID) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') { return this.$message.info('取消删除！') }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightID}`)
      if (res.meta.status !== 200) { return this.$message.error('删除权限失败！') }
      // this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // console.log(role)
      // 获取所有权限
      const { data: res } = await this.$http.get('rights/tree ')
      if (res.meta.status !== 200) { return this.$message.error('获取权限列表失败！') }
      // 把获取到的权限数据保存到 data中
      this.rightsList = res.data
      // console.log(this.rightsList)
      // 递归获取三级节点的ID
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式获取角色下所有三级权限的ID并保存到defKeys数组中
    getLeafKeys (node, arr) {
      if (!node.children) { return arr.push(node.id) }
      node.children.forEach(itme => this.getLeafKeys(itme, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed () {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) { return this.$message.error('分配权限失败！') }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
  .el-tag {
    margin:7px;
  }
  .bdTop {
    border-top: 1px solid #eee;
  }
  .bdBottom {
    border-bottom: 1px solid #eee;
  }
  .vcenter {
    display: flex;
    align-items: center;
  }
</style>
