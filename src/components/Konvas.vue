<template>
  <div>
    <div style="margin-top: 10px">
      <el-button
        type="primary"
        size="mini"
        @click="addShape"
      >Thêm shape</el-button>
      <el-button
        type="primary"
        size="mini"
        @click="addLine"
      >Thêm line</el-button>
      <el-button
        type="primary"
        size="mini"
        @click="addArrow"
      >Thêm mũi tên</el-button>
      <el-button
        type="primary"
        size="mini"
        @click="addNote"
      >Thêm note</el-button>
    </div>
    <div class="page-center">
      <v-stage
        ref="stage"
        :config="stageSize"
        @mouseup="handleMouseup"
        @mousemove="handleMousemove"
        @mousedown="handleMousedown"
        @touchend="handleMouseup"
        @touchmove="handleMousemove"
        @touchstart="handleMousedown"
        @dblclick="addNote"
        @click="touchEndLine"
      >
        <!-- image -->
        <v-layer ref="layer">
          <!-- <v-image
            ref="image"
            :config="{
            image: image
          }"
          /> -->
          <!-- <v-image
          ref="image"
          :config="{
          sceneFunc: function(context, shape) {
            context.beginPath();
            context.strokeRect(1050, 150, 450, 450);
            context.closePath();
            context.fillStrokeShape(shape);
          },
          fill: '#00D2FF',
          stroke: 'black',
          strokeWidth: 1
          }"
        >
        </v-image> -->
        </v-layer>
        <!-- layer 2 -->
        <v-layer ref="layer2">
          <!-- note -->
          <v-text
            v-for="(item) in listNote"
            :key="item.config.name"
            :ref="item.config.name"
            :config="item.config"
            @click="editText"
          />
          <!-- line -->
          <v-line
            v-for="(item) in listLine"
            :key="item.config.name"
            :ref="item.config.name"
            :config="item.config"
            @click="defineTarget"
          />
          <!-- shape -->
          <v-rect
            v-for="item in listShape"
            :key="item.config.name"
            :ref="item.config.name"
            :config="item.config"
            @click="defineTarget"
          />

          <v-arrow
            v-for="item in listArrow"
            :key="item.config.name"
            :ref="item.config.name"
            :config="item.config"
            @click="defineTarget"
          />

          <v-transformer
            ref="transformerText"
            :config="{
            anchorStroke: 'red',
            anchorFill: 'red',
            anchorSize: 5,
            borderStroke: 'red',
            borderDash: [5, 5],
            padding: 5
          }"
            @transformstart="handleTransformStart"
            @transformend="handleTransformEnd"
          />
        </v-layer>

        <v-layer ref="layerDraw"></v-layer>
      </v-stage>
    </div>
  </div>
</template>

<script>

import Konva from 'konva'

const m_width = 3000;
const m_number = 200;
const m_height = 3000;

const fontSize = 14;
const fontFamily = "Saira Semi Condensed";

