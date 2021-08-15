<template>
   <z-view   :style="[{backgroundImage:`url(${this.image})`},{backgroundSize: `80% 80%`},{backgroundColor: `transparent`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
      >
    <!-- First modal -->
    <vue-final-modal v-model="showModal" classes="modal-container" content-class="modal-content">
      <button class="modal__close" @click="showModal = false">
        <mdi-close></mdi-close>
      </button>
      <span class="modal__title">Hello, vue-final-modal</span>
      <div class="modal__content">
        <p v-for="i in 5" :key="i">
          Vue Final Modal is a renderless, stackable, detachable and lightweight modal component.
        </p>
      </div>
      <div class="modal__action">
        <v-button highlight @click="showConfirmModal = true">confirm</v-button>
        <v-button @click="showModal = false">cancel</v-button>
      </div>
    </vue-final-modal>

    <div>
      <h1>{{name}}</h1>
      <br/>
      <!-- <i v-if="quote">"{{quote}}"</i>
      <br/>
      <br/>
      <i><b>{{quoteauthor}}</b></i> -->
    </div>
    <div slot="default" style="width:85%;margin:5%;"> <vue-markdown :source="this.description"></vue-markdown></div>
    <div slot="extension">
      <z-spot v-for="(holon,index) in holons" :key="holon.id"
      :knob="editing ? true : false"
      @click.native= "holonsvalues[index]? normalize(index) : ''"
      :qty.sync="holonsvalues[index]"
      :label="holonslabels[index]"
      :angle=" 200 - (index * (220./(holons.length >1 ? holons.length - 1 : 1)))"
      size="m"
      :distance="130"
      :label-pos="index>=holons.length/2?'right':'left'"
      :to-view="editing ? '': {name:'github', params: {id: holon.id}}"
      :style="[{backgroundImage:`url(${holonsimages[index]})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`},{borderWidth:'$(holonsvalues[index])px'}]"
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
        :distance ="170"
        size ="s"
        :label ="editing ? 'Save' : 'Edit'"
        label-pos = 'top'
        @click.native="editing ? editing = false : editing = true; showModal = true">
        <i v-if="editing" class="fas fa-save"></i>
        <i v-else class="fas fa-edit"></i>
      </z-spot>
      <z-spot v-if="type == 'GITHUB'"
        button
        :angle ="250"
        :distance ="170"
        size ="s"
        label ="GitHub"
        label-pos = 'top'
        @click.native ="goToSite(this.address)"
        >
        <i class="fab fa-github"> </i>
      </z-spot>
      <z-spot
        button
        :angle="290"
        :distance="170"
        size="s"
        label-pos = 'top'
        label="Comms">
        <i class="fas fa-video"> </i>
      </z-spot>
      <!-- ------------------------- Add Button ------------------------- -->
      <z-spot
        button
        :angle="90"
        :distance="100"
        size="s"
        label-pos = 'top'
        label="Add"
        @click='showModal = true'>
        <i class="fas fa-plus"> </i>
      </z-spot>
    </div>
  </z-view>
