<script setup>
defineProps({
  msg: {
    type: String,
    required: true,
  },
});
</script>

<template>
  <div class="greetings">
    <h1 class="text-yellow-400">{{ msg }}</h1>
    <h3 class="text-2xl">
      You’ve successfully created a project with
      <a href="https://vitejs.dev/" target="_blank" rel="noopener">Vite</a> +
      <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>.
    </h3>
  </div>
</template>
<script>
import { gsap } from "gsap";

export default {
  mounted() {
    console.log("yuhoo1");
    gsap.from(".greetings", { 
      duration: 5,
      rotation: 360,
      x: 200,
      y: -200,
      repeat: 2,
      yoyo: `true`,
    });
  },
};
</script>
<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  top: -10px;
}

h3 {
  /* font-size: 1.2rem; */
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
