<template>
  <div class="stars-background">
    <canvas></canvas>
  </div>
</template>

<script>
  function createColorStyle (r, g, b) {
    return `rgba(${r}, ${g}, ${b}, 0.9)`
  }

  class Color {
    constructor (min) {
      min = min || 0
      this.r = this.colorValue(min)
      this.g = this.colorValue(min)
      this.b = this.colorValue(min)
      this.style = createColorStyle(this.r, this.g, this.b)
    }

    colorValue (min) {
      return Math.floor(Math.random() * 255 + min)
    }
  }

  class Dot {
    constructor (width, height) {
      this.x = Math.random() * width
      this.y = Math.random() * height
//      this.z = Math.random()

      this.vx = Math.random() - 0.5
      this.vy = Math.random() - 0.5
//      this.vz = Math.random() - 0.5

//      this.maxRadius = 3
//      this.radius = this.z * this.maxRadius
      this.radius = 2 * Math.random()

      this.color = new Color()
    }

    draw (ctx) {
      ctx.beginPath()
      ctx.fillStyle = this.color.style
      ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2, false)
      ctx.fill()
    }
  }

  export default {
    name: 'stars-background',
    created () {
      window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame ||
        window.webkitRequestAnimationFrame || window.msRequestAnimationFrame || function (callback) {
          window.setTimeout(callback, 1000 / 60)
        }
    },
    mounted () {
      this.$nextTick(this.init)
    },
    data () {
      return {
        canvas: {
          el: null,
          ctx: null,
          width: 0,
          height: 0
        },
        dots: {
          array: [],
          count: 0,
          radius: 150,
          distance: 100
        },
        mousePosition: {
          x: 0,
          y: 0
        }
      }
    },
    computed: {
    },
    methods: {
      init () {
        let wrap = this.$el

        let canvas = wrap.querySelector('canvas')
        let width = wrap.clientWidth
        let height = wrap.clientHeight
        let ctx = canvas.getContext('2d')

        // init canvas
        this.canvas.el = canvas
        this.canvas.width = width
        this.canvas.height = height
        this.canvas.ctx = ctx

        canvas.width = width
        canvas.height = height
        ctx.lineWidth = 0.3
        ctx.strokeStyle = (new Color(150)).style

        // init dots
        this.dots.count = Math.floor(width * height / 10000)
        this.createDots()

        this.mousePosition.x = width / 2
        this.mousePosition.y = height / 2

        this.addEvent()

        window.requestAnimationFrame(this.animateDots)
      },
      throttle (method, scope, time = 100) {
        clearTimeout(method.tId)
        method.tId = setTimeout(function () {
          method.call(scope)
        }, time)
      },
      resizeDot () {
        this.init()
        let width = this.width || this.canvas.el.offsetWidth
        let height = this.height || this.canvas.el.offsetHeight

        this.canvas.width = width
        this.canvas.height = height

        this.dots.count = width * height / 10000
        this.dots.array = []
        this.createDots()
      },
      mixComponents (comp1, weight1, comp2, weight2) {
        return (comp1 * weight1 + comp2 * weight2) / (weight1 + weight2)
      },
      averageColorStyles (dot1, dot2) {
        let color1 = dot1.color
        let color2 = dot2.color

        let r = this.mixComponents(color1.r, dot1.radius, color2.r, dot2.radius)
        let g = this.mixComponents(color1.g, dot1.radius, color2.g, dot2.radius)
        let b = this.mixComponents(color1.b, dot1.radius, color2.b, dot2.radius)
        return createColorStyle(Math.floor(r), Math.floor(g), Math.floor(b))
      },
      createDots () {
        for (let i = 0; i < this.dots.count; i++) {
          this.dots.array.push(new Dot(this.canvas.width, this.canvas.height))
        }
      },
      moveDots () {
        for (let i = 0; i < this.dots.count; i++) {
          let dot = this.dots.array[i]

          if (dot.y < 0 || dot.y > this.canvas.height) {
            dot.vy = -dot.vy
          } else if (dot.x < 0 || dot.x > this.canvas.width) {
            dot.vx = -dot.vx
          } else if (dot.z <= 0 || dot.z > 1) {
            dot.vz = -dot.vz
          }
          dot.x += dot.vx
          dot.y += dot.vy
//          dot.radius += dot.z * dot.maxRadius
        }
      },
      connectDots () {
        let iDot = 0
        let jDot = 0
        let ctx = this.canvas.ctx

        for (let i = 0; i < this.dots.count; i++) {
          for (let j = 0; j < this.dots.count; j++) {
            iDot = this.dots.array[i]
            jDot = this.dots.array[j]

            if ((iDot.x - jDot.x) < this.dots.distance &&
              (iDot.y - jDot.y) < this.dots.distance &&
              (iDot.x - jDot.x) > -this.dots.distance &&
              (iDot.y - jDot.y) > -this.dots.distance) {
              if ((iDot.x - this.mousePosition.x) < this.dots.radius &&
                (iDot.y - this.mousePosition.y) < this.dots.radius &&
                (iDot.x - this.mousePosition.x) > -this.dots.radius &&
                (iDot.y - this.mousePosition.y) > -this.dots.radius) {
                ctx.beginPath()
                ctx.strokeStyle = this.averageColorStyles(iDot, jDot)
                ctx.moveTo(iDot.x, iDot.y)
                ctx.lineTo(jDot.x, jDot.y)
                ctx.stroke()
                ctx.closePath()
              }
            }
          }
        }
      },
      drawDots () {
        for (let i = 0; i < this.dots.count; i++) {
          this.dots.array[i].draw(this.canvas.ctx)
        }
      },
      animateDots () {
        this.canvas.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
        this.moveDots()
        this.connectDots()
        this.drawDots()

        window.requestAnimationFrame(this.animateDots)
      },
      addEvent () {
        let self = this

        window.addEventListener('resize', () => {
          self.throttle(self.resizeDot)
        })

        this.canvas.el.addEventListener('mousemove', function (e) {
          self.mousePosition.x = e.pageX
          self.mousePosition.y = e.pageY
        })

        this.canvas.el.addEventListener('mouseleave', function (e) {
          self.mousePosition.x = self.canvas.width / 2
          self.mousePosition.y = self.canvas.height / 2
        })
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .stars-background {
    background: linear-gradient(#1b1f23,#000000);
    width: 100%;
    height: 100%;
  }
</style>
