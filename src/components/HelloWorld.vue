<template>
  <vue2-org-tree
    v-if="root ? true : false"
    :data="root"
    :horizontal="true"
    :collapsable="true"
    :render-content="renderContent"
    @onExpand="onExpand"
  ></vue2-org-tree>
</template>

<script>
import Vue from 'vue'
// import Vue2OrgTree from 'vue2-org-tree'
import Vue2OrgTree from './vue2-tree/components/org-tree'

Vue.use(Vue2OrgTree)

export default {

  name: 'LuongXuLy',
  components: {
    Vue2OrgTree
  },
  data () {
    return {

      root: {
        id: 1,
        type: 'editorNode',
        label: 'Nguyễn Đức Quỳnh',
        expand: true,
        children: [
          {
            id: 2,
            type: 'banHanhNode',
            expand: true,
            label: 'ban hành',
            children: [
              {
                id: 3,
                type: 'donViBanHanhNode',
                label: 'TT CNTT',
                expand: true,
                children: [
                  {
                    id: 4,
                    type: 'nodeThuong',
                    expand: true,
                    label: 'Hà Viễn Dưong',
                    children: [
                      {
                        id: 5,
                        type: 'donViBanHanhNode',
                        label: 'Nguyễn Quốc Huy',
                        expand: true,
                        children: [
                          {
                            id: 6,
                            type: 'nodeThuong',
                            label: 'Nguyễn Văn Kiên',
                            expand: true,
                            children: [
                              {
                                id: 7,
                                type: 'banHanhNode',
                                label: 'ban hành',
                                expand: true,
                                children: [
                                  {
                                    id: 6,
                                    type: 'nodeThuong',
                                    label: 'Nguyễn Văn Kiên',
                                    expand: true
                                  }
                                ]
                              }
                            ]
                          }
                        ]
                      }
                    ]
                  }
                ]
              }
            ]
          }
        ]
      }
    }
  },
  methods: {
    /**
     * build node đầu tiên thể hiện người ban hành;
     */
    renderContent (h, data) {
      console.log('render content:')
      console.log(data)
      if (data.type == 'editorNode') {
        let classes = ['node_handlder center active']

        // let doc = data.doc

        // (this.user.id == doc.editorId) && (classes.push('active'))

        let components = []

        components.push(h('div', { class: 'info' }, [h('div', { class: 'name' }, data.label)]))

        return h('div', { class: classes.join(' ') }, components)
      } else if (data.type === 'banHanhNode') { // node banHanh
        return h('div', { class: 'node_label' }, [h('span', data.label)])
      } else if (data.type == 'donViBanHanhNode') {
        let classes = ['node_handlder center']

        let components = []

        components.push(h('div', { class: 'info' }, [h('div', { class: 'name' }, data.label)]))

        return h('div', { class: classes.join(' ') }, components)
      } else { // node thuong;
        let components = []

        let classes = ['node_handlder center']

        components.push(h('i', { class: 'fas fa-check-double font-size__small color__blue' }))

        components.push(h('div', { class: 'info' }, [h('div', { class: 'name' }, data.label)]))

        return h('div', { class: classes.join(' ') }, components)
      }
    },

    onExpand: function (data) {
      console.log('on expand')
      console.log(data)
      if ('expand' in data) {
        data.expand = !data.expand
        if (!data.expand && data.children) {
          this.collapse(data.children)
        }
      } else {
        this.$set(data, 'expand', true)
      }
      this.$forceUpdate()
    },
    collapse: function (list) {
      console.log('collapse')
      console.log(list)
      var _this = this
      list.forEach(function (child) {
        console.log('child')
        console.log(child)
        if (child.expand) {
          child.expand = false
        }
        child.children && _this.collapse(child.children)
      })
      this.$forceUpdate()
    },
    expandChange: function () {
      console.log(this.data)
      this.toggleExpand(this.data, this.expandAll)
    },
    toggleExpand: function (data, val) {
      var _this = this
      if (Array.isArray(data)) {
        data.forEach(function (item) {
          _this.$set(item, 'expand', val)
          if (item.children) {
            _this.toggleExpand(item.children, val)
          }
        })
      } else {
        this.$set(data, 'expand', val)
        if (data.children) {
          _this.toggleExpand(data.children, val)
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.org-tree-node-children {
  vertical-align: middle !important;
}

h1,
h2 {
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
</style>
