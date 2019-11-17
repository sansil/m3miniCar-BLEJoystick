<template>
  <div class="flex-col flex justify-center h-full">
    <div class>
      <a class="button is-link font-medium" @click="searchBLE" v-if="!connected">
        Buscar M3Car
        <Zondicon icon="bluetooth" class="fill-current text-white w-6 ml-2" />
      </a>
      <a class="button is-danger is-outlined" @click="disconnect" v-if="connected">Desconnectar!</a>
    </div>

    <div class="flex justify-around items-center px-6 w-full self-center" v-if="connected">
      <div id="left" class></div>
      <div class="w-full -mt-64">
        <span class="text-gray-600 mb-1 inline-block uppercase font-semibold text-sm">potencia</span>
        <div class="flex w-full justify-center items-center">
          <div class="w-6 inline-block mr-3">
            <a @click="rpm>20? rpm= rpm-5: rpm" class="cursor-pointer shadow-2xl">
              <Zondicon
                icon="arrow-thick-left"
                class="fill-current text-red-600 border border-gray-700"
              />
            </a>
          </div>

          <span
            v-for="index in 19"
            :key="index"
            :class="rpm/5 > index ? classRed(): classGray()"
            style="margin: 0 0.15rem 0 0.15rem;"
          ></span>

          <div class="w-6 inline-block ml-3">
            <a @click="rpm<100? rpm= rpm+5: rpm" class="cursor-pointer shadow-2xl">
              <Zondicon
                icon="arrow-thick-right"
                class="fill-current text-red-600 shadow-2xl border-gray-700 border"
              />
            </a>
          </div>
        </div>
      </div>
      <div id="right" class></div>
    </div>
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
      configServiceUuid: "6e400001-b5a3-f393-e0a9-e50e24dcca9e",
      RxCharString: "6e400002-b5a3-f393-e0a9-e50e24dcca9e",
      RxChar: null,
      velocidadChar: null,
      servoChar: null,
      BLEdevice: null,
      connected: false,
      state_moviemiento: {},
      rpm: 30
    };
  },
  methods: {
    classGray() {
      return "w-1 h-8 bg-gray-800 border border-gray-500";
    },
    classRed() {
      return "w-1 h-8 bg-red-600 border border-red-300";
    },
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
        this.RxChar = await service.getCharacteristic(this.RxCharString);

        // cargo estado inicial de movimiento
        this.createJoystick();
      } catch (error) {
        console.log("Argh! " + error);
      }
    },
    async writeRxCharacteristic(comando) {
      try {
        if (this.connected) {
          await this.RxChar.writeValue(new TextEncoder().encode(comando));
          console.log("characteristic servo updated!");
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
        position: { left: "19%", top: "50%" },
        color: "red",
        size: 170,
        lockY: true
      });

      this.joystickR = nipplejs.create({
        zone: document.getElementById("right"),
        mode: "static",
        position: { right: "19%", top: "50%" },
        color: "red",
        size: 170,
        lockX: true
      });

      this.joystickR.on("dir", (evt, data) => {
        // Do something.
        console.log(data);
        var comando = "";
        if (data.direction.angle == "left") {
          comando = "DIRECCION_CUSTOM/19";
        } else {
          comando = "DIRECCION_CUSTOM/15";
        }
        this.writeRxCharacteristic(comando);
        console.log(data.direction);
      });

      this.joystickR.on("end", async (evt, data) => {
        console.log(data);
        console.log("send end");
        let result = 0;
        while (!result) {
          result = await this.writeRxCharacteristic("DIRECCION_CUSTOM/17");
        }
      });

      this.joystickL.on("dir", (evt, data) => {
        // Do something.
        console.log("jostick L" + data.direction);
        var comando = "";
        if (data.direction.angle == "down") {
          let rpm_a = 100 - this.rpm;
          rpm_a === 0 ? (rpm_a = 5) : rpm_a;
          comando = "SENTIDO_R/" + String(rpm_a);
        } else {
          comando = "SENTIDO_A/" + String(this.rpm);
        }
        this.writeRxCharacteristic(comando);
      });

      this.joystickL.on("end", async (evt, data) => {
        // Do something.
        console.log(data);
        console.log("send end");
        let result = 0;
        while (!result) {
          result = await this.writeRxCharacteristic("SENTIDO_R/0");
        }
      });
    }
  }
};
</script>
<style scoped>
</style>