const width = window.innerWidth
const height = window.innerHeight
let vm = {}
let stage = null
let layer = null
let layer2 = null
let context = null
let canvas = null
let textNode = null
let tr = null
export default {
  data () {
    return {
      stageSize: {
        width: width,
        height: m_height
      },

      numberRefs: 0,

      listShape: [],

      listLine: [],

      listNote: [],

      listArrow: [],

      image: null,

      isPaint: false,

      lastPointerPosition: {
        x: 0,
        y: 0
      },

      pointerPosition: {
        x: 0,
        y: 0
      },

      isAddLine: false,
      numberToCheckAddLine: 0,

      selectedShapeName: ''
    }
  },
  methods: {

    addTransformerImmediate (name_refs = '') {
      textNode = vm.$refs[name_refs][0].getNode();
      this.selectedShapeName = name_refs;
      this.updateTransformer();
    },

    addNote () {
      this.numberRefs += 1
      let new_note = {
        config: {
          width: 150,
          text: 'ghi chú mới',
          fontSize: fontSize,
          fontFamily: fontFamily,
          draggable: true,
          name: `text${this.numberRefs}`,
          x: this.lastPointerPosition.x || 100,
          y: this.lastPointerPosition.y || 0,
          ellipsis: true,
          fill: 'red'
        }
      };
      let numberNote = this.listNote.length;
      this.$set(this.listNote, numberNote, new_note)

      this.$nextTick(() => {
        this.addTransformerImmediate(this.listNote[numberNote].config.name)
      })
    },

    addLine () {
      this.isAddLine = true;
    },

    touchEndLine () {
      if (this.isAddLine) {
        this.numberToCheckAddLine += 1;
        this.pointerPosition = stage.getPointerPosition();
        if (this.numberToCheckAddLine % 2 !== 0) {
          this.numberRefs += 1;
          let new_line = {
            config: {
              x: this.pointerPosition.x || 20,
              y: this.pointerPosition.y || 200,
              points: [0, 0, 0, 0, 5, 0],
              closed: true,
              draggable: true,
              padding: 10,
              name: `line${this.numberRefs}`,
              stroke: 'red',
              strokeWidth: 3
            }
          };
          let numberLine = this.listLine.length;
          this.$set(this.listLine, numberLine, new_line)
          this.$nextTick(() => {
            this.addTransformerImmediate(this.listLine[numberLine].config.name)
          })
        } else {
          this.listLine.forEach(item => {
            if (item.config.name === this.selectedShapeName) {
              let oldValue = item.config.x
              this.$set(item.config.points, 4, this.pointerPosition.x - oldValue)
              this.addTransformerImmediate(item.config.name)
            }
          });
          this.$forceUpdate();
          this.selectedShapeName = ''
          this.updateTransformer();
          this.isAddLine = false;
        }
      }
    },

    addShape () {
      this.numberRefs += 1;
      let new_shape = {
        config: {
          x: 150,
          y: 150,
          width: 100,
          height: 25,
          stroke: 'red',
          strokeWidth: 0.5,
          name: `shape${this.numberRefs}`,
          draggable: true
        }
      };
      let numberShape = this.listShape.length;
      this.$set(this.listShape, numberShape, new_shape)
      this.$nextTick(() => {
        this.addTransformerImmediate(this.listShape[numberShape].config.name)
      })
    },

    addArrow () {
      this.numberRefs += 1;
      let new_arrow = {
        config: {
          x: 100,
          y: 100,
          points: [120, 0, 250, 0],
          pointerLength: 20,
          pointerWidth: 15,
          name: `arrow${this.numberRefs}`,
          stroke: 'red',
          fill: 'red',
          draggable: true,
          strokeWidth: 1
        }
      }
      let numberArrow = this.listArrow.length;
      this.$set(this.listArrow, numberArrow, new_arrow)
      this.$nextTick(() => {
        this.addTransformerImmediate(this.listArrow[numberArrow].config.name)
      })
    },

    allowPaint (e) {
      // draw line
      this.isPaint = true
      this.lastPointerPosition = stage.getPointerPosition()
    },

    handleMousedown (e) {
      if (e.target === e.target.getStage()) {
        // draw line
        this.allowPaint()
        // this.selectedShapeName = ''
        this.updateTransformer()
        return
      }

      // clicked on transformer - do nothing
      const clickedOnTransformer =
        e.target.getParent().className === 'Transformer'
      if (clickedOnTransformer) {
        // draw line
        this.allowPaint()
        return
      }

      // find clicked rect by its name
      const name = e.target.name()
      const rect = textNode
      if (rect) {
        this.selectedShapeName = name
      } else {
        this.selectedShapeName = ''
      }
      this.updateTransformer()
    },

    handleMouseup (e) {
      this.isPaint = false
    },

    handleMousemove (e) {
      if (!this.isPaint) {
        return
      }
      context.globalCompositeOperation = 'source-over'
      // context.globalCompositeOperation = 'destination-out';
      context.beginPath()
      context.strokeStyle = '#df4b26'
      // context.lineJoin = 'round'
      context.lineWidth = 1
      var localPos = {
        x: this.lastPointerPosition.x,
        y: this.lastPointerPosition.y
      }
      context.moveTo(localPos.x, localPos.y)
      var pos = stage.getPointerPosition()
      localPos = {
        x: pos.x,
        y: pos.y
      }
      context.lineTo(localPos.x, localPos.y)
      context.stroke()
      context.closePath()

      this.lastPointerPosition = pos
      // stage.batchDraw()
    },

    updateTransformer () {
      // here we need to manually attach or detach Transformer node
      const transformerNode = this.$refs.transformerText.getStage()
      const stage = transformerNode.getStage()
      const { selectedShapeName } = this
      console.log(selectedShapeName)

      const selectedNode = stage.findOne('.' + selectedShapeName)
      // do nothing if selected node is already attached
      if (selectedNode === transformerNode.node()) {
        return
      }

      if (selectedNode) {
        // attach to another node
        transformerNode.attachTo(selectedNode)
      } else {
        // remove transformer
        transformerNode.detach()
      }
      transformerNode.getLayer().batchDraw()
    },

    defineTarget (e) {
      vm = this
      textNode = vm.$refs[e.target.attrs.name][0].getNode();
    },

    editText (e) {
      vm = this
      textNode = vm.$refs[e.target.attrs.name][0].getNode();
      textNode.hide();
      tr.hide();
      layer2.draw();

      // create textarea over canvas with absolute position
      // first we need to find position for textarea
      // how to find it?

      // at first lets find position of text node relative to the stage:
      var textPosition = textNode.absolutePosition();

      // then lets find position of stage container on the page:
      var stageBox = stage.container().getBoundingClientRect();

      // so position of textarea will be the sum of positions above:
      var areaPosition = {
        x: stageBox.left + textPosition.x,
        y: stageBox.top + textPosition.y
      };

      // create textarea and style it
      var textarea = document.createElement('textarea');
      document.body.appendChild(textarea);

      // apply many styles to match text on canvas as close as possible
      // remember that text rendering on canvas and on the textarea can be different
      // and sometimes it is hard to make it 100% the same. But we will try...
      textarea.value = textNode.text();
      textarea.style.position = 'absolute';
      textarea.style.top = areaPosition.y + 'px';
      textarea.style.left = areaPosition.x + 'px';
      textarea.style.width = textNode.width() - textNode.padding() * 2 + 'px';
      textarea.style.height = textNode.height() - textNode.padding() * 2 + 5 + 'px';
      textarea.style.fontSize = textNode.fontSize() + 'px';
      textarea.style.border = 'none';
      textarea.style.padding = '0px';
      textarea.style.margin = '0px';
      textarea.style.overflow = 'hidden';
      textarea.style.background = 'none';
      textarea.style.outline = 'none';
      textarea.style.resize = 'none';
      textarea.style.lineHeight = textNode.lineHeight();
      textarea.style.fontFamily = textNode.fontFamily();
      textarea.style.transformOrigin = 'left top';
      textarea.style.textAlign = textNode.align();
      textarea.style.color = textNode.fill();
      var rotation = textNode.rotation();
      var transform = '';
      if (rotation) {
        transform += 'rotateZ(' + rotation + 'deg)';
      }

      var px = 0;
      // also we need to slightly move textarea on firefox
      // because it jumps a bit
      var isFirefox =
        navigator.userAgent.toLowerCase().indexOf('firefox') > -1;
      if (isFirefox) {
        px += 2 + Math.round(textNode.fontSize() / 20);
      }
      transform += 'translateY(-' + px + 'px)';

      textarea.style.transform = transform;

      // reset height
      textarea.style.height = 'auto';
      // after browsers resized it we can set actual value
      textarea.style.height = textarea.scrollHeight + 3 + 'px';

      textarea.focus();

      function removeTextarea () {
        textarea.parentNode.removeChild(textarea);
        window.removeEventListener('click', handleOutsideClick);
        textNode.show();
        tr.show();
        tr.forceUpdate();
        layer2.draw();
      }

      function setTextareaWidth (newWidth) {
        if (!newWidth) {
          // set width for placeholder
          newWidth = textNode.placeholder.length * textNode.fontSize();
        }
        // some extra fixes on different browsers
        var isSafari = /^((?!chrome|android).)*safari/i.test(
          navigator.userAgent
        );
        var isFirefox =
          navigator.userAgent.toLowerCase().indexOf('firefox') > -1;
        if (isSafari || isFirefox) {
          newWidth = Math.ceil(newWidth);
        }

        var isEdge =
          document.documentMode || /Edge/.test(navigator.userAgent);
        if (isEdge) {
          newWidth += 1;
        }
        textarea.style.width = newWidth + 'px';
      }

      textarea.addEventListener('keydown', function (e) {
        // hide on enter
        // but don't hide on shift + enter
        if (e.keyCode === 13 && !e.shiftKey) {
          textNode.text(textarea.value);
          removeTextarea();
        }
        // on esc do not set value back to node
        if (e.keyCode === 27) {
          removeTextarea();
        }
      });

      textarea.addEventListener('keydown', function (e) {
        var scale = textNode.getAbsoluteScale().x;
        setTextareaWidth(textNode.width() * scale);
        textarea.style.height = 'auto';
        textarea.style.height =
          textarea.scrollHeight + textNode.fontSize() + 'px';
      });

      function handleOutsideClick (e) {
        if (e.target !== textarea) {
          removeTextarea();
        }
      }
      setTimeout(() => {
        window.addEventListener('click', handleOutsideClick);
      });
    },


    handleTransformEnd (e) {
      console.log(e)
    },

    handleTransformStart (e) {
      console.log(e)
    }

  },

  mounted () {
    vm = this
    layer = vm.$refs.layer.getNode()

    layer2 = vm.$refs.layer2.getNode()

    stage = vm.$refs.stage.getNode()

    tr = vm.$refs.transformerText.getNode()




    var img = new Image();

    img.src = "/static/pag2.jpg";

    img.onload = () => {
      canvas = document.querySelectorAll('canvas');
      let cvs = canvas[0]
      let ctx = cvs.getContext("2d")
      console.log(img.width)
      console.log(img.height)
      this.$set(this.stageSize, 'height', img.height);
      this.$set(this.stageSize, 'width', img.width);
      cvs.width = img.width
      cvs.height = img.height
      // Draw image to the canvas
      ctx.drawImage(img, 0, 0)
    }

    canvas = document.querySelectorAll('canvas');
    canvas = canvas[canvas.length - 1]

    context = canvas.getContext('2d')

    stage.draw()
    layer.draw();

    // this.fitStageIntoParentContainer();
    // adapt the stage on any window resize
    // window.addEventListener('resize', this.fitStageIntoParentContainer);

    // console.table({ layer: layer, stage: stage, context: context, canvas: canvas, text: textNode, transform: tr });
  },

  created () {
  },

  beforeDestroy () {
    // window.removeEventListener('resize', this.fitStageIntoParentContainer);
  }
}
</script>
<style>
.page-center div {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
