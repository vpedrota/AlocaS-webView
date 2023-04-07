<template>
  <v-app>
    <default-bar />

    <default-view />
    <DefaultFooter/>
  </v-app>
</template>

<script lang="ts" setup>
  import DefaultBar from './AppBar.vue'
  import DefaultView from './View.vue'
  import DefaultFooter from './Footer.vue'
</script>
