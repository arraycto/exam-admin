<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input :placeholder="$t('table.question')" v-model="listQuery.name" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter"/>
      <el-select v-model="listQuery.questionType" :placeholder="$t('table.typeName')" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in questionTypes" :key="item.id" :label="item.name" :value="item.id"/>
      </el-select>
      <el-select v-model="listQuery.subject" :placeholder="$t('table.subjectName')" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in subjects" :key="item.id" :label="item.name" :value="item.id"/>
      </el-select>
      <el-select v-model="listQuery.knowledgePoint" :placeholder="$t('table.pointName')" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in knowledgePoints" :key="item.id" :label="item.name" :value="item.id"/>
      </el-select>
      <el-select v-model="listQuery.questionTag" :placeholder="$t('table.tagName')" clearable class="filter-item" style="width: 130px">
        <el-option v-for="item in questionTags" :key="item.id" :label="item.tagName" :value="item.id"/>
      </el-select>
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">{{ $t('table.search') }}</el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">{{ $t('table.add') }}</el-button>
    </div>

    <el-table
      v-loading="listLoading"
      :key="tableKey"
      :data="list"
      border
      fit
      style="width: 100%;">
      <el-table-column :label="$t('table.id')" prop="id" sortable="custom" align="center" width="100">
        <template slot-scope="scope">
          <span>{{ (listQuery.page-1)*listQuery.size+scope.$index+1 }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.question')" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.typeName')" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.questionTypeName }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.pointName')" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.examingPoint }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.creatorName')" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.creatorName }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.createTime')" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.status')" class-name="status-col" width="100">
        <template slot-scope="scope">
          <span v-if="scope.row.active">{{ $t('table.enable') }}</span>
          <span v-else>{{ $t('table.disable') }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.actions')" align="center" width="300" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="medium" icon="el-icon-edit" @click="handleUpdate(scope.row)">{{ $t('table.edit') }}</el-button>
          <el-button size="medium" type="danger" icon="el-icon-delete" @click="handleDelete(scope.row)">{{ $t('table.delete') }}</el-button>
          <el-button type="danger" @click="handleActive(scope.row)">{{ scope.row.active? $t('table.prohibit'): $t('table.active') }}</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.size" @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" width="40%">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="90px" style="width: 420px; margin-left:50px;">
        <el-form-item :label="$t('table.typeName')" prop="questionTypeId">
          <el-select v-model="temp.questionTypeId" placeholder="请选择题型">
            <el-option
              v-for="item in questionTypes"
              :key="item.id"
              :label="item.name"
              :value="item.id"/>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('table.pointName')" prop="examingPoint">
          <el-cascader
            ref="examingPoint"
            v-model="temp.examingPoint"
            :options="subjects"
            :props="props"
            clearable/>
        </el-form-item>
        <el-form-item :label="$t('table.question')" prop="name">
          <el-input v-model="temp.name"/>
        </el-form-item>
        <el-form-item :label="$t('table.questionContent')">
          <el-input :autosize="{ minRows: 2, maxRows: 4}" v-model="temp.content" type="textarea" placeholder="Please input"/>
        </el-form-item>
        <el-form-item :label="$t('table.tagName')">
          <el-select v-model="temp.tagId" clearable class="filter-item" style="width: 130px">
            <el-option v-for="item in questionTags" :key="item.id" :label="item.tagName" :value="item.id"/>
          </el-select>
        </el-form-item>
        <!-- 编辑不可用 -->
        <template v-if="this.dialogStatus==='create'">
          <el-form-item :label="$t('table.status')">
            <el-radio-group v-model="temp.active">
              <el-radio v-for="item in statusOptions" :label="item.key">{{ item.display_name }}</el-radio>
            </el-radio-group>
          </el-form-item>
        </template>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{ $t('table.cancel') }}</el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">{{ $t('table.confirm') }}</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import { fetchList, createOne, deleteOne, activeOne, updateOne } from '@/api/questions/question'
import { fetchAll as typeList } from '@/api/questions/type'
import { fetchAll as subjectList } from '@/api/questions/subject'
import { fetchAll as tagList } from '@/api/questions/tag'
import waves from '@/directive/waves' // Waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // Secondary package based on el-pagination

const statusOptions = [
  { key: true, display_name: '可用' },
  { key: false, display_name: '禁用' }
]

