<template>
  <div id="app">
    <div class="flex-grid">
      <h1 class="col center animated bounceInDown">Swing to Mine!</h1>
      <div class="col" />
      <h1 class="col center animated bounceInDown">Mined: {{ mined }}</h1>
    </div>
    <coins :count="mined"/>
  </div>
</template>

<script>
import GyroNorm from 'gyronorm/dist/gyronorm.complete.min.js'
import { Howl } from 'howler'
import Coins from './components/Coins.vue'

const STATES = {
  IDLE: 0,
  DOWNSWING: 1,
  UPSWING: 2
}

const MINING_SOUND_URLS = [
  '/static/audio/mining.wav',
  '/static/audio/mining.ogg'
]
const MINING_INTERVAL = 500

export default {
  components: {
    Coins
  },

  data () {
    return {
      audio: null,
      lastMined: 0,
      mined: 0,
      state: STATES.IDLE,
      log: '',
      data: {
        do: {
          alpha: 0,
          beta: 0,
          gamma: 0,
          absolute: 0
        },
        dm: {
          x: 0,
          y: 0,
          z: 0,
          gx: 0,
          gy: 0,
          gz: 0,
          alpha: 0,
          beta: 0,
          gamma: 0
        }
      }
    }
  },

  computed: {},

  async created () {
    try {
      this.audio = new Howl({ src: MINING_SOUND_URLS })
    } catch (e) {
      console.log(e)
    }

    try {
      const gn = new GyroNorm()
      await gn.init()
      gn.start(this.onDeviceMotion.bind(this))
    } catch (e) {
      console.log(e)
    }
  },

  methods: {
    onDeviceMotion (data) {
      this.data = data

      if (this.state === STATES.IDLE) {
        if (Date.now() - this.lastMined > MINING_INTERVAL && data.dm.x > 3) {
          this.lastMined = Date.now()
          this.audio.play()
          this.mined++
          this.state = STATES.DOWNSWING
        }
      } else if (this.state === STATES.DOWNSWING) {
        if (data.dm.x < -1) {
          this.state = STATES.UPSWING
        }
      } else if (this.state === STATES.UPSWING) {
        if (data.dm.x > 0) {
          this.state = STATES.IDLE
        }
      }
    }
  }
}
</script>

<style>
@font-face {
  font-family: 'Proxima Soft';
  src: url('/static/fonts/ProximaNovaSoft-Regular.otf') format('opentype');
}

* {
  color: black;
  font-family: 'Proxima Soft';
  text-shadow: 0 0 10px grey;
}

.flex-grid {
  display: flex;
}
.col {
  flex: 1;
}

.center {
  text-align: center;
}
</style>
