<template>
  <div>
    <!--    面包屑导航区域-->
    <el-breadcrumb separator-class='el-icon-arrow-right'>
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--    卡片视图-->
    <el-card>
      <!--      添加角色按钮区域-->
      <el-row>
        <el-col>
          <el-button type='primary'>添加角色</el-button>
        </el-col>
      </el-row>
      <!--      角色列表-->
      <el-table :data='roleList' border stripe>
        <!--        展开列-->
        <el-table-column type='expand'>
          <template slot-scope='scope'>
            <el-row :class="['bdbottom',i1 ===0?'bdtop':'','vcenter']" v-for='(item1,i1) in scope.row.children'
                    :key='item1.id'>
              <!--              渲染一级权限-->
              <el-col :span='5'>
                <el-tag closable @close='removeRightById(scope.row,item1.id)'>{{ item1.authName }}</el-tag>
                <i class='el-icon-caret-right'></i>
              </el-col>
              <!--              渲染二级和三级权限-->
              <el-col :span='19'>
                <!--                通过for循环嵌套二级权限-->
                <el-row :class="[i2 ===0 ?'':'bdtop' ,'vcenter']" v-for='(item2,i2) in item1.children' :key='item2.id'>
                  <el-col :span='6'>
                    <el-tag type='success' closable @close='removeRightById(scope.row,item2.id)'>{{ item2.authName }}
                    </el-tag>
                    <i class='el-icon-caret-right'></i>
                  </el-col>
                  <el-col :span='18'>
                    <el-tag type='warning' v-for='(item3,i3) in item2.children' :key='item3.id' closable
                            @close='removeRightById(scope.row,item3.id)'>{{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!--            <pre>{{ scope.row }}</pre>-->
          </template>
        </el-table-column>
        <!--        索引列-->
        <el-table-column type='index'></el-table-column>
        <el-table-column label='角色名称' prop='roleName'></el-table-column>
        <el-table-column label='角色描述' prop='roleDesc'></el-table-column>
        <el-table-column label='操作' width='300px'>
          <template slot-scope='scope'>
            <el-button size='mini' type='primary' icon='el-icon-edit'>编辑</el-button>
            <el-button size='mini' type='warming' icon='el-icon-delete'>删除</el-button>
            <el-button size='mini' type='danger' icon='el-icon-setting' @click='showSetRightDialog(scope.row)'>分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!--    分配权限对话框-->
    <el-dialog
      title='分配权限'
      :visible.sync='setRightDialogVisible'
      width='50%' @close='setRightDialogClosed'>
      <!--      树形控件-->
      <el-tree :data='rightsList' :props='treeProps' show-checkbox node-key='id' default-expand-all
               :default-checked-keys='defKeys' ref='treeRef'></el-tree>
      <span slot='footer' class='dialog-footer'>
    <el-button @click='setRightDialogVisible = false'>取 消</el-button>
    <el-button type='primary' @click='allotRight'>确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      roleList: [],
      // 分配权限对话框
      setRightDialogVisible: false,
      // 权限树
      rightsList: [],
      // 树形控件
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id
      defKeys: [],
      // 当前角色的id
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取所有觉得列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表失败！')
      }
      this.roleList = res.data
      console.log(this.roleList)
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除权限
      await this.$confirm('是否删除该权限?, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败！')
        }
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        // return this.getRolesList()
        role.children = res.data
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
      console.log(role, rightId)
    },
    // 展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限列表
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('查询列表失败！')
      }
      this.rightsList = res.data
      console.log(this.rightsList)
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)
      console.log(this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取所有觉得的三级权限id，并保存到defKes数组
    getLeafKeys(node, arr) {
      // 如果当node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听分配权限对话框的关闭
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRight() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, idStr)
      if (res.meta.status !== 200) {
        return this.$message.error('更新权限失败！')
      }
      this.$message.success('更新权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style scoped>
.el-tag {
  margin: 10px;
}

.bdtop {
  border-top: 1px solid #eeeeee;
}

.bdbottom {
  border-bottom: 1px solid #eeeeee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
