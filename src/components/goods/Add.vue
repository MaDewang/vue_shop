<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>
        <a>商品管理</a>
      </el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区 -->
    <el-card>
      <!-- 搜索添加 -->
      <el-row>
        <!-- 警告提示区 -->
        <el-alert title="消息提示的文案" type="info" center show-icon :closable="false"></el-alert>
      </el-row>
      <!-- 步骤条区域 -->
      <el-row>
        <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
      </el-row>
      <!-- tab栏区域 -->
      <el-form
        ref="addFormRef"
        :rules="addFormRules"
        :model="addForm"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类">
              <!-- 级联选择器 -->
              <el-cascader
                v-model="goodsCat"
                :options="cateList"
                :props="cateProps"
                @change="handleChang"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- tab项 -->
            <el-form-item :label="item.attr_name" v-for="item in manyTabData" :key="item.attr_id">
              <!-- 复选框 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border :label="cb" v-for="(cb, i) in item.attr_vals" :key="i"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item :label="item.attr_name" v-for="(item,i) in onlyTabData" :key="i">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="actionUploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture-card"
              :headers="headersObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
              <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本区域 -->
            <quill-editor v-model="addForm.quillData"></quill-editor>
            <!-- 添加商品按钮区域 -->
            <el-button class="addBtn" type="primary" @click="add">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewDialogVisible" width="50%" class="previewImg">
      <img :src="previewUrl" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 默认步骤
      activeIndex: 0,
      //   表单数据 添加商品所需的参数
      addForm: {
        goods_name: '',
        goods_price: '0',
        goods_number: '0',
        goods_weight: '0',
        // 图片上传后的临时路径
        pics: [],
        //   富文本的内容
        quillData: '',
        attrs: []
      },
      //   选择的分类的id数组
      goodsCat: [],
      //   表单验证规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ]
        // ,
        // goodsCat: [
        //   { required: true, message: '请输入商品分类', trigger: 'blur' }
        // ]
      },
      //   商品分类数据
      cateList: [],
      // 指定联级选择器的参数
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //   动态参数列表数据
      manyTabData: [],
      //   静态属性列表数据
      onlyTabData: [],
      //   图片上传地址
      actionUploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      //   设置图片上传的请求头
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      //   图片预览地址
      previewUrl: '',
      //   控制图片预览对话框的显示隐藏
      previewDialogVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表数据 渲染页面
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.cateList = res.data
    },
    // 当联级选择器选择时
    handleChang() {
      if (this.goodsCat.length !== 3) {
        return (this.goodsCat = [])
      }
      //   console.log(this.goodsCat)
    },
    // tab栏切换前
    beforeTabLeave(activeName, oldActiveName) {
      // 在tab栏切换之前触发，两个形参为切换前，后的tab栏name
      //   console.log(oldActiveName)
      if (oldActiveName === '0') {
        if (this.goodsCat.length !== 3) {
          this.$message.error('请选择商品分类')
          return false
        } else if (this.addForm.goods_name.trim() === '') {
          this.$message.error('请输入商品名称')
          return false
        } else if (this.addForm.goods_price.trim() === '') {
          this.$message.error('请输入商品价格')
          return false
        } else if (this.addForm.goods_number.trim() === '') {
          this.$message.error('请输入商品数量')
          return false
        } else if (this.addForm.goods_weight.trim() === '') {
          this.$message.error('请输入商品重量')
          return false
        }
      }
    },
    // 点击tab栏切换
    async tabClicked() {
      //   如果访问的是动态参数面板
      if (this.activeIndex === '1') {
        // console.log(this.activeIndex)
        // 发送请求 获取动态参数列表数据
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'many' } }
        )
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数失败')
        }
        // 将attr_vals变成数组
        res.data.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
          //   console.log(item.attr_vals)
        })
        this.manyTabData = res.data
      } else if (this.activeIndex === '2') {
        // console.log(this.activeIndex)
        // 发送请求 获取静态属性列表数据
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'only' } }
        )
        // console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性失败')
        }
        this.onlyTabData = res.data
      }
    },
    // 处理点击图片预览
    handlePreview(file) {
      //   console.log(file)
      this.previewUrl = file.response.data.url
      this.previewDialogVisible = true
    },
    // 处理点击图片删除
    handleRemove(file) {
      // 获取被删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 根据获得的临时路径查询其索引
      const i = this.addForm.pics.findIndex(x => {
        return x.pic === filePath
      })
      // 根据索引把这个图片信息对象在数组中删除
      this.addForm.pics.splice(i, 1)
      //   console.log(this.addForm.pics)
    },
    // 监听图片上传
    handleSuccess(response) {
      // 将图片信息保存为对象
      const picInfo = { pic: response.data.tmp_path }
      //   将这个对象保存到addForm.pics中
      this.addForm.pics.push(picInfo)
      //   console.log(this.addForm.pics)
    },
    // 添加商品
    add() {
      // 验证表单数据
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        this.addForm.goods_cat = this.goodsCat.join(',')
        // console.log(this.manyTabData)
        // 处理动态参数
        this.manyTabData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTabData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        // 发送请求 添加商品
        const { data: res } = await this.$http.post('goods', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }
        this.$message.success('成功添加商品！')
        // 跳转到商品列表页面
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.goodsCat.length === 3) {
        return this.goodsCat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.previewImg {
  width: 100%;
}
</style>
