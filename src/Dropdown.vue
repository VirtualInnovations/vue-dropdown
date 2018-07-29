<template>
<div class="hsy-dropdown" ref="mainPane" :class="cls" :style="{width: _width}">
  <div class="selected" @click="autoShow" :style="{backgroundPosition: (_true_width - 18) + 'px, center'}">{{ selectedText }}</div>
  <transition name="fadeIn" enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
    <div class="list" v-show="isShow" :style="{width: _width}">
      <div class="inner">
        <div class="item" v-if="!grouped" v-for="item in items" :data-title="item.label" @click="!multiple && itemClicked(item)" :class="{selected: item.selected}">
          {{ appendIdx(item) }}
          <div v-if="multiple">
            <input type="checkbox" :disabled="item.disabled" :checked="item.selected" @change="checkboxChanged(item)" :id="id(item)">
            <label :for="id(item)" @click="itemClicked(item)">{{ item.label }}</label>
          </div>
          <div v-else>
            {{ item.label }}
          </div>
        </div>
        <div v-if="grouped">
          <div v-for="group in items" class="group">
            <h3>{{ group.label }}</h3>
            <div class="item" v-for="item in group.children" :data-title="item.label" @click="!multiple && itemClicked(item)" :class="{selected: item.selected}">
              {{ appendIdx(item) }}
              <div v-if="multiple">
                <input type="checkbox" :disabled="item.disabled" :checked="item.selected" @change="checkboxChanged(item)" :id="id(item)">
                <label :for="id(item)" @click="itemClicked(item)">{{ item.label }}</label>
              </div>
              <div v-else>
                {{ item.label }}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </transition>
</div>
</template>

<script>
const EMPTY_FN = () => {}

let idx = 0

export default {
  name: 'Dropdown',

  data() {
    return {
      selected: [],
      isShow: false,
      items: [],
      mainEl:null
    }
  },
  props: {
    data: {
      type: Array,
      default: () => []
    },
    grouped: {
      type: Boolean,
      default: false
    },
    multiple: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: 'Please choose'
    },
    width: {
      default: '100%'
    },
    cbItemChanged: {
      type: Function,
      default: EMPTY_FN
    },
    cbCustomSelectedText: {
      type: Function,
      default: EMPTY_FN
    }
  },
  computed: {
    _width(){
      if (typeof this.width === 'string' || this.width instanceof String)
        return this.width;
      else if(typeof this.width === 'number')
        return this.width + 'px';
      return '100%';
    },
    _true_width(){
      return this.mainEl ? this.mainEl.offsetWidth : 150;
    },
    cls() {
      let c = {
        grouped: this.grouped,
        multiple: this.multiple
      }
      return c
    },
    selectedText() {
      let text = ''
      let fn = selected => selected.map((e) => e.label).join(', ')
      if (this.cbCustomSelectedText !== EMPTY_FN) {
        text = this.cbCustomSelectedText(this.selected, fn)
      } else {
        text = fn(this.selected)
      }
      return text === '' ? this.placeholder : text
    }
  },
  watch: {
    data: {
      immediate: true,
      handler(val) {
        this.selected = this.data.filter(d => d.selected)
        this.items = this.data.slice(0)
      }
    }
  },
  methods: {
    id(item) {
      return 'hsy-dropdown-item-' + item._idx
    },
    appendIdx(item) {
      if (item._idx === undefined) {
        item._idx = ++idx
      }
    },
    autoShow() {
      this.isShow = !this.isShow
    },
    autoHide(evt) {
      if (!this.$el.contains(evt.target)) {
        this.isShow = false
      }
    },
    findSelected() {
      if (!this.grouped) {
        return this.data.filter(d => d.selected === true)
      }
      let items = []
      return this.data.reduce((pre, e) => {
        pre = pre.concat(e.children)
        return pre
      }, items).filter(d => d.selected === true)
    },
    itemClicked(item) {
      if (!this.multiple) {
        if (this.selected.indexOf(item) === -1) {
          this.findSelected().forEach((d) => d.selected = false)
          item.selected = true
          this.selected.pop()
          this.selected.push(item)

          this.$emit('input', this.findSelected().map(d => d.value))
        }
        this.$nextTick(() => {
          this.isShow = false
        })
      }
    },
    checkboxChanged(item) {
      if (this.selected === null) {
        this.selected = []
      }

      let id = this.id(item)
      item.selected = document.querySelector('#' + id).checked

      if (!item.selected) {
        this.selected = this.selected.filter((d) => d !== item)
      } else {
        this.selected.push(item)
      }

      if (this.cbItemChanged !== EMPTY_FN) {
        this.cbItemChanged(item)
      }
      this.$emit('input', this.selected.map(d => d.value))
    }
  },
  mounted() {
    this.mainEl = this.$refs.mainPane;
    document.addEventListener('click', this.autoHide, false)
  },
  destroyed() {
    document.removeEventListener('click', this.autoHide, false)
  }
}
</script>

<style>

</style>
