<template>
  <div class="infinite">
    <div class="infinite-list" ref="list" :style="`height: ${height}px`">
      <div class="infinite-list-item" v-for="(item, index) in showItems" :style="`transform: translate3d(0, ${item.top}px, 0)`">
        <div class="infinite-animate infinite-animate-item" :style="{opacity: +item.loaded}">
          <slot name="item" :item="item"></slot>
        </div>
        <div class="infinite-animate infinite-animate-shell" :style="{opacity: +!item.loaded}">
          <slot name="shell" :item="item"></slot>
        </div>
      </div>
      <div class="infinite-pool">
        <div class="infinite-list-item" v-for="(item, index) in items" :ref="'item' + index" v-if="!item.shell && !item.height" style="visibility: hidden;top: -1000px">
          <slot name="item" :item="item"></slot>
        </div>
        <div class="vue-recyclist-item" ref="shell" style="visibility: hidden;top: -1000px">
          <slot name="shell"></slot>
        </div>
      </div>
    </div>

    <div class="infinte-loading" v-show="loading && !nomore">
      <slot name="loading"></slot>
    </div>
    <div class="infinite-nomore" v-show="nomore">
      <slot name="nomore"></slot>
    </div>

    <div class="infinite-config">
      <div>list：{{list.length}}</div>
      <div>items：{{items.length}}</div>
      <div>{{showStart}} - {{showEnd}}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'infinite',
  props: {
    list: {
      type: Array,
      required: true
    },
    size: {
      type: Number,
      default: 20
    },
    loadMore: {
      type: Function,
      required: true
    },
    nomore: {
      type: Boolean,
      default: false
    },
    showShell: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      items: [],
      start: 0,
      height: 0,
      loadings: []
    }
  },
  computed: {
    showStart () {
      return Math.max(0, this.start - this.size)
    },
    showEnd () {
      return Math.min(this.start + this.size, this.items.length)
    },
    showItems () {
      return this.items.slice(Math.max(0, this.start - this.size), Math.min(this.start + this.size, this.items.length))
    },
    shellHeight () {
      return this.$refs.shell.offsetHeight
    },
    loading () {
      return this.loadings.length
    },
    lock () {
      return this.loadings.length > 2
    }
  },
  watch: {
    list (newVal) {
      if (newVal.length) {
        this.loadings.pop()
        if (!this.loading) {
          this.loadItems()
        }
      }
    },
    items (arr) {
      if (arr.length > this.list.length) {
        this.getItems()
      }
    },
    nomore (val) {
      if (val) {
        let dif = this.items.length - this.list.length
        this.$nextTick(() => {
          this.height -= dif * this.shellHeight
        })
      }
    }
  },
  mounted () {
    this.$el.addEventListener('scroll', this.scrollHandle.bind(this))
    this.init()
  },
  methods: {
    init () {
      this.reset()
      this.load()
    },
    reset () {
      this.items = []
      this.$el.scrollTop = 0
      this.height = 0
      this.start = 0
    },
    load () {
      this.items.length += this.size
      this.loadItems()
    },
    loadItems () {
      let load = []
      let start = 0
      let end = this.showShell ? this.items.length : this.list.length
      for (let i = start; i < end; i++) {
        if (this.items[i] && this.items[i].loaded) {
          continue
        }
        this.setItem(i, this.list[i] || null)
        load.push(i)
      }
      this.$nextTick(() => {
        load.forEach(index => {
          this.updateItemHeight(index)
        })
        this.updateItemTop()
      })
    },
    getItems () {
      this.loadings.push(1)
      this.loadMore()
    },
    setItem (index, data) {
      // 如果data为null 则是小坟包
      this.$set(this.items, index, {
        data: data || {},
        height: 0,
        top: -1000,
        shell: !data,
        loaded: !!data
      })
    },
    updateItemHeight (index) {
      let cur = this.items[index]
      const dom = this.$refs['item' + index]
      if (dom && dom[0]) {
        cur.height = dom[0].offsetHeight
      } else {
        cur.height = this.shellHeight
      }
    },
    updateItemTop () {
      this.height = 0
      this.items.forEach((item, index) => {
        const pre = this.items[index - 1]
        item.top = pre ? pre.top + pre.height : 0
        this.height += item.height
      })
      this.updateIndex()
    },
    updateIndex () {
      let scrollTop = this.$el.scrollTop
      for (let i = 0, length = this.items.length; i < length; i++) {
        if (this.items[i].top >= scrollTop) {
          this.start = Math.max(0, i - 1)
          break
        }
      }
    },
    scrollHandle () {
      if (this.loading || this.nomore) {
        this.updateIndex()
        return false
      }
      const scrollTop = this.$el.scrollTop
      const scrollHeight = this.$el.scrollHeight
      const height = this.$el.offsetHeight
      if (scrollHeight - scrollTop - height < 300) {
        this.load()
      }
      this.updateIndex()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .infinite{
    overflow-x: hidden;
    overflow-y: auto;
    background: #ccc;
  }
  .infinite-list{
    position: relative;
    padding: 0;
    margin: 0;
    list-style: none;
    overflow: hidden;
  }
  .infinite-list-item{
    width: 100%;
    position: absolute;
    padding: 0;
    margin: 0;
  }
  .infinite-animate, .vue-recyclist-item{
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    opacity: 0;
  }
  .infinite-animate-item{
    z-index: 2;
    transition: opacity .3s;
  }
  .infinite-animate-shell{
    z-index: 1;
  }
  .infinite-nomore{
    margin-top: 20px;
    text-align: center;
  }
  .infinite-config{
    position: fixed;
    top: 0;
    left: 0;
    padding: 5px;
    background: #1F2D3D;
    color: #fff;
    text-align: left;
  }
</style>
