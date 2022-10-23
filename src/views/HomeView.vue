<template>
  <div class="home">
    <div>
      <label for="">Nombre</label>
      <input v-model="user.name" type="text">
    </div>
    <div>
      <label for="">Cédula</label>
      <input v-model="user.identity" type="text">
    </div>
    <div>
      <label for="">Carrera</label>
      <input v-model="user.career" type="text">
    </div>
    <div>
      <label for="">Departamento</label>
      <select name="" id="" v-model="department">
        <option v-for="dep in departments" :key="dep.id" :value="dep.peer_id">{{dep.name}}</option>
      </select>
    </div>
    <button @click="call()">Llamar</button>
    <button @click="finish()">Finalizar</button>
    <!-- Call -->
    <video id="call-in-progress" autoplay></video>
  </div>
</template>

<script>
import { Peer } from "peerjs"
import axios from "axios"
export default {
  name: 'HomeView',
  data() {
    return {
      server: {
        host: "http://192.168.1.53",
        port: 9090,
        peer_port: 6969,
      },
      peer: {},
      user: {
        name: "",
        identity: "",
        career:""
      },
      departments: [],
      department: "",
      callInProgress:{}
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      this.peer = new Peer({
        host: this.server.host.split(":")[1],
        port: this.server.peer_port,
      });

      this.peer.on("disconnected",()=>{
        console.log("deioa");
      })

      axios
        .get(`${this.server.host}:${this.server.port}/api/departments`)
        .then(({ data }) => {
          this.departments = data
        })

    },
    call() {
      const getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;
      getUserMedia({ audio: true }, this.handleMediaStreamCall, this.handleMediaStreamError);

    },
    handleMediaStreamError(error) {
      console.log(error);
    },
    handleMediaStreamCall(stream) {
      this.callInProgress = this.peer.call(this.department, stream, {
        metadata: {
          user: this.user
        }
      });
      this.callInProgress.on('stream', function (remoteStream) {
        const video = document.getElementById("call-in-progress");
        video.srcObject = remoteStream
      });
    },
    finish() {
      this.callInProgress.close();
      console.log("ss");
    }
  }
}
</script>
