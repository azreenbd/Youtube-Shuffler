<template>
  <div class="container">
    <form @submit.prevent="goToPlayer">
      <div v-for="(id, index) in playlistIds" :key="index" style="margin: .75rem 0">
        <input type="text" v-model="id.playlistId" placeholder="Enter playlist id" required>
        <button type="button" @click="removePlaylistId(index)" v-if="index < playlistIds.length-1" class="btn-round">-</button>
        <button type="button" @click="addPlaylistId()" v-if="index === playlistIds.length-1" class="btn-round">+</button>
      </div>
      <button type="submit" class="btn-play" style="margin-top: 1rem">Play</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'Home',
  data(){
    return{
        playlistIds: [
          { playlistId: '' }
        ]
    }
  },
  methods: {
    addPlaylistId() {
      this.playlistIds.push({
        playlistId: ''
      });
    },
    removePlaylistId(index) {
      this.playlistIds.splice(index, 1);
    },
    goToPlayer() {
      let url = 'player?';

      for (let [index, playlist] of this.playlistIds.entries()) {
        url += index < this.playlistIds.length-1 ? `id=${playlist.playlistId}&` : `id=${playlist.playlistId}`;
      }

      this.$router.push(url);
    }
  }
}
</script>

<style>
form {
  text-align: center;
}
.btn-round {
  height: 3rem;
  width: 3rem;
}
.btn-play {
  padding: 1rem 4rem;
}
</style>
