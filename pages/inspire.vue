<template>
  <v-layout>
    <v-flex class="text-center">
      <section class="container">
        <div>
          <div>
            <h2>お相手</h2>
            <video id="their-video" width="400" autoplay playsinline/>
          </div>
          <div>
            <h2>あなた</h2>
            <video id="my-video" muted width="400" autoplay playsinline/>
          </div>

          <div class="main">
            <h2>設定</h2>
            <p>
              マイク:
              <select v-model="selectedAudio" @change="onChange">
                <option disabled value="">選択してください</option>
                <option v-for="(audio, key, index) in audios" :key="index" :value="audio.value">
                  {{ audio.text }}
                </option>
              </select>
            </p>

            <p>
              カメラ:
              <select v-model="selectedVideo" @change="onChange">
                <option disabled value="">選択してください</option>
                <option v-for="(video, key, index) in videos" v-bind:key="index" :value="video.value">
                  {{ video.text }}
                </option>
              </select>
            </p>

            <v-btn @click="connectLocalCamera">マイクとカメラに接続する</v-btn>

            <div>
              <p>部屋ID: <span id="my-id">{{peerId}}</span></p>
              <p>他のブラウザでこのIDをコールしましょう。</p>
              <h3>コールする</h3>
              <div>
                <input v-model="callToId" placeholder="部屋IDを入力してください"/>
              </div>
              <v-btn @click="makeCall">入室する</v-btn>
            </div>
          </div>

        </div>
      </section>
    </v-flex>
  </v-layout>
</template>

<script lang="ts">
  import Peer, {MediaConnection} from 'skyway-js'
  import {Component, Vue} from 'nuxt-property-decorator'

  @Component({
    filters: {
      triple: (n: number): number => 3 * n,
    },
  })
  export default class Inspire extends Vue {
    private skywayApiKey: string = process.env.SKYWAY_API_KEY || ''
    peer: Peer | null = null
    peerId: string = ''
    callToId: string = ''
    audios: object[] = []
    videos: object[] = []
    selectedAudio: string = ''
    selectedVideo: string = ''
    name: string = 'name'
    age: number = 21
    localStream: MediaStream | undefined

    get double() {
      return this.age * 2
    }

    imgClick() {
      alert('imgClick!!')
    }

    onChange() {
      if (this.selectedAudio && this.selectedVideo) {
        this.connectLocalCamera()
      }
    }

    makeCall() {
      console.log('makeCall')
      if (this.peer) {
        const call = this.peer.call(this.callToId, this.localStream)
        this.connect(call)
      }
    }

    connect(call: MediaConnection) {
      call.on('stream', stream => {
        const el: HTMLVideoElement = <HTMLVideoElement>document.getElementById('their-video')
        el.srcObject = stream
        el.play()
      })
    }

    async connectLocalCamera() {
      const stream = await navigator.mediaDevices.getUserMedia({
        audio: true,
        video: true,
      })
      const myVideo: HTMLVideoElement = <HTMLVideoElement>document.getElementById('my-video')
      if (myVideo) {
        myVideo.srcObject = stream
      }
      this.localStream = stream
    }

    peerInit() {
      this.peer = new Peer({
        key: this.skywayApiKey,
        debug: 3,
      })
      this.peer.on('open', this.peerOnOpen)
      this.peer.on('call', this.peerOnCall)
    }

    peerOnOpen() {
      this.peerId = this.peer ? this.peer.id : ''
    }

    peerOnCall(call: MediaConnection) {
      call.answer(this.localStream)
      this.connect(call)
    }

    async mounted() {
      this.peerInit()
      const deviceInfos: MediaDeviceInfo[] = await navigator.mediaDevices.enumerateDevices()
      deviceInfos.map(deviceInfo => {
        switch (deviceInfo.kind) {
          case 'audioinput':
            const audioText: string = deviceInfo.label || `Microphone ${this.audios.length + 1}`
            this.audios.push({
              text: audioText,
              value: deviceInfo.deviceId
            })
            return
          case 'videoinput':
            const videoText: string = deviceInfo.label || `Camera  ${this.videos.length - 1}`
            this.videos.push({
              text: videoText,
              value: deviceInfo.deviceId
            })
            return
        }
      })
    }

    beforeDestroy() {
      console.log('beforeDestroy')
    }
  }
</script>