</template>
<script>
import VueMarkdown from 'vue-markdown'
import Home from '../home.json'
import VueFinalModal from 'vue-final-modal'
// import web3 from '../libs/web3.js'
import Web3 from 'web3'
// import abi from '../abi.json'
import * as contractdata from '../ContractData.json'
export default {
  watch: {
    // holons: function () {
    //   console.log('called! ' + this.holons.length)
    // },
    // deep: true,

    $route (to, from) {
      console.log(to)
      console.log(from)
    }
  },
  components: {
    VueMarkdown,
    VueFinalModal
  },
  methods: {
    goToSite (address) {
      window.open(address)
    },
    async initWeb3 () {
      this.web3 = new Web3(new Web3.providers.WebsocketProvider('wss://ropsten.infura.io/ws/v3/966b62ed84c84715bc5970a1afecad29'))
      // this.web3.eth.getAccounts(console.log)
    },
    async fetchContributors (id) {
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
      console.log('Found contributors: ' + r.json())
      return r.json()
    },
    async fetchEthInfo (address) { // compiles a compatible JSON from blockchain information at address
      var json = {}
      this.web3 = new Web3(new Web3.providers.WebsocketProvider('wss://ropsten.infura.io/ws/v3/966b62ed84c84715bc5970a1afecad29'))
      // check if it is an holon
      const code = await this.web3.eth.getCode(address)
      if (code === '0x') {
        console.log(address + 'is NOT an holon')
        json.id = address
        json.name = address // check ens
        json.description = address
        json.image = 'https://ipfs.3box.io/profile?address=' + address // check 3box
      } else {
        let holon = new this.web3.eth.Contract(contractdata.abi, address)
        // const totalappreciation = await holon.methods.totalappreciation().call()
        // console.log(totalappreciation)
        // var name = holon.methods.toName(address).call()
        // console.log(name)
        json.id = address
        json.name = address
        json.image = 'https://ipfs.3box.io/profile?address=' + address
        var members = await holon.methods.listMembers().call()
        if (members) {
          json.holons = members.map((member, index) => {
            var name = holon.methods.toName(member).call()
            var value = holon.methods.appreciation(member).call()
            console.log(value)
            return { 'id': member, 'label': name, 'value': value }
          })
        }
      }
      return json
      // check if it is a holon from ethereum
      // check if it is a dao from the graph
      // compile jsos
    },
    async fetchInfo (address) { // returns json containing info of the holon passedd in
      console.log('Requesting: ' + address)
      var type = 'N/A'
      var json = { 'id': address, 'name': 'n/a', 'image': 'notfound.jpg' }
      var base = ''
      var file = 'holon.json'
      var url
      var pathArray
      var fetchaddress

      if (address === '') { // ADDRESS IS EMPTY
        type = 'EMPTY'
        if (this.$route.params.repo) { // fetch github info from address bar if it exist
          base = 'https://raw.githubusercontent.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/master/'
          file = (this.$route.params.file ? this.$route.params.file : file)
          if (!file.endsWith('.json')) {
            file = file + '.json'
          }
          fetchaddress = base + file
        } else if (this.$route.params.address) { // try with the info in the address bar
          return this.fetchInfo(this.$route.params.address)
        } else { // showcase home + instructions
          console.log(Home)
          return { Home, type }
        }
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
        if (!address.endsWith('.json')) address += '/murmurations.json'
        fetchaddress = 'https://api.allorigins.win/get?url=' + address
      } else if (address.startsWith('0x')) { // ETHEREUM ADDRESS
        type = 'ETHEREUM'
        json = await this.fetchEthInfo(address)
        return { json, type }
        // var contract = new web3.eth.Contract(abi, '0x6B175474E89094C44Da98b954EedeAC495271d0F')
        // contract.methods.balanceOf('0x96bB7B429cF97131E624Fa32d27B45595d59b5B8').call().then(console.log)
        // (query ethereum and the graph)
      } else if (address.match(/[0-9A-Fa-f]{47}/g)) { //  IPFS ADDRESS
        type = 'IPFS'
        fetchaddress = 'https://ipfs.io/ipfs/' + address
      } else if (address.startsWith('./') || address.startsWith('/') || address.match(/[0-9A-Za-z]/)) { // RELATIVE ADDRESS
        type = 'RELATIVE'
        pathArray = this.$route.path.split('/') // source absolute path info from route
        base = 'https://raw.githubusercontent.com/' + pathArray[1] + '/' + pathArray[2] + '/master/'
        if (pathArray[3]) file = this.$route.path.slice(this.$route.path.indexOf(pathArray[2]) + pathArray[2].length)
        fetchaddress = base + file.slice(0, file.lastIndexOf('/') + 1) + address
        if (!fetchaddress.endsWith('.json')) { fetchaddress += '.json' }
      // } else if (address) { // NPM NAME
      //   // fetchaddress = 'https://registry.npmjs.org/' + address
      //   fetchaddress = base + address // temporary
      } else { // not know what to do, assume fetched from github
        type = 'NONE'
        base = 'https://raw.githubusercontent.com/' + this.$route.params.org + '/' + this.$route.params.repo + '/master/'
        file = this.$route.params.file
        if (!file.endsWith('.json')) {
          file = file + '.json'
        }
        fetchaddress = base + file
      }
      if (!fetchaddress) return { json, type }
      console.log('Fetching: ' + fetchaddress)
      var r = await fetch(fetchaddress)
      if (r.ok) {
        json = await r.json().catch(() => {
          console.log(fetchaddress + ' is not in a valid JSON format')
          return { json, type }
        })
        if (type === 'WEB' && json.contents.startsWith('{')) {
          // parse returned web value (from cors proxy)
          json = JSON.parse(json.contents)
        }
        return { json, type }
      } else {
        // try again with package.json
        if (!fetchaddress.endsWith('package.json')) {
          return this.fetchInfo(fetchaddress.slice(0, fetchaddress.lastIndexOf('/')) + '/package.json')
        } else {
          type = 'N/A'
          console.log(address + ' not found')
          return { json, type }
        }
      }
    },
    async holonInfo () { // retrieves and updates the information about this holon and its sub holons
      if (this.$zircle.getParams()) {
        this.id = this.$zircle.getParams().id
      }
      let r = await this.fetchInfo(this.id)
      console.log(r.type)
      var json = r.json
      if (json) {
        this.name = json.name
        this.description = json.description
        this.website = json.website
        this.quote = json.quote
        this.image = json.image
        this.quoteauthor = json.quoteauthor
        this.holons = json.holons
        if (json.holons) {
          // fetch names of each sub-holon
          Promise.all(
            this.holons.map(async (holon, index) => {
              let r = await this.fetchInfo(holon.id)
              if (holon.label) {
                return holon.label
              } else {
                return r.json.name
              }
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
          // fetch subholon values if available
          for (let i = 0; i < this.holons.length; i++) {
            this.holonsvalues[i] = parseInt(this.holons[i].value)
          }
        }
      }
    },
    truncate (text) {
      return text.substring(0, 6) + '..'
    },
    normalize (idx) {
      var index = idx // idx + (this.$zircle.getCurrentPageIndex() * 10)
      var qty = this.holonsvalues[index]
      var total = this.holonsvalues.reduce((total, num) => total + num)
      if (total > 100) {
        var diff = total - 100 // qty - this.previousholonsvalues[index]

        var delta
        //  if (total == 0) {//all sliders are 0 except changed, distribute change equally
        //    delta = this.holonsvalues.map(() => { return - (1/(this.holonsvalues.length-1)) * diff})
        //  }
        //  else {
        delta = this.holonsvalues.map((x) => { return -x / (total - qty) * diff })
        //  }

        var normal = this.holonsvalues.map((x, idx) => {
          return Math.floor(x + delta[idx])
        })
        normal[index] = qty // copy correct value for current slider
        this.holonsvalues = Object.assign([], this.holonsvalues, normal)
        // this.previousholonsvalues = Object.assign([],this.holonsvalues,this.holonsvalues);
        console.log(this.holonsvalues.reduce((total, num) => total + num))
      }// return this.holonsvalues.map((x) => { return (x *100) /this.holonsvalues.reduce((total, num)=> total + num);} );
    }
  },
  mounted () {
    this.holonInfo()
    this.initWeb3()
    // this.holon = new Web3.eth.Contract(data.holonabi, data.holonaddress)
  },

  data () {
    return {
      showModal: true,
      web3: null,
      holon: null,
      editing: false,
      website: null,
      type: 'GITHUB',
      id: '',
      name: '',
      description: '',
      quote: '',
      quoteauthor: '',
      image: '',
      holons: [],
      holonslabels: [],
      holonsimages: [],
      holonsvalues: []
    }
  }
}
</script>
