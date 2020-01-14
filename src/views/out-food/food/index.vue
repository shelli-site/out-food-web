<template>
    <div class="app-container">
        <!--工具栏-->
        <div class="head-container">
            <div v-if="crud.props.searchToggle">
                <!-- 搜索 -->
                <el-input v-model="query.value" clearable placeholder="输入搜索内容" style="width: 200px;" class="filter-item"
                          @keyup.enter.native="crud.toQuery"/>
                <el-select v-model="query.type" clearable placeholder="类型" class="filter-item" style="width: 130px">
                    <el-option v-for="item in queryTypeOptions" :key="item.key" :label="item.display_name"
                               :value="item.key"/>
                </el-select>
                <rrOperation :crud="crud"/>
            </div>
            <!--如果想在工具栏加入更多按钮，可以使用插槽方式， slot = 'left' or 'right'-->
            <crudOperation :permission="permission"/>
            <!--表单组件-->
            <el-dialog :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0"
                       :title="crud.status.title" width="980px">
                <el-form ref="form" :model="form" :rules="rules" status-icon size="small" label-width="80px"
                         v-loading="status.getData">
                    <el-form-item label="菜品名称" prop="name">
                        <el-input v-model="form.name" style="width: 380px"/>
                    </el-form-item>
                    <el-form-item label="菜品类型">
                        <tag-or-category-select ref="typeSelect" clearable multiple type="category"
                                                placeholder="请选择菜品类型"
                                                @change="data=>{form.type = data?data.join(','):''}"
                                                style="width: 380px"/>
                    </el-form-item>
                    <el-form-item label="标签">
                        <tag-or-category-select ref="tagSelect" clearable multiple type="tag"
                                                placeholder="请选择菜品标签"
                                                @change="data=>{form.tagsOrCategorys = data?data.join(','):''}"
                                                style="width: 380px"/>
                    </el-form-item>
                    <div style="display: flex">
                        <el-form-item label="价格" prop="price">
                            <el-input-number controls-position="right" :precision="2" :min="0" :step="1"
                                             v-model="form.price"
                                             style="width: 120px"/>
                            <span class="form-item-unit">元</span>
                        </el-form-item>
                        <el-form-item label="推荐等级" style="margin-left: 100px;">
                            <el-rate style="padding-top: 6px;"
                                     v-model="form.recommendationLevel"
                                     show-text
                                     :texts="dict.recommendation_level.map(d=>d.label)"
                                     :colors="['#99A9BF', '#F7BA2A', '#FF9900']"/>
                        </el-form-item>
                    </div>
                    <el-form-item label="详情图片">
                        <upload ref="uploadPic" @update="data=>{form.pictureIds = data?data.join(','):''}"/>
                        <!--form.pictureIds-->
                    </el-form-item>
                    <el-form-item label="描述">
                        <el-input :rows="3" v-model="form.description" type="textarea" style="width: 480px"/>
                        <div class="form-item-des">这个描述是指用户会看到的商品介绍</div>
                    </el-form-item>
                    <el-form-item label="额外信息">
                        <el-input :rows="3" v-model="form.information" type="textarea" style="width: 480px"/>
                        <div class="form-item-des">这个额外信息可以记一些自己对商品的备注（客户不会看到）</div>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button type="text" @click="crud.cancelCU">取消</el-button>
                    <el-button :loading="crud.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
                </div>
            </el-dialog>
            <!--表格渲染-->
            <el-table ref="table" v-loading="crud.loading" :data="crud.data" size="small" style="width: 100%;"
                      @selection-change="crud.selectionChangeHandler">
                <el-table-column type="selection" width="55"/>
                <el-table-column v-if="columns.visible('name')" prop="name" label="菜品名称"/>
                <el-table-column v-if="columns.visible('type')" prop="type" label="菜品类型">
                    <template slot-scope="scope">
                        <el-tag style="margin-right: 5px;"
                                v-for="(type, index) in scope.row.typeNames||[]"
                                size="mini"
                                :key="index"
                                effect="dark">
                            {{ type }}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column v-if="columns.visible('price')" prop="price" label="价格">
                    <template slot-scope="scope">
                        {{scope.row.price&&`￥${parseFloat(scope.row.price).toFixed(2)}`}}
                    </template>
                </el-table-column>
                <el-table-column v-if="columns.visible('tagsOrCategorys')" prop="tagsOrCategorys" label="标签">
                    <template slot-scope="scope">
                        <el-tag style="margin-right: 5px;"
                                v-for="(tag, index) in scope.row.tagNames||[]"
                                :key="index">
                            {{ tag }}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column v-if="columns.visible('recommendationLevel')" prop="recommendationLevel"
                                 label="推荐等级" width="300">
                    <template slot-scope="scope">
                        <el-rate v-if="scope.row.recommendationLevel"
                                 :value="parseFloat(scope.row.recommendationLevel)"
                                 disabled
                                 show-score
                                 text-color="#ff9900"
                                 score-template="{value}">
                        </el-rate>
                    </template>
                </el-table-column>
                <el-table-column v-if="columns.visible('monthlySalesVolume')" prop="monthlySalesVolume" label="月销量"/>
                <el-table-column v-permission="['admin','food:edit','food:del']" label="操作" width="150px"
                                 align="center">
                    <template slot-scope="scope">
                        <udOperation
                                :data="scope.row"
                                :permission="permission"
                        />
                    </template>
                </el-table-column>
            </el-table>
            <!--分页组件-->
            <pagination/>
        </div>
        <!--<food-form :form="form" :crud="crud" @primary="crud.submitCU"/>-->
    </div>
