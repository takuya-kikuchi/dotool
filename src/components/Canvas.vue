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
        <input type="checkbox" id="checkbox" v-model="checked"><label for="checkbox">bucket</label>
        <div>
          <input type="radio" id="black" value="#110000" v-model="color" checked/>
          <label for="black">Black</label>
          <input type="radio" id="red" value="#ff0000" v-model="color"/>
          <label for="red">Red</label>
        </div>
      </div>
    </div>
</template>

<script>
export default {
  name: 'Canvas',
  mounted () {
    this.canvas = this.$refs.canvas
    this.context = this.canvas.getContext('2d')
    this.canvasData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
    // this.scroller = new Scroller(this.drawAllNodes, { zooming: true })
  },
  data () {
    return {
      pointSize: 10,
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
      checked: false,
      color: {},
      canvasData: []
    }
  },
  methods: {
    zoomIn () {
      console.log('zoomIn!')
      var imgData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
      this.context.save()
      this.context.scale(2, 2)
      this.context.putImageData(imgData, this.canvasWidth, this.canvasHeight)
      this.context.restore()
    },
    zoomOut () {

    },
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
      // console.log(`mouseMove! ${point.x}, ${point.y}`)
      if (this.checked) return
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
    drawLine (p1, p2) {
      var from = p1.x < p2.x ? p1 : p2
      var to = p1.x < p2.x ? p2 : p1

      var dx = to.x - from.x + 1
      var dy = to.y - from.y
      var ratio = dy / dx

      var cur = from
      console.log(`drawLine ${p1.x},${p1.y} -> ${p2.x},${p2.y} dx:${dx} dy:${dy} ratio: ${ratio}`)
      while (cur.x <= to.x) {
        console.log(`${cur.x}, ${cur.y} -> ${to.x}, ${to.y}`)
        this.drawNode(cur)
        var curY = cur.y
        var nextY = from.y + ratio * (cur.x - from.x + 1)
        if (ratio > 0) {
          while (curY < nextY) {
            this.drawNode({x: cur.x, y: curY})
            curY += 1
          }
        } else {
          while (curY > nextY) {
            this.drawNode({x: cur.x, y: curY})
            curY -= 1
          }
        }
        cur = {x: cur.x + 1, y: curY}
      }
    },
    drawNode (point) {
      var startPosOffset = Math.floor(this.pointSize / 2)

      var rgb = colorCodeToRGB(this.color)
      for (var i = 0; i < this.pointSize / 2; i++) {
        for (var j = 0; j < this.pointSize / 2; j++) {
          this.drawPointInternal({ x: point.x - startPosOffset + i, y: point.y - startPosOffset + j }, rgb)
        }
      }
      this.context.putImageData(this.canvasData, 0, 0, point.x - startPosOffset, point.y - startPosOffset, this.pointSize, this.pointSize)
    },
    drawPointInternal (point, color) {
      var index = (point.x + point.y * this.canvasWidth) * 4

      this.canvasData.data[index + 0] = color.R
      this.canvasData.data[index + 1] = color.G
      this.canvasData.data[index + 2] = color.B
      this.canvasData.data[index + 3] = 0xFF
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
      // var imgData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
      var imgData = this.canvasData
      var pixelPos = (point.y * this.canvasWidth + point.x) * 4
      var baseColor = decompositColor(imgData, pixelPos)
      console.log(`${baseColor.R} ${baseColor.G} ${baseColor.B}, ${this.fillStyle}`)
      var stack = [point]
      while (stack.length > 0) {
        var current = stack.pop()
        this.fillAsFloodInternal(current, imgData, baseColor, colorCodeToRGB(this.color), stack)
      }
      this.context.putImageData(imgData, 0, 0)
    }
  }
}
function colorCodeToRGB (code) {
  var red = parseInt(code.substring(1, 3), 16)
  var green = parseInt(code.substring(3, 5), 16)
  var blue = parseInt(code.substring(5, 7), 16)
  return {R: red, G: green, B: blue}
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
