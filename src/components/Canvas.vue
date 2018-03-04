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
      msg: '0w0'
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
    mouseup (event) {
      let mouseUpPoint = this.getMousePos(event)
      // if (this.dragMode && this.overPoint) {
      //   this.dragPoint(this.overPoint, mouseUpPoint)
      //   this.dragMode = false
      // } else {
      //   this.addNode(mouseUpPoint)
      // }
      this.addNode(mouseUpPoint)
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
    mousemove (event) {
      // let point = this.getMousePos(event)
      // this.selectedPoint(point)
      // if (this.dragMode && this.overPoint) {
      //   this.dragPoint(this.overPoint, point)
      // }
    },
    mousedown (event) {
      // let point = this.getMousePos(event)
      // if (this.isMouseOnNode(point)) {
      //   // point selected
      //   this.dragMode = true
      // } else {
      //   this.dragMode = false
      // }
    },
    addNode (point) {
      this.context.save()
      this.drawNode(point)
      this.context.restore()
      this.points.push(point)
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
    }
  }
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
