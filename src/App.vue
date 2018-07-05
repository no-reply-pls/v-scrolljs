<template>
  <div id="app">
    <div class="content">
      <scroll :list="items" :fetch="fetchItem" :total="page.total" :offsetItems="30">
          <template slot="tombstone" slot-scope="props">
              <div style="height: 50px;">loading....</div>
          </template>

          <template slot="item" slot-scope="props">
              <div style="border-bottom: 1px solid #eee;padding:10px 20px;word-break: break-all;">
                  <b>{{props.index}}&nbsp;&nbsp;</b>
                  <span>{{props.data}}</span>
              </div>
          </template>
          <template slot="bottom">
                All loaded
          </template>
      </scroll>
</div>
  </div>
</template>

<script>
// import VScrolljs from './components/index.vue'
import VScrolljs from './components/index.vue'
import data from './data.js'
const MESSAGES = data.message

export default {
    name: 'App',
    components: {
        scroll: VScrolljs
    },
    data () {
        return {
            items: [],
            page: {
                total: 10000,
                pageSize: 200,
                pageNumber: 1
            }
        }
    },
    methods: {
        fetchItem () {
            return new Promise((resolve, reject) => {
                setTimeout(() => {
                    const items = MESSAGES.slice(this.items.length, this.items.length + this.page.pageSize)
                    for (var i = 0; i < items.length; i++) {
                        this.items.push(items[i])
                    }
                    resolve()
                }, 300)
            })
        }
    }
}
</script>

<style lang="less">
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    .content{
        width: 800px;
        height: 500px;
        margin: 0px auto;
        border:1px solid #eee;
    }
}
</style>
