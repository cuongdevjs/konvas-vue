<template>
  <div class="pDetail">
    <div
      class="
    pControl
    flex-row
    flex__between"
      v-if="!isWatchingNote"
    >
      <div class="pAddShape flex-row flex__left">
        <el-button class="BtnEdit" size="mini" @click="addShape">
          <img
            src="https://img.icons8.com/office/13/000000/rectangle-stroked.png"
          />
        </el-button>
        <el-button class="BtnEdit" size="mini" @click="addLine">
          <img
            src="https://img.icons8.com/ios/13/000000/horizontal-line-filled.png"
          />
        </el-button>
        <el-button class="BtnEdit" size="mini" @click="addArrow">
          <i class="fal fa-location-arrow font-size__medium"></i>
        </el-button>
        <el-button class="BtnEdit" size="mini" @click="addNote">
          <img
            src="https://img.icons8.com/material-two-tone/13/000000/text.png"
          />
        </el-button>
        <el-button
          size="mini"
          class="BtnEdit"
          :disabled="!selectedShapeName"
          @click="deleteDraw()"
        >
          <i class="fal fa-trash font-size__medium"></i>
        </el-button>
      </div>
      <div class="pEditShape flex-row flex__right">
        <el-button
          class="BtnEdit flex-row flex__left padding-top-5 padding-bottom-5 margin-right-10"
        >
          <img
            v-for="item in listFontSize"
            :key="item.value"
            @click="changeFontSizeNote(item.value)"
            class="margin-right-5
          margin-left-5 clickable"
            :src="
              'https://img.icons8.com/material/' +
                item.value +
                '/000000/circled-a.png'
            "
          />
        </el-button>
        <el-color-picker
          v-model="form.color"
          color-format="hex"
          size="mini"
          :disabled="!selectedShapeName"
        ></el-color-picker>
      </div>
    </div>
    <transition name="el-fade-in-linear">
      <VuePerfectScrollbar
        :class="[
          'pScrollDoc transition-box page-center',
          isWatchingNote ? 'pWatchingNote' : ''
        ]"
      >
        <v-stage
          ref="stage"
          :config="stageSize"
          @dblclick="addNote"
          @mouseup="handleMouseup"
          @mousemove="handleMousemove"
          @mousedown="handleMousedown"
          @touchend="handleMouseup"
          @touchstart="handleMousedown"
          @touchmove="handleMousemove"
          @click="touchEndLine"
        >
          <!-- image -->
          <v-layer ref="layer">
            <v-image
              ref="image"
              :config="{
                image: image
              }"
            />
          </v-layer>
          <!-- layer 2 -->
          <v-layer ref="layer2">
            <!-- note -->
            <v-text
              v-for="item in listNote"
              :key="item.config.name"
              :ref="item.config.name"
              :config="item.config"
              @click="editText"
              @transformend="updateShapeWhenChange"
              @dragend="updateShapeWhenChange"
            />
            <!-- line -->
            <v-line
              v-for="item in listLine"
              :key="item.config.name"
              :ref="item.config.name"
              :config="item.config"
              @click="defineTarget"
              @transformend="updateShapeWhenChange"
              @dragend="updateShapeWhenChange"
            />
            <!-- shape -->
            <v-rect
              v-for="item in listShape"
              :key="item.config.name"
              :ref="item.config.name"
              :config="item.config"
              @click="defineTarget"
              @transformend="updateShapeWhenChange"
              @dragend="updateShapeWhenChange"
            />

            <v-arrow
              v-for="item in listArrow"
              :key="item.config.name"
              :ref="item.config.name"
              :config="item.config"
              @click="defineTarget"
              @transformend="updateShapeWhenChange"
              @dragend="updateShapeWhenChange"
            />

            <v-transformer
              ref="transformerText"
              :config="{
                anchorStroke: 'red',
                anchorFill: 'red',
                anchorSize: 5,
                borderStroke: 'red',
                borderDash: [5, 5],
                enabledAnchors:
                  selectedTypeName === 'line'
                    ? [
                        'top-left',
                        'top-right',
                        'middle-right',
                        'middle-left',
                        'bottom-left',
                        'bottom-right'
                      ]
                    : [
                        'top-left',
                        'top-center',
                        'top-right',
                        'middle-right',
                        'middle-left',
                        'bottom-left',
                        'bottom-center',
                        'bottom-right'
                      ],
                rotationSnaps: [0, 90, 180, 270],
                padding: 10
              }"
            />
          </v-layer>
        </v-stage>
      </VuePerfectScrollbar>
    </transition>
  </div>
