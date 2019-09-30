<template>
  <div>
    <a class="button is-info" @click="searchBLE">BLE!</a>
    <a class="button is-danger" @click="disconnect">Desconnectar!</a>
    <div id="left"></div>
    <div class="w-12" style>
      <a @click="upVelocity" class="cursor-pointer">
        <Zondicon icon="arrow-thick-up" class="fill-current text-blue-800 hover:text-blue-600" />
      </a>
      <input
        class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
        id="username"
        type="number"
        placeholder="30"
      />
      <a @click="upVelocity" class="cursor-pointer">
        <Zondicon icon="arrow-thick-down" class="fill-current text-blue-800 hover:text-blue-600" />
      </a>
    </div>
    <div id="right"></div>
  </div>
</template>

<script>
import nipplejs from "nipplejs";
require("../assets/sass/main.scss");
import Zondicon from "vue-zondicons";

export default {
  components: {
    Zondicon
  },
  data() {
    return {
      options: "",
      joystickR: null,
      joystickL: null,
      configServiceUuid: "0000f00d-1212-efde-1523-785fef13d123",
      velocidadCharString: "0000bbef-1212-efde-1523-785fef13d123",
      servoPositionCharString: "0000bbed-1212-efde-1523-785fef13d123",
      velocidadChar: null,
      servoChar: null,
      BLEdevice: null,
      connected: false,
      state_moviemiento: {},
      movimiento: {
        velocidad: {
          rpm: 1,
          sentido: 0
        },
        servo: 0 // 0 = centro, 1 = derecha, 2 = izquierda
      }
    };
  },
  mounted() {
    this.createJoystick();
  },
  methods: {
    disconnect() {
      this.BLEdevice.gatt.disconnect();
      this.connected = false;
    },
    async searchBLE() {
      try {
        console.log("Requesting any Bluetooth Device...");
        const device = await navigator.bluetooth.requestDevice({
          // filters: [...] <- Prefer filters to save energy & show relevant devices.
          acceptAllDevices: true,
          optionalServices: [this.configServiceUuid]
        });
        console.log("Connecting to GATT Server...");
        this.BLEdevice = device;
        const server = await device.gatt.connect();
        this.connected = true;
        console.log("Getting Service...");
        const service = await server.getPrimaryService(this.configServiceUuid);

        console.log("Getting Characteristic...");
        this.velocidadChar = await service.getCharacteristic(
          this.velocidadCharString
        );
        this.servoChar = await service.getCharacteristic(
          this.servoPositionCharString
        );
        // cargo estado inicial de movimiento
        var initServoPositionValue = Uint8Array.of(
          "0x" + Number(this.movimiento.servo).toString(8)
        );
        await this.servoChar.writeValue(initServoPositionValue);

        var initVelocidadPositionValue = Uint8Array.of(
          "0x" + Number(this.movimiento.velocidad.rpm).toString(8),
          "0x" + Number(this.movimiento.velocidad.sentido).toString(8)
        );
        await this.velocidadChar.writeValue(initVelocidadPositionValue);
        console.log("characteristics updated!");
        this.createJoystick();
      } catch (error) {
        console.log("Argh! " + error);
      }
    },
    async writeServoCharacteristic() {
      try {
        var newServoValue = Uint8Array.of(
          "0x" + Number(this.movimiento.servo).toString(8)
        );
        console.log(newServoValue);
        if (this.connected) {
          await this.servoChar.writeValue(newServoValue);
          console.log("characteristic servo updated!");
          return 1;
        }
        return 0;
      } catch (error) {
        console.log("Argh! " + error);
        return 0;
      }
    },
    async writeVelocidadCharacteristic() {
      try {
        var newVelocidadValues = Uint8Array.of(
          "0x" + Number(this.movimiento.velocidad.rpm).toString(8),
          "0x" + Number(this.movimiento.velocidad.sentido).toString(8)
        );
        if (this.connected) {
          await this.velocidadChar.writeValue(newVelocidadValues);
          console.log("characteristic velocidad updated!");
          return 1;
        }
        return 0;
      } catch (error) {
        console.log("Argh! " + error);
        return 0;
      }
    },
    async createJoystick() {
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

      this.joystickR.on("dir", (evt, data) => {
        // Do something.
        console.log(data);
        if (data.direction.x == "left") {
          this.movimiento.servo = 8;
        } else {
          this.movimiento.servo = 2;
        }
        this.writeServoCharacteristic();
        console.log(data.direction);
        //console.log(evt);
      });

      this.joystickR.on("end", async (evt, data) => {
        // Do something.
        this.movimiento.servo = 4;
        console.log(data);
        console.log("termino");
        let result = 0;
        while (!result) {
          result = await this.writeServoCharacteristic();
        }
        //console.log(evt);
      });

      this.joystickL.on("dir", (evt, data) => {
        // Do something.
        console.log(data.direction);
        this.writeVelocidadCharacteristic();
        //console.log(evt);
      });

      this.joystickL.on("end", async (evt, data) => {
        // Do something.
        console.log(data);
        console.log("termino");
        let result = 0;
        while (!result) {
          result = await this.writeVelocidadCharacteristic();
        }
        //console.log(evt);
      });
    }
  }
};
</script>
<style scoped>
</style>