<template>
    <el-select v-bind="$attrs" v-model="selectValue" @change="d=>{$emit('change',d)}">
        <el-option v-for="option in options" :key="options.id" :value="option.id" :label="option.name"/>
    </el-select>
</template>

<script>
  import { initData } from '@/api/data'

  export default {
    name: 'TagOrCategorySelect',
    props: {
      type: {
        type: 'tag' | 'category',
        default: 'tag'
      }
    },
    data() {
      return {
        selectValue: '',
        options: []
      }
    },
    created() {
      this.getOptionList()
    },
    methods: {
      async getOptionList() {
        const res = await initData('api/tagOrCategory', { type: this.type, paging: false })
        this.options = res
      },
      resetValue() {
        this.selectValue = ''
      },
      initValue(value) {
        this.selectValue = value
        this.$emit('change', this.selectValue)
      }
    }

  }
</script>

<style scoped>

</style>
