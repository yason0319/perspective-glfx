<template>
<div>
  <konva-stage ref="konvaStage" :config="configKonva">
    <konva-layer ref="konvaLayer">
      <konva-circle
        :config="configCircle"
        ref='handler'
        draggable="true"
        
      ></konva-circle>
    </konva-layer>
  </konva-stage>
  <v-btn depressed @click="checkElement">button</v-btn>
</div>
</template>

<script>
import Vue from 'vue';
import VueKonva from 'vue-konva';
Vue.use(VueKonva, {prefix: 'Konva'});

// @mouseover='onHandlerMouseOver'
//         @mouseout='onHandlerMouseOut'
//         @mousedown='onHandlerMouseDown'
//         @dragmove='onHandlerDragMove'
//         @dragend='onHandlerDragEnd'
//         @mouseup='onHandlerMouseUp'

export default {
  data() {
    return {
      configKonva: {
        width: 600,
        height: 600
      },
      configCircle: {
        x: 100,
        y: 100,
        radius: 70,
        fill: "red",
        stroke: "black",
        strokeWidth: 4
      },
      stage: null,
      canvas: null,
      context: null,
      drawingLayer: null,
      drawingScope: null,
      lastPointerPosition: {},
      localPos: {
        x: 0,
        y: 0
      },
      pos: null,
      isPaint: false
    };
  },
  methods: {
    checkElement() {
      console.log(this.$refs.konvaStage);
      console.log(this.$refs.konvaLayer);
    }
  }
}
</script>