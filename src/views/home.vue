<template>
  <z-view>
    <h1>Bridge Medicine</h1>
    <div slot="extension">
      <z-spot v-for="(holon,index) in holons" :key="holon.title"
      :label="holon.title"
      :angle="180 - (index *30)"
      size="m"
      :distance="130"
      label-pos="bottom"
      :to-view="{name:'holon', params: {id: holon.title}}"
      :style="[{backgroundImage:`url(${holon.image})`},{backgroundSize: `80% 80%`},{backgroundRepeat: `no-repeat`},{backgroundPosition: `center`}]"
      class = "filtered"
       >
      <!-- <div v-bind:style="{ backgroundImage: 'url(' + holon.image + ')' }"></div> -->
      <!-- <i :href=holon.background-size: 80% 80%; background-repeat: no-repeat;></i> -->
      </z-spot>
      <!-- settings-->
      <z-spot
        :angle="150"
        :distance="190"
        size="s"
        label="Settings"
        to-view="settings">
        <i class="fas fa-sliders-h"></i>
      </z-spot>
    </div>
  </z-view>
</template>
<script>
export default {
  data () {
    return {
      holons: [
        { title: 'bridgemedicine', image: './images/bridgemedicine.png' },
        { title: 'medicine', image: './images/medicine.png' },
        { title: 'pain', image: './images/pain.png' },
        { title: 'roots', image: './images/roots.png' },
        { title: 'plants', image: './images/plants.png' },
        { title: 'me', image: './images/me.png' },
        { title: 'go', image: './images/go.png' }
      ]
    }
  }
}
</script>
