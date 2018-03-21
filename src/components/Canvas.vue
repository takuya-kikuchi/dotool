<template>
    <div class="flex">
        <div>
        <div class="hello">
            <h1>{{ msg }}</h1>
        </div>
        <canvas
            ref="canvas"
            :width="canvasWidth"
            :height="canvasHeight"
            @mousedown="mousedown"
            @mouseup="mouseup"
            @mousemove="mousemove"
        ></canvas>
        <input type="checkbox" id="checkbox" v-model="checked">
        <label for="checkbox">{{ checked }}</label>
        </div>
    </div>
</template>

<script>
export default {
  name: 'Canvas',
  mounted () {
    this.canvas = this.$refs.canvas
    this.context = this.canvas.getContext('2d')
    // this.scroller = new Scroller(this.drawAllNodes, { zooming: true })
  },
  data () {
    return {
      pointSize: 2,
      strokeStyle: 'darkgrey',
      fillStyle: 'darkgrey',
      points: [],
      canvas: {},
      context: {},
      canvasWidth: 320,
      canvasHeight: 240,
      dragging: false,
      previousPoint: {},
      msg: '0w0',
      checked: false
    }
  },
  methods: {
    getMousePos (evt) {
      // https://stackoverflow.com/questions/17130395/real-mouse-position-in-canvas
      var rect = this.context.canvas.getBoundingClientRect()
      return {
        x: parseInt((evt.clientX - rect.left) / (rect.right - rect.left) * this.canvasWidth),
        y: parseInt((evt.clientY - rect.top) / (rect.bottom - rect.top) * this.canvasHeight)
      }
    },
    mousemove (event) {
      let point = this.getMousePos(event)
      console.log(`mouseMove! ${point.x}, ${point.y}`)
      if (this.dragging) {
        this.drawLine(this.previousPoint, point)
        this.previousPoint = point
      }
    },
    mousedown (event) {
      this.previousPoint = this.getMousePos(event)
      this.dragging = true
      // let point = this.getMousePos(event)
      // if (this.isMouseOnNode(point)) {
      //   // point selected
      //   this.dragMode = true
      // } else {
      //   this.dragMode = false
      // }
    },
    mouseup (event) {
      let mouseUpPoint = this.getMousePos(event)
      this.dragging = false
      // if (this.dragMode && this.overPoint) {
      //   this.dragPoint(this.overPoint, mouseUpPoint)
      //   this.dragMode = false
      // } else {
      //   this.addNode(mouseUpPoint)
      // }
      if (this.checked) {
        this.fillAsFlood(mouseUpPoint)
      } else {
        this.addNode(mouseUpPoint)
      }
      console.log(`mouseUp! ${mouseUpPoint.x}, ${mouseUpPoint.y}`)
    },
    dragPoint (selectedPoint, newPoint) {
      this.points.map((pointToMove) => {
        if (pointToMove.x === selectedPoint.x && pointToMove.y === selectedPoint.y) {
          pointToMove.x = newPoint.x
          pointToMove.y = newPoint.y

          this.clear()
          this.drawAllNodes()
        }
      })
    },
    addNode (point) {
      this.context.save()
      this.drawNode(point)
      this.context.restore()
      this.points.push(point)
    },
    drawLine (from, to) {
      this.context.beginPath()
      this.context.moveTo(from.x, from.y)
      this.context.lineTo(to.x, to.y)
      this.context.strokeStyle = this.strokeStyle
      this.context.lineWidth = this.pointSize
      this.context.stroke()
    },
    drawNode (point) {
      this.context.beginPath()
      var startPosOffset = Math.floor(this.pointSize / 2)
      console.log(`${point.x - startPosOffset},${point.y - startPosOffset} - ${point.x + this.pointSize},${point.y + this.pointSize}`)
      this.context.moveTo(point.x - startPosOffset, point.y - startPosOffset)
      this.context.lineTo(point.x - startPosOffset, point.y + startPosOffset)
      this.context.lineTo(point.x + startPosOffset, point.y + startPosOffset)
      this.context.lineTo(point.x + startPosOffset, point.y - startPosOffset)
      this.context.lineTo(point.x - startPosOffset, point.y - startPosOffset)
      this.context.fillStyle = this.fillStyle
      this.context.fill()
      this.context.strokeStyle = this.strokeStyle
      this.context.stroke()
    },
    fillAsFloodInternal (point, imgData, baseColor, fillColor, stack) {
      var y = point.y
      var x = point.x
      var pixelPos = (y * this.canvasWidth + point.x) * 4
      var canvasWidth = this.canvasWidth
      var canvasHeight = this.canvasHeight
      while (y-- >= 0 && matchStartColor(pixelPos, imgData, baseColor)) {
        pixelPos -= canvasWidth * 4
      }
      pixelPos += canvasWidth * 4
      y++
      var reachLeft = false
      var reachRight = false
      while (y++ < canvasHeight - 1 && matchStartColor(pixelPos, imgData, baseColor)) {
        colorPixel(pixelPos, imgData, fillColor)

        if (x > 0) {
          if (matchStartColor(pixelPos - 4, imgData, baseColor)) {
            if (!reachLeft) {
              stack.push({x: x - 1, y: y})
              reachLeft = true
            }
          } else {
            reachLeft = false
          }
        }
        if (x < canvasWidth - 1) {
          if (matchStartColor(pixelPos + 4, imgData, baseColor)) {
            if (!reachRight) {
              stack.push({x: x + 1, y: y})
              reachRight = true
            }
          } else {
            reachRight = false
          }
        }
        pixelPos += canvasWidth * 4
      }
    },
    fillAsFlood (point) {
      var imgData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
      var pixelPos = (point.y * this.canvasWidth + point.x) * 4
      var baseColor = decompositColor(imgData, pixelPos)
      console.log(`${baseColor.R} ${baseColor.G} ${baseColor.B}, ${this.fillStyle}`)
      var stack = [point]
      while (stack.length > 0) {
        var current = stack.pop()
        this.fillAsFloodInternal(current, imgData, baseColor, {R: 255, G: 123, B: 0}, stack)
      }
      this.context.putImageData(imgData, 0, 0)
    }
  }
}

function colorPixel (pixelPos, imgData, baseColor) {
  imgData.data[pixelPos] = baseColor.R
  imgData.data[pixelPos + 1] = baseColor.G
  imgData.data[pixelPos + 2] = baseColor.B
  imgData.data[pixelPos + 3] = 255
}
function matchStartColor (pixelPos, imgData, baseColor) {
  var r = imgData.data[pixelPos]
  var g = imgData.data[pixelPos + 1]
  var b = imgData.data[pixelPos + 2]
  // console.log(`${r} vs ${baseColor.R}, ${g} vs ${baseColor.G}, ${b} vs ${baseColor.B}`)
  return (r === baseColor.R && g === baseColor.G && b === baseColor.B)
}

function decompositColor (imgData, pixelPos) {
  var r = imgData.data[pixelPos]
  var g = imgData.data[pixelPos + 1]
  var b = imgData.data[pixelPos + 2]
  return { R: r, G: g, B: b }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

  canvas {
    border:1px solid #bbb;
  }
  .flex{
    display: flex;
    flex-flow: row wrap;
  }
  .flex > * {
    border: 1px solid #eee;
    margin: 10px;
    padding: 10px;
  }

</style>
