<script setup>
import Player from './components/Player.vue';
import Button from './components/Button.vue';
import { io } from 'socket.io-client';
import { ref } from 'vue';
import "leaflet/dist/leaflet.css"
import { LMap, LTileLayer, LMarker, LCircleMarker, LCircle } from '@vue-leaflet/vue-leaflet';
let socket = io("ws://localhost:3000");

const refdata = ref({
  'state' : 'pre',
  'circles' : [],
  'survivors' : [{
    'username' : 'horst',
    'coords' : [1, 1]
  }]
});

navigator.geolocation.getCurrentPosition((pos) => {
  console.log(pos)
  refdata.value.coords = [
    pos.coords.latitude,
    pos.coords.longitude
  ]
  refdata.value.main_center = [
    pos.coords.latitude,
    pos.coords.longitude
  ]
})

let zoom = 15;

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

function startGame() {
  socket.emit('start-game', (refdata.value.coords));
}

socket.on('start-game', (data) => {
  refdata.value.state = 'mid';
  refdata.value.circles.push({
    'coords' : data.coords,
    'radius' : data.radius,
    'color' : '#f00'
  })
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
     <Button @click="startGame()" id="lobby-start-game" text="Start Game" />
     <Player v-for="player in refdata.players" :name="player.username"/>
  </div>
  <div v-else-if="refdata.state == 'mid'" id="mid-game-screen" class="screen">
    <div v-if="refdata.main_center">
      <div style="height:60vh; width:100vw">
       <l-map :use-global-leaflet="false" ref="map" v-model:zoom="zoom" :center="refdata.main_center">
          <l-tile-layer
            url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
            layer-type="base"
            name="OpenStreetMap"
          ></l-tile-layer>
          <l-circle v-if="refdata.circles[0]" :lat-lng="refdata.circles[0].coords" :radius="refdata.circles[0].radius" :color="refdata.circles[0].color">
            
          </l-circle>
          <l-circle-marker color="#00f" :radius="4" :lat-lng="refdata.coords"></l-circle-marker>
       </l-map>
      </div>
      <div id="mid-lower-area">
        <table id="survivor-table">
          <tr v-for="survivor in refdata.survivors">
            <td>{{ survivor.username }}</td>
          </tr>
        </table>
      </div>
    </div>
    <div v-else style="background-color: black; height: 100vh;">
      loading...
    </div>
  </div>
  <div v-else-if="refdata.state == 'post'" id="post-game-screen" class="screen">
     
  </div>
</template>


<style scoped>
#survivor-table {
  height: fit-content;
  width: 30%;
}


#mid-lower-area {
  height: 40vh;
  width: 100vw;
}


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