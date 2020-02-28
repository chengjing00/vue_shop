<template>
  <div>
    <!--   面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!--    卡片视图区-->
    <el-card class="box-card">
      <!--      搜索与添加区域-->
      <div style="margin-top: 15px;">

        <el-row :gutter="20">
          <el-col :span="7">
            <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
              <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
            </el-input>
          </el-col>
          <el-col :span="4">
            <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
          </el-col>
        </el-row>

        <!--        用户列表区域-->
        <el-table :data="userList" border stripe>
          <el-table-column type="index" label="#"></el-table-column>
          <el-table-column label="姓名" prop="username"></el-table-column>
          <el-table-column label="邮箱" prop="email"></el-table-column>
          <el-table-column label="电话" prop="mobile"></el-table-column>
          <el-table-column label="角色" prop="role_name"></el-table-column>
          <el-table-column label="状态">
            <!--            作用域插槽-->
            <template slot-scope="scope">
              <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
              </el-switch>
            </template>
          </el-table-column>


          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-tooltip effect="dark" content="修改" placement="top" :enterable="false">
                <el-button type="primary" icon="el-icon-edit" size="min" circle
                           @click="showEditDialog(scope.row.id)"></el-button>
              </el-tooltip>

              <el-tooltip effect="dark" content="删除" placement="top" :enterable="false">
                <el-button type="danger" icon="el-icon-delete" size="min" circle
                           @click="removeUserById(scope.row.id)"></el-button>
              </el-tooltip>

              <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
                <el-button type="warning" icon="el-icon-setting" size="min" circle
                           @click="setRole(scope.row)"></el-button>
              </el-tooltip>
            </template>

          </el-table-column>
        </el-table>

        <!--        分页区域-->
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum"
          :page-sizes="[1, 2, 5, 10]"
          :page-size="queryInfo.pagesize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>
    </el-card>

    <!--添加对话框-->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="40%" @close="addDialogClosed">

      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>

        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>

        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>


    <!--    修改用户对话框-->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="40%" @close="editDialogClosed">

      <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>

        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!--    分配角色对话框-->
    <el-dialog
      title="分配角色"
      :visible.sync="setRoledialogVisible"
      width="40%" @close="setRoleDialogClosed">
      <p>当前的用户:{{userInfo.username}}</p>
      <p>当前的角色:{{userInfo.role_name}}</p>

      <p>分配新角色:
        <el-select v-model="selectedRoleId" placeholder="请选择">
          <el-option v-for="item in roleslist" :key="item.item" :label="item.roleName" :value="item.id"></el-option>
        </el-select>
      </p>

      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoledialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
  export default {
    name: 'users',
    data () {
      // 验证邮箱
      var checkEmail = (rules, value, callback) => {
        const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
        if (regEmail.test(value)) {
          return callback()
        }
        callback(new Error('输入合法的邮箱'))
      }
      var checkMobile = (rules, value, callback) => {
        const regMobile = /^[1][3,4,5,6,7,8,9][0-9]{9}$/
        if (regMobile.test(value)) {
          return callback()
        }
        callback(new Error('输入正确的手机号'))
      }
      return {
        // get查询参数
        queryInfo: {
          query: '',
          pagenum: 1,
          pagesize: 2
        },
        //用户列表
        userList: [],
        total: 0,
        addDialogVisible: false,
        editDialogVisible: false,
        // 添加用户表单数据
        addForm: {
          username: '',
          password: '',
          email: '',
          mobile: ''
        },
        addFormRules: {
          username: [
            {
              required: false,
              message: '请输入用户名',
              trigger: 'blur'
            },
            {
              min: 3,
              max: 10,
              message: '长度在 6 到 10 个字符',
              trigger: 'blur'
            }
          ],
          password: [
            {
              required: true,
              message: '请输入密码',
              trigger: 'blur'
            },
            {
              min: 6,
              max: 10,
              message: '长度在 6 到 10 个字符',
              trigger: 'blur'
            }

          ],
          email: [
            {
              required: true,
              message: '请输入邮箱',
              trigger: 'blur'
            },
            {
              validator: checkEmail,
              trigger: 'blur'
            }
          ],
          mobile: [
            {
              required: true,
              message: '请输入手机',
              trigger: 'blur'
            },
            {
              validator: checkMobile,
              trigger: 'blur'
            }
          ]
        },
        // 查询到的用户对象
        editForm: {},
        //控制分配角色对话框的显示和隐藏
        setRoledialogVisible: false,
        // 需要被分配角色的用户信息
        userInfo: {},
        // 角色列表
        roleslist: [],
        selectedRoleId: ''
      }
    },
    methods: {
      async getUserList () {
        const { data: res } = await this.$http.get('users', { params: this.queryInfo })
        if (res.meta.status !== 200) {
          return this.$message.error('获取用户列表失败')
        }
        this.userList = res.data.users
        this.total = res.data.total
      },
      // 监听pagesize改变的事件
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize
        this.getUserList()
      },
      // 页码值改变的事件
      handleCurrentChange (newPage) {
        console.log(newPage)
        this.queryInfo.pagenum = newPage
        this.getUserList()
      },

      // 状态改变
      async userStateChanged (userInfo) {
        const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
        console.log(res)
        if (res.meta.status !== 200) {
          this.$message.error('更新用户状态失败')
          userInfo.mg_state = !userInfo.mg_state
        }
        this.$message.success('用户状态更新成功')
      },
      // 监听添加用户框的关闭事件
      addDialogClosed () {
        this.$refs.addFormRef.resetFields()
      },
      // 添加用户
      addUser () {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) {
            return
          }
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加用户失败')
          } else {
            this.$message.success('添加用户成功')
            this.addDialogVisible = false
            this.getUserList()
          }
        })
      },
      //展示编辑用户对话框
      async showEditDialog (id) {
        this.editDialogVisible = true
        const { data: res } = await this.$http.get('users/' + id)

        if (res.meta.status !== 200) {
          this.$message.error('获取数据失败')
        } else {
          this.editForm = res.data

        }
      },
      async editUserInfo (id) {
        this.$refs.editFormRef.validate(async valid => {
          if (!valid) {
            return
          }
          const { data: res } = await this.$http.put('users/' + this.editForm.id,
            {
              email: this.editForm.email,
              mobile: this.editForm.mobile
            })

          if (res.meta.status !== 200) {
            this.$message.error('更新用户信息失败')
          } else {
            this.$message.success('更新用户信息成功')
            this.editDialogVisible = false
            this.getUserList()
          }

        })
      },

      // 修改对话框的关闭事件
      editDialogClosed () {
        this.$refs.editFormRef.resetFields()
      },
      // 根据Id删除数据
      async removeUserById (id) {
        const confirmReuslt = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => {
          return err
        })

        if (confirmReuslt !== 'confirm') {
          return this.$message.info('已经取消删除')
        }
        // 调用删除接口删除用户
        const { data: res } = await this.$http.delete('users/' + id)
        console.log('res', res)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success('删除成功')
        this.getUserList()
      },
      // 展示分配角色的对话框
      async setRole (userInfo) {
        this.userInfo = userInfo
        // 获取所有的角色列表
        const { data: res } = await this.$http.get('roles')
        if (res.meta.status !== 200) {
          return this.$message.error('获取角色列表失败')
        }
        this.roleslist = res.data
        this.setRoledialogVisible = true
      },
      // 点击按钮分配角色
      async saveRoleInfo () {
        if (!this.selectedRoleId) {
          return this.$message.info('请选择角色')
        }
        const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })

        if (res.meta.status !== 200) {
          return this.$message.error('分配角色错误')
        }
        this.setRoledialogVisible = false;
        this.getUserList()
        return this.$message.success('更新角色成功')
      },
      // 监听分配角色对框关闭
      setRoleDialogClosed(){
        this.selectedRoleId = ''
        this.userList = {}
      }

    },
    created () {
      this.getUserList()
    }
  }
</script>

<style lang="less" scoped>

</style>
