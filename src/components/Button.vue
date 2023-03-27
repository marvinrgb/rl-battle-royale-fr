<template>
  <div class="box">
    <div class="text">{{ props.text }}</div>
  </div>
</template>

<script setup>
const props = defineProps(['text']);
</script>

<style scoped>
.box {
  padding: 1vh 2vw;
  display: grid;
  place-items: center;
  border: 2px solid white;
  width: fit-content;
  border-radius: 1vh;
  cursor: pointer;
  background-color: black;
  color: white;
  margin: 1vh;
}
</style>