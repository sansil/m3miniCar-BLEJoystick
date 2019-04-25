<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <a class="button is-info" @click="searchBLE">BLE!</a>
  </div>
</template>

<script>
// @ is an alias to /src

require("../assets/sass/main.scss");

export default {
  name: "home",
  components: {},
  data() {
    return {
      configServiceUuid: "0000f00d-1212-efde-1523-785fef13d123"
    };
  },
  mounted() {},
  methods: {
    searchBLE() {
      console.log(navigator);
      navigator.bluetooth
        .requestDevice({
          acceptAllDevices: true,
          optionalServices: [this.configServiceUuid]
        })
        .then(device => {
          console.log(device);
        })
        .catch(error => {
          console.log("Argh! " + error);
        });
    }
  }
};
</script>