</template>

<script>
import VuePerfectScrollbar from "vue-perfect-scrollbar";
import { TYPE, STYLE } from "./const-shape";
import { EventBus } from "../../../event-bus.js";

let vm = {};
let stage = null;
let layer = null;
let layer2 = null;
let textNode = null;
let tr = null;
let itemSelected = null;

export default {
  components: {
    VuePerfectScrollbar
  },

  props: [
    "urlImg",
    "listShapeDoc",
    "listLineDoc",
    "listArrowDoc",
    "listNoteDoc",
    "isWatchingNote"
  ],

  beforeDestroy() {
    !this.isWatchingNote && this.sendDataItemChange();
  },

  data() {
    return {
      isFillData: false,

      form: {
        color: "#FF0000",
        width: null,
        fontSize: null
      },

      stageSize: {
        width: 1000,
        height: 1000
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

      selectedShapeName: "",

      selectedTypeName: ""
    };
  },

  methods: {
    sendDataItemChange() {
      if (
        this.listShape.length ||
        this.listLine.length ||
        this.listNote.length ||
        this.listArrow.length
      ) {
        this.$emit("modifiedDoc", {
          url: this.urlImg,
          listShape: this.listShape,
          listLine: this.listLine,
          listArrow: this.listArrow,
          listNote: this.listNote,
          base64: stage.toDataURL()
        });
      }
    },

    addTransformerImmediate(nameShape = "", typeShape = "") {
      textNode = vm.$refs[nameShape][0].getNode();
      this.selectedShapeName = nameShape;
      this.selectedTypeName = typeShape;
      this.updateTransformer();
    },

    addNote() {
      if (!this.isWatchingNote) {
        this.numberRefs += 1;
        let new_note = {
          config: {
            width: 150,
            text: "Nhấn để chính sửa",
            fontSize: STYLE.FONT_SIZE.MEDIUM.value,
            fontFamily: STYLE.FONT_FAMILY,
            draggable: true,
            name: `note${this.numberRefs}`,
            x: this.lastPointerPosition.x || 100,
            y: this.lastPointerPosition.y || 0,
            ellipsis: true,
            fill: STYLE.COLOR,
            type: TYPE.NOTE
          }
        };
        let numberNote = this.listNote.length;
        this.$set(this.listNote, numberNote, new_note);

        this.$nextTick(() => {
          this.addTransformerImmediate(
            this.listNote[numberNote].config.name,
            this.listNote[numberNote].config.type
          );
        });
      }
    },

    addLine() {
      if (!this.isWatchingNote) {
        this.isAddLine = true;
      }
    },

    touchEndLine() {
      if (!this.isWatchingNote) {
        if (this.isAddLine) {
          this.pointerPosition = stage.getPointerPosition();
          // each click number += 1, if even number => first point (add line), otherwise => last point (update length line (position last - first))
          if (this.numberToCheckAddLine % 2 === 0) {
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
                stroke: STYLE.COLOR,
                strokeWidth: 4,
                type: TYPE.LINE
              }
            };
            let numberLine = this.listLine.length;
            this.$set(this.listLine, numberLine, new_line);
            this.$nextTick(() => {
              this.addTransformerImmediate(
                this.listLine[numberLine].config.name,
                this.listLine[numberLine].config.type
              );
            });
          } else {
            this.listLine.forEach(item => {
              if (item.config.name === this.selectedShapeName) {
                this.isAddLine = false;
                this.$set(
                  item.config.points,
                  4,
                  this.pointerPosition.x - item.config.x
                );
                console.log(item.config);
                this.addTransformerImmediate(
                  item.config.name,
                  item.config.type
                );
                this.updateTransformer();
              }
            });
            this.$forceUpdate();
          }
          this.numberToCheckAddLine += 1;
        }
      }
    },

    addShape() {
      if (!this.isWatchingNote) {
        this.numberRefs += 1;
        let new_shape = {
          config: {
            x: 150,
            y: 150,
            width: 100,
            height: 25,
            stroke: STYLE.COLOR,
            strokeWidth: 1,
            name: `shape${this.numberRefs}`,
            draggable: true,
            type: TYPE.SHAPE,
            strokeScaleEnabled: false
          }
        };
        let numberShape = this.listShape.length;
        this.$set(this.listShape, numberShape, new_shape);
        this.$nextTick(() => {
          this.addTransformerImmediate(
            this.listShape[numberShape].config.name,
            this.listShape[numberShape].config.type
          );
        });
      }
    },

    addArrow() {
      if (!this.isWatchingNote) {
        this.numberRefs += 1;
        let new_arrow = {
          config: {
            x: 100,
            y: 100,
            points: [0, 0, 70, 0],
            pointerLength: 5,
            pointerWidth: 10,
            name: `arrow${this.numberRefs}`,
            stroke: STYLE.COLOR,
            fill: STYLE.COLOR,
            draggable: true,
            strokeWidth: 1,
            type: TYPE.ARROW,
            rotation: 0
          }
        };
        let numberArrow = this.listArrow.length;
        this.$set(this.listArrow, numberArrow, new_arrow);
        this.$nextTick(() => {
          this.addTransformerImmediate(
            this.listArrow[numberArrow].config.name,
            this.listArrow[numberArrow].config.type
          );
        });
      }
    },

    handleMousedown(e) {
      if (!this.isAddLine && !this.isWatchingNote) {
        var pos = stage.getPointerPosition();
        this.$set(this.lastPointerPosition, "x", pos.x);
        this.$set(this.lastPointerPosition, "y", pos.y);

        if (e.target === e.target.getStage()) {
          this.selectedShapeName = "";
          this.selectedTypeName = "";
          this.updateTransformer();
          return;
        }
        // clicked on transformer - do nothing
        const clickedOnTransformer =
          e.target.getParent().className === "Transformer";
        if (clickedOnTransformer) {
          return;
        }
        // find clicked rect by its name
        const name = e.target.name();
        const rect = textNode;
        if (rect) {
          this.selectedShapeName = name;
          this.selectedTypeName = e.target.attrs.type;
        } else {
          this.selectedShapeName = "";
          this.selectedTypeName = "";
        }
        this.updateTransformer();
      }
    },

    handleMouseup(e) {
      this.isPaint = false;
    },

    handleMousemove(e) {},

    updateTransformer() {
      if (!this.isAddLine && !this.isWatchingnote) {
        // here we need to manually attach or detach Transformer node
        const transformerNode = this.$refs.transformerText.getStage();
        const stage = transformerNode.getStage();
        const { selectedShapeName } = this;

        const selectedNode = stage.findOne("." + selectedShapeName);

        if (selectedNode === transformerNode.node()) {
          return; // do nothing if selected node is already attached
        }
        if (selectedNode) {
          this.isFillData = true;
          this.fillDataShape(this.selectedShapeName, this.selectedTypeName);
          this.setRefsShapeSelected();
          transformerNode.attachTo(selectedNode); //attach transformer to target
        } else {
          transformerNode.detach(); // remove transformer
        }
        transformerNode.getLayer().batchDraw();
      }
    },

    defineTarget(e) {
      vm = this;
      textNode = vm.$refs[e.target.attrs.name][0].getNode();
    },

    editText(e) {
      this.defineTarget(e);
      textNode.hide();
      tr.hide();
      layer2.draw();

      // create textarea over canvas with absolute position
      // first we need to find position for textarea
      // how to find it?

      // at first lets find position of text node relative to the stage:
      var textPosition = textNode.absolutePosition();

      // create textarea and style it
      var textarea = document.createElement("textarea");
      var eleParent = document.querySelector(".konvajs-content");
      eleParent.appendChild(textarea);

      // apply many styles to match text on canvas as close as possible
      // remember that text rendering on canvas and on the textarea can be different
      // and sometimes it is hard to make it 100% the same. But we will try...
      textarea.value = textNode.text();
      textarea.style.position = "absolute";
      textarea.style.top = textPosition.y + "px";
      textarea.style.left = textPosition.x + "px";
      textarea.style.width = textNode.width() - textNode.padding() * 2 + "px";
      textarea.style.height =
        textNode.height() - textNode.padding() * 2 + 5 + "px";
      textarea.style.fontSize = textNode.fontSize() + "px";
      textarea.style.border = "none";
      textarea.style.padding = "0px";
      textarea.style.margin = "0px";
      textarea.style.overflow = "hidden";
      textarea.style.background = "none";
      textarea.style.outline = "none";
      textarea.style.resize = "none";
      textarea.style.lineHeight = textNode.lineHeight();
      textarea.style.fontFamily = textNode.fontFamily() || "";
      textarea.style.transformOrigin = "left top";
      textarea.style.textAlign = textNode.align() || "left";
      textarea.style.color = textNode.fill() || "gray";
      var rotation = textNode.rotation() || 0;
      var transform = "";
      if (rotation) {
        transform += "rotateZ(" + rotation + "deg)";
      }

      var px = 0;
      // also we need to slightly move textarea on firefox
      // because it jumps a bit
      var isFirefox = navigator.userAgent.toLowerCase().indexOf("firefox") > -1;
      if (isFirefox) {
        px += 2 + Math.round(textNode.fontSize() / 20);
      }
      transform += "translateY(-" + px + "px)";

      textarea.style.transform = transform;

      // reset height
      textarea.style.height = "auto";
      // after browsers resized it we can set actual value
      textarea.style.height = textarea.scrollHeight + 3 + "px";

      textarea.focus();

      function removeTextarea() {
        textarea.parentNode.removeChild(textarea);
        window.removeEventListener("click", handleOutsideClick);
        textNode.show();
        tr.show();
        tr.forceUpdate();
        layer2.draw();
      }

      function setTextareaWidth(newWidth) {
        if (!newWidth) {
          // set width for placeholder
          newWidth = textNode.placeholder.length * textNode.fontSize();
        }
        // some extra fixes on different browsers
        var isSafari = /^((?!chrome|android).)*safari/i.test(
          navigator.userAgent
        );
        var isFirefox =
          navigator.userAgent.toLowerCase().indexOf("firefox") > -1;
        if (isSafari || isFirefox) {
          newWidth = Math.ceil(newWidth);
        }

        var isEdge = document.documentMode || /Edge/.test(navigator.userAgent);
        if (isEdge) {
          newWidth += 1;
        }
        textarea.style.width = newWidth + "px";
      }

      textarea.addEventListener("keydown", function(e) {
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

      textarea.addEventListener("keydown", function(e) {
        var scale = textNode.getAbsoluteScale().x;
        setTextareaWidth(textNode.width() * scale);
        textarea.style.height = "auto";
        textarea.style.height =
          textarea.scrollHeight + textNode.fontSize() + "px";
      });

      function handleOutsideClick(e) {
        if (e.target !== textarea) {
          removeTextarea();
        }
      }
      setTimeout(() => {
        window.addEventListener("click", handleOutsideClick);
      });
    },

    findIndexItemSelected(typeName, shapeName) {
      switch (typeName) {
        case TYPE.LINE:
          return this.listLine.findIndex(
            item => item.config.name === shapeName
          );
        case TYPE.NOTE:
          return this.listNote.findIndex(
            item => item.config.name === shapeName
          );
        case TYPE.ARROW:
          return this.listArrow.findIndex(
            item => item.config.name === shapeName
          );
        case TYPE.SHAPE:
          return this.listShape.findIndex(
            item => item.config.name === shapeName
          );
        default:
          break;
      }
    },

    deleteDraw() {
      if (this.selectedShapeName && this.selectedTypeName) {
        try {
          let index = this.findIndexItemSelected(
            this.selectedTypeName,
            this.selectedShapeName
          );
          switch (this.selectedTypeName) {
            case TYPE.LINE:
              this.listLine.splice(index, 1);
              break;
            case TYPE.NOTE:
              this.listNote.splice(index, 1);
              break;
            case TYPE.ARROW:
              this.listArrow.splice(index, 1);
              break;
            case TYPE.SHAPE:
              this.listShape.splice(index, 1);
              break;
            default:
              break;
          }
        } catch (error) {
          console.log(error);
        } finally {
          this.selectedShapeName = "";
          this.selectedTypeName = "";
          this.updateTransformer();
          this.$forceUpdate();
        }
      }
    },

    fillDataShape(shapeName = "", typeName = "") {
      try {
        let index = this.findIndexItemSelected(typeName, shapeName);
        switch (typeName) {
          case TYPE.LINE:
            this.$set(this.form, "color", this.listLine[index].config.stroke);
            break;
          case TYPE.NOTE:
            this.$set(
              this.form,
              "fontSize",
              this.listNote[index].config.fontSize
            );
            this.$set(this.form, "width", this.listNote[index].config.width);
            this.$set(this.form, "color", this.listNote[index].config.fill);
            break;
          case TYPE.ARROW:
            this.$set(this.form, "color", this.listArrow[index].config.stroke);
            break;
          case TYPE.SHAPE:
            this.$set(this.form, "color", this.listShape[index].config.stroke);
            break;
          default:
            this.form = {};
            break;
        }
      } catch (err) {
        console.log(err);
      } finally {
        this.isFillData = false;
      }
    },

    // downloadURI(uri, name) {
    // var link = document.createElement('a');
    // link.download = name;
    // link.href = uri;
    // document.body.appendChild(link);
    // // link.click();
    // document.body.removeChild(link);
    // link = ""
    // },

    // exportToImage() {
    // this.selectedShapeName = "";
    // this.updateTransformer();
    // var dataURL = stage.toDataURL();
    // // console.log(dataURL);
    // this.downloadURI(dataURL, 'stage.png');
    // },

    updateDataShape(shapeName, typeName, typeData, value) {
      try {
        let index = this.findIndexItemSelected(typeName, shapeName);
        switch (typeData) {
          case "color":
            this.updateColorItemByType(typeName, index, value);
            break;
          case "fontSize":
            this.$set(this.listNote[index].config, "fontSize", value);
            break;
          case "width":
            this.$set(this.listNote[index].config, "width", value);
            break;
          default:
            break;
        }
      } catch (error) {
      } finally {
        this.$forceUpdate();
      }
    },

    updateColorItemByType(typeItem, indexItem, newValue) {
      try {
        switch (typeItem) {
          case TYPE.LINE:
            this.$set(this.listLine[indexItem].config, "stroke", newValue);
            break;
          case TYPE.NOTE:
            this.$set(this.listNote[indexItem].config, "fill", newValue);
            break;
          case TYPE.ARROW:
            this.$set(this.listArrow[indexItem].config, "stroke", newValue);
            this.$set(this.listArrow[indexItem].config, "fill", newValue);
            break;
          case TYPE.SHAPE:
            this.$set(this.listShape[indexItem].config, "stroke", newValue);
            break;
          default:
            break;
        }
      } catch (err) {
        console.log(err);
      }
    },

    setRefsShapeSelected() {
      if (this.selectedTypeName && this.selectedShapeName) {
        let index = this.findIndexItemSelected(
          this.selectedTypeName,
          this.selectedShapeName
        );
        let nameRefs = null;
        switch (this.selectedTypeName) {
          case TYPE.LINE:
            nameRefs = this.listLine[index].config.name;
            break;
          case TYPE.NOTE:
            nameRefs = this.listNote[index].config.name;
            break;
          case TYPE.ARROW:
            nameRefs = this.listArrow[index].config.name;
            break;
          case TYPE.SHAPE:
            nameRefs = this.listShape[index].config.name;
            break;
          default:
            break;
        }
        nameRefs && (itemSelected = this.$refs[nameRefs][0].getNode());
      }
    },

    getChangeOfShape() {
      return {
        x: itemSelected.x(),
        y: itemSelected.y(),
        width: itemSelected.width(),
        height: itemSelected.height(),
        rotation: itemSelected.rotation(),
        scaleX: itemSelected.scaleX(),
        scaleY: itemSelected.scaleY()
      };
    },

    updateShapeWhenChange(e) {
      let change = this.getChangeOfShape();
      if (this.selectedShapeName && this.selectedTypeName) {
        try {
          let index = this.findIndexItemSelected(
            this.selectedTypeName,
            this.selectedShapeName
          );
          switch (this.selectedTypeName) {
            case TYPE.LINE:
              this.listLine[index].config.points[4] *= change.scaleX;
              this.listLine[index].config.x = change.x;
              this.listLine[index].config.y = change.y;
              break;
            case TYPE.NOTE:
              this.listNote[index].config.width *= change.scaleX;
              this.listNote[index].config.x = change.x;
              this.listNote[index].config.y = change.y;
              break;
            case TYPE.ARROW:
              this.listArrow[index].config.points[2] *= change.scaleX;
              this.listArrow[index].config.x = change.x;
              this.listArrow[index].config.y = change.y;
              this.listArrow[index].config.rotation = change.rotation;
              break;
            case TYPE.SHAPE:
              this.listShape[index].config.width *= change.scaleX;
              this.listShape[index].config.height *= change.scaleY;
              this.listShape[index].config.x = change.x;
              this.listShape[index].config.y = change.y;
              break;
            default:
              break;
          }
          itemSelected.scaleX(1);
          itemSelected.scaleY(1);
          this.updateTransformer();
          this.$forceUpdate();
        } catch (error) {}
      }
    },

    changeFontSizeNote(value) {
      if (
        value &&
        !this.isFillData &&
        this.selectedShapeName &&
        this.selectedTypeName
      ) {
        this.updateDataShape(
          this.selectedShapeName,
          this.selectedTypeName,
          "fontSize",
          parseInt(value)
        );
      }
    },

    initState() {
      this.$nextTick(() => {
        vm = this;
        layer = this.$refs.layer.getNode();
        layer2 = this.$refs.layer2.getNode();
        stage = this.$refs.stage.getNode();
        tr = this.$refs.transformerText.getNode();

        let parentNode = document.querySelector(".konvajs-content");
        parentNode.style.position = "absolute";
        parentNode.style.top = 0;

        let wrapper = document.querySelector(".pScrollDoc");
        var img = new Image();
        img.setAttribute("crossOrigin", "anonymous");
        img.src = this.urlImg;
        img.onload = () => {
          let ratio = img.width / img.height;
          img.width = wrapper.clientWidth;
          img.height = img.width / ratio;
          this.$set(this.stageSize, "height", img.height);
          this.$set(this.stageSize, "width", img.width);
          this.image = img;
        };

        stage.draw();
        layer.draw();
        layer2.draw();
      });
    }
  },

  mounted() {
    this.initState();
    !this.isWatchingNote &&
      EventBus.$on("getChangeLastItem", () => {
        this.sendDataItemChange();
      });
  },

  watch: {
    urlImg: {
      immediate: true,
      deep: true,
      handler(value) {
        this.selectedShapeName = "";
        this.selectedTypeName = "";
        this.listShape = [].concat(this.listShapeDoc);
        this.listNote = [].concat(this.listNoteDoc);
        this.listArrow = [].concat(this.listArrowDoc);
        this.listLine = [].concat(this.listLineDoc);
        this.urlImg = value;
      }
    },

    "form.color": function(newValue) {
      if (
        newValue &&
        !this.isFillData &&
        this.selectedShapeName &&
        this.selectedTypeName
      ) {
        this.updateDataShape(
          this.selectedShapeName,
          this.selectedTypeName,
          "color",
          newValue.toString()
        );
      }
    }
  },

  computed: {
    listFontSize: function() {
      let arr = [];
      Object.keys(STYLE.FONT_SIZE).forEach(key => {
        arr.push(STYLE.FONT_SIZE[key]);
      });
      return arr;
    }
  }
};
</script>
<style></style>
