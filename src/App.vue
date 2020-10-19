<template>
  <div>
    <svg
      ref="canv"
      width="500"
      height="500"
      @pointerdown="down"
      @pointermove="move"
      @pointerup="up"
      @wheel="zoom"
    >
      <defs>
        <!-- arrowhead marker definition -->
        <marker
          id="arrow"
          viewBox="0 0 10 10"
          refX="5"
          refY="5"
          markerWidth="6"
          markerHeight="6"
          orient="auto-start-reverse"
        >
          <path d="M 0 0 L 10 5 L 0 10 z" />
        </marker>
      </defs>
      <line
        v-for="(v,index) in screenVectors"
        :key="index"
        :x1="center.x"
        :y1="center.y"
        :x2="v.x"
        :y2="v.y"
        marker-end="url(#arrow)"
      />
    </svg>
    <div>{{(mouse.x | 0)}}, {{(mouse.y | 0)}}</div>
    <div>ViewBox {{viewBox.left.toFixed(1)}}, {{viewBox.top.toFixed(1)}}, {{viewBox.right.toFixed(1)}}, {{viewBox.bottom.toFixed(1)}}</div>
    <div>Scale {{scale.toFixed(2)}}</div>
    <div>
      <button @click="setScale(1)">100%</button>
      <button @click="setScale(2)">200%</button>
    </div>
  </div>
</template>
<script>
import { Rect, Vec2, Transform } from "paintvec";

export default {
  data() {
    return {
      vectors: [{ x: 100, y: 100 }, { x: 50, y: -50 }],
      oldOffset: { x: 0, y: 0 },
      drag: null,
      mouse: {
        x: 0,
        y: 0
      },
      transformRaw: new Transform().members //no-op
    };
  },
  watch: {},
  computed: {
    scale() {
      return new Rect(new Vec2(0, 0), new Vec2(1, 1)).transform(this.transform)
        .width;
    },
    transform: {
      get() {
        return new Transform(...this.transformRaw);
      },
      set(newValue) {
        this.transformRaw = newValue.members;
      }
    },
    screenVectors() {
      return this.vectors.map(i =>
        new Vec2(i.x, i.y).transform(this.transform)
      );
    },
    invert() {
      return this.transform.invert();
    },
    viewBox() {
      return new Rect(
        new Vec2(0, 0).transform(this.transform.invert()),
        new Vec2(500, 500).transform(this.transform.invert()) //todo width, height
      );
    },
    center() {
      return new Vec2(0, 0).transform(this.transform);
    }
  },
  methods: {
    setScale(scale) {
      const viewOffset = this.viewBox.center.transform(this.transform);

      this.transform = this.transform
        .translate(viewOffset.neg)
        .scale(new Vec2(scale / this.scale))
        .translate(viewOffset);
    },
    zoom(ev) {
      //https://github.com/seanchas116/sketch-glass/blob/941972032c90e6fd59de997c2b27151c31b56d20/src/view/CanvasView.ts#L110
      const scale = Math.pow(0.5, ev.deltaY / 256);
      const center = new Vec2(ev.offsetX, ev.offsetY);
      this.transform = this.transform
        .translate(center.neg)
        .scale(new Vec2(scale, scale))
        .translate(center);
    },
    up() {
      this.drag = null;
    },
    down(e) {
      this.$refs.canv.setPointerCapture(e.pointerId);
      this.oldOffset = this.transformRaw;
      this.drag = {
        x: e.offsetX,
        y: e.offsetY
      };
    },
    move(e) {
      if (this.drag) {
        const startPos = new Vec2(this.drag.x, this.drag.y);
        const pos = new Vec2(e.offsetX, e.offsetY);
        const mouseDiff = pos.sub(startPos);
        this.transform = new Transform(...this.oldOffset).translate(mouseDiff);
      }
      this.mouse = new Vec2(e.offsetX, e.offsetY).transform(this.invert);
    }
  }
};
</script>
    <style>
body {
  background: #ccc;
}
svg {
  background: white;
}
line {
  stroke-width: 1;
  stroke: black;
}
</style>