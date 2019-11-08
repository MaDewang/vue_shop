<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>
        <a href="/">商品管理</a>
      </el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-row :gutter="20">
        <el-col :span="4">
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效的模板 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            style="color: lightgreen"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <!-- 排序的模板 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="min" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="min" v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag size="min" v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
        </template>
        <!-- 操作的模板 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!--  添加分类的的对话框 -->
    <el-dialog
      title="提示"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClose"
    >
      <el-form
        ref="addCateFormRef"
        :model="addCateForm"
        label-width="100px"
        :rules="addCateFormRules"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            expand-trigger="hover"
            v-model="selectedKeys"
            :options="ParentCateList"
            :props="cascaderProps"
            @change="ParentCateChange"
            clearable
            change-on-select
          ></el-cascader>
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
  data() {
    return {
      // 获取商品列表的参数
      queryInfo: {
        // 搜索关键字
        type: 3,
        // 当前页码
        pagenum: 1,
        // 每页显示的行数
        pagesize: 5
      },
      // 获取的分类数据
      cateList: [],
      // 总数据条数
      total: 0,
      // 为tabal制定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示 将当前列定义为模板列
          type: 'template',
          // 表示当前列的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          // 表示 将当前列定义为模板列
          type: 'template',
          // 表示当前列的模板名称
          template: 'order'
        },
        {
          label: '操作',
          // 表示 将当前列定义为模板列
          type: 'template',
          // 表示当前列的模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示
      addCateDialogVisible: false,
      // 表单数据
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        // 分类等级 默认一级 0
        cat_level: 0
      },
      // 分类名称验证规则
      addCateFormRules: {
        // 验证规则
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      ParentCateList: [],
      // 指定联级选择器的参数
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的Id数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取用户列表数据 渲染页面
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      // console.log(res)
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 修改页面显示数据的数量
    handleSizeChange(newsize) {
      // console.log(newsize)
      this.queryInfo.pagesize = newsize
      this.getCateList()
    },
    // 更改页码
    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage
      this.getCateList()
    },
    // 点击添加分类按钮
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级数据列表失败')
      }
      this.ParentCateList = res.data
    },
    // 选择项发生变化时
    ParentCateChange() {
      // console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        // 分类父 ID
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 分类层级
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 分类父 ID
        this.addCateForm.cat_pid = 0
        // 分类层级
        this.addCateForm.cat_level = 0
      }
    },
    // 点击确定 添加分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // console.log(this.addCateForm)
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听添加分类对话框的关闭事件 q清空对话框表单
    addCateDialogClose() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_name = ''
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>

<style lang="less" scoped>
.el-pagination {
  margin-top: 15px;
}
.el-row {
  margin-bottom: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
