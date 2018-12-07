<template>
  <div class="uploader-file">
       <slot
      :file="file"
      :list="list"
      :status="status"
      :paused="paused"
      :error="error"
      :average-speed="averageSpeed"
      :formated-average-speed="formatedAverageSpeed"
      :current-speed="currentSpeed"
      :is-complete="isComplete"
      :is-uploading="isUploading"
      :size="size"
      :formated-size="formatedSize"
      :uploaded-size="uploadedSize"
      :progress="progress"
      :progress-style="progressStyle"
      :progressing-class="progressingClass"
      :time-remaining="timeRemaining"
      :formated-time-remaining="formatedTimeRemaining"
      :type="type"
      :extension="extension"
      :file-category="fileCategory"
      >       
        <div class="inner__uploaderprogress">
          <div class="uploader__progress my-2">
            <v-toolbar  class="uploader_details py-1 px-1">
              <div class="v-toolbar-title videothum">
                <img src="../images/VID.png">
              </div>
              <v-spacer class="text my-1 mx-2">
                <div class="uploader_title">
                  <p class="subheading font-weight-medium">
                    {{file.name}}
                  </p>  
                </div>
                <div class="progress">
                  <v-progress-linear :color="getProgressColor(status)" height="2" :value="progressStyle.progress"></v-progress-linear>
                </div>
                <div class="uploader_text">
                  <template v-if="status !== 'uploading'">
                      <div class="caption">
                        <p v-bind:class="getProgressClass(status)+' d-inline mx-2'">
                          {{statusText}}
                        </p>
                      </div>
                  </template>
                  <template v-if="status === 'uploading'">
                    <div class="caption">
                        File size {{formatedSize}}
                      <p v-bind:class="getProgressClass(status)+' d-inline mx-2'">
                         - Progress {{progressStyle.progress}}
                      </p>
                      <p v-bind:class="getProgressClass(status)+' d-inline mx-2'">
                         - Avarage speed {{formatedAverageSpeed}}
                      </p>
                      <p v-bind:class="getProgressClass(status)+' d-inline mx-2'">
                         - Remaining time {{formatedTimeRemaining}}
                      </p>
                    </div>
                  </template>
                </div>
              </v-spacer>
              <div class="v-toolbar-title btnholder">
                <div class="text-xs-center">        
                  <v-btn round color="primary" dark @click="dialog = true" v-if="status === 'success'">
                    <v-icon class="edit">create</v-icon>
                  </v-btn>
                  <v-btn round color="primary" dark @click="pause" class="uploader-file-pause" v-if="status === 'uploading'">
                    <v-icon>pause_circle_filled</v-icon>
                  </v-btn>
                  <v-btn round color="primary" dark @click="resume" class="uploader-file-resume" v-if="status === 'paused'">
                    <v-icon>play_circle_filled</v-icon>
                  </v-btn>
                  <v-btn round color="primary" dark  @click="retry" class="uploader-file-retry" v-if="status === 'error'">
                    <v-icon>replay</v-icon>
                  </v-btn>
                  <v-btn round color="primary" dark @click="remove" class="uploader-file-remove" v-if="status === 'error' || status === 'uploading' || status === 'paused' || status === 'waiting'">
                    <v-icon>delete_sweep</v-icon>
                  </v-btn>
                </div>
              </div>
            </v-toolbar>
          </div>
        </div>
    </slot>
</div>
</template>

