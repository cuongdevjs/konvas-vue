<template>
  <div
    class="org-tree-container"
    id="luongXuLy"
  >
    <div
      class="org-tree"
      :class="{horizontal, collapsable}"
    >
      <org-tree-node
        :data="data"
        :props="props"
        :horizontal="horizontal"
        :label-width="labelWidth"
        :collapsable="collapsable"
        :render-content="renderContent"
        :label-class-name="labelClassName"
        @on-expand="$emit('onExpand', $event)"
      ></org-tree-node>
    </div>
  </div>
</template>

<script>
import render from './node'

export default {
  name: 'Vue2OrgTree',
  components: {
    OrgTreeNode: {
      render,
      functional: true
    }
  },
  props: {
    data: {
      type: Object,
      required: true
    },
    props: {
      type: Object,
      default: () => ({
        label: 'label',
        expand: 'expand',
        children: 'children'
      })
    },
    horizontal: Boolean,
    collapsable: Boolean,
    renderContent: Function,
    labelWidth: [String, Number],
    labelClassName: [Function, String]
  },
  data () {
    return {
      settings: {
        maxScrollbarLength: 750
      }
    }
  }
}
</script>

<style scoped>
.history-area {
  position: relative;
  background-color: #fff;
  width: 100%;
  max-height: 400px;
  height: 100%;
  margin: auto;
}

.org-tree-container {
  display: inline-block;
  background-color: #fff;
  width: 100%;
  height: 100%;
  margin: 10px 0;
}

.node_label {
  width: 8em;
  max-width: 9em;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.25);
  background-color: white;
  color: #abb4bd;
  border: 1px solid #abb4bd;
  padding: 3px 7px;
  border-radius: 25px;
  font-size: 12px;
}

.node_handlder {
  background-color: #e3e8ee;
  padding: 5px 3px;
  height: 5em;
  min-width: 4.5em;
  width: auto;
  border: 1px solid #e3e8ee;
  /* box-shadow: -0.5px 0 0 0 #e3e8ee; */
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.25);
  border-radius: 4px;
  max-width: 7em;
  max-height: 6em;
  cursor: pointer;
  transition: 0.3s;
  position: relative;
  display: grid;
}

.node_handlder:hover {
  transform: translateY(1px);
  box-shadow: none;
  transition: 0.3s;
}

.node_handlder.active {
  border: 1px solid #2b2d50;
  box-shadow: -0.5px 0 0 0 #e3e8ee;
}
.node_handlder .status {
  position: absolute;
  top: 0;
  color: #39ca74;
  right: 0;
  margin-right: 1px;
  margin-top: 1px;
}
.node_handlder .fa-check-double {
  font-size: 12px;
}

