<template>
  <div>
    <button @click="conectarBT">START BLE</button>
    <div id="left"></div>
    <div id="right"></div>
  </div>
</template>

<script>
import nipplejs from "nipplejs";
import { isNull } from "util";
export default {
  data() {
    return {
      options: "",
      joystickR: null,
      joystickL: null
    };
  },
  methods: {
    conectarBT() {
      navigator.bluetooth
        .requestDevice({
          filters: [{ services: ["heart_rate"] }]
        })
        .then(device => {})
        .catch(error => {
          console.log("Argh! " + error);
        });
    }
  },
  mounted() {
    this.joystickL = nipplejs.create({
      zone: document.getElementById("left"),
      mode: "static",
      position: { left: "30%", top: "50%" },
      color: "green",
      size: 150,
      lockY: true
    });

    this.joystickR = nipplejs.create({
      zone: document.getElementById("right"),
      mode: "static",
      position: { right: "30%", top: "50%" },
      color: "rgb(35, 153, 213)",
      size: 150,
      lockX: true
    });
    // console.log(joystickR);

    this.joystickL.on("move", function(evt, data) {
      // Do something.
      console.log(data.direction);
      //console.log(evt);
    });

    this.joystickR.on("move", function(evt, data) {
      // Do something.
      console.log(data.direction);
      //console.log(evt);
    });
  }
};
</script>
<style scoped>
</style>s