<script>
  import Uploader from 'simple-uploader.js'
  import events from '../common/file-events'
  import { secondsToStr } from '../common/utils'

  const COMPONENT_NAME = 'uploader-file'

  export default {
    name: COMPONENT_NAME,
    props: {
      file: {
        type: Object,
        default () {
          return {}
        }
      },
      list: {
        type: Boolean,
        default: false
      }
    },
    data () {
      return {
        paused: false,
        error: false,
        averageSpeed: 0,
        currentSpeed: 0,
        isComplete: false,
        isUploading: false,
        size: 0,
        formatedSize: '',
        uploadedSize: 0,
        progress: 0,
        timeRemaining: 0,
        type: '',
        extension: '',
        progressingClass: ''
      }
    },
    computed: {
      fileCategory () {
        const extension = this.extension
        const isFolder = this.file.isFolder
        let type = isFolder ? 'folder' : 'unknown'
        const categoryMap = this.file.uploader.opts.categoryMap
        const typeMap = categoryMap || {
          image: ['gif', 'jpg', 'jpeg', 'png', 'bmp', 'webp'],
          video: ['mp4', 'm3u8', 'rmvb', 'avi', 'swf', '3gp', 'mkv', 'flv'],
          audio: ['mp3', 'wav', 'wma', 'ogg', 'aac', 'flac'],
          document: ['doc', 'txt', 'docx', 'pages', 'epub', 'pdf', 'numbers', 'csv', 'xls', 'xlsx', 'keynote', 'ppt', 'pptx']
        }
        Object.keys(typeMap).forEach((_type) => {
          const extensions = typeMap[_type]
          if (extensions.indexOf(extension) > -1) {
            type = _type
          }
        })
        return type
      },
      progressStyle () {
        const progress = Math.floor(this.progress * 100)
        const style = `translateX(${Math.floor(progress - 100)}%)`
        return {
          progress: `${progress}%`,
          webkitTransform: style,
          mozTransform: style,
          msTransform: style,
          transform: style
        }
      },
      formatedAverageSpeed () {
        return `${Uploader.utils.formatSize(this.averageSpeed)} / s`
      },
      status () {
        const isUploading = this.isUploading
        const isComplete = this.isComplete
        const isError = this.error
        const paused = this.paused
        if (isComplete) {
          return 'success'
        } else if (isError) {
          return 'error'
        } else if (isUploading) {
          return 'uploading'
        } else if (paused) {
          return 'paused'
        } else {
          return 'waiting'
        }
      },
      statusText () {
        const status = this.status
        return this.file.uploader.fileStatusText[status] || status
      },
      formatedTimeRemaining () {
        const timeRemaining = this.timeRemaining
        const file = this.file
        if (timeRemaining === Number.POSITIVE_INFINITY || timeRemaining === 0) {
          return ''
        }
        let parsedTimeRemaining = secondsToStr(timeRemaining)
        const parseTimeRemaining = file.uploader.opts.parseTimeRemaining
        if (parseTimeRemaining) {
          parsedTimeRemaining = parseTimeRemaining(timeRemaining, parsedTimeRemaining)
        }
        return parsedTimeRemaining
      }
    },
    watch: {
      status (newStatus, oldStatus) {
        if (oldStatus && newStatus === 'uploading' && oldStatus !== 'uploading') {
          this.tid = setTimeout(() => {
            this.progressingClass = 'uploader-file-progressing'
          }, 200)
        } else {
          clearTimeout(this.tid)
          this.progressingClass = ''
        }
      }
    },
    methods: {
      _actionCheck () {
        this.paused = this.file.paused
        this.error = this.file.error
        this.isUploading = this.file.isUploading()
      },
      pause () {
        this.file.pause()
        this._actionCheck()
        this._fileProgress()
      },
      resume () {
        this.file.resume()
        this._actionCheck()
      },
      remove () {
        this.file.cancel()
      },
      retry () {
        this.file.retry()
        this._actionCheck()
      },
      fileEventsHandler (event, args) {
        const rootFile = args[0]
        const file = args[1]
        const target = this.list ? rootFile : file
        if (this.file === target) {
          if (this.list && event === 'fileSuccess') {
            return
          }
          this[`_${event}`].apply(this, args)
        }
      },
      _fileProgress () {
        this.progress = this.file.progress()
        this.averageSpeed = this.file.averageSpeed
        this.currentSpeed = this.file.currentSpeed
        this.timeRemaining = this.file.timeRemaining()
        this.uploadedSize = this.file.sizeUploaded()
        this._actionCheck()
      },
      _fileSuccess () {
        this._fileProgress()
        this.error = false
        this.isComplete = true
        this.isUploading = false
      },
      _fileComplete () {
        this._fileSuccess()
      },
      _fileError () {
        this._fileProgress()
        this.error = true
        this.isComplete = false
        this.isUploading = false
      },
      getProgressColor (status) {
        if (status === 'waiting' || status === 'uploading') {
          return 'info'
        } else if (status === 'success') {
          return 'success'
        } else if (status === 'paused') {
          return 'warning'
        } else if (status === 'error') {
          return 'error'
        }
        return 'info'
      },
      getProgressClass (status) {
        if (status === 'waiting' || status === 'uploading') {
          return 'notifation_Progress'
        } else if (status === 'success') {
          return 'notifation_success'
        } else if (status === 'paused') {
          return 'notifation_warning'
        } else if (status === 'error') {
          return 'notifation_error'
        }
        return 'notifation_Progress'
      }
    },
    mounted () {
      const staticProps = ['paused', 'error', 'averageSpeed', 'currentSpeed']
      const fnProps = [
        'isComplete',
        'isUploading',
        {
          key: 'size',
          fn: 'getSize'
        },
        {
          key: 'formatedSize',
          fn: 'getFormatSize'
        },
        {
          key: 'uploadedSize',
          fn: 'sizeUploaded'
        },
        'progress',
        'timeRemaining',
        {
          key: 'type',
          fn: 'getType'
        },
        {
          key: 'extension',
          fn: 'getExtension'
        }
      ]
      staticProps.forEach(prop => {
        this[prop] = this.file[prop]
      })
      fnProps.forEach((fnProp) => {
        if (typeof fnProp === 'string') {
          this[fnProp] = this.file[fnProp]()
        } else {
          this[fnProp.key] = this.file[fnProp.fn]()
        }
      })

      const handlers = this._handlers = {}
      const eventHandler = (event) => {
        handlers[event] = (...args) => {
          this.fileEventsHandler(event, args)
        }
        return handlers[event]
      }
      events.forEach((event) => {
        this.file.uploader.on(event, eventHandler(event))
      })
    },
    destroyed () {
      events.forEach((event) => {
        this.file.uploader.off(event, this._handlers[event])
      })
      this._handlers = null
    }
  }
</script>