<template>
  <div class='search-bar'>
    <div class='input-icon-group'>
      <input class='input-search hint' type='text' readonly autocomplete="off" spellcheck="false" dir="ltr" :placeholder="hint">
      <input class='input-search input active native-key-bindings' type="text" :placeholder="placeholder" v-model="searchkey" ref='input' @keydown="onKeyDown" @keydown.tab.prevent="autocomplete" @keydown.up.prevent="prevCurrent" @keydown.down.prevent="nextCurrent" @keydown.enter="onEnter" >
      <i class='icon' :class="[icon || 'icon-search']"></i>
    </div>
    <div class='typeahead-list select-list' v-show="hasItems">
      <scroller class='list-group search-bar-typeahead-scroller' :horizontal="horizontal" :scrollable="scrollable" :items="suggestions" ref='scroller' v-on:current-change="current = $event">
        <div class='item' slot-scope="{item, index}" v-on:mouseenter="$refs['scroller'].setCurrent(index)" v-on:mouseleave="$refs['scroller'].clearCurrent()" @click="done">
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
      current: -1,
      completed: false
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
    },
    disableIME() {
      return this.$attrs["ime-off"] !== undefined
    }
  },
  watch: {
    searchkey: function(newValue) {
      if (this.completed == 2) {
        this.completed = false
      }
      if (this.completed) {
        // this is revoked after complete = true. We need to keep complete != false until next change.
        this.completed = 2;
        return;
      }
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
      if (this.current >= 0 && this.hasItems) {
        this.searchkey = this.label(this.suggestions[this.current]);
      }
      this.$emit("complete", { result: this.suggestions[this.current], query: this.searchkey });
      this.suggestions = [];
      this.completed = true;
      // mouse click
      this.$refs['input'].focus()
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
    prevCurrent(event) {
      if (!this.completed && this.hasItems) {
        event.stopPropagation();
      }
      this.$refs['scroller'].prevCurrent() || this.clearCurrent();
    },
    nextCurrent(event) {
      if (!this.completed && this.hasItems) {
        event.stopPropagation();
      }
      this.$refs['scroller'].nextCurrent() || this.clearCurrent();
    },
    setCurrent(i) {
      this.$refs['scroller'].setCurrent(i)
    },
    clearCurrent() {
      this.$refs['scroller'].clearCurrent()
    },
    onKeyDown(event) {
      if (!this.disableIME) return;
      //it may be in composing already
      if (event.keyCode == 229) {
        if (event.key.length == 1) {
          this.searchkey += event.key
        } else {
          this.searchkey += event.code.replace(/key/i, "").toLowerCase();
        }
      }
      if (/[a-z0-9]/i.test(String.fromCharCode(event.keyCode))) {
        this.searchkey += String.fromCharCode(event.keyCode).toLowerCase();
      }
      if (event.keyCode == 8) {
        this.searchkey = this.searchkey.substr(0, this.searchkey.length - 1)
      }

      // make it readonly to disable ime
      this.$refs['input'].setAttribute('readonly', 'readonly')
    },
    onEnter(event) {
      if (event.isComposing) return;
      this.done();
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
    &:before {
      margin-right: 0;
    }
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
  .scroller-wrapper.list-group {
    margin-top: 10px;
  }
}
</style>
