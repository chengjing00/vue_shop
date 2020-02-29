<template>
  <div>
    <!--   面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!--      添加分类按钮-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>


      <!--    分类列表-->
      <tree-table class='treeTable' :data="catelist" :columns="columns" :selection-type="false"
                  :expand-type="false" show-index index-text="#" border>
        <!--是否有效-->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen;"></i>
          <i class="el-icon-error" v-else style="color:red;"></i>
        </template>

        <!--        排序-->
        <template slot="order" slot-scope="scope">
          <el-tag type="primary" v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag type="warning" v-else>三级</el-tag>
        </template>

        <!--        操作-->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="min">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="min">删除</el-button>
        </template>
      </tree-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible" width="40%" @close="addCataDialogClosed">
      <!--      添加分类表单-->
      <el-form :model="addCateForm" :rules="addCateRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>

        <el-form-item label="父级分类">
          <!--          options 指定数据源-->
          <!--          props 用来指定配置对象-->
          <el-cascader expand-trigger="hover" :options="parentCateList" v-model="selectedKeys"
                       :props="cascaderProps"
                       @change="parentCateChanged" clearable change-on-select></el-cascader>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
    <el-button @click="addCateDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addCate">确 定</el-button>
  </span>
    </el-dialog>


  </div>
</template>

<script>
  export default {
    name: 'Cate',
    data () {
      return {
        // 查询条件
        queryInfo: {
          type: 2,
          pagenum: 1,
          pagesize: 5,
        },
        // 商品分类列表
        catelist: [],
        //总数据长度
        total: 0,
        columns: [
          {
            label: '分类名称',
            prop: 'cat_name',
            width: '200px',
          },
          {
            label: '是否有效',
            // prop: 'cat_deleted',
            type: 'template',
            template: 'isok',
            width: '200px',
          },
          {
            label: '排序',
            type: 'template',
            template: 'order',
            width: '200px',
          },
          {
            label: '操作',
            type: 'template',
            template: 'opt',
            width: '200px',
          },
        ],
        //控制添加分类对话框的显示与隐藏
        addCateDialogVisible: false,
        // 添加分类表单对喜庆
        addCateForm: {
          cat_name: '',
          cat_pid: 0,
          cat_level: 0
        },
        // 校验规则
        addCateRules: {
          cat_name: [
            {
              required: true,
              message: '请输入分类名称',
              trigger: 'blur'
            },
            {
              min: 1,
              max: 15,
              message: '长度在 1 到 15 个字符',
              trigger: 'blur'
            }]
        },
        //父级分类列表
        parentCateList: [],
        // 指定级联选择器
        cascaderProps: {
          value: 'cat_id',
          label: 'cat_name',
          children: 'children'
        },
        // 选中的父级分类
        selectedKeys: []

      }
    },
    methods: {
      async getCateList () {
        const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
        if (res.meta.status != 200) {
          return this.$message.error('获取商品分类失败')
        }
        this.catelist = res.data.result
        this.total = res.data.total

      },
      //监听pagesize 改变的事件
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      //当前页码发生改变
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      // 展示添加分类对话框
      showAddCateDialog () {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      // 获取父级分类
      async getParentCateList () {
        const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
        console.log('res', res)
        if (res.meta.status != 200) {
          this.$message.error('获取父级分类失败')
        }
        this.parentCateList = res.data.slice(1, 15)

      },
      // 选择项发生变化触发个函数
      parentCateChanged () {
        debugger
        if (this.selectedKeys.length > 0) {
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
        } else {
          //没有选择父级分类
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }
      },
      //添加分类
      async addCate () {
        this.$refs.addCateFormRef.validate(async valid => {
          if (!valid) {
            return
          }
          console.log(this.addCateForm)
          debugger
          const { data: res } = await this.$http.post('categories', this.addCateForm)
          if (res.meta.status !== 201) {
            return this.$message.error('添加分类成功')
          }
          this.getCateList()
          this.addCateDialogVisible = false
          this.$message.success('添加分类成功')
        })

      },
      //监听对话框的关闭事件
      addCataDialogClosed () {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      }

    },
    created () {
      this.getCateList()
    }
  }
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }

  .el-cascader {
    width: 100%;
  }

  .el-cascader-panel {
    height: 50%;
  }
</style>
