<template>
  <div id="box">
    <div class="name">{{ props.name }}</div>
  </div>
</template>

<script setup>
const props = defineProps(['name']);
</script>

<style scoped>
#box {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #222;
  border: 1px solid #aaa;
}
#box div {
  padding: 1vh 3vw;
}
</style>