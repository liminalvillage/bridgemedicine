<template>
   <z-view   :style="[{backgroundImage:`url(${this.image})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
      >
    <div v-if="!description">
      <h1>{{name}}</h1>
      <br/>
      <!-- <i v-if="quote">"{{quote}}"</i>
      <br/>
      <br/>
      <i><b>{{quoteauthor}}</b></i> -->
    </div>
    <div v-else slot="default" style="width:85%;margin:5%;"> <vue-markdown :source="this.description"></vue-markdown></div>
    <div slot="extension">
      <z-spot v-for="(holon,index) in holons" :key="holon.id"
      :knob="editing ? true : false"
      @click.native= "knobValue[index]? normalize(index) :''"
      :qty.sync="knobValue[index]"
      :label="holonslabels[index]"
      :angle=" 200 - (index * (220./(holons.length-1)))"
      size="m"
      :distance="130"
      :label-pos="index>=holons.length/2?'right':'left'"
      :to-view="editing ? '': {name:'github', params: {id: holon.id}}"
      :style="[{backgroundImage:`url(./images/${holonsimages[index]})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`},{borderWidth:'$(knobValue[index])px'}]"
      >
      <!-- <img v-if=holonsimages[index] :src="`./images/${holonsimages[index]}`" style="width: 80%; margin-top:auto; margin-bottom:auto;"> -->
       <!-- :imagePath = "holonsimages[index]?'./images/'+ holonsimages[index]:''" -->
      </z-spot>
      <!-- settings-->
      <!-- <z-spot
        :angle="270"
        :distance="140"
        size="s"
        label="Settings"
        to-view="settings">
        <i class="fas fa-sliders-h"></i>
      </z-spot> -->
      <!-- Edit signals -->
       <z-spot
        button
        :angle="90"
        :distance="50"
        size="s"
        :label="editing ? 'Save' : 'Edit'"
        @click.native="editing ? editing = false : editing = true">
        <i v-if="editing" class="fas fa-save"></i>
        <i v-else class="fas fa-edit"></i>
      </z-spot>
      <z-spot v-if="github"
        button
        :angle="130"
        :distance="50"
        size="s"
        label="GitHub">
        <i class="fab fa-github"> </i>
      </z-spot>
      <z-spot
        button
        :angle="50"
        :distance="50"
        size="s"
        label="Comms">
        <i class="fas fa-video"> </i>
      </z-spot>
    </div>
  </z-view>
</template>
<script>
import VueMarkdown from 'vue-markdown'

export default {
  components: {
    VueMarkdown
  },
  methods: {
    async fetchGithubContributors (id) {
      // fetch(id, {
      //   method: 'POST',
      //   headers: {
      //     'Content-Type': 'text/plain',
      //     'Accept': 'application/vnd.github.v3+json'
      //   }
      // })
      //   .then(response => response.json()) // Converting the response to a JSON object
      //   .then(data => document.body.append())
      //   .catch(error => console.error(error))

      var r = await fetch(id, {
        method: 'GET',
        headers: {
          'Content-Type': 'text/plain',
          'Accept': 'application/vnd.github.v3+json'
        }
      })
      console.log('asdasd' + r.json())
      return r.json()
    },
    async fetchInfo (id) {
      var json = {}
      var url = new URL(id, 'https://raw.githubusercontent.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/master/')
      // Fill in information from router
      var org = this.$route.params.org
      var repo = this.$route.params.repo
      var file = this.$route.params.file || id || 'package'

      console.log(id + ' - ' + url.hostname + ' - ' + url.pathname + ' - ' + this.$route.params.file + ' - ')
      //
      var pathArray = url.pathname.split('/')
      if (url.hostname === 'github.com') {
        org = pathArray[1]
        repo = pathArray[2]
        pathArray[3] ? file = pathArray[3] : file = 'package'
      }
      // } else if (url.hostname === 'raw.githubusercontent.com') {
      //   org = pathArray[1]
      //   repo = pathArray[2]
      //   file = id
      // }
      // this.fetchGithubContributors(id)
      // if (id.includes('github')) {
      //   var res = await this.fetchGithubContributors(id)
      //   console.log(res)
      //   json.name = res.login
      //   json.image = res.avatar
      // } else {
      const address = 'https://raw.githubusercontent.com/' + org + '/' + repo + '/master/' + file + '.json'
      console.log('fetching: ' + address)
      var r = await fetch(address)
      if (r.ok) {
        json = await r.json().catch(() => {
          console.log(id + ' is not in json format')
          json = {}
          json.name = repo
          json.id = id
          json.image = 'notfound.png'
          return json
        })
        return json
      } else {
        console.log(id + ' not found')
        json.name = repo
        json.id = id
        json.image = 'notfound.png'
        return json
      }
      // }
    },
    async holonInfo () {
      if (this.$zircle.getParams()) {
        this.id = this.$zircle.getParams().id
      } // else {
      //   this.id = 'holon'
      // }
      var json = await this.fetchInfo(this.id)
      if (json) {
        this.name = json.name
        this.description = json.description
        this.quote = json.quote
        this.image = json.image
        this.quoteauthor = json.quoteauthor
        this.holons = json.holons
        if (json.holons) {
          // fetch names of each sub-holon
          Promise.all(
            this.holons.map(async (holon, index) => {
              var json = await this.fetchInfo(holon.id)
              return json.name
            })
          ).then((values) => {
            this.holonslabels = values
          })
          // fetch images of each sub-holon
          Promise.all(
            this.holons.map(async (holon, index) => {
              var json = await this.fetchInfo(holon.id)
              return json.image
            })
          ).then((values) => {
            this.holonsimages = values
          })
        }
      }
    },
    truncate (text) {
      return text.substring(0, 6) + '..'
    },
    normalize (idx) {
      var index = idx // idx + (this.$zircle.getCurrentPageIndex() * 10)
      var qty = this.knobValue[index]
      var total = this.knobValue.reduce((total, num) => total + num)
      if (total > 100) {
        var diff = total - 100 // qty - this.previousKnobValue[index]

        var delta
        //  if (total == 0) {//all sliders are 0 except changed, distribute change equally
        //    delta = this.knobValue.map(() => { return - (1/(this.knobValue.length-1)) * diff})
        //  }
        //  else {
        delta = this.knobValue.map((x) => { return -x / (total - qty) * diff })
        //  }

        var normal = this.knobValue.map((x, idx) => {
          return Math.floor(x + delta[idx])
        })
        normal[index] = qty // copy correct value for current slider
        this.knobValue = Object.assign([], this.knobValue, normal)
        // this.previousKnobValue = Object.assign([],this.knobValue,this.knobValue);
        console.log(this.knobValue.reduce((total, num) => total + num))
      }// return this.knobValue.map((x) => { return (x *100) /this.knobValue.reduce((total, num)=> total + num);} );
    }
  },
  mounted () {
    this.holonInfo()
  },

  data () {
    return {
      editing: false,
      github: true,
      id: '',
      language: 'en',
      name: '',
      description: '',
      quote: '',
      quoteauthor: '',
      image: '',
      holons: [],
      holonslabels: [],
      holonsimages: [],
      knobValue: []
    }
  }
}
</script>
