<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <label class="el-form-item-label">链接地址</label>
        <el-input v-model="query.url" clearable placeholder="链接地址" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">标题</label>
        <el-input v-model="query.title" clearable placeholder="标题" style="width: 185px;" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <label class="el-form-item-label">分类</label>
        <el-select v-model="query.kind" filterable placeholder="请选择">
          <el-option
            v-for="item in dict.nav_kind"
            :key="item.id"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
        <rrOperation :crud="crud" />
      </div>
      <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
      <crudOperation :permission="permission" />
      <!--表单组件-->
      <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="500px">
        <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
          <el-form-item label="链接地址" prop="url">
            <el-input v-model="form.url" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="标题">
            <el-input v-model="form.title" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="描述">
            <el-input v-model="form.description" :rows="3" type="textarea" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="logo">
            <el-input v-model="form.logo" style="width: 370px;" />
          </el-form-item>
          <el-form-item label="分类">
            <el-select v-model="form.kind" filterable placeholder="请选择">
              <el-option
                v-for="item in dict.nav_kind"
                :key="item.id"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
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
        <el-table-column prop="logo" label="logo" width="200">
          <template slot-scope="scope">
            <el-image
              style="width: 30px; height: 30px"
              :src="scope.row.logo"
              :fit="contain"
            />
          </template>
        </el-table-column>
        <el-table-column prop="kind" label="分类">
          <template slot-scope="scope">
            {{ dict.label.nav_kind[scope.row.kind] }}
          </template>
        </el-table-column>
        <el-table-column prop="url" label="链接地址" />
        <el-table-column prop="title" label="标题" />
        <!-- <el-table-column prop="description" label="描述" /> -->
        <el-table-column v-if="checkPer(['admin','blogNav:edit','blogNav:del'])" label="操作" width="150px" align="center">
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
import crudBlogNav from '@/api/blogNav'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'

const defaultForm = { id: null, url: null, title: null, description: null, logo: null, createTime: null, updateTime: null, kind: null }
export default {
  name: 'BlogNav',
  components: { pagination, crudOperation, rrOperation, udOperation },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  dicts: ['nav_kind'],
  cruds() {
    return CRUD({ title: '博客导航栏', url: 'api/blogNav', idField: 'id', sort: 'id,desc', crudMethod: { ...crudBlogNav }})
  },
  data() {
    return {
      permission: {
        add: ['admin', 'blogNav:add'],
        edit: ['admin', 'blogNav:edit'],
        del: ['admin', 'blogNav:del']
      },
      rules: {
        id: [
          { required: true, message: 'id不能为空', trigger: 'blur' }
        ],
        url: [
          { required: true, message: '链接地址不能为空', trigger: 'blur' }
        ]
      },
      queryTypeOptions: [
        { key: 'url', display_name: '链接地址' },
        { key: 'title', display_name: '标题' },
        { key: 'kind', display_name: '分类' }
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
