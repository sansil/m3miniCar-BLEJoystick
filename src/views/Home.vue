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
    async searchBLE() {
      try {
        console.log("Requesting any Bluetooth Device...");
        const device = await navigator.bluetooth.requestDevice({
          // filters: [...] <- Prefer filters to save energy & show relevant devices.
          acceptAllDevices: true,
          optionalServices: [this.configServiceUuid]
        });
        console.log("Connecting to GATT Server...");
        const server = await device.gatt.connect();

        console.log("Getting Service...");
        const service = await server.getPrimaryService(this.configServiceUuid);
      } catch (error) {
        document.querySelector("#writeButton").disabled = true;
        console.log("Argh! " + error);
      }
    }
  }
};
</script>
