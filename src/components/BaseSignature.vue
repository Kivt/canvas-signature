<template>
  <div class="signature-simple">
    <h2>Signature</h2>

    <div class="signature-simple__menu">
      <label class="signature-simple__menu-range">
        Width
        <div>
          <input
            v-model="lineWidth"
            type="range"
            min="1"
            max="72"
            @input="changeLineWidth"
          />
          <span>{{ lineWidth }}px</span>
        </div>
      </label>

      <label class="signature-simple__menu-colorpicker">
        <div>Color</div>
        <input v-model="lineColor" type="color" @input="changeColor" />
      </label>

      <button
        :disabled="!history.length"
        class="signature-simple__menu-button"
        @click="undo"
      >
        Undo
      </button>

      <button
        :disabled="!history.length"
        class="signature-simple__menu-button"
        @click="clearCanvas(true)"
      >
        Clear
      </button>

      <button
        :disabled="!previewSrc"
        class="signature-simple__menu-button"
        @click="downloadAsImage"
      >
        Download
      </button>
    </div>

    <h3>Draw</h3>

    <canvas
      ref="canvas"
      class="signature-simple__canvas"
      @mousedown="startDrawing"
      @mouseup="stopDrawing"
      @mousemove="drawLine"
      @touchstart="startDrawing"
      @touchend="stopDrawing"
      @touchmove.prevent.stop="onTouchMove"
      width="600"
      height="200"
    ></canvas>

    <h3>Preview</h3>

    <div v-show="previewSrc" ref="preview" class="signature-simple__preview">
      <img
        :src="previewSrc"
        alt="preview"
        class="signature-simple__preview-image"
      />
    </div>
    <div v-show="!previewSrc" style="min-height: 200px">Draw signature</div>
  </div>
</template>

<script>
export default {
  name: "SignatureSimple",

  data() {
    return {
      ctx: null,
      lineWidth: 2,
      lineColor: "#000000",
      isDrawing: false,
      prevX: 0,
      prevY: 0,
      previewSrc: null,
      history: [],
      points: [],
      resizeTimeout: null,
    };
  },

  mounted() {
    this.ctx = this.$refs.canvas.getContext("2d");
    this.setCtxConfig();
    this.addResizeListener();
    this.onWindowResize();
  },

  methods: {
    setCtxConfig() {
      this.ctx.lineCap = "round";
      this.ctx.lineJoin = "round";
      this.ctx.strokeStyle = this.lineColor;
      this.ctx.lineWidth = this.lineWidth;
    },

    startDrawing(e) {
      this.isDrawing = true;
      this.points = [];
      this.prevX = e.offsetX;
      this.prevY = e.offsetY;
      this.points.push({
        x: e.offsetX,
        y: e.offsetY,
        lineWidth: this.lineWidth,
        lineColor: this.lineColor,
      });
    },

    stopDrawing() {
      this.isDrawing = false;
      this.history.push(this.points);
      this.changePreview();
    },

    drawLine(e) {
      if (this.isDrawing) {
        const newX = e.offsetX;
        const newY = e.offsetY;
        this.ctx.beginPath();
        this.ctx.moveTo(this.prevX, this.prevY);
        this.ctx.lineTo(newX, newY);
        this.ctx.closePath();
        this.ctx.stroke();
        this.points.push({ x: e.offsetX, y: e.offsetY });
        this.prevX = newX;
        this.prevY = newY;
      }
    },

    undo() {
      if (this.history.length) {
        this.history.splice(-1, 1);
        this.drawHistory();
      }
    },

    drawHistory() {
      this.clearCanvas();
      this.history.forEach((path) => {
        this.ctx.beginPath();
        this.ctx.strokeStyle = path[0].lineColor;
        this.ctx.lineWidth = path[0].lineWidth;
        this.ctx.moveTo(path[0].x, path[0].y);
        for (let i = 1; i < path.length; i++) {
          this.ctx.strokeStyle = path[i].lineColor;
          this.ctx.lineWidth = path[i].lineWidth;
          this.ctx.lineTo(path[i].x, path[i].y);
        }
        this.ctx.stroke();
      });
      this.ctx.strokeStyle = this.lineColor;
      this.ctx.lineWidth = this.lineWidth;
      this.changePreview();
    },

    changeLineWidth() {
      this.ctx.lineWidth = this.lineWidth;
    },

    changeColor() {
      this.ctx.strokeStyle = this.lineColor;
    },

    onTouchMove(e) {
      const touch = e.touches[0];
      this.drawLine({
        offsetX: touch.clientX - this.$refs.canvas.offsetLeft,
        offsetY: touch.clientY - this.$refs.canvas.offsetTop,
      });
    },

    clearCanvas(clearHistory = false) {
      this.ctx.clearRect(
        0,
        0,
        this.$refs.canvas.width,
        this.$refs.canvas.height
      );
      this.previewSrc = null;

      if (clearHistory) {
        this.history = [];
        this.points = [];
      }
    },

    changePreview() {
      if (this.history.length) {
        this.previewSrc = this.$refs.canvas.toDataURL("image/png");
      } else {
        this.previewSrc = null;
      }
    },

    downloadAsImage() {
      if (!this.previewSrc) {
        return;
      }

      const downloadLink = document.createElement("a");
      downloadLink.download = "canvas_image.png";
      downloadLink.href = this.previewSrc;
      downloadLink.click();
    },

    addResizeListener() {
      window.addEventListener("resize", this.onWindowResize);
    },

    onWindowResize() {
      clearTimeout(this.resizeTimeout);
      // Wait for resize to finish
      this.resizeTimeout = setTimeout(() => {
        if (window.innerWidth < 620) {
          this.$refs.canvas.width = window.innerWidth - 40;
          this.$refs.preview.style.width = window.innerWidth - 40 + "px";
        } else {
          this.$refs.canvas.width = 600;
          this.$refs.preview.style.width = 600 + "px";
        }

        this.setCtxConfig();
        this.drawHistory();
      }, 200);
    },
  },
};
</script>

<style scoped>
.signature-simple {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-bottom: 2rem;
}

.signature-simple__menu {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  margin-bottom: 1rem;
}

.signature-simple__menu > * {
  margin: 1rem;
}

.signature-simple__menu-range {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.signature-simple__menu-range > div {
  display: flex;
  align-items: center;
}

.signature-simple__menu-range > div > span {
  margin-left: 0.25rem;
}

.signature-simple__menu-colorpicker {
  cursor: pointer;
}

.signature-simple__canvas {
  border: 1px solid #d3d3d3;
  border-radius: 0.75rem;
  cursor: crosshair;
}

.signature-simple__menu-button {
  border: 1px solid #d3d3d3;
  background-color: white;
  height: 27px;
  border-radius: 0.25rem;
  padding: 0.25rem 0.5rem;
  cursor: pointer;
}

.signature-simple__menu-button:not(:disabled):hover {
  background-color: whitesmoke;
}

.signature-simple__menu-button:disabled {
  cursor: not-allowed;
}

.signature-simple__preview {
  border: 1px solid #d3d3d3;
  border-radius: 0.75rem;
  height: 200px;
}

.signature-simple__preview-image {
  width: 100%;
  height: 100%;
}
</style>
