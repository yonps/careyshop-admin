<template>
  <div class="cs-p">
    <el-form
      :inline="true"
      size="small">
      <el-form-item v-if="auth.add">
        <el-button
          icon="el-icon-plus"
          :disabled="loading"
          @click="handleCreate">新增等级</el-button>
      </el-form-item>

      <el-form-item>
        <el-button-group>
          <el-button
            v-if="auth.del"
            icon="el-icon-delete"
            :disabled="loading"
            @click="handleDelete(null)">删除</el-button>

          <el-button
            icon="el-icon-refresh"
            :disabled="loading"
            @click="$emit('refresh')">刷新</el-button>
        </el-button-group>
      </el-form-item>

      <cs-help
        :router="$route.path"
        style="padding-bottom: 19px;">
      </cs-help>
    </el-form>

    <el-table
      :data="currentTableData"
      :highlight-current-row="true"
      @selection-change="handleSelectionChange">
      <el-table-column align="center" type="selection" width="55"/>

      <el-table-column
        label="名称"
        prop="name">
        <template slot-scope="scope">
          <span>{{scope.row.name}}</span>
          <el-image
            v-if="scope.row.icon"
            class="level-icon"
            :src="scope.row.icon"
            fit="fill">
            <div slot="error" class="image-slot">
              <i class="el-icon-picture-outline"/>
            </div>
          </el-image>
        </template>
      </el-table-column>

      <el-table-column
        label="消费金额"
        prop="amount">
      </el-table-column>

      <el-table-column
        label="折扣(%)"
        prop="discount">
      </el-table-column>

      <el-table-column
        label="描述"
        prop="description"
        min-width="220">
      </el-table-column>

      <el-table-column
        label="操作"
        align="center"
        min-width="100">
        <template slot-scope="scope">
          <el-button
            v-if="auth.set"
            @click="handleUpdate(scope.$index)"
            size="small"
            type="text">编辑</el-button>

          <el-button
            v-if="auth.del"
            @click="handleDelete(scope.$index)"
            size="small"
            type="text">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
      :append-to-body="true"
      :close-on-click-modal="false"
      width="600px">
      <el-form
        :model="form"
        :rules="rules"
        ref="form"
        label-width="80px">
        <el-form-item
          label="名称"
          prop="name">
          <el-input
            v-model="form.name"
            placeholder="请输入等级名称"
            :clearable="true"/>
        </el-form-item>

        <el-form-item
          label="等级图标"
          prop="icon">
          <el-input
            v-model="form.icon"
            placeholder="可输入等级图标"
            :clearable="true">
            <el-dropdown
              slot="append"
              :show-timeout="50"
              @command="handleCommand">
              <el-button icon="el-icon-upload"/>
              <el-dropdown-menu slot="dropdown">
                <el-dropdown-item command="storage" icon="el-icon-finished">资源选择</el-dropdown-item>
                <el-dropdown-item command="upload" icon="el-icon-upload2">上传资源</el-dropdown-item>
              </el-dropdown-menu>
            </el-dropdown>
          </el-input>
        </el-form-item>

        <el-row>
          <el-col :span="12">
            <el-form-item
              label="消费金额"
              prop="amount">
              <el-input-number
                v-model="form.amount"
                controls-position="right"
                :precision="2"
                :min="0"/>
            </el-form-item>
          </el-col>

          <el-col :span="12">
            <el-form-item
              label="折扣(%)"
              prop="discount">
              <el-input-number
                v-model="form.discount"
                controls-position="right"
                :min="0"
                :max="100"/>
            </el-form-item>
          </el-col>
        </el-row>

        <el-form-item
          label="描述"
          prop="description">
          <el-input
            v-model="form.description"
            placeholder="可输入等级描述"
            type="textarea"
            :rows="3">
          </el-input>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button
          @click="dialogFormVisible = false"
          size="small">取消</el-button>

        <el-button
          v-if="dialogStatus === 'create'"
          type="primary"
          :loading="dialogLoading"
          @click="create"
          size="small">确定</el-button>

        <el-button
          v-else type="primary"
          :loading="dialogLoading"
          @click="update"
          size="small">修改</el-button>
      </div>

      <cs-storage
        ref="storage"
        style="display: none;"
        :limit="1"
        @confirm="_getStorageFileList">
      </cs-storage>

      <cs-upload
        style="display: none;"
        ref="upload"
        type="slot"
        accept="image/*"
        :limit="1"
        :multiple="false"
        @confirm="_getUploadFileList">
      </cs-upload>
    </el-dialog>
  </div>
</template>

<script>
import {
  addUserLevelItem,
  delUserLevelList,
  setUserLevelItem
} from '@/api/user/level'
import util from '@/utils/util'

