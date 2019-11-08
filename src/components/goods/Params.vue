<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>
        <a href="/">商品管理</a>
      </el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数" type="warning" :closable="false"></el-alert>
      <el-row>
        <!-- 选择商品分类区域 -->
        <el-col class="cat_opt">
          <span>选择商品分类</span>
          <!-- 商品分类联级选择器 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChang"
          ></el-cascader>
        </el-col>
        <!-- tab 栏区域 -->
        <el-col class="cat_opt">
          <el-tabs v-model="activeName" @tab-click="handleClick">
            <el-tab-pane label="动态参数" name="many">
              <el-button
                type="primary"
                size="mini"
                :disabled="isBtnDisabled"
                @click="addCatedialogVisible"
              >添加参数</el-button>
              <!-- 动态参数的表格 -->
              <el-table :data="manyTableData" border stripe>
                <!-- 展开区域 -->
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <!-- 渲染的tag标签 -->
                    <el-tag
                      closable
                      v-for="(item,i) in scope.row.attr_vals"
                      :key="i"
                      @close="handleClose(i,scope.row)"
                    >{{item}}</el-tag>
                    <!-- 动态编辑标签 -->
                    <el-input
                      class="input-new-tag"
                      v-if="scope.row.inputVisible"
                      v-model="scope.row.inputValue"
                      ref="saveTagInput"
                      size="small"
                      @keyup.enter.native="handleInputConfirm(scope.row)"
                      @blur="handleInputConfirm(scope.row)"
                    ></el-input>
                    <el-button
                      v-else
                      class="button-new-tag"
                      size="small"
                      @click="showInput(scope.row)"
                    >+ New Tag</el-button>
                  </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                <el-table-column label="操作">
                  <template slot-scope="scope">
                    <el-button
                      type="primary"
                      icon="el-icon-edit"
                      size="mini"
                      @click="editCateDialogVisible(scope.row.attr_id)"
                    >编辑</el-button>
                    <el-button
                      type="danger"
                      icon="el-icon-delete"
                      size="mini"
                      @click="deleteCateDialogVisible(scope.row.attr_id)"
                    >删除</el-button>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
            <el-tab-pane label="静态属性" name="only">
              <el-button
                type="primary"
                size="mini"
                :disabled="isBtnDisabled"
                @click="addCatedialogVisible"
              >添加属性</el-button>
              <!-- 静态属性的表格 -->
              <el-table :data="onlyTableData" border stripe>
                <!-- 展开区域 -->
                <el-table-column type="expand">
                  <template slot-scope="scope">
                    <!-- 渲染的tag标签 -->
                    <el-tag
                      closable
                      v-for="(item,i) in scope.row.attr_vals"
                      :key="i"
                      @close="handleClose(i,scope.row)"
                    >{{item}}</el-tag>
                    <!-- 动态编辑标签 -->
                    <el-input
                      class="input-new-tag"
                      v-if="scope.row.inputVisible"
                      v-model="scope.row.inputValue"
                      ref="saveTagInput"
                      size="small"
                      @keyup.enter.native="handleInputConfirm(scope.row)"
                      @blur="handleInputConfirm(scope.row)"
                    ></el-input>
                    <el-button
                      v-else
                      class="button-new-tag"
                      size="small"
                      @click="showInput(scope.row)"
                    >+ New Tag</el-button>
                  </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                <el-table-column label="操作">
                  <template slot-scope="scope">
                    <el-button
                      type="primary"
                      icon="el-icon-edit"
                      size="mini"
                      @click="editCateDialogVisible(scope.row.attr_id)"
                    >编辑</el-button>
                    <el-button
                      type="danger"
                      icon="el-icon-delete"
                      size="mini"
                      @click="deleteCateDialogVisible(scope.row.attr_id)"
                    >删除</el-button>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-col>
      </el-row>
    </el-card>
    <!-- 添加静态属性或者添加动态数据的对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="dialogVisible"
      width="50%"
      @close="addCatedialogClosed"
    >
      <el-form ref="addFormRef" :rules="addFormRules" :model="addForm" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改静态属性或者添加动态数据的对话框 -->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editCatedialogClosed"
    >
      <el-form ref="editFormRef" :rules="addFormRules" :model="editForm" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 分类列表数据
      cateList: [],
      // 分类列表数据的条数
      total: 0,
      // 联级选择器 选中的数据
      selectedCateKeys: [],
      // 联级选择器的配置
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // tab栏选中的页面的名称
      activeName: 'only',
      // 动态 参数栏的数据
      manyTableData: [],
      // 静态参数栏的数据
      onlyTableData: [],
      // 控制添加对话框显示隐藏
      dialogVisible: false,
      // 控制添加对话框显示隐藏
      editDialogVisible: false,
      // 储存参数id
      attr_id: '',
      // 添加参数表单数据
      addForm: {
        attr_name: ''
      },
      // 修改参数表单数据
      editForm: {
        attr_name: ''
      },
      // 对话框表单的验证规则
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有商品分类列表数据 渲染页面
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败')
      }
      console.log(res)
      this.cateList = res.data
    },
    // 级联选择框选中项变化
    async handleChang() {
      // 获取参数的数据列表
      this.getParamsData()
    },
    // tab栏切换
    handleClick() {
      // console.log(this.activeName)
      // 获取参数的数据列表
      this.getParamsData()
    },
    // 获取参数的数据列表
    async getParamsData() {
      // 判断是否选中的是第三级分类 如果不是  清空
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // console.log(this.selectedCateKeys)
      // 发送请求 获取参数列表
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      // console.log(res)
      // 将attr_vals 变成数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示隐藏
        item.inputVisible = false
        // 文本框的内容
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 显示添加参数对话框
    addCatedialogVisible() {
      this.dialogVisible = true
    },
    // 监听关闭添加参数对话框
    addCatedialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 监听关闭修改参数对话框
    editCatedialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 点击确定 添加参数
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          { attr_name: this.addForm.attr_name, attr_sel: this.activeName }
        )
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.dialogVisible = false
        this.getParamsData()
      })
    },
    // 显示修改参数对话框
    async editCateDialogVisible(id) {
      this.attr_id = id
      // 发送请求 获取当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${id}`,
        { params: { attr_sel: this.activeName } }
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      this.editForm.attr_name = res.data.attr_name
      this.editDialogVisible = true
    },
    // 点击确定 修改参数
    async editParams() {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${this.attr_id}`,
        { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('编辑参数失败')
      }
      this.$message.success('编辑参数成功')
      this.attr_id = ''
      this.getParamsData()
      this.editDialogVisible = false
    },
    // 点击确定 删除参数
    async deleteCateDialogVisible(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 发送请求 删除参数
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${id}`
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败')
      }
      this.$message.success('删除参数成功')
      this.getParamsData()
      this.editDialogVisible = false
    },
    // 动态编辑标签 input 点击或者摁下 Enter 都会触发
    async handleInputConfirm(row) {
      // 如果没有输入内容
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果输入了内容 就保存到数组中
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发送请求 保存到服务器端
      this.saveAtrrVals()
    },
    // 将对attr_vals 的操作保存到服务器端
    async saveAtrrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('编辑参数项失败！')
      }
      this.$message.success('编辑参数项成功！')
    },
    // 删除对于的参数可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAtrrVals()
    },
    // 动态编辑标签 点击button显示input
    showInput(row) {
      // console.log(row)
      row.inputVisible = true
      // nextTick 当页面重新渲染时
      // 让文本框获得焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  },
  computed: {
    // 根据联级选择器选中的数据长度  返回true 或false
    isBtnDisabled() {
      if (this.selectedCateKeys.length === 3) {
        return false
      }
      return true
    },
    // 计算 获取参数列表的分类id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[this.selectedCateKeys.length - 1]
      }
      return null
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === 'only') {
        return '静态属性'
      }
      return '动态参数'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 0 10px;
}
.input-new-tag {
  width: 120px;
}
</style>
