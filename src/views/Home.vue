<template>
  <div class="home">
    <ToDo />
  </div>
</template>

<script>
import ToDo from "@/components/ToDo/ToDo.vue";

export default {
  name: "Home",
  components: {
    ToDo,
  },
};
</script>
