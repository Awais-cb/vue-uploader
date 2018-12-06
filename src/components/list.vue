<template>
  <div class="uploader-list">
    <slot :file-list="fileList">
      <div>
        <v-flex lg12 class="uploaderprogress__block">
          <div v-for="file in fileList" :key="file.id">
            <uploader-file :file="file" :list="true"></uploader-file>
          </div>
        </v-flex>
      </div>
    </slot>
  </div>
</template>

<script>
  import { uploaderMixin } from '../common/mixins'
  import UploaderFile from './file.vue'

  const COMPONENT_NAME = 'uploader-list'

  export default {
    name: COMPONENT_NAME,
    mixins: [uploaderMixin],
    computed: {
      fileList () {
        return this.uploader.fileList
      }
    },
    components: {
      UploaderFile
    }
  }
</script>

<style>
  .uploader-list {
    position: relative;
  }
</style>