</template>

<script>
  import crudFood from '@/api/food'
  import CRUD, { crud, form, header, presenter } from '@crud/crud'
  import rrOperation from '@crud/RR.operation'
  import crudOperation from '@crud/CRUD.operation'
  import udOperation from '@crud/UD.operation'
  import pagination from '@crud/Pagination'
  import upload from '../components/upload/upload'
  import TagOrCategorySelect from '../components/TagOrCategorySelect/TagOrCategorySelect'

  // crud交由presenter持有
  const defaultCrud = CRUD({
    title: '菜品',
    url: 'api/food',
    sort: ['sort,asc', 'createTime,desc'],
    crudMethod: { ...crudFood },
    optShow: {
      add: true,
      edit: false,
      del: true,
      download: false
    }
  })
  const defaultForm = {
    id: null,
    name: null,
    type: null,
    price: null,
    recommendationLevel: null,
    tagsOrCategorys: null,
    pictureIds: null,
    description: null,
    information: null
  }
  export default {
    name: 'Food',
    components: { pagination, crudOperation, rrOperation, udOperation, upload, TagOrCategorySelect },
    mixins: [presenter(defaultCrud), header(), form(defaultForm), crud()],
    dicts: ['recommendation_level'],
    data() {
      return {
        permission: {
          add: ['admin', 'food:add'],
          edit: ['admin', 'food:edit'],
          del: ['admin', 'food:del']
        },
        rules: {
          name: [{ required: true, message: '请输入菜品名称', trigger: 'blur' }]
        },
        queryTypeOptions: [
          { key: 'name', display_name: '菜品名称' },
          { key: 'type', display_name: '菜品类型' }
        ],
        status: {
          getData: false
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
        return true
      },
      // 清除组件状态
      [CRUD.HOOK.afterSubmit]() {
        this.resetComponents()
        return true
      },
      [CRUD.HOOK.afterAddCancel]() {
        this.resetComponents()
        return true
      },
      [CRUD.HOOK.afterEditCancel]() {
        this.resetComponents()
        return true
      },
      [CRUD.HOOK.beforeToEdit]() {
        this.status.getData = true
      },
      [CRUD.HOOK.afterToEdit]() {
        crudFood.get(this.form.id).then(res => {
          // 修改组件状态
          this.initComponents(res)
          this.crud.resetForm({
            ...res, ...{
              type: res.types.map(t => t.id).join(','),
              tagsOrCategorys: res.tagsOrCategorys.map(t => t.id).join(',')
            }
          })
          this.status.getData = false
        })
      },
      resetComponents() {
        this.$refs['uploadPic'] && this.$refs['uploadPic'].resetFileList()
        this.$refs['typeSelect'] && this.$refs['typeSelect'].resetValue()
        this.$refs['tagSelect'] && this.$refs['tagSelect'].resetValue()
      },
      initComponents(data) {
        this.$refs['uploadPic'] && this.$refs['uploadPic'].initFileList(data.pictures.map(res => ({
          status: 'success',
          url: res.url,
          response: res
        })))
        this.$refs['typeSelect'] && this.$refs['typeSelect'].initValue(data.types.map(type => type.id))
        this.$refs['tagSelect'] && this.$refs['tagSelect'].initValue(data.tagsOrCategorys.map(tag => tag.id))
      }
    }
  }
</script>

<style lang="scss" scoped>
    @import "../outfood";
</style>
