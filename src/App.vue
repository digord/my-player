<template>
  <v-app>
    <my-player></my-player>
  </v-app>
</template>

<script>
import MyPlayer from './components/MyPlayer.vue'

export default {
  name: 'app',
  components: {
    MyPlayer
  }
}
</script>