export default {
  components: {
    'csUpload': () => import('@/components/cs-upload'),
    'csStorage': () => import('@/components/cs-storage')
  },
  props: {
    loading: {
      default: false
    },
    tableData: {
      default: () => []
    }
  },
  data() {
    return {
      currentTableData: [],
      multipleSelection: [],
      auth: {
        add: false,
        set: false,
        del: false
      },
      dialogLoading: false,
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: '编辑等级',
        create: '新增等级'
      },
      form: {
        name: undefined,
        icon: undefined,
        amount: undefined,
        discount: undefined,
        description: undefined
      },
      rules: {
        name: [
          {
            required: true,
            message: '名称不能为空',
            trigger: 'blur'
          },
          {
            max: 30,
            message: '长度不能大于 30 个字符',
            trigger: 'blur'
          }
        ],
        icon: [
          {
            max: 512,
            message: '长度不能大于 200 个字符',
            trigger: 'blur'
          }
        ],
        amount: [
          {
            required: true,
            message: '消费金额不能为空',
            trigger: 'blur'
          }
        ],
        discount: [
          {
            required: true,
            message: '折扣不能为空',
            trigger: 'blur'
          }
        ],
        description: [
          {
            max: 200,
            message: '长度不能大于 200 个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  watch: {
    tableData: {
      handler(val) {
        this.currentTableData = val
      },
      immediate: true
    }
  },
  mounted() {
    this._validationAuth()
  },
  methods: {
    // 验证权限
    _validationAuth() {
      this.auth.add = this.$permission('/member/user/level/add')
      this.auth.set = this.$permission('/member/user/level/set')
      this.auth.del = this.$permission('/member/user/level/del')
    },
    // 获取列表中的编号
    _getIdList(val) {
      if (val === null) {
        val = this.multipleSelection
      }

      let idList = []
      if (Array.isArray(val)) {
        val.forEach(value => {
          idList.push(value.user_level_id)
        })
      } else {
        idList.push(this.currentTableData[val].user_level_id)
      }

      return idList
    },
    // 资源下拉框事件
    handleCommand(command) {
      switch (command) {
        case 'storage':
          this.$refs.storage.handleStorageDlg([0, 2])
          break

        case 'upload':
          this.$refs.upload.handleUploadDlg()
          break
      }
    },
    // 获取上传资源
    _getUploadFileList(files) {
      if (!files.length) {
        return
      }

      const response = files[0].response
      if (!response || response.status !== 200) {
        return
      }

      if (response.data[0].type === 0) {
        this.form.icon = util.checkUrl(response.data[0].url)
      }
    },
    // 获取选择资源
    _getStorageFileList(files) {
      if (!files.length) {
        return
      }

      for (const value of files) {
        if (value.type === 0) {
          this.form.icon = util.checkUrl(value.url)
          break
        }
      }
    },
    // 选中数据项
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    // 批量删除
    handleDelete(val) {
      let user_level_id = this._getIdList(val)
      if (user_level_id.length === 0) {
        this.$message.error('请选择要操作的数据')
        return
      }

      this.$confirm('确定要执行该操作吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        closeOnClickModal: false
      })
        .then(() => {
          delUserLevelList(user_level_id)
            .then(() => {
              util.deleteDataList(this.currentTableData, user_level_id, 'user_level_id')
              this.$message.success('操作成功')
            })
        })
        .catch(() => {
        })
    },
    // 弹出新建对话框
    handleCreate() {
      this.form = {
        name: '',
        amount: 0,
        icon: '',
        discount: 0,
        description: ''
      }

      this.$nextTick(() => {
        if (this.$refs.form) {
          this.$refs.form.clearValidate()
        }

        this.dialogStatus = 'create'
        this.dialogLoading = false
        this.dialogFormVisible = true
      })
    },
    // 请求创建
    create() {
      this.$refs.form.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          addUserLevelItem({ ...this.form })
            .then(res => {
              this.currentTableData.push(res.data)
              this.dialogFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    },
    // 弹出编辑对话框
    handleUpdate(index) {
      this.currentIndex = index
      this.form = { ...this.currentTableData[index] }

      this.$nextTick(() => {
        if (this.$refs.form) {
          this.$refs.form.clearValidate()
        }

        this.dialogStatus = 'update'
        this.dialogLoading = false
        this.dialogFormVisible = true
      })
    },
    // 请求编辑
    update() {
      this.$refs.form.validate(valid => {
        if (valid) {
          this.dialogLoading = true
          setUserLevelItem({ ...this.form })
            .then(res => {
              this.$set(
                this.currentTableData,
                this.currentIndex,
                {
                  ...this.currentTableData[this.currentIndex],
                  ...res.data
                })

              this.dialogFormVisible = false
              this.$message.success('操作成功')
            })
            .catch(() => {
              this.dialogLoading = false
            })
        }
      })
    }
  }
}
</script>

<style scoped>
  .level-icon {
    margin-left: 5px;
    line-height: 0;
    vertical-align: text-bottom;
  }
</style>
