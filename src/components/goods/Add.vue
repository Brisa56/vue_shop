<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert
        title="添加商品信息"
        type="info"
        :closeable="false"
        show-icon
      >
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps
        align-center
        :space="200"
        :active="activeIndex-0"
        finish-status="success"
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 表单区域 -->
      <!-- 只能form嵌套tab -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-position="top"
      >
        <!-- tab栏区域 -->
        <el-tabs
          tab-position="left"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <!-- 基本信息 -->
          <el-tab-pane
            name='0'
            label="基本信息"
          >
            <el-form-item
              label="商品名称"
              prop="goods_name"
            >
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item
              label="商品价格"
              prop="goods_price"
            >
              <el-input
                v-model="addForm.goods_price"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品重量"
              prop="goods_weight"
            >
              <el-input
                v-model="addForm.goods_weight"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品数量"
              prop="goods_number"
            >
              <el-input
                v-model="addForm.goods_number"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="商品参数"
              prop="goods_cat"
              style="width:400px"
            >
              <el-cascader
                v-model="addForm.goods_cat"
                :options="catelist"
                :props="catProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>

          </el-tab-pane>
          <!-- 商品参数 -->
          <el-tab-pane
            name='1'
            label="商品参数"
          >
            <!-- 渲染表单的Item项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="(item) in manyTableData"
              :key="item.attr_id"
              label-position="top"
            >
              <!-- 复选框组 -->
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb,i) in item.attr_vals"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>

            </el-form-item>
          </el-tab-pane>
          <!-- 商品属性 -->
          <el-tab-pane
            name='2'
            label="商品属性"
          >
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals">
              </el-input>
            </el-form-item>
          </el-tab-pane>
          <!-- 商品图片 -->
          <el-tab-pane
            name='3'
            label="商品图片"
          >
            <!-- action表示图片要上传到后台的API地址 -->
            <el-upload
              :headers="headerObj"
              class="upload-demo"
              :action="uploadUrl"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              list-type="picture"
            >
              <el-button
                size="small"
                type="primary"
              >点击上传</el-button>

            </el-upload>
          </el-tab-pane>
          <!-- 商品内容 -->
          <el-tab-pane
            name='4'
            label="商品内容"
          >
            <!-- 富文本编辑器 -->
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button
              type="primary"
              class="btnAdd"
              @click="add"
            >添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>

    </el-card>
    <!-- 图片预览 -->
    <el-dialog
      title="提示"
      :visible.sync="previewVisible"
      width="50%"
    >
      <img
        :src="previewPath"
        alt=""
        class="previewImg"
      >
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'Add',
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属的分类数组
        goods_cat: [],
        //图片的数组
        pics: [],
        //商品的详情描述
        goods_introduce: '',
        attrs: [],
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' },
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' },
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' },
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' },
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' },
        ],
      },
      //商品分类数据
      catelist: [],
      // 指定级联选择器的配置对象
      catProps: {
        expandTrigger: 'hover',
        // checkStrictly: 'true',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
      },
      //动态参数列表
      manyTableData: [],
      //静态参数列表
      onlyTableData: [],
      //   上传图片的地址
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      //图片上传的请求头Obj
      headerObj: {
        Authorization: window.sessionStorage.getItem('token'),
      },
      //图片预览地址
      previewPath: '',
      //图片预览显示控制
      previewVisible: false,
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取所有商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200)
        return this.$message.error('获取商品分类失败')
      // 赋值
      this.catelist = res.data
    },
    //级联选择器选中项变化会触发这个函数
    handleChange() {
      // console.log(this.addForm.goods_cat)
      if (this.addForm.goods_cat.length !== 3) this.addForm.goods_cat = []
    },
    //
    beforeTabLeave(activeName, oldActiveName) {
      // console.log(activeName, oldActiveName)
      if (oldActiveName === '0' && this.addForm.goods_cat.length != 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    //
    async tabClicked() {
      // console.log(this.activeIndex)
      if (this.activeIndex === '1') {
        // console.log('动态参数面板')
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'many' } }
        )
        if (res.meta.status !== 200)
          return this.$message.error('获取动态参数列表失败')
        //   console.log(res.data)
        res.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          { params: { sel: 'only' } }
        )
        if (res.meta.status !== 200)
          return this.$message.error('获取静态属性列表失败')
        this.onlyTableData = res.data
        //   console.log(res.data)
      }
    },
    //处理图片预览效果
    handlePreview(file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    //处理图片移除的操作
    handleRemove(file) {
      // 1、获取将要删除的图片的临时路径
      const filePath = file.response.data.tmp_path
      // 2、从pics数组中找到这个图片对应的索引值
      const i = this.addForm.pics.findIndex((x) => {
        x.pic === filePath
      })
      // 3、调用数组的splice方法把图片信息对象从pics数组中移除
      this.addForm.pics.splice(i, 1)
    },
    //监听图片上传成功的事件
    handleSuccess(response) {
      // 1、拼接得到一个图片信息
      const picInfo = { pic: response.data.tmp_path }
      // 2、将图片信息push到pics中
      this.addForm.pics.push(picInfo)
    },
    //添加商品
    add() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) {
          return this.$message.error('请填写必要的字段')
        }
        //执行添加

        //级联选择器中的v-model要是数组
        //先做一个深拷贝
        //lodash cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        //处理动态参数和静态属性
        this.manyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' '),
          }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals,
          }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败')
        this.$message.success('添加商品成功')
        //跳转
        this.$router.push('/goods')
      })
    },
  },
  computed: {
    //获取其分类id
    cateId() {
      if (this.addForm.goods_cat.length === 3) return this.addForm.goods_cat[2]
      return null
    },
  },
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 5px 0 0 !important;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>