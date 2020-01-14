<template>
    <div class="app-container">
        <el-row :gutter="10">
            <el-col :xs="24" :sm="24" :md="10" :lg="10" :xl="10" style="margin-bottom: 10px">
                <category/>
            </el-col>
            <el-col :xs="24" :sm="24" :md="14" :lg="14" :xl="14" style="margin-bottom: 10px">
                <tag/>
            </el-col>
        </el-row>
    </div>
</template>

<script>
  import Category from './category'
  import Tag from './tag'

  export default {
    name: 'TagOrCategory',
    components: { Category, Tag }

  }
</script>

<style scoped>

</style>