.node_handlder .info {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.node_handlder .name {
  font-size: 12px;
  color: #2d3e4f;
  font-weight: 700;
}

.node_handlder .info .tag {
  border-radius: 4px;
  background-color: #ffa94b;
  display: inline-block;
  padding: 2px 5px;
  border: 1px solid #e47e30;
  color: white;
  margin-top: 5px;
  font-size: 10px;
}

.org-tree {
  display: table;
  text-align: center;
}
.org-tree:after,
.org-tree:before {
  content: "";
  display: table;
}
.org-tree:after {
  clear: both;
}
.org-tree-node,
.org-tree-node-children {
  position: relative;
  margin: 0;
  padding: 0;
  list-style-type: none;
}
.org-tree-node-children:after,
.org-tree-node-children:before,
.org-tree-node:after,
.org-tree-node:before {
  transition: all 0.35s;
}
.org-tree-node-label {
  position: relative;
  display: inline-block;
}
.org-tree-node-label .org-tree-node-label-inner {
  /* padding: 10px 15px;
  text-align: center;
  border-radius: 3px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.15); */
}
.org-tree-node-btn {
  position: absolute;
  top: 100%;
  left: 50%;
  width: 15px;
  height: 15px;
  z-index: 10;
  margin-left: -11px;
  margin-top: 9px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 50%;
  box-shadow: 0 0 2px rgba(58, 56, 56, 0.15);
  cursor: pointer;
  transition: all 0.35s ease;
}
.org-tree-node-btn:hover {
  background-color: #e7e8e9;
  /* -webkit-transform: scale(1.1); */
  /* transform: scale(1.15); */
}
.org-tree-node-btn:after,
.org-tree-node-btn:before {
  content: "";
  position: absolute;
}
.org-tree-node-btn:before {
  top: 49%;
  left: 4px;
  right: 4px;
  height: 0;
  border-top: 1px solid #ccc;
}
.org-tree-node-btn:after {
  top: 4px;
  left: 46%;
  bottom: 4px;
  width: 0;
  border-left: 1px solid #ccc;
}
.org-tree-node-btn.expanded:after {
  border: none;
}
.org-tree-node {
  padding-top: 20px;
  display: table-cell;
  vertical-align: top;
}
.org-tree-node.collapsed,
.org-tree-node.is-leaf {
  padding-left: 10px;
  padding-right: 10px;
}
.org-tree-node:after,
.org-tree-node:before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 50%;
  height: 19px;
}
.org-tree-node:after {
  left: 50%;
  border-left: 1px solid #ddd;
}
.org-tree-node:not(:first-child):before,
.org-tree-node:not(:last-child):after {
  border-top: 1px solid #ddd;
}
.collapsable .org-tree-node.collapsed {
  padding-bottom: 30px;
}
.collapsable .org-tree-node.collapsed .org-tree-node-label:after {
  content: "";
  position: absolute;
  top: 100%;
  left: 0;
  width: 50%;
  height: 20px;
  border-right: 1px solid #ddd;
}
.org-tree > .org-tree-node {
  padding-top: 0;
}
.org-tree > .org-tree-node:after {
  border-left: 0;
}
.org-tree-node-children {
  padding-top: 20px;
  display: table;
}
.org-tree-node-children:before {
  content: "";
  position: absolute;
  top: 0;
  left: 50%;
  width: 0;
  height: 20px;
  border-left: 1px solid #ddd;
}
.org-tree-node-children:after {
  content: "";
  display: table;
  clear: both;
}
.horizontal .org-tree-node {
  display: table-cell;
  float: none;
  padding-top: 0;
  padding-left: 20px;
}
.horizontal > .org-tree-node {
  padding-left: 0;
}

.horizontal .org-tree-node.collapsed,
.horizontal .org-tree-node.is-leaf {
  padding-top: 10px;
  padding-bottom: 10px;
}
.horizontal .org-tree-node:after,
.horizontal .org-tree-node:before {
  width: 19px;
  height: 50%;
}
.horizontal .org-tree-node:after {
  top: 50%;
  left: 0;
  border-left: 0;
}
.horizontal .org-tree-node:only-child:before {
  top: 0px;
  border-bottom: 1px solid #ddd;
}
.horizontal .org-tree-node:not(:first-child):before,
.horizontal .org-tree-node:not(:last-child):after {
  border-top: 0;
  border-left: 1px solid #ddd;
}
.horizontal .org-tree-node:not(:only-child):after {
  border-top: 1px solid #ddd;
}
.horizontal .org-tree-node .org-tree-node-inner {
  display: table;
}
.horizontal .org-tree-node-label {
  display: table-cell;
  vertical-align: middle;
}
.horizontal.collapsable .org-tree-node.collapsed {
  padding-right: 30px;
}
.horizontal.collapsable .org-tree-node.collapsed .org-tree-node-label:after {
  top: 0;
  left: 100%;
  width: 20px;
  height: 50%;
  border-right: 0;
  border-bottom: 1px solid #ddd;
}
.horizontal .org-tree-node-btn {
  top: 51%;
  left: 100%;
  margin-top: -8px;
  margin-left: 12px;
}
.horizontal > .org-tree-node:only-child:before {
  border-bottom: 0;
}
.horizontal .org-tree-node-children {
  display: table-cell;
  padding-top: 0;
  padding-left: 20px;
}
.horizontal .org-tree-node-children:before {
  top: 50%;
  left: 0;
  width: 20px;
  height: 0;
  border-left: 0;
  border-top: 1px solid #ddd;
}
.horizontal .org-tree-node-children:after {
  display: none;
}
.horizontal .org-tree-node-children > .org-tree-node {
  display: block;
}

.horizontal .org-tree-node-children {
  vertical-align: middle;
}
/*# sourceMappingURL=style.css.map */
</style>
