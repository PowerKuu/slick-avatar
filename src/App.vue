<script setup lang="ts">
import { ref } from "vue"
import SlickAvatar, { type PersonProp } from "./components/SlickAvatar.vue"

const person = ref<PersonProp>(null)
</script>

<template>
  <SlickAvatar :size="500" v-model="person"/>
</template>