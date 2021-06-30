<template>
   <z-view  :style="[{backgroundImage:`url(./images/${this.image})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
      >
    <div v-if="!text">
      <h1>{{name}}</h1>
      <br/>
      <i v-if="quote">"{{quote}}"</i>
      <br/>
      <br/>
      <i><b>{{quoteauthor}}</b></i>
    </div>
    <div v-else slot="default" style="width:85%;margin:5%;"> {{text}} </div>
    <div slot="extension">
      <z-spot v-for="(holon,index) in holons" :key="holon.id"
      :label="holonslabels[index]"
      :angle=" 180 - (index * (180./(holons.length-1)))"
      size="m"
      :distance="130"
      :label-pos="index>=holons.length/2?'right':'left'"
      :to-view="{name:'holon', params: {id: holon.id}}"
      :style="[{backgroundImage:`url(./images/${holonsimages[index]})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
      >
       <!-- :imagePath = "holonsimages[index]?'./images/'+ holonsimages[index]:''"
      -->
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
    </div>
  </z-view>
</template>
<script>
export default {
  methods: {
    holonInfo () {
      var id
      if (this.$zircle.getParams() !== undefined) {
        id = this.$zircle.getParams().id
      } else {
        id = 'home'
      }
      fetch('/text/' + id + '.json')
        .then(r => r.json())
        .then(json => {
          this.id = id
          this.name = json.name
          this.text = json.text
          this.quote = json.quote
          this.image = json.image
          this.quoteauthor = json.quoteauthor
          this.holons = json.holons
          // fetch names of each sub-holon
          Promise.all(
            this.holons.map(async (holon) => {
              var r = await fetch('/text/' + holon.id + '.json')
              var json = await r.json()
              this.holonslabels.push(json.name)
              this.holonsimages.push(json.image)
            })
          )
          // fetch('/text/' + this.holons[i].id + '.json')
          //   .then(r => r.json())
          //   .then(json => {
          //     this.holonslabels[i] = json.name
          //     this.holonsimages.push(json.image)
          //   })
        },
        response => {
          console.log('Error loading json:', response)
        })
    }
  },
  mounted () {
    this.holonInfo()
  },

  data () {
    return {
      id: '',
      name: '',
      text: '',
      quote: '',
      quoteauthor: '',
      image: '',
      holons: [],
      holonslabels: [],
      holonsimages: []
    }
  }
}
</script>
