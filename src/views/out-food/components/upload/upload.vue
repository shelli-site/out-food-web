<template>
    <div>
        <el-upload :action="uploadUrl"
                   :headers="header"
                   list-type="picture-card"
                   :file-list="upLoad.fileList"
                   :show-file-list="true"
                   :on-success="handleAvatarSuccess"
                   :before-upload="beforeAvatarUpload"
                   :on-change="fileChange"
                   :auto-upload="true">
            <i slot="default" class="el-icon-plus"></i>
            <div slot="file" slot-scope="{file}"
                 style="position:relative;top: 50%;transform: translateY(-50%);">
                <el-progress v-if="file.status!=='success'" type="circle"
                             :percentage="file.percentage===100?99:file.percentage"
                             class="upload-progress"></el-progress>
                <template v-else>
                    <img class="el-upload-list__item-thumbnail"
                         :src="file.url" alt="">
                    <span class="el-upload-list__item-actions">
                    <span class="el-upload-list__item-preview"
                          @click="handlePictureCardPreview(file)">
                    <i class="el-icon-zoom-in"></i></span>
                    <span v-if="!upLoad.disabled"
                          class="el-upload-list__item-delete"
                          @click="handleRemove(file)">
                    <i class="el-icon-delete"></i>
                    </span>
                    </span>
                </template>
            </div>
        </el-upload>

        <el-image ref="img"
                  style="display:block;width: 0px; height: 0px;line-height:0px;"
                  :src="''" fit="contain"
                  :preview-src-list="[upLoad.dialogImageUrl]">
        </el-image>
    </div>
</template>

<script>
  export default {
    name: 'upload',
    data() {
      return {
        upLoad: {
          dialogImageUrl: '',
          dialogVisible: false,
          disabled: false,
          fileList: []
        }
      }
    },
    computed: {
      uploadUrl() {
        return this.$store.getters.imagesUploadApi
      },
      header() {
        return {
          Authorization: this.$store.getters.token
        }
      }
    },
    methods: {
      handleRemove(file) {
        this.upLoad.fileList.splice(this.upLoad.fileList.findIndex(f => f == file), 1)
      },
      handleAvatarSuccess(response, file, fileList) {
        this.upLoad.fileList = fileList
      },
      beforeAvatarUpload(file) {
        return true
      },
      handlePictureCardPreview(file) {
        this.upLoad.dialogImageUrl = file.url
        this.$refs['img'].showViewer = true
      },
      fileChange(file, fileList) {
        this.upLoad.fileList = fileList
        this.$emit('update', fileList.reduce((previous, current) => {
          (current.status === 'success') && previous.push(current.response.id)
          return previous
        }, []))
      },
      resetFileList() {
        this.upLoad.fileList = []
      },
      initFileList(fileList) {
        this.upLoad.fileList = fileList
        this.$emit('update', fileList.reduce((previous, current) => {
          (current.status === 'success') && previous.push(current.response.id)
          return previous
        }, []))
      },
      getPercentage(file) {
        console.log(JSON.parse(JSON.stringify(file)))
      }
    }
  }
</script>

<style scoped>
    .upload-progress {
        /*background-color: #1f2d3d;*/
    }
</style>
