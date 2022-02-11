<template>
  <div
    class="container"
  >
    <van-nav-bar
      title="PDF预览"
      :fixed="true"
      left-arrow
      @click-left="goBack"
    />
    <div class="pdfContent" ref="scale-element" id="gesture-area" offset-top='12vw'>
      <div
        ref="pdfView"
        class="pdfView"
        id="scale-element"
      >
        <pdf
          :src="pdfSrc"
          :page="currentPage"
          @num-pages="pageCount=$event"
          @page-loaded="currentPage=$event"
          @loaded="loadPdfHandler"
        />
      </div>
    </div>
    <!--页码-->
    <div
      v-show="fileType === 'pdf'"
      class="pdf"
    >
      <div class="pagesize">
        <!--上一页-->
        <span
          class="turn"
          :class="{grey: currentPage==1}"
          @click="changePdfPage(0)"
        >上一页</span>
        {{ currentPage }} / {{ pageCount }}
        <!--下一页-->
        <span
          class="turn"
          :class="{grey: currentPage==pageCount}"
          @click="changePdfPage(1)"
        >下一页</span>
      </div>
    </div>
  </div>
</template>
<script>
import pdf from 'vue-pdf' // 引入pdf组件
import interact from 'interactjs'
export default {
  metaInfo: {
    title: 'This is the test',
    meta: [
      { charset: 'utf-8' },
      // { name: 'viewport', content: 'width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=2,user-scalable=yes' }
    ]
  },
  components: { pdf }, // 引入pdf组件
  data () {
    return {
      pdfLoading: null,
      currentPage: 0, // pdf文件页码
      pageCount: 0, // pdf文件总页数
      fileType: 'pdf', // 文件类型
      pdfSrc: 'xxx.pdf'
    }
  },
  updated () {
    this.$nextTick(() => {
      this.pdfLoading.close()
      this.getData()
    })
  },
  created () {
    // this.getFilePathName()
    this.pdfLoading = this.$toast.loading({
      className: 'toast-default-size',
      message: '加载中...',
      forbidClick: true,
      loadingType: 'spinner',
      duration: 0
    })
  },
  methods: {
    // 返回上级
    goBack () {
      this.$router.back()
    },
    // 改变PDF页码,val传过来区分上一页下一页的值,0上一页,1下一页
    changePdfPage (val) {
      // console.log(val)
      if (val === 0 && this.currentPage > 1) {
        this.currentPage--
      }
      if (val === 1 && this.currentPage < this.pageCount) {
        this.currentPage++
      }
    },
    // pdf加载时
    loadPdfHandler (e) {
      this.currentPage = 1 // 加载的时候先加载第一页
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
          modifiers: [
            interact.modifiers.restrictRect({
              restriction: 'parent',
              endOnly: true
            })
          ],
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
<style lang="less" scoped>
  .container{
    position: relative;
    width:100%;
    overflow-x: hidden;
  }
  #gesture-area{
    width:750px;
    margin-top: 12vw;
  }
  /*#scale-element{*/
  /*  width:375px;*/
  /*}*/
  .pagesize{
    position: fixed;
    width:100%;
    bottom: 5%;
    text-align: center;
  }
  .pdf-box {
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    width: 100%;
    margin: 0 auto;
    overflow-y: auto;
    font-size: .28rem;
    span {
      width: 100%;
    }
  }
</style>
