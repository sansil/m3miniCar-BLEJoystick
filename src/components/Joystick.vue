<template>
  <div class="w-full flex-col h-screen flex justify-center">
    <div class>
      <a class="button is-info" @click="searchBLE" v-if="!connected">Buscar M30Car!</a>
      <a class="button is-danger" @click="disconnect" v-if="connected">Desconnectar!</a>
    </div>

    <div class="flex justify-around items-center px-6 w-full self-center" v-if="!connected">
      <div id="left" class></div>

      <div class="flex w-full justify-center items-center -mt-64">
        <div class="w-8 inline-block">
          <a @click="rpm>20? rpm= rpm-5: rpm" class="cursor-pointer shadow-2xl">
            <Zondicon
              icon="arrow-thick-left"
              class="fill-current text-gray-400 hover:text-red-400 shadow-2xl"
            />
          </a>
        </div>

        <span
          v-for="index in 19"
          :key="index"
          :class="rpm/5 > index ? classRed(): classGray()"
          style="margin: 0 0.15rem 0 0.15rem;"
        ></span>

        <div class="w-8 inline-block">
          <a @click="rpm<100? rpm= rpm+5: rpm" class="cursor-pointer shadow-2xl">
            <Zondicon
              icon="arrow-thick-right"
              class="fill-current text-gray-400 hover:text-red-400 shadow-2xl"
            />
          </a>
        </div>
      </div>
      <!-- <div class="w-12" style>
        <a @click="rpm<100? rpm= rpm+5: rpm" class="cursor-pointer shadow-2xl">
          <Zondicon
            icon="arrow-thick-up"
            class="fill-current text-red-500 hover:text-red-400 shadow-2xl"
          />
        </a>
        <input
          class="shadow-2xl appearance-none border text-sm rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
          id="username"
          type="number"
          disabled="disabled"
          :value="rpm"
        />
        <a @click="rpm>20? rpm= rpm-5: rpm" class="cursor-pointer shadow-2xl">
          <Zondicon icon="arrow-thick-down" class="fill-current text-red-500 hover:text-red-400" />
        </a>
      </div>-->
      <div id="right" class="shadow-2xl"></div>
    </div>
  </div>
</template>

<script>
import nipplejs from "nipplejs";
require("../assets/sass/main.scss");
const CENTER = 40; // 40 centro
// const LEFT = 55;
// const RIGHT = 30;
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
      velocidadCharString: "0000bbef-1212-efde-1523-785fef13d123",
      RxCharString: "6e400002-b5a3-f393-e0a9-e50e24dcca9e",
      servoPositionCharString: "0000bbed-1212-efde-1523-785fef13d123",
      RxChar: null,
      velocidadChar: null,
      servoChar: null,
      BLEdevice: null,
      connected: false,
      state_moviemiento: {},
      rpm: 30,
      movimiento: {
        velocidad: {
          rpm: 1,
          sentido: 0
        },
        servo: CENTER // 0 = centro, 1 = derecha, 2 = izquierda
      }
    };
  },
  mounted() {
    this.createJoystick();
  },
  methods: {
    classGray() {
      return "w-1 h-8 bg-gray-800 border-2 border-gray-500";
    },
    classRed() {
      return "w-1 h-8 bg-red-700 border-2 border-red-500";
    },
    upVelocity() {},
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
      // console.log(joystickR);

      this.joystickR.on("dir", (evt, data) => {
        // Do something.
        console.log(data);
        var comando = "";
        if (data.direction.angle == "left") {
          comando = "DIRECCION_CUSTOM/8";
        } else {
          comando = "DIRECCION_CUSTOM/6";
        }
        this.writeRxCharacteristic(comando);
        console.log(data.direction);
        //console.log(evt);
      });

      this.joystickR.on("end", async (evt, data) => {
        // Do something.
        this.movimiento.servo = CENTER;
        console.log(data);
        console.log("termino");
        let result = 0;
        while (!result) {
          result = await this.writeRxCharacteristic("DIRECCION_CUSTOM/7");
        }
        //console.log(evt);
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
        //console.log(evt);
      });

      this.joystickL.on("end", async (evt, data) => {
        // Do something.
        console.log(data);
        console.log("termino");
        let result = 0;
        while (!result) {
          result = await this.writeRxCharacteristic("SENTIDO_R/0");
        }
        //console.log(evt);
      });
    }
  }
};
</script>
<style scoped>
</style>