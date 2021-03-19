<template>
  <view class="simple-mde" :class="classes">
    <textarea ref="mde"></textarea>
  </view>
</template>

<script>
import 'font-awesome/css/font-awesome.min.css'
import SimpleMDE from './lib/simpleMDE/js/simplemde'
import 'simplemde/dist/simplemde.min.css'
import marked from 'marked'
import 'github-markdown-css/github-markdown.css'

export default {
  name: 'MarkdownEditor',
  props: {
    value: {
      type: String,
      default: '',
    },
    border: {
      type: Boolean,
      default: false,
    },
    config: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      mde: null,
      defaultConfig: {},
    }
  },
  computed: {
    classes() {
      return [
        {
          'i-mde-no-border': !this.border,
        },
      ]
    },
  },
  mounted() {
    this.init()
  },
  beforeDestroy() {
    this.mde = null
  },
  methods: {
    init() {
      console.log(this.$refs.mde.$el.querySelector('textarea'))
      const _self = this
      const config = Object.assign({}, this.defaultConfig, this.config)
      this.mde = new SimpleMDE({
        ...config,
        initialValue: this.value,
        element: this.$refs.mde.$el.querySelector('textarea'),
        autoDownloadFontAwesome: false,
        promptURLs: true,
        uploadCallBack(res) {
          let editor = this
          _self.$myCloud
            .uploadFile({
              filePath: res.tempFilePaths[0],
              cloudPath: `mm_${new Date().getTime()}.jpg`,
            })
            .then((res) => {
              console.log(res)
              let { fileID } = res
              return _self.$myCloud.getTempFileURL({
                fileList: [fileID],
              })
            })
            .then((res) => {
              console.log(res)
              let { download_url } = res.fileList[0]
              editor.options.promptTexts['image'] = download_url
              editor.createImageUrl(editor)
            })
        },
      })
      this.mde.codemirror.on('change', () => {
        this.$emit('input', this.mde.value())
        this.$emit('on-change', this.mde.value())

        const mdHtml = marked(this.mde.value() || '', {
          renderer: new marked.Renderer(),
          gfm: true,
          tables: true,
          breaks: false,
          pedantic: false,
          sanitize: false,
          smartLists: true,
          smartypants: false,
        })

        this.$emit(
          'show-html',
          '<div class="markdown-body" >' + mdHtml + '</div>',
        )
      })
    },
    add(val) {
      if (this.mde) {
        this.mde.value(this.value + val)
      }
    },
    replace(val) {
      if (this.mde) {
        this.mde.value(val)
      }
    },
  },
}
</script>

<style lang="scss" scoped>
.simple-mde {
  textarea {
    // min-height: 150px;
    height: 300px;
    // overflow: auto;
    overflow-y: scroll;
  }
}
/deep/ .editor-toolbar.fullscreen {
  z-index: 999999;
}
</style>
