
<template>
  <z-canvas :views='$options.components'></z-canvas>
</template>
<script>
// TO SET UP VUE-ROUTER UNCOMMENT THE FOLLOWING LINES
// import Vue from 'vue'
// import Router from 'vue-router'
// Vue.use(Router)
// const router = new Router()
export default {
  components: {
    home: () => import('./views/home'),
    holon: () => import('./views/holon'),
    settings: () => import('./views/settings')
  },
  mounted () {
    this.$zircle.config({
      mode: 'full',
      style: {
        theme: 'black',
        mode: 'dark-outline'
      },
      debug: true
    })
    this.$zircle.toView('home')
    //
    //   to: 'holon',
    //   params: { id: 'home' } // Optional
    // })
  }
}
</script>
<style>
@import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro:400,700');
@import url('https://use.fontawesome.com/releases/v5.1.0/css/all.css');

  a {
    color:  var( --primary);
    font-style: bold
  }
  #z-container {
    background-image:url('../public/images/green-blur.jpg');
    background-size:cover;
    background-position:center;
  }

  .z-content {
    /*background-color: rgba(255, 255, 255, 0.1) !important;*/
    background: radial-gradient(40% 40% at 50% 50%,hsla(0,0%,100%,.1) 0,rgba(0,0,0,.1) 100%)
  }

  .z-spot {
    background-color: rgba(255, 255, 255, 0.1) !important;
  }

  .z-content.maincontent {
    background-color: rgba(255, 255, 255, 0.2) !important;
  }

  .primary {
    background-color: rgba(255, 255, 255, 0.4) !important;
  }

  .inside {
    background-color: rgba(255, 255, 255, 0.4) !important;
  }

  .overflow.img {
    opacity:0.5;
  }

</style>
