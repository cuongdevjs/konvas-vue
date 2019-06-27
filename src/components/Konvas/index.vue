<template>
  <dialogTemplate
    :openDialog="isOpen"
    :widthDialog="'1450px'"
    :classDialog="'dialog-doc-out-finish'"
    @closeDialog="$emit('closeDialog')"
    @closed="$emit('closed')"
  >
    <template #body>
      <el-row :gutter="12" class="">
        <el-col :span="5" class="pListThumb">
          <div class="pListThumb-Title margin-top-10">
            <h2 class="margin-botom-10 margin-top-10">
              {{ !isWatchingNote ? "Ghi chú văn bản" : "Xem chi tiết văn bản" }}
            </h2>
            <el-select
              v-if="isWatchingNote"
              class="pSelectThumb input-font__bold fix-width__small margin-top-10 margin-right-10"
              v-model="typeWatch"
              size="mini"
              filterable
              placeholder="-"
            >
              <el-option
                label="Đã chỉnh sửa"
                value="modified"
              ></el-option>
              <el-option
                label="Tất cả"
                value="all"
              ></el-option>
            </el-select>
          </div>
          <VuePerfectScrollbar class="pScroll">
            <pThumb
              v-for="(item, $index) in listImages"
              :key="$index"
              :item="item"
              :index="$index"
              :active="urlDocSelected.url == item.url ? true : false"
              :isModified="item.isModify ? true : false"
              v-if="!isWatchingNote || typeWatch == 'all' || (item.isModify && typeWatch == 'modified')"
              @selectedDoc="changeSelectedDoc"
            ></pThumb>
          </VuePerfectScrollbar>
        </el-col>
        <el-col :span="19" class="pDetailDocs">
          <detailDocs
            v-if="urlDocSelected.url"
            :isWatchingNote="isWatchingNote"
            :key="urlDocSelected.url"
            :urlImg="urlDocSelected.url"
            :listShapeDoc="urlDocSelected.listShapeDoc"
            :listArrowDoc="urlDocSelected.listArrowDoc"
            :listNoteDoc="urlDocSelected.listNoteDoc"
            :listLineDoc="urlDocSelected.listLineDoc"
            @modifiedDoc="modifiedDoc"
          />
        </el-col>
      </el-row>
    </template>
    <template #footer>
      <div class="flex-row flex__left form-button__action">
        <div v-if="!isWatchingNote">

          <el-button class="dialogBtnAlert" @click="refuseDocs">
          <span class="flex-row flex__center">
            <i class="font-size__small far fa-arrow-alt-circle-right"></i>
            &nbsp;Từ chối
          </span>
          </el-button>
          <el-button class="dialogBtnSecondary">
          <span class="flex-row flex__center">
            &nbsp;Huỷ
          </span>
          </el-button>
        </div>
        <div v-else>
          <el-button class="dialogBtnSecondary" @click="$emit('closeDialog')">
          <span class="flex-row flex__center">
            &nbsp;Xác nhận
          </span>
          </el-button>
        </div>
      </div>
    </template>
  </dialogTemplate>
</template>

