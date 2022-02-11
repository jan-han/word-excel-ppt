<template>
  <div class="word-container">
    <van-nav-bar
      title="word查看"
      left-arrow
      :fixed="true"
      :placeholder="true"
      @click-left="back"
    />
    <div id="gesture-area" ref="gesture-area">
      <div ref="docPreview" class="docView" id="scale-element"/>
    </div>
  </div>
</template>
<script>
import mammoth from 'mammoth'
import interact from 'interactjs'
export default {
  metaInfo: {
    title: 'This is the test',
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=2,user-scalable=yes' }
    ]
  },
  name: 'WordView',
  props: {
  },
  data () {
    return {
      url: 'xx.docx'
    }
  },
  mounted () {
    this.previewFile()
    this.$nextTick(() => {
      this.getData()
    })
  },
  updated () {
    console.log('**1')
  },
  methods: {
    back () {
      this.$router.back()
    },
    previewFile () {
      const _this = this
      try {
        var xhr = new XMLHttpRequest()
        xhr.open('GET', this.url)
        xhr.responseType = 'arraybuffer'
        xhr.onload = function (e) {
          var arrayBuffer = xhr.response // arrayBuffer
          mammoth
            .convertToHtml({ arrayBuffer: arrayBuffer })
            .then(displayResult)
            .done()

          function displayResult (result) {
            _this.$refs.docPreview.innerHTML = result.value || ''
          }
        }
        xhr.send()
      } catch (e) {
        console.log(e)
        _this.$emit('errorPreview')
      }
    },
    getData () {
      const self = this
      var angleScale = {
        angle: 0,
        scale: 1
      }
      var gestureArea = document.getElementById('gesture-area')
      var scaleElement = document.getElementById('scale-element')
      var resetTimeout

      interact(gestureArea)
        .gesturable({
          listeners: {
            start (event) {
              angleScale.angle -= event.angle

              clearTimeout(resetTimeout)
              scaleElement.classList.remove('reset')
            },
            move (event) {
              // document.body.appendChild(new Text(event.scale))
              // var currentAngle = event.angle + angleScale.angle
              var currentScale = event.scale * angleScale.scale

              scaleElement.style.transform = 'scale(' + currentScale + ')'

              // uses the dragMoveListener from the draggable demo above
              self.dragMoveListener(event)
            },
            end (event) {
              angleScale.angle = angleScale.angle + event.angle
              angleScale.scale = angleScale.scale * event.scale

              resetTimeout = setTimeout(reset, 10000)
              scaleElement.classList.add('reset')
            }
          },
          modifiers: [
            // minimum size
            interact.modifiers.restrictSize({
              min: { width: 200, height: 300 }
            })
          ]
        })
        .draggable({
          listeners: { move: self.dragMoveListener }
        })

      function reset () {
        scaleElement.style.transform = 'scale(1)'

        angleScale.angle = 0
        angleScale.scale = 1
      }
    },
    dragMoveListener (event) {
      var target = event.target
      // keep the dragged position in the data-x/data-y attributes
      var x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx
      var y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy

      // translate the element
      target.style.transform = 'translate(' + x + 'px, ' + y + 'px)'

      // update the posiion attributes
      target.setAttribute('data-x', x)
      target.setAttribute('data-y', y)
    }
  }
}
</script>
<style lang="less">
  .word-container{
    position: relative;
    width:100%;
  }
  .docView{
    margin-top: 20px;
    width:90%;
    padding: 20px;
    img{
      width:100%;
    }
    div, p{

    }
  }
</style>
