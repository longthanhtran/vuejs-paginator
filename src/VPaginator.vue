<template>
  <div class="v-paginator">
    <button class="btn btn-default" @click="fetchData(prev_page_url)" :disabled="!prev_page_url">
      {{config.previous_button_text}}
    </button>
    <span>Page {{current_page}} of {{last_page}}</span>
    <button class="btn btn-default" @click="fetchData(next_page_url)" :disabled="!next_page_url">
      {{config.next_button_text}}
    </button>
  </div>
</template>

<script>
import {utils} from './utils'
export default {
  props: {
    resource: {
      required: true,
      twoWay: true
    },
    resource_url: {
      type: String,
      required: true
    },
    custom_template : '',
    options: {
      type: Object,
      required: false,
      default () {
        return {}
      }
    }
  },
  data () {
    return {
      current_page: '',
      last_page: '',
      next_page_url: '',
      prev_page_url: '',
      config: {
          remote_data: 'data',
          remote_current_page: 'current_page',
          remote_last_page: 'last_page',
          remote_next_page_url: 'next_page_url',
          remote_prev_page_url: 'prev_page_url',
          previous_button_text: 'Previous',
          next_button_text: 'Next'
      }
    }
  },
  methods: {
    fetchData (pageUrl) {
      pageUrl = pageUrl || this.resource_url
      this.$http.get(pageUrl, {}, { headers: this.config.headers })
      .then(function (response) {
        this.handleResponseData(response.data)
      }).catch(function (response) {
        console.log('Fetching data failed.', response)
      })
    },
    handleResponseData (response) {
      this.makePagination(response)
      this.$set('resource', utils.getNestedValue(response, this.config.remote_data))
    },
    makePagination (data) {
      this.current_page = utils.getNestedValue(data, this.config.remote_current_page)
      this.last_page = utils.getNestedValue(data, this.config.remote_last_page)
      this.next_page_url = (this.current_page === this.last_page) ? null : utils.getNestedValue(data, this.config.remote_next_page_url);
      this.prev_page_url = (this.current_page === 1) ? null : utils.getNestedValue(data, this.config.remote_prev_page_url);
    },
    initConfig(){
      this.config = utils.merge_objects(this.config, this.options)
    }
  },
  watch : {
    resource_url () {
      this.fetchData()
    }
  },
  created () {
    this.initConfig()
    this.fetchData()
  }
}
</script>