<script>
  import constants from "../../../constants";
  import buildAttachmentURL from "../../../filters/buildAttachmentURL";
  import vbAttachmentTokenService from "../../../services/vbAttachmentTokenService";
  import attachmentService from "../../../services/attachmentService";
  import {EventBus} from "../../../event-bus.js";

  export default {
    name: "draw",
    props: ["isOpen", "vbAttachmentId", "isWatchingNote", "listPageNumberChange"],
    filters: {buildAttachmentURL},

    components: {
      VuePerfectScrollbar: () => import("vue-perfect-scrollbar"),
      dialogTemplate: () => import("../custom-dialog"),
      pThumb: () => import("./pThumbnail.vue"),
      detailDocs: () => import("./detailDocs.vue")
    },
    data() {
      return {
        token: null,
        from: 0,
        to: 1,
        typeWatch: "modified",
        urlDocSelected: "",

        listImages: []
      };
    },

    methods: {

      isShowThumb(index) {
        if (this.typeWatch == "all") {
          return true;
        } else {
          return this.isWatchingNote ? this.listPageNumberChange.indexOf(index) != -1 : false;
        }
      },

      async refuseDocs() {
        await EventBus.$emit("getChangeLastItem", {});
        await this.$emit("finishNoteDoc", this.listImages);
      },

      initToken() {
        this.getToken();
      },

      getPageSize() {
        return attachmentService
          .getPageSize(this.vbAttachmentId)
          .then(response => {
            this.pageSize = response.data[0].pageSize || this.to;
          })
          .catch(() => {
            this.pageSize = 24;
          });
      },

      changeImg(obj) {
        let index = this.listImages.findIndex(item => item.url == obj.url);
        this.$set(this.listImages[index], "uri", obj.uri);
        this.$set(this.listImages[index], "isModify", true);
      },

      getToken() {
        return vbAttachmentTokenService
          .getToken(this.vbAttachmentId)
          .then(response => {
            this.token = response.data.token;
            this.getPageSize().then(() => this.downloadImage());
          });
      },

      downloadImage() {
        // console.log(this.isWatchingNote)
        let token = this.token;
        for (let i = this.from; i < this.pageSize; i++) {
          this.listImages.push({
            url: this.$options.filters.buildAttachmentURL(token, {
              type: constants.DOWNLOAD_OPTION.TYPE.IMAGE,
              imageStatus: this.isWatchingNote == true ? "comment" : "original",
              pageNumber: i
            }),
            id: i,
            name: "Trang " + (i + 1),
            listShapeDoc: [],
            listLineDoc: [],
            listArrowDoc: [],
            listNoteDoc: [],
            base64: null,
            isModify: this.isWatchingNote ? this.listPageNumberChange.indexOf(i) != -1 : false
          });
        }
        this.urlDocSelected = this.listImages[0];
      },

      changeSelectedDoc(value) {
        this.urlDocSelected = null;
        this.urlDocSelected = value;
      },

      modifiedDoc(data) {
        let index = this.listImages.findIndex(item => item.url === data.url);
        this.$set(this.listImages[index], "listShapeDoc", data.listShape);
        this.$set(this.listImages[index], "listArrowDoc", data.listArrow);
        this.$set(this.listImages[index], "listLineDoc", data.listLine);
        this.$set(this.listImages[index], "listNoteDoc", data.listNote);
        this.$set(this.listImages[index], "base64", data.base64);
        this.$set(this.listImages[index], "isModify", true);
      }
    },

    computed: {
      listUrl: function () {
        return this.listImages.filter(item => item.url);
      }
    },

    created() {
      this.initToken();
      // this.fakeData();
    }
  };
</script>

<style lang="css">
  .pListThumb {
    max-height: calc(100vh - 100px);
    height: auto;
  }

  .pListThumb-Title {

  }

  .pScroll {
    max-height: calc(100vh - 153px);
    height: auto;
    margin-top: 15px;
    margin-bottom: 10px;
  }

  .pThumb {
    max-width: calc(100% - 10px);
    height: 100%;
    max-height: 350px;
    margin-bottom: 10px;
    border-radius: 8px;
    border: 1px solid #ceced2;
    position: relative;
    transition: all .25s ease-in-out;
  }


  .pThumb.active {
    border: 2px solid #007aff;
    transition: all .25s ease-in-out;
  }

  .pThumb .markIsModified.off {
    visibility: hidden;
  }

  .pThumb .markIsModified.on {
    visibility: visible;
  }

  .pThumb .markIsModified {
    position: absolute;
    width: 12px;
    height: 12px;
    flex: 0 0 auto;
    background-color: #e54d42;
    top: 3px;
    right: 3px;
    border-radius: 50%;
  }

  .pThumb .pBgThumb {
    max-width: 90%;
    width: 100%;
    height: 150px;
    margin: 0 10px;
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
  }

  .pThumb .pIndex {
    position: absolute;
    width: 100%;
    background-image: linear-gradient(to bottom, rgba(255, 255, 255, 0), rgb(255, 255, 255));
    height: 60px;
    bottom: 0;
    z-index: 0;
    border-radius: 0 0 8px 8px;
    padding-bottom: 10px;
    align-items: flex-end;
    justify-content: center;
    color: #2d3e4f;
  }


  .pDetailDocs {
    max-height: calc(100vh - 100px);
    height: auto;
  }

  .pControl {
    padding: 10px 0;
  }

  .pDetail {
    max-height: calc(100vh - 100px);
    height: auto;
  }

  .pScrollDoc {
    max-height: calc(100vh - 153px);
    height: auto;
    border: 1px solid #abb4bd;
    border-bottom-color: transparent;
    border-radius: 8px 0 0 0;
  }

  .pScrollDoc.pWatchingNote {
    margin-top: 50px;
  }

  .page-center > div {
    display: flex;
    justify-content: center;
    align-items: center;
    max-height: calc(100vh - 140px);
    height: calc(100vh - 140px);
    position: relative;
  }

  .BtnEdit {
    background-color: #efeff4;
    border: none;
  }
</style>
