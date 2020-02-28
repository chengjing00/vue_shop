<template xmlns:el-col="http://www.w3.org/1999/html">
  <div>
    <!--   面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!--      添加角色按钮-->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
    </el-card>

    <!--    角色列表-->
    <el-table :data="roleslist" border stripe>
      <!--      展开列-->
      <el-table-column type="expand">

        <template slot-scope="scope">
          <!--          衫格系统-->
          <el-row :class="['dbbottom',i == 0?'bdtop':'','vcenter']" v-for="(item ,i) in scope.row.children"
                  :key="item.id">
            <!--            一级权限-->
            <el-col :span="5">
              <el-tag closable @close="removeRightById(scope.row,item.id)">
                {{item.authName}}
              </el-tag>
              <i class="el-icon-caret-right"></i>

            </el-col>
            <!--            二三级权限-->
            <el-col :span="19">
              <!--              二级权限-->
              <el-row :class="[i2==0?'':'bdtop','vcenter']" v-for="(item2, i2) in item.children" :key="item2.id">

                <el-col :span="6">
                  <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">
                    {{item2.authName}}
                  </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>

                <el-col :span="18">
                  <el-tag v-for="(item3 ,i3) in item2.children" :key="item3.id" type="warning"
                          closable @close="removeRightById(scope.row,item3.id)">
                    {{item3.authName}}<i class="el-icon-caret-right"></i>
                  </el-tag>

                </el-col>
              </el-row>
            </el-col>
          </el-row>

        </template>
      </el-table-column>

      <el-table-column type="index" label="#"></el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>

      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="min">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="min">删除</el-button>
          <el-button type="warning" icon="el-icon-setting" size="min" @click="showSetRightDialog(scope.row)">分配权限
          </el-button>
        </template>

      </el-table-column>
    </el-table>

    <!--分配权限对话框-->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="40%" @close="setRightDialogClose">
      <!--      树形控件-->
      <el-tree :data="roleslist" :props="treeProps" show-checkbox node-key="id"
               :default-expand-all="true" :default-checked-keys="defKeys" ref="treeRef"></el-tree>

      <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    name: 'Roles',
    data () {
      return {
        roleslist: [],
        setRightDialogVisible: false,
        // 所有权限的数据
        rightsList: [],
        // 树型控件
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        //默认选中的idKey值
        defKeys: [],
        // 分配权限的roleId
        roleId: ''
      }
    },
    methods: {
      async getRolesList () {
        const { data: res } = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.roleslist = res.data
        console.log(res)
        return this.$message.success('获取角色列表成功')

      },
      // 根据id 删除对应的权限
      async removeRightById (role, id) {
        // 弹框提示用户是否删除
        const confirmReuslt = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => {
          return err
        })

        if (confirmReuslt !== 'confirm') {
          return this.$message.info('已经取消删除')
        }
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${id}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败')
        }
        this.$message.success('删除权限成功')
        role.children = res.data
      },
      // 展示分配权限的对话框
      async showSetRightDialog (role) {
        const { data: res } = await this.$http.get('rights/tree')
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限列表失败')
        }
        this.roleslist = res.data
        this.roleId = role.id

        //递归获取三级节点id
        this.getLeftKeys(role, this.defKeys)

        this.setRightDialogVisible = true
      },
      // 通过递归获取所有三级权限id
      getLeftKeys (node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }

        node.children.forEach(item => {
          this.getLeftKeys(item, arr)
        })

      },
      // 监听分配权限对话框的关闭
      setRightDialogClose () {
        this.defKeys = []
        this.getRolesList()
      },
      // 为角色分配权限
      async allotRights () {
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys(),]

        const idStr = keys.join(',')
        const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
        if (res.meta.status !== 200) {
          return this.$message.error('添加权限失败')
        }
        this.setRightDialogVisible = false
        return this.$message.success('添加权限成功')
      }
    },
    created () {
      this.getRolesList()
    }
  }
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }

  .bdtop {
    border-top: 1px solid #eee;
  }

  .dbbottom {
    border-bottom: 1px solid #eee;
  }

  .vcenter {
    display: flex;
    align-items: center;
  }

</style>
