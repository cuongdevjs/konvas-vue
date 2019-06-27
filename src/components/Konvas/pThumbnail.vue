<template functional>
  <div
    :class="[
      'pThumb flex-col flex__center clickable margin-right-10',
      props.active ? 'active' : ''
    ]"
    @click="listeners['selectedDoc'](props.item)"
  >
    <div
      class="pBgThumb"
      :style="`background-image: url( ${props.item.url} )`"
    ></div>
    <div class="pIndex flex-row font-size__small font-weight__bold">
      Trang {{ props.index +1}}
    </div>
    <div
      :class="['markIsModified', props.item.isModify == true ? 'on' : 'off']"
    ></div>
  </div>
</template>

<script>
export default {
  name: "listThumb",
  props: {
    item: {
      type: Object,
      default: () => {
        return {};
      }
    },
    index: {
      type: Number,
      default: 0
    },
    active: {
      type: Boolean,
      default: false
    }
  }
};
</script>

<style lang="css" scoped></style>
