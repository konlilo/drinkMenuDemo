<template>
  <div class="h2 text-success">飲料訂單 - Drink Menu</div>
  <DrinkMenu msg="Welcome to Your Vue.js App" />
</template>

<script>
// @ is an alias to /src
import DrinkMenu from "@/components/DrinkMenu.vue";

export default {
  name: "Home",
  components: {
    DrinkMenu,
  },
};
</script>
