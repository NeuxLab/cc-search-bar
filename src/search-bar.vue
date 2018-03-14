<template>
  <div class='search-bar '>
    <div class='input-icon-group'>
      <input class='input-search hint' type='text' readonly autocomplete="off" spellcheck="false" dir="ltr" :placeholder="hint">
      <input class='input-search input active' type="text" :placeholder="placeholder" v-model="searchkey" ref='input' @keydown.tab.prevent="autocomplete" @keydown.up.prevent="prevCurrent" @keydown.down.prevent="nextCurrent" @keydown.enter="done">
      <i class='icon' :class="[icon || 'icon-search']"></i>
    </div>
    <div class='typeahead-list select-list' v-show="hasItems">
      <scroller class='list-group search-bar-typeahead-scroller' :horizontal="horizontal" :scrollable="scrollable" :items="suggestions" ref='scroller' v-on:current-change="current = $event" v-on:item-click="autocomplete">
        <div class='item' slot-scope="{item, index}" v-on:mouseenter="$refs['scroller'].setCurrent(index)" v-on:mouseleave="$refs['scroller'].clearCurrent()">
          <slot :index="index" :item="item"></slot>
        </div>
      </scroller>
    </div>
  </div>
</template>

<script>
import Scroller from 'cc-scroller'

export default {
  name: 'search-bar',
  components: {
    scroller: Scroller
  },
  data() {
    return {
      searchkey: null,
      suggestions: [],
      current: -1
    }
  },
  props: ['data', 'label', 'scrollable', 'horizontal', 'tooltip', 'icon'],
  computed: {
    hint() {
      var v = this.suggestions[this.current]
      var label = v !== undefined && this.label(v);
      if (label && label.indexOf(this.searchkey) == 0) return label;
    },
    hasItems() {
      return this.suggestions.length > 0
    },
    placeholder(){
      if (this.current == -1){
        return this.tooltip;
      } else {
        return ''
      }
    }
  },
  watch: {
    searchkey: function(newValue) {
      let vm = this

      this.data(newValue).then((data) => {
        this.suggestions = data
        this.$nextTick( () => {
          if (newValue) {
            this.setCurrent(0)
          } else {
          }
        })
      })
    }
  },
  methods: {
    autocomplete() {
      if (this.hint && this.searchkey !== this.hint) {
        this.searchkey = this.hint
      }
    },
    done() {
      this.$emit("complete", { result: this.suggestions[this.current], query: this.searchkey });
    },
    reset(v) {
      this.searchkey = v || ""
      this.clearCurrent();
    },
    focus(){
      this.$nextTick(() => {
        this.$refs['input'].focus()
      })
    },
    blur(){
      this.$nextTick(() => {
        this.$refs['input'].blur()
      })
    },
    prevCurrent() {
      this.$refs['scroller'].prevCurrent() || this.clearCurrent();
    },
    nextCurrent() {
      this.$refs['scroller'].nextCurrent() || this.clearCurrent();
    },
    setCurrent(i) {
      this.$refs['scroller'].setCurrent(i)
    },
    clearCurrent() {
      this.$refs['scroller'].clearCurrent()
    }
  }
}
</script>

<style lang="less">
.search-bar {
  .input-icon-group .icon {
    position: absolute;
    top: 0;
    width: 2em;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .hint {
    position: absolute;
    top: 0;
    left: 0;
    border-color: transparent;
    box-shadow: none;
    opacity: 1;
    overflow: hidden;
  }
  .input {
    position: relative;
    vertical-align: top;
    background-color: transparent;
  }

}
</style>
