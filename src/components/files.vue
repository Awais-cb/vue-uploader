<template>
  <div class="uploader-files">
    <slot :files="files">
      <div>
        <v-flex lg12 class="uploaderprogress__block">
          <div v-for="file in files" :key="file.id">
            <uploader-file :file="file"></uploader-file>
          </div>
      </v-flex>
    </slot>
  </div>
</template>

<script>
  import { uploaderMixin } from '../common/mixins'
  import UploaderFile from './file.vue'

  const COMPONENT_NAME = 'uploader-files'

  export default {
    name: COMPONENT_NAME,
    mixins: [uploaderMixin],
    computed: {
      files () {
        return this.uploader.files
      }
    },
    components: {
      UploaderFile
    }
  }
</script>

<style>
  .uploader-files {
    position: relative;
  }
</style>
