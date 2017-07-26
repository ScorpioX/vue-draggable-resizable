<template>
  <div id="app">
    <div id="xx" style="height: 500px; margin: 20px; border: 1px solid red; position: relative;">
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
        x: 3,
        y: 6,
        w: 4,
        h: 3
      },
      grid: [ 20, 20 ]
    }
  }
}
</script>

<style>
  .active {
    border: 1px dashed black;
  }
  .vdr { background-color: yellow }
  #xx { width: 500px; }
</style>