export default {
  name: 'ComplexTable',
  components: { Pagination },
  directives: { waves },
  filters: {
  },
  data() {
    return {
      props: {
        multiple: true,
        label: 'name',
        value: 'id',
        children: 'points'
      },
      tableKey: 0,
      list: [],
      total: 0,
      questionTypes: [],
      subjects: [],
      questionTags: [],
      listLoading: true,
      listQuery: {
        page: 1,
        size: 10,
        name: undefined,
        questionType: undefined,
        subject: undefined,
        questionTag: undefined,
        knowledgePoint: undefined
      },
      statusOptions,
      temp: {
        name: '',
        content: '',
        duration: 0,
        points: 0,
        answer: '',
        difficulty: 0,
        analysis: '',
        reference: '',
        examingPoint: [],
        keyword: '',
        questionTypeId: 0,
        tagId: '',
        active: true
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      rules: {
        questionTypeId: [{ required: true, message: 'Question Type is required', trigger: 'blur' }],
        name: [{ required: true, message: 'Question Name is required', trigger: 'blur' }],
        examingPoint: [{ required: true, message: 'ExamingPoint is required', trigger: 'blur' }]
      }
    }
  },
  computed: {
    knowledgePoints: function() {
      const subjectId = this.listQuery.subject
      for (const index in this.subjects) {
        if (subjectId == this.subjects[index].id) {
          return this.subjects[index].points
        }
      }
    }
  },
  created() {
    this.getList()
    typeList().then(response => {
      this.questionTypes = response.data.data
    })
    subjectList().then(response => {
      this.subjects = response.data.data
    })
    tagList().then(response => {
      this.questionTags = response.data.data
    })
  },
  methods: {
    getList() {
      this.listLoading = true
      fetchList(this.listQuery).then(response => {
        this.listQuery.name = undefined

        const data = response.data.data
        if (data != null) {
          this.list = data.items
          this.total = data.total
        } else {
          this.list = null
          this.total = 0
        }

        // Just to simulate the time of the request
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    resetTemp() {
      this.temp = {
        name: '',
        content: '',
        duration: 0,
        points: 0,
        answer: '',
        difficulty: 0,
        analysis: '',
        reference: '',
        examingPoint: '',
        keyword: '',
        questionTypeId: '',
        tagId: '',
        active: true
      }
    },
    handleActive(row) {
      const tempData = {
        id: row.id,
        active: !row.active
      }
      activeOne(tempData).then(response => {
        const data = response.data
        if (data.data) {
          this.$notify({
            title: '成功',
            message: '操作成功',
            type: 'success',
            duration: 1500
          })
          this.getList()
        } else {
          this.$notify.error({
            title: '失败',
            message: '操作失败',
            duration: 2000
          })
        }
      })
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const examingPoint = this.$refs['examingPoint'].getCheckedNodes(true).map(item => item.value).join(',')
          const tempData = Object.assign({}, this.temp)
          tempData.examingPoint = examingPoint
          createOne(tempData).then(response => {
            this.dialogFormVisible = false
            const data = response.data
            if (data.data) {
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 1500
              })
              this.getList()
            } else {
              this.$notify.error({
                title: '失败',
                message: '删除失败',
                duration: 2000
              })
            }
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const examingPoint = this.$refs['examingPoint'].getCheckedNodes(true).map(item => item.value).join(',')
          const tempData = Object.assign({}, this.temp)
          tempData.examingPoint = examingPoint
          updateOne(tempData).then(() => {
            for (const v of this.list) {
              if (v.id === this.temp.id) {
                const index = this.list.indexOf(v)
                this.list.splice(index, 1, this.temp)
                break
              }
            }
            this.dialogFormVisible = false
            this.$notify({
              title: '成功',
              message: '更新成功',
              type: 'success',
              duration: 1500
            })
          })
        }
      })
    },
    handleDelete(row) {
      deleteOne(row.id).then(response => {
        const data = response.data
        if (data.data) {
          this.$notify({
            title: '成功',
            message: '删除成功',
            type: 'success',
            duration: 1500
          })
          this.getList()
        } else {
          this.$notify.error({
            title: '失败',
            message: '删除失败',
            duration: 2000
          })
        }
      })
    }
  }
}
</script>
