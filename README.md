# Slick-avatar

Create avatars with one single component. Random or self made avatars.
Just copy ![SlickAvatars.vue](/src/components/SlickAvatar.vue) and the widgets!

![showcase](/showcase.gif)

## So simple and slick 😎
```vue
<script setup lang="ts">
import { ref } from "vue"
import SlickAvatar, { type PersonProp } from "./components/SlickAvatar.vue"

const person = ref<PersonProp>(null)
</script>

<template>
  <SlickAvatar :size="500" v-model="person"/>
</template>
```