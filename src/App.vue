<script setup lang="ts">
import { ref } from "vue"
import SlickAvatar from "./components/SlickAvatar.vue"

const person = ref(null)
</script>

<template>
  <SlickAvatar v-model="person"/>
</template>

<style scoped lang="scss">

</style>