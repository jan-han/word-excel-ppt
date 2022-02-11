<template>
  <div
    class="excel-container"
  >
    <van-nav-bar
      left-text="返回"
      title="excel查看"
      left-arrow
      :fixed="true"
      :placeholder="true"
      @click-left="back"
    />
    <div id="gesture-area">
      <div class="xlsxView" id="scale-element">
        <van-tabs v-model="active" @change="onChange">
          <van-tab :title="item" :name="index" v-for="(item, index) in tabs" :key="index">
          </van-tab>
        </van-tabs>
        <div ref="preview" ></div>
      </div>
    </div>
  </div>
</template>
<style lang="css">
  #scale-element {
    display: block;
  }

  #scale-element.reset {
    transition: transform 0.3s ease-in-out;
  }
  .excel-container {
    width: 100%;
  }
  .xlsxView{
    position: relative;
    width:100%;
    height: 100vw;
  }
  table {
    width:100%;
    table-layout: fixed !important;
    border-collapse:collapse;
    border: 1px solid #000;
    /*border:none;*/
    font-size:0.23rem;
  }

  td,th {
    width:1px;
    white-space:nowrap; /* 自适应宽度*/
    word-break:keep-all; /* 避免长单词截断，保持全部 */
    border:solid #676767 1px;
    text-align:center;
    white-space:pre-line;
    word-break:break-all !important;
    word-wrap:break-word !important;
    display:table-cell;
    vertical-align:middle !important;
    white-space: normal !important;
    height:auto;
    padding:2px 2px 0 2px;
  }
</style>
<style lang="less" scoped>
  /deep/ .van-tab--active{
    background: #7b8aa9;
    color: #fff!important;
  }
  /deep/ .van-tabs__wrap .van-tabs__line{
    background: none;
  }
</style>
<script>
import XLSX from 'xlsx'
import interact from 'interactjs'
export default {
  name: 'xlsxView',
  props: {},
  data () {
    return {
      active: 0,
      htmlString: '',
      url: 'xx.xlsx', // excel文件地址
      tabs: [],
      wb: null
    }
  },
  updated () {
    console.log('*&&&')
  },
  created () {
    this.readWorkbookFromRemoteFile(this.url)
    this.$nextTick(() => {
      this.getData()
    })
  },
  mounted () {
  },
  methods: {
    // 返回键
    back () {
      this.$router.back()
    },
    async readWorkbookFromRemoteFile (url) {
      var xhr = new XMLHttpRequest()
      xhr.open('get', url, true)
      xhr.responseType = 'arraybuffer'
      const _this = this
      xhr.onload = function (e) {
        var binary = ''
        if (xhr.status === 200) {
          var bytes = new Uint8Array(xhr.response)
          var length = bytes.byteLength
          for (var i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i])
          }
          _this.wb = XLSX.read(binary, { type: 'binary' })
          const sheetNames = _this.wb.SheetNames
          _this.tabs = sheetNames
          _this.onTabWorkbook(0)
        }
      }
      xhr.send()
    },
    onTabWorkbook (index) {
      var wsname = this.wb.SheetNames[index]
      console.log(wsname)
      var ws = this.wb.Sheets[wsname]
      var HTML = XLSX.utils.sheet_to_html(ws)
      console.log(HTML)
      if (this.$refs.preview) {
        this.$refs.preview.innerHTML = HTML
      }
    },
    onChange (val) {
      this.onTabWorkbook(val)
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
              //
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
