<template>
  <div>
    <template v-if="isRoomJoined && memberStreams.length < 2">
      <div class="container" v-if="memberStreams.length > 0">
        <h4>メンバーID</h4>
        <ul class="member-list">
          <li v-for="m in memberStreams" :key="m.peerId">{{ m.peerId }}</li>
        </ul>
      </div>
      <div class="container" v-else>
        <h4>まだ誰もいません</h4>
      </div>
    </template>

    <div v-show="isRoomJoined" class="my-video-container" @click="changeViewMode">
      <video id="my-video" ref="myVideo" :srcObject.prop="stream" autoplay playsinline/>
    </div>

    <div class="video-member-container">
      <div class="video-box" v-for="m in memberStreams" :key="m.peerId">
        <!--div class="user-label">{{ m.peerId }}</div-->
        <video :id="m.peerId" :srcObject.prop="m.stream" autoplay playsinline/>
      </div>
    </div>

    <div class="footer container">
      <template v-if="isMenuShow">
        <div>
          <label>
            ルームID：
            <input type="text" v-model="roomId" :disabled="isRoomJoined">
          </label>
        </div>
        <div>
          <label>
            ニックネーム：
            <input type="text" v-model="nickname" :disabled="isRoomJoined">
          </label>
        </div>
        <div>
          <v-btn v-if="isRoomJoined" @click="newRoom">退室する</v-btn>
          <v-btn v-else :disabled="!roomId || !nickname" @click="newRoom">入室する</v-btn>
          <v-btn @click="isMenuShow = false">メニューを閉じる</v-btn>
        </div>
      </template>
      <template v-else>
        <v-btn @click="isMenuShow = true">メニューを開く</v-btn>
      </template>
    </div>
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
    stream: MediaStream | null = null
    isRoomJoined: boolean = false
    isMenuShow: boolean = true
    room: any
    roomId: string = 'koba'
    nickname: string = 'koba'
    memberStreams: Array<MemberStream> = []

    newRoom() {
      if (this.stream) {
        this.room = this.peer.joinRoom(this.roomId, {
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
      } else {
        alert('カメラとマイクを許可してください')
      }
    }

    onStream(stream: RoomStream) {
      console.log('stream peerId', stream.peerId)
      const memberStream = this.memberStreams.find(m => m.peerId === stream.peerId)
      if (!memberStream) {
        this.memberStreams.push({
          peerId: stream.peerId,
          stream: <MediaStream>stream
        })
      }
      if (this.memberStreams.length > 1) {
        this.isMenuShow = false
      }
    }

    onRoomOpen() {
      console.log('onRoomOpen')
      this.isRoomJoined = true
    }

    onPeerJoin(peerId: string) {
      console.log('onPeerJoin peerId', peerId)
    }

    onPeerLeave(peerId: string) {
      console.log('onPeerLeave peerId', peerId)
      this.memberStreams = this.memberStreams.filter(member => member.peerId !== peerId)
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

    changeViewMode() {
      console.log('changeViewMode')
    }

    public get myVideoStyle(): Object {
      console.log('this.$refs', this.$refs)
      if (this.$ref !== {} && this.$refs.myVideo) {
        const videoHeight = this.$refs.myVideo.videoHeight
        const videoWidth = this.$refs.myVideo.videoWidth
        return videoHeight > videoWidth ? {
          height: `${videoHeight}px`,
          width: `${videoHeight}px`,
        } : {
          height: `${videoWidth}px`,
          width: `${videoWidth}px`,
        }
      } else {
        return {
          height: '100vh',
          width: '100vw',
        }
      }
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

  input[type=text] {
    border: solid 1px #000000;
    padding-left: 5px;
  }

  .container {
    padding: 10px;

    div {
      padding-top: 5px;
    }
  }

  .my-video-container {
    position: fixed;
    right: 10px;
    bottom: 10px;
    z-index: 10;

    #my-video {
      background-color: black;
      border-radius: 50%;
      border: solid 5px red;
    }
  }

  .video-member-container {
    position: fixed;
    height: 100vh;
    width: 100vw;
    display: grid;
    grid-row: 1 / 4;
    grid-column: 1 / 4;

    .video-box {
      position: relative;
      top: 0;
      max-height: 40vh;

      .user-label {
        position: relative;
        top: 35px;
        left: 5px;
        height: 24px;
        width: 200px;
        text-align: center;
        color: white;
        background-color: black;
        border-radius: 20px;
        z-index: 5;
      }

      video {
        max-height: 40vh;
        max-width: 100%;
        height: auto;
      }
    }
  }

  .footer {
    position: fixed;
    bottom: 10px;
  }
</style>
