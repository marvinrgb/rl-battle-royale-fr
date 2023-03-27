<script setup>
import Player from './components/Player.vue';
import Button from './components/Button.vue';
import { io } from 'socket.io-client';
import { ref } from 'vue';
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer } from '@vue-leaflet/vue-leaflet';
let socket = io("ws://localhost:3000");

const refdata = ref({
  'state' : 'mid'
});

let zoom = 16;

window.localStorage.getItem('player_id') ? signIn() : signUp();

function signIn() {
  let player_id = window.localStorage.getItem('player_id');
  socket.emit('sign-in', {"player_id" : player_id});
}

function signUp() {
  let username = prompt('Enter username: ');
  socket.emit('sign-up', {"username" : username});
}
socket.on('sign-up-new-id', (data) => {
  window.localStorage.setItem('player_id', data.player_id);
});

socket.on('reset-client-auth', () => {
  window.localStorage.removeItem('player_id');
  location.reload();
})

function joinGame() {
  let code = prompt('Game Code: ');
  socket.emit('join-game', ({ 'code' : code }));
}

socket.on('game-joined', (data) => {
  refdata.value.state = 'lobby';
  refdata.value.code = data.code;
  refdata.value.players = data.players
  console.log(data)
})
</script>


<template>
  <div v-if="refdata.state == 'pre'" id="pre-game-screen" class="screen">
    <h1 class="pre-heading">Reallife Battle Royale</h1>
    <Button @click="socket.emit('create-game')" text="Create Game"/>
    <Button @click="joinGame()" text="Join Game"/>
  </div>
  <div v-else-if="refdata.state == 'lobby'" id="lobby-game-screen" class="screen">
     <h1 id="lobby-code">Game {{ refdata.code }}</h1>
     <Button id="lobby-start-game" text="Start Game" />
     <Player v-for="player in refdata.players" :name="player.username"/>
  </div>
  <div v-else-if="refdata.state == 'mid'" id="mid-game-screen" class="screen">
     <div style="height:60vh; width:100vw">
      <l-map ref="map" v-model:zoom="zoom" :center="[50.599, 8.417]">
        <l-tile-layer
          url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
          layer-type="base"
          name="OpenStreetMap"
        ></l-tile-layer>
      </l-map>
     </div>
  </div>
  <div v-else-if="refdata.state == 'post'" id="post-game-screen" class="screen">
     
  </div>
</template>


<style scoped>
#lobby-start-game {
  margin-bottom: 5vh;
}
#lobby-game-screen {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
.screen {
  background-color: black;
  height: 100vh;
  width: 100vw;
}
.pre-heading {
  margin-bottom: 5vh;
}
#pre-game-screen {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}
</style>


<style>
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  color: white;
}
</style>