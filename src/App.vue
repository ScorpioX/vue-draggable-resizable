<template>
  <div id="app">
    <div id="xx" style="margin: 20px; border: 1px solid red; position: relative;">
      <vue-draggable-resizable :cfg="cfg" :grid="grid">
        <p>Component</p>
      </vue-draggable-resizable>
    </div>
    <div>{{cfg}}</div>
  </div>
</template>

<script>
import VueDraggableResizable from './components/vue-draggable-resizable'

export default {
  name: 'app',
  components: {
    'vue-draggable-resizable': VueDraggableResizable
  },
  data () {
    return {
      cfg: {
        x: 1,
        y: 1,
        w: 4,
        h: 4
      },
      grid: [ 20, 20 ]
    }
  },
  mounted () {
    this.resize()
    window.addEventListener('resize', this.resize)
  },
  methods: {
    resize () {
      let w = this.$el.parentNode.clientWidth
      this.grid = [w / 20, 20]
    }
  }
}
</script>

<style>
  .active {
    border: 1px dashed black;
  }
  .vdr { background-color: yellow }
  #xx { width: 50%; height: 480px; }
</style>
