<template>
  <div>
    <el-card>
      <el-row>
        <el-col :span="10">
          <el-form :inline="true" :model="getAPIFrom">
            <el-form-item label="">
              <el-input v-model="apiName" placeholder="请输入用例名" />
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="getAPIListEvent">查询</el-button>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="$router.push({name:'addCaseAPI', params: {case_id:caseId}})">新增
              </el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :offset="10" :span="3">
          <el-select v-model="hostId" placeholder="请选择测试环境">
            <el-option
              v-for="item in hostList"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />

          </el-select>
        </el-col>
      </el-row>

      <el-table :data="apiList" style="width: 100%">
        <el-table-column prop="name" label="用例名">
          <!--<template slot-scope="scope">-->
          <!--<el-link type="primary" @click="$router.push({name:'testAPIAPI',params:{api_id:scope.row.id}})">{{-->
          <!--scope.row.name }}-->
          <!--</el-link>-->
          <!--</template>-->
        </el-table-column>
        <el-table-column prop="method" label="请求方式" />
        <el-table-column prop="url" label="接口地址" />

        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.status"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="updateAPIEvent(scope.row.id, scope.row)"
            />
          </template>
        </el-table-column>
        <el-table-column label="测试结果">
          <template slot-scope="scope">
            <el-link v-if="scope.row.result===2" type="info" disabled>未运行</el-link>
            <el-link
              v-else-if="scope.row.result===1"
              type="success"
              @click="apiResultDialogOpenEvent(scope.row)"
            >查看结果
            </el-link>
            <el-link v-else-if="scope.row.result===0" type="danger" @click="apiResultDialogOpenEvent(scope.row)">运行失败
            </el-link>
          </template>
        </el-table-column>
        <el-table-column show-overflow-tooltip label="操作">
          <template slot-scope="scope">
            <el-button size="mini" type="success" @click="runAPI(scope.row)">运行</el-button>
            <el-button size="mini" @click="$router.push({name:'updateCaseAPI',params:{api_id:scope.row.id}})">修改
            </el-button>
            <el-button type="danger" size="mini" @click="openDeleteAPIDialogEvent(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页器-->
      <el-row>
        <el-col :span="9" :offset="13">
          <el-pagination
            :current-page="currentPage"
            :page-sizes="[10, 20, 30, 50]"
            :page-size="size"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
            @size-change="sizeChangeEvent"
            @current-change="currentChangeEvent"
          />
        </el-col>
      </el-row>
    </el-card>

    <!--新增用例对话框-->
    <el-dialog
      title="新增用例"
      :visible.sync="addAPIDialogVisible"
      width="30%"
      :before-close="addAPIDialogCloseEvent"
    >
      <!--新增用例集表单-->
      <el-form ref="apiForm" label-width="80px" :model="apiForm" :rules="apiRules">
        <el-form-item label="用例名" prop="name">
          <el-input v-model="apiForm.name" />
        </el-form-item>
        <el-form-item label="用例描述" prop="description">
          <el-input v-model="apiForm.description" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addAPIDialogCloseEvent">取 消</el-button>
        <el-button type="primary" @click="addAPIEvent">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改用例集对话框-->
    <el-dialog
      title="修改用例"
      :visible.sync="updateAPIDialogVisible"
      width="30%"
      :before-close="updateAPIDialogCloseEvent"
    >
      <!--修改用例集表单-->
      <el-form ref="apiForm" label-width="100px" :model="apiForm" :rules="apiRules">
        <el-form-item label="用例名" prop="name">
          <el-input v-model="apiForm.name" />
        </el-form-item>
        <el-form-item label="用例描述" prop="description">
          <el-input v-model="apiForm.description" />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="updateAPIDialogCloseEvent">取 消</el-button>
        <el-button type="primary" @click="updateAPIEvent(apiId, apiForm)">确 定</el-button>
      </span>
    </el-dialog>

    <!--删除主机-->
    <el-dialog
      title="删除用例集"
      :visible.sync="deleteAPIDialogVisible"
      width="30%"
      :before-close="deleteAPIDialogCloseEvent"
    >
      <span>是否确认删除该用例集？</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deleteAPIDialogCloseEvent">取 消</el-button>
        <el-button type="primary" @click="deleteAPIEvent">确 定</el-button>
      </span>
    </el-dialog>
    <!--查看测试结果对话框-->
    <el-dialog
      title="测试结果"
      :visible.sync="apiResultDialogVisible"
      width="80%"
      :before-close="apiResultDialogCloseEvent"
    >
      <el-collapse v-model="apiResultCollapseActiveNames">
        <el-collapse-item title="请求报文" name="1">

          <el-form label-width="100px">
            <el-form-item label="请求方法：">
              {{ apiResult.request_method }}
            </el-form-item>
            <el-form-item label="请求url：">
              {{ apiResult.request_url }}
            </el-form-item>
            <el-form-item label="请求头：">
              <json-viewer :value="apiResult.request_headers" />
            </el-form-item>
            <el-form-item label="请求体：">
              <json-viewer :value="apiResult.request_body" />
            </el-form-item>
          </el-form>
        </el-collapse-item>
        <el-collapse-item title="响应报文" name="2">
          <el-form label-width="100px">
            <el-form-item label="响应状态码：">
              {{ apiResult.status_code }}
            </el-form-item>
            <el-form-item label="响应头：">
              <json-viewer :value="apiResult.response_headers" />
            </el-form-item>
            <el-form-item label="响应正文：">
              <json-viewer :value="apiResult.response_body" />
            </el-form-item>
          </el-form>
        </el-collapse-item>
        <el-collapse-item title="断言" name="3">
          <el-form label-width="100px">
            <template v-for="item in apiResult.assert_list">
              <el-form-item :key="item.index" label="断言方式：">
                {{ item.type | assertType }}
              </el-form-item>
              <el-form-item :key="item.index" label="匹配规则：">
                {{ item.pattern }}
              </el-form-item>
              <el-form-item :key="item.index" label="预期结果：">
                {{ item.expect }}
              </el-form-item>
              <el-divider :key="item.index" />
            </template>
            <el-form-item label="通过与否：">
              {{ apiResult.assert_result ? '通过': '不通过' }}
            </el-form-item>
          </el-form>
        </el-collapse-item>
        <el-collapse-item title="接口变量提取" name="4">
          <el-form label-width="100px">
            <template v-for="item in apiResult.relate_list">
              <el-form-item :key="item.index" label="变量名：">
                {{ item.type | relateType }}
              </el-form-item>
              <el-form-item :key="item.index" label="断言方式：">
                {{ item.type | relateType }}
              </el-form-item>
              <el-form-item :key="item.index" label="匹配规则：">
                {{ item.pattern }}
              </el-form-item>
              <el-form-item :key="item.index" label="变量值：">
                {{ item.value }}
              </el-form-item>
              <el-divider :key="item.index" />
            </template>
          </el-form>
        </el-collapse-item>
      </el-collapse>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="apiResultDialogCloseEvent">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  apiListRequest,
  addAPIRequest,
  updateAPIRequest,
  deleteAPIRequest,
  hostListRequest,
  runAPIRequest,
  getAPIResultRequest
} from '@/api/project.js'
import { sleep } from '@/utils/sleep.js'
import { mapGetters } from 'vuex'

