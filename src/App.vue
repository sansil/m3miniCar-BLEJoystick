<template>
  <div id="app" class>
    <div class="background fixed my-bg w-full h-full"></div>
    <router-view v-if="landscape" class="w-full h-full" />
    <div v-else>
      <div class="flex flex-col justify-center items-center">
        <span class="my-20 text-xl text-white">Por favor gire su telefono</span>
        <img
          src="https://raw.githubusercontent.com/NordicPlayground/nrf52-quadcopter/master/Web/img/rotate.png"
        />
      </div>
    </div>
  </div>
</template>



<script>
export default {
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

.my-bg {
  background-image: url("https://www.rutamotor.com/wp-content/uploads/2017/12/58-1000x567.jpg");
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  background-color: #42b983;
  -webkit-filter: blur(5px);
  -moz-filter: blur(5px);
  -o-filter: blur(5px);
  -ms-filter: blur(5px);
  filter: blur(5px);
}
</style>
