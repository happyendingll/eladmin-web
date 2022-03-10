<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">题号</label>
        <el-input v-model="query.questionNo" clearable placeholder="题号" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">题目标题</label>
        <el-input v-model="query.questionTitle" clearable placeholder="题目标题" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker
          v-model="query.createTime"
          start-placeholder="createTimeStart"
          end-placeholder="createTimeStart"
          class="date-item"
        />
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="题目类型">
            <el-radio v-for="item in dict.question_type" :key="item.id" v-model="form.type" :label="item.value">{{ item.label }}</el-radio>
          </el-form-item>
          <el-form-item label="选项A">
            <el-input v-model="form.optionA" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="选项B">
            <el-input v-model="form.optionB" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="选项C">
            <el-input v-model="form.optionC" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="选项D">
            <el-input v-model="form.optionD" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="答案">
            <el-input v-model="form.answer" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="答案解析">
            <el-input v-model="form.answerExplain" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="题目标题">
            <el-input v-model="form.questionTitle" style="width: 370px;" />
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button type="text" @click="crud.cancelCU">取消</el-button>
          <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
        </div>
      </el-dialog>
      <!--表格渲染-->
      <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;" @selection-change="crud.selectionChangeHandler">
        <el-table-column type="selection" width="55" />
        <el-table-column prop="questionNo" label="题号" />
        <el-table-column prop="type" label="题目类型">
          <template slot-scope="scope">
            {{ dict.label.question_type[scope.row.type] }}
          </template>
        </el-table-column>
        <el-table-column prop="optionA" label="选项A" />
        <el-table-column prop="optionB" label="选项B" />
        <el-table-column prop="optionC" label="选项C" />
        <el-table-column prop="optionD" label="选项D" />
        <el-table-column prop="answer" label="答案" />
        <el-table-column prop="answerExplain" label="答案解析" />
        <el-table-column prop="creator" label="创建者" />
        <el-table-column prop="createTime" label="创建时间" />
        <el-table-column prop="updateTime" label="更新时间" />
        <el-table-column prop="questionTitle" label="题目标题" />
        <el-table-column v-if="checkPer(['admin','busQuestion:edit','busQuestion:del'])" label="操作" width="150px" align="center">
          <template slot-scope="scope">
            <udOperation
              :data="scope.row"
              :permission="permission"
            />
          </template>
        </el-table-column>
      </el-table>
      <!--分页组件-->
      <pagination />
    </div>
  </div>
</template>

<script>
import crudBusQuestion from '@/api/busQuestion'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, questionNo: null, type: null, optionA: null, optionB: null, optionC: null, optionD: null, answer: null, answerExplain: null, creator: null, createTime: null, updateTime: null, questionTitle: null }
export default {
  name: 'BusQuestion',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['question_type'],
  cruds() {
    return CRUD({ title: '业务: 试题', url: 'api/busQuestion', idField: 'id', sort: 'id,desc', crudMethod: { ...crudBusQuestion }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'busQuestion:add'],
        edit: ['admin', 'busQuestion:edit'],
        del: ['admin', 'busQuestion:del']
      },
      rules: {
      },
      queryTypeOptions: [
        { key: 'questionNo', display_name: '题号' },
        { key: 'questionTitle', display_name: '题目标题' }
      ]
    }
  },
  methods: {
    // 钩子：在获取表格数据之前执行，false 则代表不获取数据
    [CRUD.HOOK.beforeRefresh]() {
      return true
    }
  }
}
</script>

<style scoped>

</style>
