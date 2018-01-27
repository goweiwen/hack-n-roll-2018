<template>
  <div id="app">
    <audio ref="audio">
      <source src="/static/audio/mining.m4a" type="audio/m4a">
      <source src="/static/audio/mining.ogg" type="audio/ogg">
      <source src="/static/audio/mining.wav" type="audio/wav">
    </audio>
    <h1>Mine mine mine</h1>
    <p>Mined: {{ mined }}</p>
    <p>State: {{ state === 0 ? 'idle' : state === 1 ? 'downswing' : state === 2 ? 'mining' : 'upswing' }}</p>
    <p>{{ data }}</p>
  </div>
</template>

<script>
import GyroNorm from 'gyronorm/dist/gyronorm.complete.min.js'

const STATES = {
  IDLE: 0,
  DOWNSWING: 1,
  MINING: 2,
  UPSWING: 3
}

const MINING_INTERVAL = 1000

export default {
  data () {
    return {
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
    const gn = new GyroNorm()
    await gn.init()
    gn.start(this.onDeviceMotion.bind(this))
  },

  methods: {
    onDeviceMotion (data) {
      this.data = data

      if (this.state === STATES.IDLE) {
        if (Date.now() - this.lastMined > MINING_INTERVAL && data.dm.z > 2) {
          this.state = STATES.DOWNSWING
          this.$refs.audio.play()
        }
      } else if (this.state === STATES.DOWNSWING) {
        if (data.dm.z < 0) {
          this.mined++
          this.state = STATES.MINING
        }
      } else if (this.state === STATES.MINING) {
        if (data.dm.z < -1) {
          this.state = STATES.UPSWING
        }
      } else if (this.state === STATES.UPSWING) {
        if (data.dm.z > 0) {
          this.state = STATES.IDLE
        }
      }
    }
  }
}
</script>

<style>

</style>
