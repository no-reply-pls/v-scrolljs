<template src="./index.html"></template>
<script>
import './index.less'

export default {
    props: {
        list: {
            type: Array,
            default () {
                return []
            }
        },
        total: {
            type: Number,
            default: 1
        },
        fetch: {
            type: Function
        },
        offsetItems: {
            type: Number,
            default: 20
        },
        itemHeight: {
            type: Number,
            default: 0
        },
        resize: {
            type: Boolean,
            default: false
        }
    },

    data () {
        return {
            anchorItem: {index: 0, offset: 0},
            lastScreenItem: {index: 0, offset: 0},
            anchorScrollTop: 0,
            runway: 0,
            items: [],
            loading: false,
            tombstoneSize: 0,
            firstAttachedIndex: 0,
            lastAttachedIndex: 0,
            scrollEl: null,
            completed: false,
            bottomHeight: 180
        }
    },
    computed: {
        visibleItems () {
            const items = this.items.slice(this.firstAttachedIndex, this.lastAttachedIndex)
            items.forEach((item, index) => {
                const data = this.list[this.firstAttachedIndex + index]
                item.data = data
            })
            return items
        }
    },
    watch: {
        completed (completed) {
            if (completed) {
                const endItem = this.items[Math.max(0, this.total - 1)]
                if (endItem) {
                    this.runway = endItem.top + endItem.realHeight + this.bottomHeight
                } else {
                    this.runway = this.bottomHeight
                }
            }
        },
        total () {
            this.completed = false
            this.onScroll()
        }
    },
    mounted () {
        this.scrollEl = this.$el.querySelector('.item-list')
        this.scrollEl.addEventListener('scroll', this.onScroll)
        window.addEventListener('resize', this.onResize)
        this.onResize(null, true)
    },

    destroyed () {
        this.scrollEl.removeEventListener('scroll', this.onScroll)
        window.removeEventListener('resize', this.onResize)
    },

    methods: {
        onResize (e, force = false) {
            if (this.resize || force) {
                this.tombstoneSize = this.$refs.tombstone.offsetHeight
                this.items.forEach(item => {
                    item.realHeight = 0
                    item.height = this.tombstoneSize
                    item.completed = 0
                })
                this.completed = false
                this.onScroll(undefined, true)
            }
        },
        onScroll (e, resize = false) {
            const delta = this.scrollEl.scrollTop - this.anchorScrollTop
            if (this.scrollEl.scrollTop === 0) {
                this.anchorItem = {index: 0, offset: 0}
            } else {
                this.anchorItem = this.calculateAnchoredItem(this.anchorItem, delta)
            }

            this.anchorItem.index = Math.min(this.total, this.anchorItem.index)
            this.anchorScrollTop = this.scrollEl.scrollTop

            this.lastScreenItem = this.calculateAnchoredItem(this.anchorItem, this.scrollEl.offsetHeight)

            if (!resize && this.anchorItem.index >= this.firstAttachedIndex && this.lastScreenItem.index < this.lastAttachedIndex) {
                return
            }
            this.firstAttachedIndex = Math.max(0, this.anchorItem.index - this.offsetItems)
            this.lastAttachedIndex = Math.min(this.lastScreenItem.index + this.offsetItems, this.total)

            this.addTombstone()
            this.calculateTop()
            this.loadItems()
            const lastItem = this.items[Math.max(0, this.total - 1)]
            if (this.total === 0 || (lastItem && lastItem.completed)) {
                this.completed = true
            }
            this.recalculateItemHeight(this.firstAttachedIndex, this.lastAttachedIndex)
        },
        calculateAnchoredItem (initialAnchor, delta) {
            if (delta === 0) return initialAnchor
            delta += initialAnchor.offset
            let i = initialAnchor.index
            let tombstones = 0
            if (delta < 0) {
                while (delta < 0 && i > 0 && this.items[i - 1].height) {
                    delta += this.items[i - 1].height
                    i--
                }
                tombstones = Math.max(-i, Math.ceil(Math.min(delta, 0) / this.tombstoneSize))
            } else {
                while (delta > 0 && i < this.items.length && this.items[i].height && this.items[i].height < delta) {
                    delta -= this.items[i].height
                    i++
                }
                if (i >= this.items.length || !this.items[i].height) {
                    tombstones = Math.floor(Math.max(delta, 0) / this.tombstoneSize)
                }
            }
            i += tombstones
            delta -= tombstones * this.tombstoneSize
            return {
                index: i,
                offset: delta
            }
        },
        calculateTop () {
            this.anchorScrollTop = 0
            for (let i = 0; i < this.anchorItem.index; i++) {
                this.anchorScrollTop += this.items[i].height
            }
            this.anchorScrollTop += this.anchorItem.offset

            let curPos = this.anchorScrollTop - this.anchorItem.offset
            let i = this.anchorItem.index
            while (i > this.firstAttachedIndex) {
                curPos -= this.items[i - 1].height
                i--
            }
            while (i < this.firstAttachedIndex) {
                curPos += this.items[i].height
                i++
            }
            for (i = this.firstAttachedIndex; i < this.lastAttachedIndex; i++) {
                const item = this.items[i]
                item.top = curPos
                curPos += item.height
            }
            if (!this.completed) {
                this.runway = Math.max(this.runway, curPos + this.bottomHeight)
                this.scrollEl.scrollTop = this.anchorScrollTop
            }
        },
        async loadItems () {
            // console.lot(this.items)
            if (this.loading || !this.fetch) return
            const len = this.list.length
            const itemsNeeded = this.lastAttachedIndex - len
            if (itemsNeeded <= 0) return

            const itemLen = this.items.length
            if (itemLen >= this.total) {
                this.lastAttachedIndex = this.total
                this.items.splice(this.total, itemLen - this.total)
                console.log('.........')
                return
            }

            this.loading = true
            try {
                await this.fetch()
                if (!this.completed) {
                    this.recalculateItemHeight(this.firstAttachedIndex, this.lastAttachedIndex)
                }
                this.loadItems()
            } catch (error) {
                console.log(error)
            }
            this.loading = false
        },

        addTombstone () {
            const len = this.lastAttachedIndex - this.items.length
            for (let i = 0; i < len; i++) {
                this.items.push({
                    data: null,
                    top: 0,
                    height: this.tombstoneSize,
                    realHeight: this.itemHeight,
                    completed: 0
                })
            }
        },

        recalculateItemHeight (firstAttachedIndex, lastAttachedIndex) {
            const fn = (firstAttachedIndex, lastAttachedIndex) => {
                let needCalculate = false

                if (this.itemHeight) {
                    for (let i = firstAttachedIndex; i < lastAttachedIndex; i++) {
                        const item = this.items[i]
                        if (item.completed) continue
                        item.height = item.realHeight
                        needCalculate = true
                    }
                } else {
                    this.$el.querySelectorAll('.flash-list > .item').forEach(node => {
                        const i = node.getAttribute('index')
                        const item = this.items[i]
                        if (!item.completed) {
                            const h = node.offsetHeight
                            item.realHeight = h
                            item.height = h
                            needCalculate = true
                        }
                    })
                }

                if (needCalculate) {
                    this.calculateTop()
                    for (let i = firstAttachedIndex; i < lastAttachedIndex; i++) {
                        const item = this.items[i]
                        if (item.realHeight && item.data) {
                            item.completed = 1
                        }
                    }
                }
                const lastItem = this.items[Math.max(0, this.total - 1)]
                if (this.total === 0 || (lastItem && lastItem.completed)) {
                    this.completed = true
                }
            }
            this.$nextTick(() => {
                fn(firstAttachedIndex, lastAttachedIndex)
            })
        },

        init () {
            this.anchorItem = {index: 0, offset: 0}
            this.lastScreenItem = {index: 0, offset: 0}
            this.anchorScrollTop = 0
            this.runway = 0
            this.items = []
            this.loading = false
            this.firstAttachedIndex = 0
            this.lastAttachedIndex = 0
            this.completed = false
            this.scrollEl.scrollTop = 0
            this.$nextTick(() => this.onResize(null, true))
        },

        anchorScroll (index) {
            const item = this.items[index]
            if (item && item.completed) {
                this.scrollEl.scrollTop = item.top
                this.onScroll()
            }
        }
    }

}
</script>
