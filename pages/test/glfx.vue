<template>
  <div>
    <div>
      <div ref="container" class="canvas_inline">
        <canvas width="600" height="600" ref="canvas" />
      </div>
      <div class="canvas_inline canvas_margin">
        <canvas ref="result" width="400" height="566" />
      </div>
    </div>
    <div>
      <span>{{ nowCorner }}</span>
    </div>
  </div>
</template>

<script>
import { Component, Vue, Watch } from 'nuxt-property-decorator';
import Konva from 'konva';

import * as fx from 'glfx';

@Component
export default class KonvaTop extends Vue {
  coords = [
   [202, 123],
   [544, 103],
   [45, 432],
   [520, 498]
  ];

  width = 600;
  height = 600;
  stage = null;
  canvas = null;
  drawingLayer = null;
  drawingScope = null;

  imageObj = null;
  backgroundLayer = null;
  backgroundImageScope = null;

  line = null;
  corners = [];

  nowCorner = '';

  // perspective
  perspectiveNubs = [];
  baseCanvas = null;
  image = null;
  texture = null;
  resCanvas = null;

  getNowCorner() {
    this.nowCorner = `{ a:{${this.coords[0][0]},${this.coords[0][1]}} b:{${this.coords[1][0]},${this.coords[1][1]}} c:{${this.coords[2][0]},${this.coords[2][1]}} d:{${this.coords[3][0]},${this.coords[3][1]}} }`
  }

  mounted() {
    var container = this.$refs.container;
    this.stage = new Konva.Stage({
      container,
      width: this.width,
      height: this.height 
    });
    this.drawingLayer = new Konva.Layer();
    this.stage.add(this.drawingLayer);

    this.canvas = this.$refs.canvas;
    this.drawingScope = new Konva.Image({
      image: this.canvas,
      stroke: 'black'
    });
    this.drawingLayer.add(this.drawingScope);
    this.stage.draw();

    /** 追加 */
    this.imageObj = new Image();
    this.imageObj.addEventListener('load', this.imageOnload);
    this.imageObj.src = require('../../static/IMG_0866.JPG');
  }

  imageOnload() {
    // 背景レイヤ
    this.backgroundLayer = new Konva.Layer()

    // 背景イメージ（xとy座標はthis.drawingScopeと同じにする）
    this.backgroundImageScope = new Konva.Image({
      image: this.imageObj,
      width: this.canvas.width,
      height: this.canvas.height
    })

    // 背景レイヤに背景イメージを追加
    this.backgroundLayer.add(this.backgroundImageScope)
    this.stage.add(this.backgroundLayer)

    // 背景イメージを最背面に移動。これをしないとペンの描画が画像の下に潜ってしまう。
    this.backgroundLayer.moveToBottom();

    this.drawCircle();
    this.initDrawLine();
    this.initPerspectiveImage();
  }

  drawCircle() {
    for (let i = 0; i < 4; i++){
      this.corners[i] = new Konva.Circle({
        radius: 10,
        x: this.coords[i][0],
        y: this.coords[i][1],
        // fill: 'red',
        stroke: 'red',
        strokeWidth: 2,
        draggable: true
      });

      switch(i) {
        case 0:
          this.corners[i].on('dragmove', this.moveCircle1);
          break;
        case 1:
          this.corners[i].on('dragmove', this.moveCircle2);
          break;
        case 2:
          this.corners[i].on('dragmove', this.moveCircle3);
          break;
        case 3:
          this.corners[i].on('dragmove', this.moveCircle4);
          break;
        default: break;
      }
      
      this.drawingLayer.add(this.corners[i]);
    }

    this.stage.add(this.drawingLayer);

    console.log(this.corners[0].x());
    this.getNowCorner();
  }

  moveCircle1(node) {
    this.moveCircleBase(node, 0);
  };

  moveCircle2(node) {
    this.moveCircleBase(node, 1);
  }

  moveCircle3(node) {
    this.moveCircleBase(node, 2);
  }

  moveCircle4(node) {
    this.moveCircleBase(node, 3);
  }

  moveCircleBase(node, i) {
    this.coords[i][0] = node.target.attrs.x;
    this.coords[i][1] = node.target.attrs.y;
    this.getNowCorner();
    this.drawLine();
    this.updatePerspectiveImage();
  }

  setPerspectiveNubs() {
    this.perspectiveNubs = [
      this.coords[0][0] * 2, this.coords[0][1] * 2,
      this.coords[1][0] * 2, this.coords[1][1] * 2,
      this.coords[2][0] * 2, this.coords[2][1] * 2,
      this.coords[3][0] * 2, this.coords[3][1] * 2,
    ];
  }

  initDrawLine() {
    this.line = new Konva.Line({
      points: [
        this.coords[0][0], this.coords[0][1],
        this.coords[1][0], this.coords[1][1],
        this.coords[3][0], this.coords[3][1],
        this.coords[2][0], this.coords[2][1],
        this.coords[0][0], this.coords[0][1],
      ],
      stroke: 'red'
    });

    this.drawingLayer.add(this.line);

    this.stage.add(this.drawingLayer);
  }

  drawLine() {
    // update coordinates
    this.line.attrs.points = [
      this.coords[0][0], this.coords[0][1],
      this.coords[1][0], this.coords[1][1],
      this.coords[3][0], this.coords[3][1],
      this.coords[2][0], this.coords[2][1],
      this.coords[0][0], this.coords[0][1],
    ];

    this.drawingLayer.add(this.line);
    this.stage.add(this.drawingLayer);
  }

  initPerspectiveImage() {
    try {
      this.baseCanvas = fx.canvas();
    } catch (e) {
      alert(e);
      return;
    }

    this.image = this.backgroundLayer.canvas._canvas;
    this.texture = this.baseCanvas.texture(this.image);
    this.resCanvas = this.$refs.result.getContext('2d');

    this.updatePerspectiveImage();

  }

  updatePerspectiveImage() {

    this.setPerspectiveNubs();
    const before = this.perspectiveNubs;

    const after = [ 0, 0, 400, 0, 0, 566, 400, 566];
    this.baseCanvas
      .draw(this.texture)
      .perspective(before, after)
      .update()

    // ImageDataを取得して、別canvasに描画
    const data = this.baseCanvas.getPixelArray();
    const img = new ImageData(new Uint8ClampedArray(data), this.width*2, this.height*2);
    this.resCanvas.putImageData(img, 0, 0);
  }
}
</script>

<style scoped>
.canvas_inline {
  display: inline-block;
  vertical-align: top;
}

.canvas_margin {
  margin-left: 10px;
}
</style>