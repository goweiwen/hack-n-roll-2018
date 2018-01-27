<template>
  <div class="container" ref="container">
    <img src="../../static/img/logo.png" />
  </div>
</template>

<script>
import {
  Engine,
  Render,
  World,
  Bodies,
  Common,
  MouseConstraint
} from 'matter-js'

export default {
  props: ['count'],

  watch: {
    count (curr, prev) {
      while (curr > prev) {
        this.addCoin(Math.random() * window.innerWidth, 100)
        prev++
      }
    }
  },

  data () {
    return {
      engine: null
    }
  },

  methods: {
    addCoin (x, y) {
      World.add(
        this.engine.world,
        Bodies.circle(x, y, 46, {
          friction: 0.01,
          restitution: 0.4,
          render: {
            strokeStyle: '#ffffff',
            sprite: {
              texture:
                '../../static/img/coin.png'
            }
          }
        })
      )
    }
  },

  mounted () {
    const canvasContainer = this.$refs.container

    const Demo = {}

    let _engine
    let _render
    let _sceneName = 'coins'
    let _sceneWidth = window.innerWidth
    let _sceneHeight = window.innerHeight
    let _deviceOrientationEvent

    Demo.init = function () {
      _engine = Engine.create()

      _render = Render.create({
        element: canvasContainer,
        engine: _engine,
        options: {
          width: _sceneWidth,
          height: _sceneHeight,
          background: '#8e94e4',
          showAngleIndicator: false,
          wireframes: false
        }
      })
      Render.run(_render)

      setTimeout(function () {
        let runner = Engine.run(_engine)

        // pass through runner as timing for debug rendering
        _engine.metrics.timing = runner

        Demo.updateScene()
      }, 800)

      window.addEventListener(
        'deviceorientation',
        function (event) {
          _deviceOrientationEvent = event
          Demo.updateGravity(event)
        },
        true
      )

      window.addEventListener(
        'orientationchange',
        function () {
          Demo.updateGravity(_deviceOrientationEvent)
          Demo.updateScene()
        },
        false
      )
    }

    Demo.coins = function () {
      let _world = _engine.world

      Demo.reset()

      World.add(_world, MouseConstraint.create(_engine))
    }

    Demo.updateScene = function () {
      if (!_engine) return

      let boundsMax = _engine.world.bounds.max
      let renderOptions = _render.options
      let canvas = _render.canvas

      boundsMax.x = _sceneWidth
      boundsMax.y = _sceneHeight

      canvas.width = renderOptions.width = _sceneWidth
      canvas.height = renderOptions.height = _sceneHeight

      Demo[_sceneName]()
    }

    Demo.updateGravity = function (event) {
      if (!_engine) return

      let orientation = window.orientation
      let gravity = _engine.world.gravity

      if (orientation === 0) {
        gravity.x = Common.clamp(event.gamma, -90, 90) / 90
        gravity.y = Common.clamp(event.beta, -90, 90) / 90
      } else if (orientation === 180) {
        gravity.x = Common.clamp(event.gamma, -90, 90) / 90
        gravity.y = Common.clamp(-event.beta, -90, 90) / 90
      } else if (orientation === 90) {
        gravity.x = Common.clamp(event.beta, -90, 90) / 90
        gravity.y = Common.clamp(-event.gamma, -90, 90) / 90
      } else if (orientation === -90) {
        gravity.x = Common.clamp(-event.beta, -90, 90) / 90
        gravity.y = Common.clamp(event.gamma, -90, 90) / 90
      }
    }

    Demo.reset = function () {
      let _world = _engine.world

      Common._seed = 2

      World.clear(_world)
      Engine.clear(_engine)

      var offset = 20
      World.addBody(
        _world,
        Bodies.rectangle(_sceneWidth * 0.5, -offset, _sceneWidth + 0.5, 50.5, {
          isStatic: true
        })
      )
      World.addBody(
        _world,
        Bodies.rectangle(
          _sceneWidth * 0.5,
          _sceneHeight + offset,
          _sceneWidth + 0.5,
          50.5,
          { isStatic: true }
        )
      )
      World.addBody(
        _world,
        Bodies.rectangle(
          _sceneWidth + offset,
          _sceneHeight * 0.5,
          50.5,
          _sceneHeight + 0.5,
          { isStatic: true }
        )
      )
      World.addBody(
        _world,
        Bodies.rectangle(
          -offset,
          _sceneHeight * 0.5,
          50.5,
          _sceneHeight + 0.5,
          { isStatic: true }
        )
      )
    }

    Demo.init()
    this.engine = _engine
    console.log(this)
  }
}
</script>

<style scoped>
.container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: -100;
}

img {
  max-width:100%;
  max-height:100%;
  position:absolute;
  top:0; left:0; right:0; bottom:0;
  margin:auto;
}

</style>
