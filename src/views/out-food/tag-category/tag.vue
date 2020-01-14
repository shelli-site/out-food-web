<template>
    <el-card class="box-card">
        <div slot="header" style="display: flex;justify-content: space-between;">
            <div>标签管理</div>
            <el-button style="padding: 4px 10px;"
                       v-if="crud.optShow.add"
                       v-permission="permission.add"
                       class="filter-item"
                       size="mini"
                       type="primary"
                       icon="el-icon-plus"
                       @click="crud.toAdd">
                新增
            </el-button>
        </div>
        <div style="margin-bottom: 10px;">
            <!-- 搜索 -->
            <el-input v-model="query.name"
                      clearable
                      placeholder="输入搜索内容"
                      style="width: 200px;"
                      class="filter-item"
                      @keyup.enter.native="crud.toQuery"/>
            <rrOperation :crud="crud"/>
        </div>
        <!--表单组件-->
        <el-dialog
                :close-on-click-modal="false"
                :before-close="crud.cancelCU"
                :visible.sync="crud.status.cu > 0"
                :title="crud.status.title"
                width="500px">
            <el-form ref="form" :model="form" :rules="rules" size="small" label-width="80px">
                <el-form-item label="名称" prop="name">
                    <el-input v-model="form.name" style="width: 370px;"/>
                </el-form-item>
                <el-form-item label="排序">
                    <el-input-number :min="0" :step="10" v-model="form.sort" style="width: 140px;"/>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button type="text" @click="crud.cancelCU">取消</el-button>
                <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
            </div>
        </el-dialog>
        <!--表格渲染-->
        <el-table
                ref="table"
                v-loading="crud.loading"
                :data="crud.data"
                size="small"
                style="width: 100%;"
                @selection-change="crud.selectionChangeHandler">
            <el-table-column type="selection" width="55"/>
            <el-table-column v-if="columns.visible('name')" prop="name" label="名称"/>
            <el-table-column v-if="columns.visible('sort')" prop="sort" label="排序"/>
            <el-table-column
                    v-permission="['admin','tagOrCategory:edit','tagOrCategory:del']"
                    label="操作"
                    width="150px"
                    align="center">
                <template slot-scope="scope">
                    <udOperation :data="scope.row" :permission="permission"/>
                </template>
            </el-table-column>
        </el-table>
        <!--分页组件-->
        <pagination/>
    </el-card>
</template>

<script>
  import crudTagOrCategory from '@/api/tagOrCategory'
  import CRUD, { presenter, header, form, crud } from '@crud/crud'
  import rrOperation from '@crud/RR.operation'
  import crudOperation from '@crud/CRUD.operation'
  import udOperation from '@crud/UD.operation'
  import pagination from '@crud/Pagination'

  // crud交由presenter持有
  const defaultCrud = CRUD({
    title: '标签',
    url: 'api/tagOrCategory',
    sort: ['sort,asc', 'createTime,desc'],
    crudMethod: { ...crudTagOrCategory }
  })
  const defaultForm = { id: null, name: null, type: 'tag', sort: null, createTime: null }
  export default {
    name: 'Tag',
    components: { pagination, crudOperation, rrOperation, udOperation },
    mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
    data() {
      return {
        permission: {
          add: ['admin', 'tagOrCategory:add'],
          edit: ['admin', 'tagOrCategory:edit'],
          del: ['admin', 'tagOrCategory:del']
        },
        rules: {
          name: [
            { required: true, message: '名称不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      // 获取数据前设置好接口地址
      [CRUD.HOOK.beforeRefresh]() {
        const query = this.query
        if (query.type && query.value) {
          this.crud.params[query.type] = query.value
        }
        this.crud.params.type = 'tag'
        return true
      }
    },
    mounted() {
    }
  }
</script>

<style scoped>

</style>
