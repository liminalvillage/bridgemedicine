<template>
   <z-view   :style="[{backgroundImage:`url(${this.image})`},{backgroundSize: `80% 80%`},{backgroundColor: `transparent`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
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
      @click.native= "knobValue[index]? normalize(index) : ''"
      :qty.sync="knobValue[index]"
      :label="holonslabels[index]"
      :angle=" 200 - (index * (220./(holons.length >1 ? holons.length - 1 : 1)))"
      size="m"
      :distance="130"
      :label-pos="index>=holons.length/2?'right':'left'"
      :to-view="editing ? '': {name:'github', params: {id: holon.id}}"
      :style="[{backgroundImage:`url(${holonsimages[index]})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`},{borderWidth:'$(knobValue[index])px'}]"
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
        :angle ="270"
        :distance ="120"
        size ="s"
        :label ="editing ? 'Save' : 'Edit'"
        label-pos = 'top'
        @click.native="editing ? editing = false : editing = true">
        <i v-if="editing" class="fas fa-save"></i>
        <i v-else class="fas fa-edit"></i>
      </z-spot>
      <z-spot v-if="github"
        button
        :angle ="250"
        :distance ="120"
        size ="s"
        label ="GitHub"
        label-pos = 'top'
        @click.native ="window.open(address, '_blank');"
        >
        <i class="fab fa-github"> </i>
      </z-spot>
      <z-spot
        button
        :angle="290"
        :distance="120"
        size="s"
        label-pos = 'top'
        label="Comms">
        <i class="fas fa-video"> </i>
      </z-spot>
    </div>
  </z-view>
</template>
<script>
import VueMarkdown from 'vue-markdown'

export default {
  watch: {
    $route (to, from) {
      console.log(to)
      console.log(from)
    }
  },
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
    async fetchInfo (address) { // returns json containing info of the holon passedd in
      console.log('requesting: ' + address)
      var json = {}
      var base = ''
      var file = 'holon.json'
      var url
      var pathArray
      var fetchaddress
      var type = ''
      if (address === '') { // ADDRESS EMPTY
        type = 'EMPTY'
        base = 'https://raw.githubusercontent.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/master/'
        file = (this.$route.params.file ? this.$route.params.file : file)
        if (!file.endsWith('.json')) {
          file = file + '.json'
        }
        fetchaddress = base + file
      } else if (address.includes('github.com')) { // GITHUB ADDRESS
        type = 'GITHUB'
        url = new URL(address)
        pathArray = url.pathname.split('/')
        base = 'https://raw.githubusercontent.com/' + pathArray[1] + '/' + pathArray[2] + '/master/'
        if (pathArray[3]) file = url.pathname.slice(url.pathname.indexOf(pathArray[2]) + pathArray[2].length)
        if (!file.endsWith('.json')) file += '.json'
        fetchaddress = base + file
      } else if (address.includes('gitlab')) { // GITLAB ADDRESS
        type = 'GITLAB'
        url = new URL(address)
        pathArray = url.pathname.split('/')
        base = 'https://gitlab.com/' + pathArray[1] + '/' + pathArray[2] + '/-/raw/master/'
        if (pathArray[3]) file = url.pathname.slice(url.pathname.indexOf(pathArray[2]) + pathArray[2].length)
        if (!file.endsWith('.json')) file += '.json'
        fetchaddress = base + file
      } else if (address.startsWith('http')) { // WEB ADDRESS
        type = 'WEB'
        fetchaddress = address
      // eslint-disable-next-line no-constant-condition
      } else if (address.startsWith('./') || address.startsWith('/') || address.match(/[0-9A-Za-z]/)) { // RELATIVE ADDRESS
        type = 'RELATIVE'
        pathArray = this.$route.path.split('/') // source absolute path info from route
        base = 'https://raw.githubusercontent.com/' + pathArray[1] + '/' + pathArray[2] + '/master/'
        fetchaddress = base + address
        if (!fetchaddress.endsWith('.json')) { fetchaddress += '.json' }
      } else if (address.startsWith('0x')) { // ETHEREUM ADDRESS
        type = 'ETHEREUM'
        // (query ethereum and  graph)
      } else if (address.match(/[0-9A-Fa-f]{47}/g)) { //  IPFS ADDRESS
        type = 'IPFS'
        fetchaddress = 'https://ipfs.io/ipfs/' + address
      // } else if (address) { // NPM NAME
      //   // fetchaddress = 'https://registry.npmjs.org/' + address
      //   fetchaddress = this.base + address // temporary
      } else { // not know what to do, assume fetched from github
        type = 'NONE'
        base = 'https://raw.githubusercontent.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/master/'
        file = this.$route.params.file
        if (!file.endsWith('.json')) {
          file = file + '.json'
        }
        fetchaddress = this.base + file
      }

      console.log('Fetching: ' + fetchaddress)
      var r = await fetch(fetchaddress)
      if (r.ok) {
        json = await r.json().catch(() => {
          console.log(address + ' is not in json format')
          json = {}
          json.name = 'n/a'
          json.id = address
          json.image = 'notfound.png'
          return { json, type, base }
        })
        return { json, type, base }
      } else {
        // try again with package.json
        if (!fetchaddress.endsWith('package.json')) {
          return this.fetchInfo(fetchaddress.slice(0, fetchaddress.lastIndexOf('/')) + '/package.json')
        } else {
          type = 'N/A'
          console.log(address + ' not found')
          json.name = 'n/a'
          json.id = address
          json.image = 'notfound.png'
          return { json, type, base }
        }
      }
    },
    async fetchInfoOld (id) {
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
      }
      let r = await this.fetchInfo(this.id)
      console.log(r.type)
      var json = r.json
      if (json) {
        this.name = json.name
        this.description = json.text
        this.quote = json.quote
        this.image = json.image
        this.quoteauthor = json.quoteauthor
        this.holons = json.holons
        if (json.holons) {
          // fetch names of each sub-holon
          Promise.all(
            this.holons.map(async (holon, index) => {
              let r = await this.fetchInfo(holon.id)
              return r.json.name
            })
          ).then((values) => {
            this.holonslabels = values
          })
          // fetch images of each sub-holon
          Promise.all(
            this.holons.map(async (holon, index) => {
              let r = await this.fetchInfo(holon.id)
              return r.json.image
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
    // var org = this.$route.params.org
    // var repo = this.$route.params.repo
    // var file = this.$route.params.file || id || 'package'
    // if (this.id === '') this.id = 'http://github.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/' + this.$route.params.file
    this.holonInfo()
  },

  data () {
    return {
      editing: false,
      github: true,
      base: '',
      id: '',
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
