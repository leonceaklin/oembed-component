<template>
  <div ref="container" class="__v-oembed">
    <div class="__v-oembed-loading" :style="boxStyle" v-if="!resJson.provider_name && !errorMessage">Loading....</div>
    <div class="__v-oembed-error" :style="boxStyle" v-if="errorMessage">{{errorMessage}}</div>
  </div>
</template>

<script>
import EndPoints from '../endpoints-regexp.json'

export default {
  name: 'o-embed',
  props: {
    url: {
      type: String,
      required: true
    },
    proxy: {
      type: String,
      default: ''
    },
    inlineStyle: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      resJson: Object,
      errorMessage: null
    }
  },
  computed: {
    boxStyle () {
      if (this.inlineStyle) {
        return {
          fontFamily: '-apple-system, system-ui, Roboto, sans-serif',
          padding: '15px',
          border: '1px solid grey',
          color: 'grey',
          borderRadius: '3px',
          textAlign: 'center'
        }
      }
    }
  },
  mounted () {
    this.setup()
  },
  watch: {
    url () {
      this.setup()
    },
    proxy () {
      this.setup()
    }
  },
  methods: {
    getOembedEnpoint: (url) => {
      return Object.keys(EndPoints).filter(endpoint => {
        let foundPattern = EndPoints[endpoint].filter(pattern => new RegExp(pattern).test(url))
        return foundPattern.length > 0
      })[0]
    },
    setInnerHtml: (elm, html) => {
      if (html === undefined) {
        this.errorMessage = 'No HTML Specified'
      }
      elm.innerHTML = html
      this.$emit('load')
      this.$nextTick(() => {
        Array.from(elm.querySelectorAll('script')).forEach(el => {
          let newEl = document.createElement('script')
          Array.from(el.attributes).forEach(el => {
            newEl.setAttribute(el.name, el.value)
          })
          newEl.appendChild(document.createTextNode(el.innerHTML))
          el.parentNode.replaceChild(newEl, el)
        })
      })
    },
    async setup () {
      let matchedPattern = this.getOembedEnpoint(this.url)
      if (matchedPattern) {
        try {
          matchedPattern = matchedPattern.replace('http://', 'https://')
          let res = await fetch(this.proxy + matchedPattern + '?format=json&url=' + this.url)
          this.resJson = await res.json()
          this.setInnerHtml(this.$el, '<div class="__v-oembed-content">' + this.resJson.html + '</div>')
        } catch (error) {
          this.errorMessage = 'Domain is not allowed'
        }
      } else {
        this.errorMessage = 'Endpoint isn\'t supported'
      }
    }
  }
}
</script>
