<template>
  <div id="app">
    <router-view v-if="landscape" class="w-full h-full" />
    <div v-else>
      <span>Por favor gire su telefono</span>
      <img
        src="https://raw.githubusercontent.com/NordicPlayground/nrf52-quadcopter/master/Web/img/rotate.png"
      />
    </div>
  </div>
</template>



<script>
import Zondicon from "vue-zondicons";
export default {
  components: {
    Zondicon
  },
  mounted() {
    window.addEventListener("orientationchange", this.handleOrientation, true);
    window.addEventListener("deviceorientation", this.initOrientation, true);
  },
  methods: {
    upVelocity() {},
    handleOrientation() {
      window.innerWidth > window.innerHeight
        ? (this.landscape = false)
        : (this.landscape = true);
      console.log("change");
    },
    initOrientation() {
      console.log("init");
      window.innerWidth > window.innerHeight
        ? (this.landscape = true)
        : (this.landscape = false);
    }
  },
  data() {
    return {
      landscape: true
    };
  }
};
</script>



<style lang="scss">
#app {
  text-align: center;
  color: #2c3e50;
  min-height: 100vh;
}
#nav {
  padding: 30px;
  a {
    font-weight: bold;
    color: #2c3e50;
    &.router-link-exact-active {
      color: #42b983;
    }
  }
}
</style>
