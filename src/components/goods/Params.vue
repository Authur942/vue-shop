<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-alert show-icon title="注意：只允许为第三级分类设置相关参数" type="warning" :closable="false">
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat-opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader expand-trigger="hover" v-model="selectedKeys" :options="catelist" :props="cateProps" @change="handleChange" clearable change-on-select="true"></el-cascader>
        </el-col>
      </el-row>

      <!-- tab 标签页区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数面板 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数按钮 -->
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParanmsDialog">添加参数</el-button>
          <!-- 动态参数表格数据 -->
          <el-table :data="manyTabData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="attrvalsTagDelete(i,scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button icon="el-icon-edit" type="primary" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button icon="el-icon-delete" type="danger" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性面板 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性按钮 -->
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addParanmsDialog">添加属性</el-button>
          <!-- 静态属性表格数据 -->
          <el-table :data="onlyTabData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="attrvalsTagDelete(i,scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button icon="el-icon-edit" type="primary" size="mini" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button icon="el-icon-delete" type="danger" size="mini" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数 对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addParamsDialogVisible"
      width="50%"
      @close="addParamsDialogClosed">
      <el-form ref="addParamsFormRef" :model="addParamsForm" label-width="100px" :rules="addParamsFormRules">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑参数 对话框 -->
    <el-dialog
      :title="'编辑' + titleText"
      :visible.sync="editParamsDialogVisible"
      width="50%"
      @close=editParamsDialogClosed>
      <el-form ref="editParamsFormRef" :model="editParamsForm" label-width="100px" :rules="editParamsFormRules">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  data () {
    return {
      // 商品分类列表
      catelist: [],
      // 级联选择框的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 这是级联选择框双向绑定的数组
      selectedKeys: [],
      // 静态标签页选中对象
      activeName: 'many',
      // 动态参数数据
      manyTabData: [],
      // 静态属性数据
      onlyTabData: [],
      // 表单数据对象
      addParamsForm: {
        attr_name: ''
      },
      // 表单数据验证规则对象
      addParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur'}
        ]
      },
      // 添加参数对话框控制对象
      addParamsDialogVisible: false,
      // 编辑表单的数据对象
      editParamsForm: {},
      // 编辑表单数据验证规则对象
      editParamsFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur'}
        ]
      },
      // 编辑参数对话框控制对象
      editParamsDialogVisible: false,
      // 这是控制标签与输入框的隐藏显示
      inputVisible: false,
      // 这是输入框文本对象
      inputValue: ''
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取商品分类数据
    async getCateList () {
      const { data: res} = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 级联选择框选中项变化，会触发这个函数
    handleChange () {
      this.getParamsData()
    },
    // 点击Tab标签页触发的事件
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 获取参数的列表数据
    async getParamsData () {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTabData = []
        this.onlyTabData = []
        return
      }
      console.log(this.selectedKeys)
      // 根据所选分类的ID，和当前所处的面板，获取对应的参数
      const { data: res} = await this.$http.get(`categories/${this.catId}/attributes`, { params: { sel: this.activeName}})
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      // 在赋值之前要对参数数据内的attr_vals属性进行字符串切割成数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制输入文本框
        item.inputVisible = false
        // 要输入的文本
        item.inputValue = ''
      })
      console.log(res.data)
      // 对数据根据不同的展示面板进行判断
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
    },
    // 点击打开 添加参数对话框
    addParanmsDialog () {
      this.addParamsDialogVisible = true
    },
    // 监听对话框关闭时表单数据重置的事件
    addParamsDialogClosed () {
      this.$refs.addParamsFormRef.resetFields()
    },
    // 点击确定提交添加参数数据
    addParams () {
      this.$refs.addParamsFormRef.validate(async valid => {
        if (!valid) return
        const { data: res} = await this.$http.post(`categories/${this.catId}/attributes`, {
          attr_name: this.addParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.addParamsDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击展开 编辑参数对话框
    async showEditDialog (attrId) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.catId}/attributes/${attrId}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }
      this.editParamsForm = res.data
      console.log('当前参数信息', this.editParamsForm)
      this.editParamsDialogVisible = true
    },
    // 点击按钮，修改参数
    editParams () {
      // 先进行预验证
      this.$refs.editParamsFormRef.validate(async valid => {
        if (!valid) return
        const { data: res} = await this.$http.put(`categories/${this.catId}/attributes/${this.editParamsForm.attr_id}`, {
          attr_name: this.editParamsForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数信息失败！')
        }
        this.$message.success('修改参数信息成功！')
        this.getParamsData()
        this.editParamsDialogVisible = false
      })
    },
    // 关闭对话框编辑表单重置事件
    editParamsDialogClosed () {
      this.$refs.editParamsFormRef.resetFields()
    },
    // 删除参数事件
    async removeParams (attrId) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        // 用户取消了删除操作
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        // 删除的业务逻辑
        const { data: res} = await this.$http.delete(`categories/${this.catId}/attributes/${attrId}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除参数失败！')
        }
        this.$message.success('删除参数成功！')
        this.getParamsData()
    },
    async handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // 如果没有return 那么证明输入的内容需要做后续的处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 需要发起请求，保存本次操作
      this.saveAttrvals(row)
    },
    // 删除参数项attr_vals的操作
    async attrvalsTagDelete (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrvals(row)
    },
    // 保存attr_vals的操作
    async saveAttrvals (row) {
      const { data: res} = await this.$http.put(`categories/${this.catId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改参数项失败！')
      }
      this.$message.success('修改参数项成功！')
    },
    // 监听切换标签与输入框的事件
    showInput (row) {
      row.inputVisible = true
      // 让文本框自动获取焦点
      // $nextTick方法的作用就是当元素被重新渲染之后才会执行回调函数中的代码
      this.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    }
  },
  // 计算器
  computed: {
    // 如果selectedKeys选中的长度不等于3，则返回true禁用,如果是则返回false取消禁用
    isBtnDisabled () {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    catId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat-opt{
  margin: 15px 0;
}

.el-tag{
  margin: 10px;
}

.input-new-tag {
  width: 100px;
}
</style>
