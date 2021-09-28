<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <Player/>
  </div>
</template>

<script>
import Player from './components/Player.vue'

export default {
  name: 'App',
  components: {
    Player
  },
  data(){
    return{
      playlistIds: [
        { id: '' }
      ],
    }
  },
  methods: {
    addPlaylistId() {
      this.playlistIds.push({id:''});
    },
    removePlaylistId(index) {
      array.splice(index, 1);
    }
  }
}
</script>

<style>
body {
  background: #121312 !important;
}
</style>
