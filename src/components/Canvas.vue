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
        @wheel="wheel"
        @gesturestart="gesturestart"
        @gesturechange="gesturechange"
        @gestureend="gestureend"
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
        <button v-on:click="zoomIn">ZoomIn</button>
        <button v-on:click="zoomOut">ZoomOut</button>
      </div>
    </div>
  </div>
</template>

<script>
  import Scroller from 'scroller'

  export default {
    name: 'Canvas',
    mounted () {
      this.canvas = this.$refs.canvas
      this.context = this.canvas.getContext('2d')
      this.context.imageSmoothingEnabled = false
      this.canvasData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
      this.scroller = new Scroller.Scroller(
        this.renderCallback,
        {
          scrollingX: true,
          scrollingY: true,
          animating: false,
          locking: false,
          zooming: true,
          maxZoom: 64
        })
      this.canvas.oncontextmenu = function (e) {
        e.preventDefault()
      }
      var rect = this.canvas.getBoundingClientRect()
      this.scroller.setPosition(rect.left + this.canvas.clientLeft, rect.top + this.canvas.clientTop)
      this.scroller.setDimensions(100, 100, this.canvasWidth, this.canvasHeight)
      this.canvasPos = {left: 0, top: 0, zoom: 1}
    },
    data () {
      return {
        scroller: {},
        pointSize: 4,
        strokeStyle: 'darkgrey',
        fillStyle: 'darkgrey',
        points: [],
        canvas: {},
        context: {},
        canvasWidth: 1024,
        canvasHeight: 400,
        dragging: false,
        scrolling: false,
        previousPoint: {},
        msg: '0w0',
        checked: false,
        color: {},
        canvasData: [],
        zoom: 1,
        dragStartPos: [],
        dragCurrentPos: [],
        canvasPos: [],
      }
    },
    methods: {
      zoomIn () {
        this.zoom *= 2
        console.log(`zoomIn!${this.zoom}`)
        this.scroller.zoomBy(2.0, true)
      },
      zoomOut () {
        this.zoom /= 2
        console.log(`zoomOut!${this.zoom}`)
        this.scroller.zoomBy(0.5, true)
      },
      // touchstart (event) {
      //   this.scroller.doTouchStart(event.touches, event.timeStamp)
      //   event.preventDefault()
      // },
      // touchmove (event) {
      //   console.log('touchmove')
      //   this.scroller.doTouchMove(event.touches, event.timeStamp, event.scale)
      // },
      // touchend (event) {
      //   this.scroller.doTouchEnd(event.timeStamp)
      // },
      // touchcancel (event) {
      //   this.scroller.doTouchEnd(event.timeStamp)
      // },
      isRightClicked (evt) {
        return (evt.button === 2)
      },
      getMousePos (evt) {
        // https://stackoverflow.com/questions/17130395/real-mouse-position-in-canvas
        var rect = this.context.canvas.getBoundingClientRect()
        return {
          x: parseInt((evt.clientX - rect.left) / (rect.right - rect.left) * this.canvasWidth),
          y: parseInt((evt.clientY - rect.top) / (rect.bottom - rect.top) * this.canvasHeight)
        }
      },
      gesturestart (event) {
        event.preventDefault()
        console.log('gesturestart')
        console.log(event)
      },
      gesturechange (event) {
        event.preventDefault()
        console.log('gesturechange')
        console.log(event)
      },
      gestureend (event) {
        event.preventDefault()
        console.log('gestureend')
        console.log(event)
      },
      doZooming (zoomLevel) {
          this.zoom = zoomLevel;
          this.canvasPos = {left: Math.floor(this.canvasPos.left), top: Math.floor(this.canvasPos.top), zoom: this.zoom}
          this.render()
      },
      wheel (event) {
        event.preventDefault()
        if (event.ctrlKey === true) {
          var delta = 1.04
          this.doZooming(event.wheelDelta > 0 ? this.zoom * delta : this.zoom / delta);
          // if(event.wheelDelta > 0) {
          //   this.zoom *= delta
          //   console.log(`zoomIn!${this.zoom}`)
          //   this.scroller.zoomBy(delta, false)
          // } else {
          //   this.zoom /= delta
          //   console.log(`zoomIn!${this.zoom}`)
          //   this.scroller.zoomBy(1/delta, false)
          // }

        } else if (this.scrolling === false) {
          let point = this.getMousePos(event)
          console.log('start scrolling')
          this.dragStartPos = point
          this.dragCurrentPos = this.dragStartPos
          this.scroller.doTouchStart([{
            pageX: this.dragCurrentPos.x,
            pageY: this.dragCurrentPos.y
          }], event.timeStamp)
          this.scrolling = true
        } else {
          var deltaX = event.deltaX / this.zoom
          var deltaY = event.deltaY / this.zoom
          this.dragCurrentPos = {x: this.dragCurrentPos.x - deltaX * 2, y: this.dragCurrentPos.y - deltaY * 2}
          console.log(`scrolling ${this.dragCurrentPos.x}, ${this.dragCurrentPos.y}`)
          this.scroller.doTouchMove([{
            pageX: this.dragCurrentPos.x,
            pageY: this.dragCurrentPos.y
          }], event.timeStamp)
        }
      },
      mousemove (event) {
        let point = this.getMousePos(event)
        if (this.checked) return
        if (this.dragging) {
          this.execToDrawLine(this.previousPoint, point)
          this.previousPoint = point
        }
      },
      mousedown (event) {
        if (this.isRightClicked(event)) {
        } else {
          this.previousPoint = this.getMousePos(event)
          this.dragging = true
        }
      },
      mouseup (event) {
        let mouseUpPoint = this.getMousePos(event)
        this.dragging = false
        if (this.checked) {
          this.execToFill(mouseUpPoint)
        } else {
          this.execToDrawPoint(mouseUpPoint)
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
      execToDrawPoint (point) {
        this.drawByBrush(point)
        this.render()
      },
      execToDrawLine (p1, p2) {
        var from = p1.x < p2.x ? p1 : p2
        var to = p1.x < p2.x ? p2 : p1

        var dx = to.x - from.x + 1
        var dy = to.y - from.y + 1
        var ratio = dy / dx

        var cur = from
        console.log(`drawLine ${p1.x},${p1.y} -> ${p2.x},${p2.y} dx:${dx} dy:${dy} ratio: ${ratio}, color: ${this.color}`)
        while (cur.x <= to.x) {
          console.log(`${cur.x}, ${cur.y} -> ${to.x}, ${to.y}`)
          this.drawByBrush(cur)
          var curY = cur.y
          var nextY = from.y + ratio * (cur.x - from.x + 1)
          if (ratio > 0) {
            while (curY < nextY) {
              this.drawByBrush({x: cur.x, y: curY})
              curY += 1
            }
          } else {
            while (curY > nextY) {
              this.drawByBrush({x: cur.x, y: curY})
              curY -= 1
            }
          }
          cur = {x: cur.x + 1, y: curY}
        }
        this.render()
      },
      drawByBrush (point) {
        var x = Math.floor(point.x / this.canvasPos.zoom)
        var y = Math.floor(point.y / this.canvasPos.zoom)
        var startPosOffset = Math.floor(this.pointSize / 2)
        var pointSize = Math.floor(this.pointSize)
        var rgb = colorCodeToRGB(this.color)
        for (var i = 0; i < pointSize; i++) {
          for (var j = 0; j < pointSize; j++) {
//              console.log(`drawByBrush: ${x - startPosOffset + i}, ${y - startPosOffset + j}`)
            this.drawPointToData({x: x - startPosOffset + i, y: y - startPosOffset + j}, rgb)
          }
        }
      },
      drawPointToData (point, color) {
        var x = point.x + this.canvasPos.left
        var y = point.y + this.canvasPos.top
        var index = (x + y * this.canvasWidth) * 4

        if (x > this.canvasWidth || y > this.canvasHeight || this.canvasData.data.length < index) {
            console.log('invalid point');
            return
        }
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
      execToFill (rawPoint) {
        // var imgData = this.context.getImageData(0, 0, this.canvasWidth, this.canvasHeight)
        var point = getPointInCanvas(rawPoint, this.canvasPos)
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
        this.render()
      },
      renderCallback (left, top, _) {
        this.canvasPos = {left: Math.floor(left), top: Math.floor(top), zoom: this.zoom}
        this.render()
      },

      render () {
        var left = this.canvasPos.left
        var top = this.canvasPos.top
        var zoom = this.canvasPos.zoom
        console.log(`render: ${left}, ${top}, ${zoom}, ${left % 1}, ${top % 1}`)
        var newCanvas = document.createElement('canvas')
        newCanvas.width = this.canvasWidth
        newCanvas.height = this.canvasHeight
        newCanvas.getContext('2d').putImageData(this.canvasData, 0, 0)

        this.context.save()
        this.context.clearRect(0, 0, this.canvasWidth, this.canvasHeight)
        this.context.scale(zoom, zoom)
        this.context.drawImage(newCanvas, -left, -top)
        this.context.restore()
        if(zoom >= 8) {
          for(var x = left % 1; x < this.canvasWidth; x += zoom) {
            this.context.beginPath();
            this.context.moveTo(x, 0)
            this.context.lineTo(x, this.canvasHeight)
            this.context.stroke()
          }
          for(var y = top % 1; y < this.canvasHeight; y += zoom) {
            this.context.beginPath();
            this.context.moveTo(0, y)
            this.context.lineTo(this.canvasWidth, y)
            this.context.stroke()
          }
        }
      },
    }
  }

  function getPointInCanvas(point, canvasPos) {
    return {
      x: Math.floor(point.x / canvasPos.zoom) + canvasPos.left,
      y: Math.floor(point.y / canvasPos.zoom) + canvasPos.top
    }
  }

  function colorCodeToRGB(code) {
    var red = parseInt(code.substring(1, 3), 16)
    var green = parseInt(code.substring(3, 5), 16)
    var blue = parseInt(code.substring(5, 7), 16)
    return {R: red, G: green, B: blue}
  }
  function colorPixel(pixelPos, imgData, baseColor) {
    imgData.data[pixelPos] = baseColor.R
    imgData.data[pixelPos + 1] = baseColor.G
    imgData.data[pixelPos + 2] = baseColor.B
    imgData.data[pixelPos + 3] = 255
  }
  function matchStartColor(pixelPos, imgData, baseColor) {
    var r = imgData.data[pixelPos]
    var g = imgData.data[pixelPos + 1]
    var b = imgData.data[pixelPos + 2]
    // console.log(`${r} vs ${baseColor.R}, ${g} vs ${baseColor.G}, ${b} vs ${baseColor.B}`)
    return (r === baseColor.R && g === baseColor.G && b === baseColor.B)
  }

  function decompositColor(imgData, pixelPos) {
    var r = imgData.data[pixelPos]
    var g = imgData.data[pixelPos + 1]
    var b = imgData.data[pixelPos + 2]
    return {R: r, G: g, B: b}
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  html, body {
    max-width: 100%;
    overflow-x: hidden;
  }

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
    border: 1px solid #bbb;
  }

  .flex {
    display: flex;
    flex-flow: row wrap;
  }

  .flex > * {
    border: 1px solid #eee;
    margin: 10px;
    padding: 10px;
  }

</style>
