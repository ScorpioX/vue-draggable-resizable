<template>
  <div class="vdr" :class="{ draggable: draggable, resizable: resizable, active: active }" @mousedown="elmDown" :style="style">
    <div
      class="handle"
      v-if="resizable"
      v-for="handle in handles"
      :class="'handle-' + handle"
      :style="{ display: active ? 'block' : 'none'}"
      @mousedown.stop.prevent="handleDown(handle, $event)"
    ></div>
    <slot></slot>
  </div>
</template>

<script>
import Vue from 'vue'

export default {
  replace: true,
  name: 'vue-draggable-resizable',
  props: {
    cfg: {
      type: Object
    },
    axis: {
      type: String,
      default: 'both',
      validator: function (val) {
        return ['x', 'y', 'both'].indexOf(val) !== -1
      }
    },
    grid: {
      type: Array,
      default: function () {
        return [1, 1]
      }
    }
  },
  created: function () {
    this.parentX = 0
    this.parentW = 9999
    this.parentY = 0
    this.parentH = 9999

    this.mouseX = 0
    this.mouseY = 0

    this.lastMouseX = 0
    this.lastMouseY = 0

    this.mouseOffX = 0
    this.mouseOffY = 0

    this.elmX = 0
    this.elmY = 0

    this.elmW = 0
    this.elmH = 0
  },
  mounted: function () {
    let x = this.cfg.x * this.grid[0]
    let y = this.cfg.y * this.grid[1]
    let w = this.cfg.w * this.grid[0]
    let h = this.cfg.h * this.grid[1]

    document.documentElement.addEventListener('mousemove', this.handleMove, true)
    document.documentElement.addEventListener('mousedown', this.deselect, true)
    document.documentElement.addEventListener('mouseup', this.handleUp, true)

    if (this.minw > this.w) this.width = this.minw
    if (this.minh > this.h) this.height = this.minh

    if (this.parent) {
      // const style = window.getComputedStyle(this.$el.parentNode, null)

      // const parentW = parseInt(style.getPropertyValue('width'), 10)
      // const parentH = parseInt(style.getPropertyValue('height'), 10)
      const node = this.$el.parentNode

      let parentW = node.clientWidth
      let parentH = node.clientHeight

      this.parentW = parentW
      this.parentH = parentH

      if (w > this.parentW) this.width = parentW
      if (h > this.parentH) this.height = parentH
      if ((x + w) > this.parentW) this.width = parentW - x
      if ((y + h) > this.parentH) this.height = parentH - y
    }

    this.top = y
    this.left = x
    this.width = w
    this.height = h

    this.$emit('resizing', this.left, this.top, this.width, this.height)
  },
  beforeDestroy: function () {
    document.documentElement.removeEventListener('mousemove', this.handleMove, true)
    document.documentElement.removeEventListener('mousedown', this.deselect, true)
    document.documentElement.removeEventListener('mouseup', this.handleUp, true)
  },
  data: function () {
    return {
      draggable: true,
      resizable: true,
      handles: ['tl', 'tm', 'tr', 'mr', 'br', 'bm', 'bl', 'ml'],
      parent: true,

      top: 0,
      left: 0,
      width: 0,
      height: 0,
      resizing: false,
      dragging: false,
      active: false,
      opacity: 1,
      handle: null,
      zIndex: 1
    }
  },
  methods: {
    elmDown: function (e) {
      if (!this.active) {
        this.zIndex += 1
        this.active = true

        this.$emit('activated')
      }

      this.elmX = parseInt(this.$el.style.left)
      this.elmY = parseInt(this.$el.style.top)
      this.elmW = this.$el.offsetWidth || this.$el.clientWidth
      this.elmH = this.$el.offsetHeight || this.$el.clientHeight

      if (this.draggable) {
        this.opacity = 0.6
        this.dragging = true
      }
    },
    deselect: function (e) {
      let target = e.target || e.srcElement
      let regex = new RegExp('handle-([trmbl]{2})', '')

      if (target !== this.$el && !regex.test(target.className)) {
        if (this.active) {
          this.active = false

          this.$emit('deactivated')
        }
      }
    },
    handleDown: function (handle, e) {
      this.handle = handle

      if (e.stopPropagation) e.stopPropagation()
      if (e.preventDefault) e.preventDefault()

      this.resizing = true
    },
    handleMove: function (e) {
      if (e.preventDefault) e.preventDefault()

      this.mouseX = e.pageX || e.clientX + document.documentElement.scrollLeft
      this.mouseY = e.pageY || e.clientY + document.documentElement.scrollTop

      let diffX = this.mouseX - this.lastMouseX + this.mouseOffX
      let diffY = this.mouseY - this.lastMouseY + this.mouseOffY

      this.mouseOffX = this.mouseOffY = 0

      this.lastMouseX = this.mouseX
      this.lastMouseY = this.mouseY

      let dX = diffX
      let dY = diffY

      if (this.resizing) {
        if (this.handle.indexOf('t') >= 0) {
          if (this.elmH - dY < this.minh) this.mouseOffY = (dY - (diffY = this.elmH - this.minh))
          else if (this.elmY + dY < this.parentY) this.mouseOffY = (dY - (diffY = this.parentY - this.elmY))
          this.elmY += diffY
          this.elmH -= diffY
        }

        if (this.handle.indexOf('b') >= 0) {
          if (this.elmH + dY < this.minh) this.mouseOffY = (dY - (diffY = this.minh - this.elmH))
          else if (this.elmY + this.elmH + dY > this.parentH) this.mouseOffY = (dY - (diffY = this.parentH - this.elmY - this.elmH))
          this.elmH += diffY
        }

        if (this.handle.indexOf('l') >= 0) {
          if (this.elmW - dX < this.minw) this.mouseOffX = (dX - (diffX = this.elmW - this.minw))
          else if (this.elmX + dX < this.parentX) this.mouseOffX = (dX - (diffX = this.parentX - this.elmX))
          this.elmX += diffX
          this.elmW -= diffX
        }

        if (this.handle.indexOf('r') >= 0) {
          if (this.elmW + dX < this.minw) this.mouseOffX = (dX - (diffX = this.minw - this.elmW))
          else if (this.elmX + this.elmW + dX > this.parentW) this.mouseOffX = (dX - (diffX = this.parentW - this.elmX - this.elmW))
          this.elmW += diffX
        }

        this.left = (Math.round(this.elmX / this.grid[0]) * this.grid[0])
        this.top = (Math.round(this.elmY / this.grid[1]) * this.grid[1])

        this.width = (Math.round(this.elmW / this.grid[0]) * this.grid[0])
        this.height = (Math.round(this.elmH / this.grid[1]) * this.grid[1])

        // this.$emit('resizing', this.left, this.top, this.width, this.height)
      } else if (this.dragging) {
        if (this.elmX + dX < this.parentX) this.mouseOffX = (dX - (diffX = this.parentX - this.elmX))
        else if (this.elmX + this.elmW + dX > this.parentW) this.mouseOffX = (dX - (diffX = this.parentW - this.elmX - this.elmW))

        if (this.elmY + dY < this.parentY) this.mouseOffY = (dY - (diffY = this.parentY - this.elmY))
        else if (this.elmY + this.elmH + dY > this.parentH) this.mouseOffY = (dY - (diffY = this.parentH - this.elmY - this.elmH))

        this.elmX += diffX
        this.elmY += diffY

        if (this.axis === 'x' || this.axis === 'both') {
          this.left = (Math.round(this.elmX / this.grid[0]) * this.grid[0])
        }
        if (this.axis === 'y' || this.axis === 'both') {
          this.top = (Math.round(this.elmY / this.grid[1]) * this.grid[1])
        }

        // this.$emit('dragging', this.left, this.top)
      }
    },
    handleUp: function (e) {
      this.handle = null
      if (this.resizing) {
        this.resizing = false
        // this.$emit('resizestop', this.left, this.top, this.width, this.height)
        let c = this.cfg
        c.x = this.cx
        c.y = this.cy
        c.w = this.cw
        c.h = this.ch
      }
      if (this.dragging) {
        this.dragging = false
        // this.$emit('dragstop', this.left, this.top)
        let c = this.cfg
        c.x = this.cx
        c.y = this.cy
      }
      this.opacity = 1

      this.elmX = this.left
      this.elmY = this.top
    }
  },
  computed: {
    cx () { return Math.round(this.left / this.grid[0]) },
    cy () { return Math.round(this.top / this.grid[1]) },
    cw () { return Math.round(this.width / this.grid[0]) },
    ch () { return Math.round(this.height / this.grid[1]) },
    minw () { return this.grid[0] },
    minh () { return this.grid[1] },
    style: function () {
      return {
        top: this.top + 'px',
        left: this.left + 'px',
        width: this.width + 'px',
        height: this.height + 'px',
        zIndex: this.zIndex,
        opacity: this.opacity
      }
    }
  }
}
</script>

<style scoped>
  .vdr {
    position: absolute;
    box-sizing: border-box;
  }
  .draggable:hover {
    cursor: move;
  }
  .handle {
    box-sizing: border-box;
    display: none;
    position: absolute;
    width: 10px;
    height: 10px;
    font-size: 1px;
    background: #EEE;
    border: 1px solid #333;
  }
  .handle-tl {
    top: -10px;
    left: -10px;
    cursor: nw-resize;
  }
  .handle-tm {
    top: -10px;
    left: 50%;
    margin-left: -5px;
    cursor: n-resize;
  }
  .handle-tr {
    top: -10px;
    right: -10px;
    cursor: ne-resize;
  }
  .handle-ml {
    top: 50%;
    margin-top: -5px;
    left: -10px;
    cursor: w-resize;
  }
  .handle-mr {
    top: 50%;
    margin-top: -5px;
    right: -10px;
    cursor: e-resize;
  }
  .handle-bl {
    bottom: -10px;
    left: -10px;
    cursor: sw-resize;
  }
  .handle-bm {
    bottom: -10px;
    left: 50%;
    margin-left: -5px;
    cursor: s-resize;
  }
  .handle-br {
    bottom: -10px;
    right: -10px;
    cursor: se-resize;
  }
</style>
