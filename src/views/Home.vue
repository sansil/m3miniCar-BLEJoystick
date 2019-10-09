<template>
  <div class="h-screen w-full">
    <joystick></joystick>
  </div>
</template>

<script>
// @ is an alias to /src

require("../assets/sass/main.scss");
import Joystick from "../components/JoystickBLE";

export default {
  components: { Joystick },
  data() {
    return {
      connected: false
    };
  },
  mounted() {},
  methods: {}
};
</script>
