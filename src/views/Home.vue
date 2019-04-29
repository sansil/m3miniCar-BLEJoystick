<template>
  <div class="home">
    <joystick></joystick>
  </div>
</template>

<script>
// @ is an alias to /src

require("../assets/sass/main.scss");
import Joystick from "../components/Joystick";

export default {
  name: "home",
  components: { Joystick },
  data() {
    return {};
  },
  mounted() {},
  methods: {}
};
</script>