export default {
  name: 'APIList',
  data() {
    return {
      apiName: '',
      projectList: [],
      apiList: [],
      apiResult: {},
      CaseId: null,
      apiId: null,
      hostId: null,
      loading: false,
      addAPIDialogVisible: false,
      updateAPIDialogVisible: false,
      deleteAPIDialogVisible: false,
      apiResultDialogVisible: false,
      apiResultCollapseActiveNames: [],
      currentPage: 1,
      hostList: [],
      size: 10,
      total: 0,
      getAPIFrom: {
        projectName: ''
      },
      apiForm: {
        name: '',
        api: '',
        description: ''
      },
      apiRules: {
        name: [{ required: true, message: '请输入环境名称', trigger: 'blur' }],
        api: [{ required: true, message: '请输入主机名称', trigger: 'blur' }],
        description: []
      }

    }
  },
  computed: {
    ...mapGetters([
      'projectId'
    ])
  },
  created() {
    if (this.$route.params.case_id) {
      this.caseId = this.$route.params.case_id
      this.getAPIListEvent()
    }
    this.getHostListEvent()
  },
  methods: {
    getHostListEvent() {
      if (this.projectId) {
        hostListRequest({ project_id: this.projectId }).then(response => {
          this.hostList = response.data.results
          this.$message.success(response.message)
        })
      } else {
        this.$message.error('查询失败主机失败，请重新选择用例集')
      }
    },
    getAPIListEvent() {
      var query = {}
      if (this.caseId) {
        query = { case_id: this.caseId }
      }
      if (this.apiName !== '') {
        query['name'] = this.apiName
      }
      apiListRequest(query).then(response => {
        this.total = response.data.count
        this.apiList = response.data.results
        this.$message.success(response.message)
      })
    },

    openAddAPIDialogEvent() {
      if (this.caseId) {
        this.addAPIDialogVisible = true
      } else {
        this.$message.error('请重新选择用例')
      }
    },
    openUpdateAPIDialogEvent(row) {
      this.apiId = row.id
      const { name, description } = row
      this.apiForm = { name, description }
      this.updateAPIDialogVisible = true
    },
    openDeleteAPIDialogEvent(row) {
      this.apiId = row.id
      this.deleteAPIDialogVisible = true
    },
    addAPIDialogCloseEvent() {
      this.$refs.apiForm.resetFields()
      this.addAPIDialogVisible = false
    },
    updateAPIDialogCloseEvent() {
      this.$refs.apiForm.resetFields()
      this.updateAPIDialogVisible = false
    },
    deleteAPIDialogCloseEvent() {
      this.deleteAPIDialogVisible = false
    },
    apiResultDialogCloseEvent() {
      this.apiResultDialogVisible = false
    },
    apiResultDialogOpenEvent(row) {
      getAPIResultRequest(row.id).then(response => {
        this.apiResult = response.data
        try {
          this.apiResult.request_headers = JSON.parse(this.apiResult.request_headers)
          this.apiResult.response_headers = JSON.parse(this.apiResult.response_headers)
          this.apiResult.response_body = JSON.parse(this.apiResult.response_body)
          this.apiResult.request_body = JSON.parse(this.apiResult.request_body)
        } catch (e) {
          console.log(e)
        }
        this.apiResultDialogVisible = true
      })
    },

    addAPIEvent() {
      this.$refs.apiForm.validate(valid => {
        if (valid) {
          var data = JSON.parse(JSON.stringify(this.apiForm))
          data['suite_id'] = this.suiteId
          addAPIRequest(data).then(res => {
            if (res.code === 20000) {
              this.getAPIListEvent()
              this.addAPIDialogVisible = false
              this.$message.success(res.message)
            } else {
              this.$message.error(res.message)
            }
          })
        } else {
          this.$message.error('数据验证失败')
        }
      })
    },
    updateAPIEvent(id, row) {
      updateAPIRequest(id, row).then(result => {
        this.$message.success(result.message)
        this.updateAPIDialogVisible = false
        this.getAPIListEvent()
      }).catch((result) => {
        sleep(500).then(() => {
          this.getAPIListEvent()
        })
      })
    },
    deleteAPIEvent() {
      deleteAPIRequest(this.apiId).then(response => {
        this.$message.success('删除成功')
        this.deleteAPIDialogCloseEvent()
        this.getAPIListEvent()
      }).catch(res => {
        this.deleteAPIDialogCloseEvent()
      })
    },
    sizeChangeEvent(size) {
      this.size = size
      this.getAPIListEvent()
    },
    currentChangeEvent(page) {
      this.currentPage = page
      this.getAPIListEvent()
    },
    runAPI(row) {
      if (this.hostId) {
        var data = { api_id: row.id, host_id: this.hostId }
        runAPIRequest(data).then(response => {
          this.$message.success('运行成功')
          this.getAPIListEvent()
        })
      } else {
        this.$message.error('请选择运行环境')
      }
    }

  }

}
</script>

<style scoped>

</style>
