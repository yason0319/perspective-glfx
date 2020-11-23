<template>
   <div>
    <v-col
      class="d-flex"
      cols="12"
      sm="6"
    >
      <v-select
        v-model="selectedItem"
        item-text="label"
        item-value="value"
        :items="items"
        return-object
      ></v-select>
      <v-btn
        depressed
      >
        color
      </v-btn>
      <v-btn
        depressed
        :disabled="undoState"
        @click="undo"
      >
        undo
      </v-btn>
      <v-btn
        depressed
        :disabled="redoState"
        @click="redo"
      >
        redo
      </v-btn>
    </v-col>
    <div>{{selectedItem.value}}</div>
    <div>
      <div ref="container" class="canvas_inline">
        <canvas id="baseImg" :width="width/2" :height="height/2" ref="canvas" />
      </div>
      <div class="canvas_inline">
        <canvas id="result" width="300" height="300" />
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
  items = [
    { value: 'brush', label: '手書き' },
    { value: 'eraser', label: '消しゴム' },
    { value: 'line', label: '直線' }
  ];
  selectedItem = this.items[0];

  a = [
   [175, 156],
   [496, 55],
   [161, 279],
   [504, 330]
  ]

  perspectiveNubs = [];
  zoom = [ 100, 100, 500, 100, 100, 500, 500, 500 ];

  mode = {
    type: String,
    default: 'line'
  };
  brushColor = {
    type: String,
    default: 'black'
  };

  width = window.innerWidth;
  height = window.innerWidth;
  stage = null;
  canvas = null;
  context = null;
  drawingLayer = null;
  drawingScope = null;
  lastPointerPosition = {};
  localPos = {
    x: 0,
    y: 0
  };
  pos = null;
  isPaint = false;

  imageObj = null;
  backgroundLayer = null;
  backgroundImageScope = null;

  line = null;

  corners = [];

  nowCorner = '';

  canvas = null;
  image = null;
  texture = null;
  resCanvas = null;

  /** 追加 */
  undoDataStack = []; // 元に戻す配列
  redoDataStack = []; // やり直し配列
  
  get undoState() {
    return this.undoDataStack.length > 0 ? false : true;
  }

  get redoState() {
    return this.redoDataStack.length > 0 ? false : true;
  }

  getNowCorner() {
    this.nowCorner = `{ a:{${this.a[0][0]},${this.a[0][1]}} b:{${this.a[1][0]},${this.a[1][1]}} c:{${this.a[2][0]},${this.a[2][1]}} d:{${this.a[3][0]},${this.a[3][1]}} }`
  }

  mounted() {
    var container = this.$refs.container;
    console.log(this.width);
    console.log(this.height);
    this.stage = new Konva.Stage({
      container,
      width: this.width,
      height: this.height 
    })
    this.drawingLayer = new Konva.Layer();
    this.stage.add(this.drawingLayer);

    this.canvas = this.$refs.canvas;
    this.drawingScope = new Konva.Image({
      image: this.canvas,
      x: this.width / 4,
      y: 5,
      stroke: 'black'
    });
    this.drawingLayer.add(this.drawingScope);
    this.stage.draw();

    /** 追加 */
    // this.imageObj = new Image()
    // this.imageObj.addEventListener('load', this.imageOnload)
    // this.imageObj.src = require('../../static/postman.png');

    this.context = this.canvas.getContext('2d')
    this.context.strokeStyle = this.brushColor
    this.context.lineJoin = 'round'
    this.context.lineWidth = 5

    // イベント追加
    this.drawingScope.on('mousedown', this.mousedown)
    this.stage.addEventListener('mouseup', this.mouseup)
    this.stage.addEventListener('mousemove', this.mousemove)
    this.drawingScope.on('touchstart', this.mousedown)
    this.stage.addEventListener('touchend', this.mouseup)
    this.stage.addEventListener('touchmove', this.mousemove)
  }

  imageOnload() {
    // 背景レイヤ
    this.backgroundLayer = new Konva.Layer()

    // 背景イメージ（xとy座標はthis.drawingScopeと同じにする）
    this.backgroundImageScope = new Konva.Image({
      image: this.imageObj,
      // x: this.width / 4,
      // y: 5,
      width: this.canvas.width,
      height: this.canvas.height
    })

    // 背景レイヤに背景イメージを追加
    this.backgroundLayer.add(this.backgroundImageScope)
    this.stage.add(this.backgroundLayer)

    // 背景イメージを最背面に移動。これをしないとペンの描画が画像の下に潜ってしまう。
    this.backgroundLayer.moveToBottom();

  }

  mousedown() {
    this.isPaint = true

    // マウスダウン時の座標を取得しておく
    this.lastPointerPosition = this.stage.getPointerPosition();

    // 戻す配列に描画前のキャンバスの状態を保存
    this.undoDataStack.push(this.context.getImageData(0, 0, this.canvas.width, this.canvas.height))
  };
  mouseup() {
    this.isPaint = false
  };
  mousemove() {
    if (!this.isPaint) {
      return;
    }
    // ペンモード時
    if (this.isTargetMode('brush') || this.isTargetMode('line')) {
      this.context.globalCompositeOperation = 'source-over';
    }
    // 消しゴムモード時
    if (this.isTargetMode('eraser')) {
      this.context.globalCompositeOperation = 'destination-out';
    }

    this.context.beginPath()

    this.localPos.x = this.lastPointerPosition.x - this.drawingScope.x()
    this.localPos.y = this.lastPointerPosition.y - this.drawingScope.y()

    // 描画開始座標を指定する
    this.context.moveTo(this.localPos.x, this.localPos.y)

    this.pos = this.stage.getPointerPosition()
    this.localPos.x = this.pos.x - this.drawingScope.x()
    this.localPos.y = this.pos.y - this.drawingScope.y()

    // 描画開始座標から、lineToに指定された座標まで描画する
    this.context.lineTo(this.localPos.x, this.localPos.y)
    this.context.closePath()
    this.context.stroke()
    this.drawingLayer.draw()

    this.lastPointerPosition = this.pos
  };

  // 現在のモードが指定されたモードと一致するかどうか
  isTargetMode(targetMode) {
    return this.selectedItem.value === targetMode
  };

  /** 元に戻す */
  undo() {
    // 戻す配列が空の場合は何もしない
    if (this.undoDataStack.length <= 0) {
      return
    }

    // 戻す前の状態をやり直し配列に保存
    this.redoDataStack.push(this.context.getImageData(0, 0, this.canvas.width, this.canvas.height))

    // 元に戻す（戻す配列からキャンバスの状態を取り出して反映）
    this.context.putImageData(this.undoDataStack.pop(), 0, 0);
    this.drawingLayer.draw();
  };

  /** やり直す */
  redo() {
    // やり直し配列が空の場合は何もしない
    if (this.redoDataStack.length <= 0) {
      return
    }

    // やり直す前の状態を戻す配列に保存
    this.undoDataStack.push(this.context.getImageData(0, 0, this.canvas.width, this.canvas.height))

    // やり直す（やり直し配列からキャンバスの状態を取り出して反映）
    this.context.putImageData(this.redoDataStack.pop(), 0, 0)
    this.drawingLayer.draw()
  }

}
</script>

<style scoped>
.canvas_inline {
  display: inline-block; 
}
</style>