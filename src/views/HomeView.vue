<template>
  <div class="home">

    <div class="search row">
      <div class="col-6">
        <input type="text" class="form-control" placeholder="Search for something" v-model="search_input">
      </div>
      <div class="col-3">
        <input type="number" class="form-control" v-model="search_length">
      </div>
      <div class="col-2">
        <button class="btn btn-primary" v-if="search"  @click="search_func">Search</button>
        <button class="btn btn-primary" v-if="search_reverse" disabled>Search in {{ time }}</button>
      </div>
    </div>

    <hr>

    <div class="result">
      <div class="row" v-if="visible">
        <div class="col-md-8">
          <div class="card">
            <div class="card-body">
              <h1>{{ search_title }}</h1>
            </div>
            <div class="card-body">
              <p v-for="result in search_results" v-bind:key="result">{{ result }}</p>
            </div>
            <div class="card-body">
              <a :href="search_url">Read More</a>
            </div>
          </div>
        </div>
        <div class="col-md-4">
          <div class="card" v-for="image in search_image" :key="image">
            <img :src="image" alt="" style="width:360px">
          </div>
        </div>
      </div>
    </div>

  </div>
</template>




<script>
import { defineComponent } from 'vue'

export default defineComponent({

  data() {
    return {
      visible: false,
      visible_reverse: true,

      search:true,
      search_reverse: false,
      time:11,

      search_length: 10,
      search_input: "",

      search_results: [],
      search_title: "",
      search_image: [],
      search_url: ""
    }
  },

  methods: {

    disable_search: function() {
      this.search = false
      this.search_reverse = true
      this.counter()
    },

    enable_search: function() {
      this.search = true
      this.search_reverse = false
      this.time = 11
    },

    counter: function() {
      if(this.time > 0) {
        this.time--
      }
      else {
        this.enable_search()
      }
      setTimeout(this.counter, 1000)
    },

    search_func: function () {

      this.search_results = []
      this.search_title = ""
      this.search_image = []
      this.searc_url = ""
      this.visible = true
      this.visible_reverse = true
      this.disable_search()

      fetch(`https://wiki-briefs.p.rapidapi.com/search?q=${this.search_input}&topk=${this.search_length}`, {
        method: "GET",
        headers: {
          'X-RapidAPI-Key': '941f406cf1msh25d5e4addbb8f7dp1dda02jsnc59c17972b62',
          'X-RapidAPI-Host': 'wiki-briefs.p.rapidapi.com'
        }
      }).then(res => { return res.json() }).then(
        data => {
          if(data['title'] == null || data['title'] == "" || data['title'].length == 0 ) {
            this.search_title = "Oops no search results found"
          }
          else {
            this.search_title = data['title']
          }
          this.search_url = data['url']
          let search_tem = data['summary']
          for (let i = 0; i < search_tem.length; i++) {
            this.search_results.push(search_tem[i])
          }
        }
      ).catch(err => { console.log(err) });

      fetch(`https://joj-image-search.p.rapidapi.com/v2/?q=${this.search_input}&hl=en`, {
        method: "GET",
        headers: {
          'X-RapidAPI-Key': '941f406cf1msh25d5e4addbb8f7dp1dda02jsnc59c17972b62',
          'X-RapidAPI-Host': 'joj-image-search.p.rapidapi.com'
        }
      }).then(res => { return res.json() }).then(data => {
        for (let i = 0; i < this.search_length; i++) {
          this.search_image.push(data['response']['images'][i]['image']['url'])
        }
      })

    }

  }

})
</script>



