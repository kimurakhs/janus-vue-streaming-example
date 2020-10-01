<template>
  <div>
    <select v-model="streamList.selected" :disabled="stream">
      <option v-for="option in streamList.options" :value="option.id" :key="option.id">
        {{ option.description }}
      </option>
    </select>
    <button @click="start" :disabled="stream">Start</button>
    <button @click="stop" :disabled="!stream">Stop</button>
    <video autoplay :srcObject.prop="stream" v-if="stream"></video>
    <div v-if="!stream">No video</div>
    <div v-if="status">Status: {{ status }}</div>
    <div v-if="error">{{ error }}</div>
  </div>
</template>

<script>
import { Janus } from 'janus-gateway'

const JANUS_URL = 'http://raspberrypi.local:8088/janus'

export default {
  name: 'App',
  data() {
    return {
      janus: null,
      plugin: null,
      stream: null,
      error: null,
      status: null,
      streamList: {
        selected: null,
        options: []
      }
    }
  },
  mounted() {
    Janus.init({
      debug: true,
      dependencies: Janus.useDefaultDependencies(),
      callback: () => {
        this.connect(JANUS_URL)
      }
    })
  },
  methods: {
    connect(server) {
      this.janus = new Janus({
        server,
        success: () => {
          this.attachPlugin()
        },
        error: (error) => {
          this.onError('Failed to connect to janus server', error)
        },
        destroyed: () => {
          window.location.reload()
        }
      })
    },
    attachPlugin() {
      this.janus.attach({
        plugin: "janus.plugin.streaming",
        opaqueId: 'thisisopaqueid',
        success: (plugin) => {
          this.plugin = plugin
          this.updateStreamsList()
        },
        error: (error) => {
          this.onError('Error attaching plugin... ', error)
        },
        onmessage: (msg, jsep) => {
          if (msg && msg.result) {
            const result = msg.result
            if (result.status) {
              this.status = result.status
            }
          } else if (msg && msg.error) {
            this.onError(msg.error)
          }
          if (jsep) {
            let stereo = (jsep.sdp.indexOf("stereo=1") !== -1);
            this.plugin.createAnswer({
              jsep: jsep,
              media: { audioSend: false, videoSend: false },
              customizeSdp: (jsep) => {
                if(stereo && jsep.sdp.indexOf("stereo=1") === -1) {
                  jsep.sdp = jsep.sdp.replace("useinbandfec=1", "useinbandfec=1;stereo=1");
                }
              },
              success: (jsep) => {
                var body = { request: "start" };
                this.plugin.send({ message: body, jsep: jsep });
              },
              error: (error) => {
                this.onError("WebRTC error:", error);
              }
            })
          }
        },
        onremotestream: (stream) => {
          this.onRemoteStream(stream)
        },
        oncleanup: () => {
          this.onCleanup()
        }
      })
    },
    updateStreamsList() {
      this.plugin.send({
        message: { request: 'list' },
        success: (result) => {
          if (!result) {
            this.onError("Got no response to our query for available streams")
          }
          this.streamList.options = result.list
          if (result.list.length) {
            this.streamList.selected = this.streamList.options[0].id
          }
        }
      })
    },
    start() {
      this.plugin.send({ message: { request: "watch", id: this.streamList.selected } })
    },
    stop() {
      this.plugin.send({ message: { request: "stop" } })
      this.plugin.hangup()
      this.onCleanup()
    },
    onRemoteStream(stream) {
      if (stream.active) {
        this.stream = stream
      } else {
        this.stream = null
      }
    },
    onCleanup() {
      this.stream = null
      this.status = null
      this.error = null
    },
    onError(message, error='') {
      Janus.error(message, error)
      this.error = message + error
    }
  }
}
</script>

<style>
body {
  background-color: #333;
  font-family: sans-serif;
  color: #ddd;
  text-align: center;
}
select {
  background-color: #ddd;
}
video {
  width: 100%;
}
</style>
