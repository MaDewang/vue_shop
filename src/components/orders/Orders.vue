<template>
  <div>
    <h3>订单列表</h3>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索框区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query">
            <el-button slot="append" icon="el-icon-search" @click="searchOrders"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="ordersList" style="width: 100%" border>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number" label="订单编号" width="180"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">{{scope.row.create_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" prop="is_send" width="120px">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
            <el-button
              type="success"
              icon="el-icon-location-information"
              size="mini"
              @click="showProgressDialog"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 40]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog title="修改地址" :visible.sync="dialogVisible" width="50%" @close="addRessFormClose">
      <el-form
        ref="addRessFormRef"
        :rules="addRessFormRules"
        :model="addRessForm"
        label-width="100px"
      >
        <!-- 联级选择器 -->
        <el-form-item label="省市区/县">
          <el-cascader
            v-model="addRessForm.addRess1"
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
            @change="handleChange"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址">
          <el-input v-model="addRessForm.addRess2"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="progressDialogVisible" width="50%">
      <!-- 时间线 -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressList"
          :key="index"
          :timestamp="activity.time"
        >{{activity.context}}</el-timeline-item>
      </el-timeline>
      <span slot="footer" class="dialog-footer">
        <el-button @click="progressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="progressDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'

export default {
  data() {
    return {
      //   获取订单数据 所需的参数
      queryInfo: {
        // 查询参数
        query: '',
        // 当前页码
        pagenum: 1,
        // 每一页数据的数量
        pagesize: 10
      },
      // 订单数据总数
      total: 0,
      //   订单列表数据
      ordersList: [],
      //   对话框显示隐藏
      dialogVisible: false,
      //   修改地址的表单
      addRessForm: {
        addRess1: [],
        addRess2: ''
      },
      //   表单验证规则
      addRessFormRules: {
        addRess1: [
          { required: true, message: '请选择省市区/县', tirgger: 'blur' }
        ],
        addRess2: [
          { required: true, message: '请选择省市区/县', tirgger: 'blur' }
        ]
      },
      //   联级选择器的数据
      cityData,
      //   物流对话框 显示隐藏
      progressDialogVisible: false,
      //   物流数据
      progressList: []
    }
  },
  created() {
    this.getOrdersList()
  },
  methods: {
    async getOrdersList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单数据失败！')
      }
      // console.log(555)
      this.total = res.data.total
      this.ordersList = res.data.goods
    },
    handleSizeChange(val) {
      //   console.log(`每页 ${val} 条`)
      this.queryInfo.pagesize = val
      this.getOrdersList()
    },
    handleCurrentChange(val) {
      //   console.log(`当前页: ${val}`)
      this.queryInfo.pagenum = val
      this.getOrdersList()
    },
    // 点击显示对话框
    showBox() {
      this.dialogVisible = true
    },
    // 联级选择去选择后
    handleChange() {
      console.log(this.addRessForm.addRess1)
    },
    // 监听修改地址对话框
    addRessFormClose() {
      this.addRessForm.addRess1 = []
      this.addRessForm.addRess2 = ''
    },
    // 点击 显示物流进度对话框
    async showProgressDialog() {
      // 查询物流信息
      const { data: res } = await this.$http.get('/kuaidi/804909574412544580')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单数据失败！')
      }
      this.progressList = res.data
      this.progressDialogVisible = true
    },
    // 点击搜索 更新页面
    searchOrders() {
      console.log(this.queryInfo.query.trim().length)
      if (this.queryInfo.query.trim().length !== 0) {
        this.getOrdersList()
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
