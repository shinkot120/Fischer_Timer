<template>
  <v-app>
    <v-main>
      <!-- GlobalNav -->
      <v-app-bar color="blue darken-4" dark>
        <v-app-bar-nav-icon @click="drawer = true">
          <v-icon>mdi-cog</v-icon>
        </v-app-bar-nav-icon>
        <v-toolbar-title>フィッシャータイマー</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-icon v-show="!isMute" @click="isMute = !isMute">mdi-volume-high</v-icon>
        <v-icon v-show="isMute" @click="isMute = !isMute">mdi-volume-off</v-icon>
      </v-app-bar>

      <!-- Sidebar -->
      <v-navigation-drawer v-model="drawer" :clipped="clipped" fixed app>
        <v-list nav>
          <v-list-item-group active-class="deep-purple--text text--accent-4">
            <v-list-item>
              <v-list-item-icon>
                <v-icon>mdi-cog</v-icon>
              </v-list-item-icon>
              <v-list-item-title class="sidebar-title">タイマー設定</v-list-item-title>
            </v-list-item>
            <v-container fluid>
              <h3 class="text-center mt-5">先手</h3>
              <v-row align="center">
                <v-col class="d-flex">
                  <v-select v-model="sm1" :items="this.minutes" label="分" outlined></v-select>
                  <v-select v-model="ss1" :items="this.seconds" label="秒" outlined></v-select>
                </v-col>
              </v-row>
              <h3 class="text-center mt-5">後手</h3>
              <v-row align="center">
                <v-col class="d-flex">
                  <v-select v-model="sm2" :items="this.minutes" label="分" outlined></v-select>
                  <v-select v-model="ss2" :items="this.seconds" label="秒" outlined></v-select>
                </v-col>
              </v-row>
            </v-container>
          </v-list-item-group>
        </v-list>
      </v-navigation-drawer>

      <!-- Alert -->
      <v-alert dense tile color="success" class="alert text-center pa-4">
        スペースキーを押すと対局が開始されます。手番交代もスペースキーで行うことができます。
      </v-alert>

      <!-- Timer -->
      <v-container>
        <v-row class="pa-5" style="height: 400px;" justify="center" align-content="center">
          <v-row class="text-center">
            <v-col cols="12" sm="6">
              <h3 class="player-name">先手</h3>
              <h3 class="time-text">{{ timeText1 }}</h3>
            </v-col>
            <v-col cols="12" sm="6">
              <h3 class="player-name">後手</h3>
              <h3 class="time-text">{{ timeText2 }}</h3>
            </v-col>
          </v-row>
        </v-row>

        <div class="my-5 text-center">
          <v-btn @click="reset" large color="success" dark>リセット</v-btn>
        </div>

      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  data: () => ({
    clipped: false,
    drawer: false,
    sm1: null,
    ss1: null,
    sm2: null,
    ss2: null,
    player: false,
    elapsed1: 0,
    elapsed2: 0,
    intervalId1: null,
    intervalId2: null,
    timeText1: null,
    timeText2: null,
    isFirst: true,
    isMute: false, //ミュートフラグ
    minutes: [
      0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11,
      12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23,
      24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35,
      36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47,
      48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59
    ],
    seconds: [0,10,20,30,40,50],
  }),

  computed: {
    limit1() {
      let limit1 = ((this.sm1*60*1000)+(this.ss1*1000))
      return limit1 ? limit1 : 300000
    },
    limit2() {
      let limit2 = ((this.sm2*60*1000)+(this.ss2*1000))
      return limit2 ? limit2 : 300000
    }
  },

  methods: {
    updateTime(limit, elapsed) {
      const ms = (limit - elapsed) % 100
      const s = Math.floor((limit - elapsed) / 1000) % 60
      const m = Math.floor((limit - elapsed) / (1000*60)) % 60

      const msStr = ms.toString().padStart(2, '0')
      const sStr = s.toString().padStart(2, '0')
      const mStr = m.toString().padStart(2, '0')

      return `${mStr}:${sStr}:${msStr}`
    },

    startTimer() {
      if (!this.player) {
        if (this.intervalId1 !== null) { return }
        let pre = new Date()
        this.intervalId1 = setInterval(() => {
          const now = new Date()
          this.elapsed1 += now - pre
          pre = now
          this.timeText1 = this.updateTime(this.limit1, this.elapsed1)
        }, 10)
      } else {
        if (this.intervalId2 !== null) { return }
        let pre = new Date()
        this.intervalId2 = setInterval(() => {
          const now = new Date()
          this.elapsed2 += now - pre
          pre = now
          this.timeText2 = this.updateTime(this.limit2, this.elapsed2)
        }, 10)
      }
    },

    stopTimer() {
      if (!this.player) {
        if (this.limit1 === null) { return }
        clearInterval(this.intervalId1)
        this.intervalId1 = null
        if (!this.isFirst) this.elapsed1 -= 5000
        this.timeText1 = this.updateTime(this.limit1, this.elapsed1)
      } else {
        if (this.limit2 === null) { return }
        clearInterval(this.intervalId2)
        this.intervalId2 = null
        this.elapsed2 -= 5000
        this.timeText2 = this.updateTime(this.limit2, this.elapsed2)
      }
      //ゲームスタート時はプレイヤー変更しない
      if (this.isFirst) {
        this.isFirst = false
        return
      }
      //プレイヤー変更
      this.player = !this.player
    },

    sound() {
      let audio = new Audio(require('./assets/sounds/push.mp3'))
      audio.play()
    },

    init() {
      this.player = false
      this.elapsed1 = 0
      this.elapsed2 = 0
      this.isFirst = true
      this.timeText1 = this.updateTime(this.limit1, this.elapsed1)
      this.timeText2 = this.updateTime(this.limit2, this.elapsed2)
    },

    reset() {
      this.stopTimer()
      this.init()
    }
  },

  watch: {
    limit1: function() {
      this.init()
    },
    limit2: function() {
      this.init()
    }
  },

  mounted() {
    this.init()

    window.addEventListener("keypress", e => {
      if (e.keyCode === 32) {
        if (!this.isMute) this.sound()
        if (this.isFirst) {
          this.isFirst = false
          this.startTimer()
        } else {
          this.stopTimer()
          this.startTimer()
        }
      }
    })
  }
};
</script>

<style scoped>
  .sidebar-title {
    font-size: 16px;
    font-weight: bold;
  }

  div.alert {
    color: #fff;
    font-weight: bold;
  }

  .player-name {
    font-size: 40px;
    letter-spacing: 10px;
    padding: 0 1rem 0.5rem;
  }

  .time-text {
    font-size: 40px;
    color: rgb(51, 51, 51);
    font-family: 'DSEG';
    border: 1px solid #333;
    border-radius: 5px;
    padding: 2rem;
    margin-bottom: 1rem;
  }

  /* width 860以下 */
  @media screen and (max-width: 860px) {
    .player-name {
      font-size: 28px;
      letter-spacing: 5px;
    }

    .time-text {
      font-size: 32px;
      padding: 1rem;
    }
  }

</style>