# v-scrolljs

> Infinite scroll list for Vue.js.

## Installation

###

``` bash
npm install -S v-scrolljs
```

## Import

``` javascript
import VScrolljs from 'v-scrolljs'
import 'v-scrolljs/dist/static/css/v-scrolljs.css'

export default {
  ...
  components: {
    scroll: VScrolljs,
  },
  ...
}
```


## Usage
``` html
<template>
  <div>
    ...
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
    ...
  </div>
</template>

<script>
import VScrolljs from 'v-scrolljs.js'
import 'v-scrolljs/dist/static/css/v-scrolljs.css'
const MESSAGES = [
    'data1',
    'data2',
    'data3',
]
export default {
    name: 'App',
    components: {
        scroll: VScrolljs
    },
    data () {
        return {
            items: [],
            page: {
                total: 3,
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
```

## Options

| Directive   | Type     | Default  |                                                                           |
| ---------   | ---------| -------- | ------------------------------------------------------------------------- |
| list        | Array    | required | List of items                                                             |
| total       | Number   | 1        | The number of total list                                                  |
| offsetItems | Number   | 20       | Number of additional nodes loaded outside the visual area                 |
| fetch       | Function | false    | The function of loading more items(async function)                        |
| resize      | Boolean  | false    | recalculate the height of each item When the resize event is triggered    |
| itemHeight  | Number   | 0        | Setting each item to the same height will reduce unnecessary calculations.|

## Development

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```