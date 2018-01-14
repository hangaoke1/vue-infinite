<template>
  <div class="hello">
    <infinite class="list" :size="size" :nomore="nomore" :list="list" :loadMore="loadMore">
      <template slot="item" slot-scope="props">
        <div class="item-wrap">
          <div class="item-box">
            <p>姓名: {{props.item.data.name}}</p>
            <p>年龄: {{props.item.data.age}}</p>
          </div>
        </div>
      </template>
      <template slot="shell" slot-scope="props">
        <div class="item-wrap item-shell">
        <div class="item-box">
          <p></p>
          <p></p>
        </div>
      </div>
      </template>
      <template slot="nomore">
        <p>没有更多啦</p>
      </template>
      <template slot="loading">
        <p>加载中...</p>
      </template>
    </infinite>
  </div>
</template>

<script>
import infinite from '@/components/infinite/infinite'
export default {
  name: 'HelloWorld',
  components: {
    infinite
  },
  data () {
    return {
      list: [],
      size: 10,
      nomore: false,
      page: 1
    }
  },
  methods: {
    loadMore () {
      // if (this.page > 2) {
      //   return false
      // }
      setTimeout(() => {
        var arr = []
        var count = 10
        // if (this.page === 2) {
        //   count = 8
        // }
        for (var i = 0; i < count; i++) {
          arr.push({
            name: '小王' + this.page + '序号' + i,
            age: parseInt(50 * Math.random())
          })
        }
        this.list = [...this.list, ...arr]
        this.page += 1
        // if (this.page === 3) {
        //   this.nomore = true
        // }
      }, 2000)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.list{
  width: 300px;
  height: 400px;
  margin: 0 auto;
  border: 1px solid red;
}
.item-wrap{
  box-sizing: border-box;
  padding: 20px;
}

p{
  margin: 0 0 10px 0;
  background: #fff;
}
.item-shell p{
  height: 50px;
}
</style>
