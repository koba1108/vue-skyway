<template>
  <div>
    {{ memberStreams }}
    <video id="my-video" autoplay playsinline/>
    <video id="other-video" autoplay playsinline/>
    <video id="other2-video" autoplay playsinline/>
    <label>
      <input style="border: solid #000000" v-model="roomName">
    </label>
    <v-btn :disabled="!roomName" @click="newRoom">newRoom</v-btn>
  </div>
</template>

<script lang="ts">
  import Peer, {RoomStream} from 'skyway-js'
  import {Component, Vue} from 'nuxt-property-decorator'

  const MODE_SFU = 'sfu'

  interface DataObject {
    src: string
    data: any
  }

  interface MemberStream {
    peerId: string
    stream?: MediaStream
  }

  @Component({
    layout: 'clear'
  })
  export default class Index extends Vue {
    private skywayApiKey: string = process.env.SKYWAY_API_KEY || ''
    peer: Peer = new Peer({
      key: this.skywayApiKey,
    })
    stream: MediaStream | undefined
    room: any
    roomName: string = ''
    memberStreams: Array<MemberStream> = []

    newRoom() {
      this.room = this.peer.joinRoom(this.roomName, {
        mode: MODE_SFU,
        stream: this.stream,
      })
      this.room.on('open', this.onRoomOpen)
      this.room.on('peerJoin', this.onPeerJoin)
      this.room.on('peerLeave', this.onPeerLeave)
      this.room.on('stream', this.onStream)
      this.room.on('log', this.onLog)
      this.room.on('data', this.onData)
      this.room.on('close', this.onClose)
    }

    onStream(stream: RoomStream) {
      console.log('stream peerId', stream.peerId)
      const memberStream = this.memberStreams.find(m => m.peerId === stream.peerId)
      if (!memberStream) {
        this.memberStreams.push({
          peerId: stream.peerId,
        })
      }
      if (this.memberStreams.length == 2) {
        const other2Video: HTMLVideoElement = <HTMLVideoElement>document.getElementById('other2-video')
        if (other2Video) {
          other2Video.srcObject = stream
        }
      } else if (this.memberStreams.length == 1) {
        const otherVideo: HTMLVideoElement = <HTMLVideoElement>document.getElementById('other-video')
        if (otherVideo) {
          otherVideo.srcObject = stream
        }
      }
    }

    onRoomOpen() {
      console.log('onRoomOpen')
      const myVideo: HTMLVideoElement = <HTMLVideoElement>document.getElementById('my-video')
      if (myVideo) {
        myVideo.srcObject = <MediaStream>this.stream
      }
    }

    onPeerJoin(peerId: string) {
      console.log('onPeerJoin peerId', peerId)
      this.memberStreams.push({
        peerId: peerId,
      })
    }

    onPeerLeave(peerId: string) {
      console.log('onPeerLeave peerId', peerId)
      this.memberStreams = this.memberStreams.filter(member => member.peerId === peerId)
    }

    onLog(logs: Array<string>) {
      logs.map(l => console.log('logs', JSON.parse(l)))
    }

    onData(data: DataObject) {
      console.log('data', data)
    }

    onClose() {
      console.log('onClose')
    }

    async mounted() {
      this.stream = await navigator.mediaDevices.getUserMedia({
        audio: false,
        video: true
      })
    }
  }
</script>

<style lang="scss" scoped>
  video {
    transform: scale(-1, 1);
  }
</style>
