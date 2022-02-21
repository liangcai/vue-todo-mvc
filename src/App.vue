<template>
  <ToDo />
  <HelloWorld />
</template>

<script>
import ToDo from "./components/ToDo.vue";
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "App",
  components: {
    ToDo,
    HelloWorld,
  },
};
</script>
