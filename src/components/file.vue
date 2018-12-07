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
                <v-btn round color="primary" dark @click="pause" v-if="status === 'uploading'">
                  <v-icon>pause_circle_filled</v-icon>
                </v-btn>
                <v-btn round color="primary" dark @click="resume" v-if="status === 'paused'">
                  <v-icon>play_circle_filled</v-icon>
                </v-btn>
                <v-btn round color="primary" dark  @click="retry" v-if="status === 'error'">
                  <v-icon>replay</v-icon>
                </v-btn>
                <v-btn round color="primary" dark @click="remove" v-if="status === 'error' || status === 'uploading' || status === 'paused' || status === 'waiting'">
                  <v-icon>delete_sweep</v-icon>
                </v-btn>
              </div>
            </div>
          </v-toolbar>
        </div>
      </div>
      <v-layout row justify-center>
        <!-- ***************************** -->
        <!-- Edit video block -->
        <!-- ***************************** -->
        <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition">
          <v-card>
                <v-toolbar dark color="primary">
                  <v-btn icon dark @click="dialog = false">
                    <v-icon>close</v-icon>
                  </v-btn>
                  <v-toolbar-title>Settings</v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-toolbar-items>
                    <v-btn dark flat @click="dialog = false">Save</v-btn>
                  </v-toolbar-items>
                </v-toolbar>
            <v-container grid-list-md>
              <div class="headline d-block text__Required">Required fields</div>
                <v-layout class="Uploaderformopen" py-4 px-3 fill-height>
                  <v-flex lg8 md8 xs12 sm12>
                    <div class="my-2">
                      <v-text-field v-model="title" label="Title"></v-text-field>
                    </div>
                    <div class="my-2">
                      <v-text-field v-model="description" label="Description"></v-text-field>
                    </div>
                    <div class="my-2">
                      <v-text-field v-model="tags" label="Tags" tags clearable>
                      </v-text-field>
                    </div>
                    <div class="my-2">
                      <v-btn slot="activator" color="primary" dark @click="dialog2 = true">
                        Subtitles Section
                      </v-btn>
                      <v-btn slot="activator" color="primary" dark @click="dialog3 = true">
                        Date recorded & Location
                      </v-btn>
                      <v-btn slot="activator" color="primary" dark @click="dialog4 = true">
                        Sharing and privacy options
                      </v-btn>
                    </div>
                    <v-layout row wrap mt-4 class="uploadthumbsforms">
                      <v-flex lg12 class="upload__thumb">
                        <div class="text-md-center my-2">
                          <v-btn class="uploadinput">
                            <v-icon>control_point</v-icon>
                            Add new Thumb
                          </v-btn>
                        </div>
                      </v-flex>
                      <!-- thumb area (show dynamic thumbs here) -->
                      <!-- <v-flex lg3 class="Video_Thumbnails">
                        <div class="selected_img">
                          <div class="vodthumb">
                            <div class="on_hover">
                              <v-btn flat small><v-icon>delete</v-icon></v-btn>
                            </div>
                            <img class="v-image__image" src="~/static/images/defultthumb/2.png">
                          </div>
                          <v-radio-group v-model="radioGroup">
                            <v-radio></v-radio>
                          </v-radio-group>
                          <span class="caption">thumb-1.png</span>
                        </div>
                      </v-flex> -->
                      <!-- thumb area (show dynamic thumbs here) -->
                    </v-layout>
                  </v-flex>
                  <v-flex lg4 md4 xs12 sm12 ml-5>
                    <div class="Video__Category">
                      <div class="headline">
                        Video Category
                      </div>
                      <span class="caption">You May Select Up To 4 Categories</span>
                      <div class="select_Video__Category app-card mb-4">
                          <div class="ais-refinement-list">
                            <div class="list-item-inner my-2 py-2">
                              <div class="list-item-one">
                                <div class="v-toolbar__content"> 
                                  <div class="v-toolbar__title">
                                    <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                  </div>
                                  <span class="spacer text">Action</span>
                                  <!-- arow_click -->
                                  <div class="v-toolbar__items open-sublist">
                                    <v-icon class="openicon">expand_more</v-icon>
                                  </div>
                                </div>
                                <!-- list-item-sub-cat --> 
                                <div class="list-item-sub action">
                                  <div class="v-toolbar__content sub-cat">
                                    <div class="v-toolbar__title">
                                      <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                    </div>
                                    <span class="spacer text">Romantic Comedies</span>
                                  </div>
                                  <div class="v-toolbar__content sub-cat">
                                    <div class="v-toolbar__title">
                                      <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                    </div>
                                    <span class="spacer text">New Releases</span>
                                  </div>
                                  <div class="v-toolbar__content sub-cat">
                                    <div class="v-toolbar__title">
                                      <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                    </div>
                                    <span class="spacer text">Exciting Movies</span>
                                  </div>
                                </div>
                              </div>
                              <!-- second_category with out sub cat -->
                              <div class="list-item-one">
                                <div class="v-toolbar__content"> 
                                  <div class="v-toolbar__title">
                                    <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                  </div>
                                  <span class="spacer text">Music & Musicals</span>
                                </div>    
                              </div>
                              <!-- second_category with out sub cat -->
                              <div class="list-item-one">
                                <div class="v-toolbar__content"> 
                                  <div class="v-toolbar__title">
                                    <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                  </div>
                                  <span class="spacer text">Romance</span>
                                </div>    
                              </div>
                              <!-- second_category with out sub list -->
                              <div class="list-item-one">
                                <div class="v-toolbar__content"> 
                                  <div class="v-toolbar__title">
                                    <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                  </div>
                                  <span class="spacer text">Romance</span>
                                  <div class="v-toolbar__items">
                                    <v-icon>expand_more</v-icon>
                                  </div>
                                </div>    
                              </div>
                              <!-- second_category with out sub list -->
                              <div class="list-item-one">
                                <div class="v-toolbar__content"> 
                                  <div class="v-toolbar__title">
                                      <v-checkbox class="d-inline" v-model="checkbox"></v-checkbox>
                                  </div>
                                  <span class="spacer text">Comedies</span>
                                </div>    
                              </div>
                            </div>
                          </div>
                      </div>
                    </div>
                  </v-flex>
                </v-layout>
              </div>
            </v-container>
          </v-card>
        </v-dialog>
        <!-- ***************************** -->
        <!-- Edit video end here-->
        <!-- ***************************** -->
        <!-- ***************************** -->
        <!-- Subtitles_Section_Form  -->
        <!-- ***************************** -->
        <v-dialog class="Subtitles_Section" v-model="dialog2" max-width="600px">
          <v-card>
            <v-card-title>
              <span class="headline">Subtitles Section</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap align-center justify-center row>
                  <span class="subheading">Upload your subtitle files to show subtitles in this video. You can upload maximum of 2 files, each file can be 100 KB in size.</span>
                  <div class="text-xs-center my-2">
                    <v-btn class="inputsubtitle">
                      <v-icon class="mx-1">attachment</v-icon>
                      Select Subtitle File
                    </v-btn>
                  </div>
                  <v-flex xs12 my-2>
                    <v-text-field
                    v-model="first"
                    label="Language of Subtitle file. e:g English(uk)"
                    solo
                    ></v-text-field>
                  </v-flex>
                </v-layout>
              </v-container>
              <small></small>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat  @click="dialog2=false">Close</v-btn>
              <v-btn color="blue darken-1" flat  @click="dialog2=false">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- ***************************** -->
        <!-- Subtitles_Section_Form end here-->
        <!-- ***************************** -->
        <!-- ***************************** -->
        <!-- Date recorded & Location_dialog -->
        <!-- ***************************** -->
        <v-dialog class="Location__dialog" v-model="dialog3" max-width="600px">
          <v-card>
              <v-card-title>
                <span class="headline">Date recorded & Location</span>
              </v-card-title>
              <v-card-text>
                <v-container grid-list-md>
                  <v-layout wrap align-center justify-center row>
                    <v-flex xs12>
                      <v-select
                      :items="items"
                      v-model="Blockedcountries"
                      label="Blocked in countries"
                      ></v-select>
                    </v-flex>
                    <v-flex xs12>
                      <v-select
                      :items="items"
                      v-model="Uploadedfrom"
                      label="Uploaded from"
                      ></v-select>
                    </v-flex>
                    <v-flex xs12>
                      <v-menu
                      ref="menu"
                      :close-on-content-click="false"
                      v-model="menu"
                      :nudge-right="40"
                      :return-value.sync="date"
                      lazy
                      transition="scale-transition"
                      offset-y
                      full-width
                      min-width="290px"
                      >
                      <v-text-field
                      slot="activator"
                      v-model="date"
                      label="Picker in menu"
                      prepend-icon="event"
                      readonly
                      ></v-text-field>
                      <v-date-picker v-model="date" no-title scrollable>
                        <v-spacer></v-spacer>
                        <v-btn flat color="primary" @click="menu = false">Cancel</v-btn>
                        <v-btn flat color="primary" @click="$refs.menu.save(date)">OK</v-btn>
                      </v-date-picker>
                    </v-menu>
                  </v-flex>
                </v-layout>
              </v-container>
              <small></small>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat  @click="dialog3=false">Close</v-btn>
              <v-btn color="blue darken-1" flat  @click="dialog3=false">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- ***************************** -->
        <!-- Date recorded & Location_dialog end here -->
        <!-- ***************************** -->
        <!-- ***************************** -->
        <!-- sharing block -->
        <!-- ***************************** -->
        <v-dialog class="Sharing__options" v-model="dialog4" max-width="600px">
          <v-card>
            <v-card-title>
              <span class="headline">Sharing and privacy options</span>
            </v-card-title>
            <v-card-text>
              <v-container grid-list-md>
                <v-layout wrap>
                  <div class="sectionContent">
                    <v-flex lg12 class="option__inners">  
                      <v-flex d-inline-block>
                        <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                      </v-flex>

                      <v-flex d-inline-block>
                        Public - Share your video with Everyone! (Recommended)
                      </v-flex>
                    </v-flex>

                    <v-flex lg12 class="option__inners">  
                      <v-flex d-inline-block>
                        <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                      </v-flex>

                      <v-flex d-inline-block>
                        Private - Viewable by you and your friends only.
                      </v-flex>
                    </v-flex>

                    <v-flex lg12 class="option__inners">  
                      <v-flex d-inline-block>
                        <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                      </v-flex>

                      <v-flex d-inline-block>
                        Public - Share your video with Everyone! (Recommended)
                      </v-flex>
                    </v-flex>

                    <v-flex lg12 class="option__inners">  
                      <v-flex d-inline-block>
                        <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                      </v-flex>

                      <v-flex d-inline-block>
                        Unlisted (anyone with the link and/or password can view)
                      </v-flex>
                    </v-flex>

                    <v-flex lg12 class="option__inners">  
                      <v-flex d-inline-block>
                        <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                      </v-flex>

                      <v-flex d-inline-block>
                        Logged only (only logged in users can watch)
                      </v-flex>
                    </v-flex>
                  </div>
                  <div class="videopassword">
                    <v-flex d-block >
                      <v-text-field 
                      v-model="videoPassword"
                      label="Video Password"
                      solo
                      ></v-text-field>
                    </v-flex>
                  </div>
                  <div class="uservideo">
                    <v-flex d-block >
                      <v-text-field 
                      v-model="videoUsers"
                      label="Video users"
                      solo
                      ></v-text-field>
                    </v-flex>
                  </div>
                  <div class="sectionContent">
                    <v-flex lg12 class="option__inners_group">
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                    </v-flex>

                    <v-flex lg12 class="option__inners_group">
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                    </v-flex>

                    <v-flex lg12 class="option__inners_group">
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                    </v-flex>

                    <v-flex lg12 class="option__inners_group">
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                      <v-layout wrap>
                        <div>
                          <v-radio-group v-model="radioGroup"> <v-radio v-for=""></v-radio></v-radio-group>
                        </div>

                        <div>
                          Allow Comments Voting
                        </div>
                      </v-layout>
                    </v-flex> 
                  </div>
                </v-layout>
              </v-container>
              <small></small>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" flat  @click="dialog4=false">Close</v-btn>
              <v-btn color="blue darken-1" flat  @click="dialog4=false">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <!-- ***************************** -->
        <!-- sharing block ends here-->
        <!-- ***************************** -->
      </v-layout> 
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
        // *********
        // edit file
        // *********
        // dialog boxes
        dialog: false,
        dialog2: false,
        dialog3: false,
        dialog4: false,
        // dialog boxes
        date: new Date().toISOString().substr(0, 7),
        menu: false,
        // Edit video
        title: this.file.name,
        description: this.file.name,
        tags: [this.file.name],
        radioGroup: '',
        videoPassword: '',
        videoUsers: '',
        first: '',
        Blockedcountries: '',
        Uploadedfrom: '',
        checkbox: false,
        items: [
          {
            action: 'local_activity',
            title: 'Attractions'
          }
        ],
        states: [
          'Alabama', 'Alaska', 'American Samoa', 'Arizona',
          'Arkansas', 'California'
        ],
        // *********
        // edit file
        // *********
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
      // style watcher removed
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