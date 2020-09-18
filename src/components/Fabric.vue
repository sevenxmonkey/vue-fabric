<template>
  <div class="whole">
    <div class="top-menu">
      <el-button
        @click="handleCropping"
        :plain="mode!=='cropping'"
        :disabled="!fimg"
        type="primary"
        size="mini"
      >Cropping</el-button>
      <el-button
        @click="handleErase"
        :disabled="!fimg"
        :plain="mode!=='erase'"
        type="primary"
        size="mini"
      >Erase</el-button>
      <el-button @click="resetEdit" :disabled="!fimg" plain size="mini">Reset</el-button>
      <el-button @click="importImage" size="mini">Submit</el-button>
    </div>
    <canvas class="canvas" id="c" width="600" height="600"></canvas>
    <el-upload class="image-uploader" action="/api/bg_remove" :before-upload="fileUpload">
      <el-button :plain="mode!=='init'" type="primary" size="mini">Add image</el-button>
    </el-upload>
    <img v-if="brushURL" :src="brushURL" width="600" height="600" />
  </div>
</template>

<script>
import { fabric } from "fabric";
import Jimp from "jimp";

export default {
  name: "Fabric",
  data() {
    return {
      fcanvas: null,
      fimg: null,
      frect: null,

      fbrushArr: [],

      file: null,

      mode: "init",

      rect: {
        from: {},
        to: {},
      },

      brushURL: "",
    };
  },
  mounted() {
    this.fcanvas = new fabric.Canvas("c", {
      selectable: false,
      defaultCursor: "crosshair",
    });
    this.init();
  },
  methods: {
    handleCropping() {
      this.fcanvas.isDrawingMode = false;
      if (this.mode === "cropping") {
        this.mode = "init";
      } else {
        this.mode = "cropping";
      }
    },
    handleErase() {
      if (this.mode === "erase") {
        this.mode = "init";
        this.fcanvas.isDrawingMode = false;
      } else {
        this.mode = "erase";
        this.fcanvas.isDrawingMode = true;

        this.fcanvas.freeDrawingBrush.color = "rgb(245, 245, 245)";
        // this.fcanvas.freeDrawingBrush.color = "black";
        this.fcanvas.freeDrawingBrush.width = 30;
      }
    },
    reset() {
      this.testURL = "";
      this.fcanvas.clear();
      this.init();
    },
    resetEdit() {
      this.testURL = "";
      this.fcanvas.clear();
      this.init();
      if (this.file) {
        this.fileUpload(this.file);
      }
    },

    init() {
      const self = this;
      this.frect = new fabric.Rect({
        fill: "white",
        selectable: false,
        absolutePositioned: true,
      });
      this.fcanvas.add(this.frect);
      this.fcanvas.bringToFront(this.frect);

      this.fcanvas.on("mouse:down", function (ev) {
        const point = self.fcanvas.getPointer(ev.e);
        self.fcanvas.forEachObject((o) => {
          o.selectable = false;
        });

        if (self.mode === "cropping") {
          self.rect.from = {
            x: point.x,
            y: point.y,
          };
        } else if (self.mode === "erase") {
          //TO DO
        }
      });

      this.fcanvas.on("mouse:up", function (ev) {
        const point = self.fcanvas.getPointer(ev.e);

        if (self.mode === "cropping") {
          self.rect.to = {
            x: point.x,
            y: point.y,
          };
          self.updateFRect();
          self.fcanvas.bringToFront(self.frect);
          self.clip();
        } else if (self.mode === "erase") {
          // TO DO
        }
      });

        this.fcanvas.on("path:created", function (options) {
          if (self.mode === "erase") {
            self.fbrushArr.push(options.path);
          }
        });

      this.fcanvas.forEachObject((o) => {
        o.selectable = false;
      });
    },

    fileUpload(file) {
      this.file = file;
      this.reset();
      //Handle file upload
      this.mode = "init";
      const self = this;
      var img = new Image();
      var reader = new FileReader();
      reader.onload = (e) => {
        img.src = e.target.result;
      };
      img.onload = () => {
        if (self.fimg) {
          self.fcanvas.remove(self.fimg);
        }
        const ajustObj = self.computeWidth(img.width, img.height);
        self.fimg = new fabric.Image(img, {
          defaultCursor: "crosshair",
          selectable: false,
        });
        self.fimg.scale(ajustObj.scale);
        self.fimg.set({
          top: ajustObj.top,
          left: ajustObj.left,
        });
        self.fcanvas.add(self.fimg);
      };
      reader.readAsDataURL(file);
    },
    computeWidth(w, h) {
      const obj = {
        height: 600,
        width: 600,
      };
      if (w < h) {
        obj.width = (600 * w) / h;
        obj.left = (600 - obj.width) / 2;
        obj.top = 0;
      } else {
        obj.width = 600;
        obj.left = 0;
        obj.top = 300 * (1 - h / w);
      }
      obj.scale = obj.width / w;
      return obj;
    },

    updateFRect() {
      const left = Math.min(this.rect.from.x, this.rect.to.x);
      const top = Math.min(this.rect.from.y, this.rect.to.y);
      const width = Math.abs(this.rect.from.x - this.rect.to.x);
      const height = Math.abs(this.rect.from.y - this.rect.to.y);

      if (width < 20 || height < 20) {
        return;
      }

      this.frect.left = left;
      this.frect.top = top;
      this.frect.width = width;
      this.frect.height = height;

      this.fcanvas.requestRenderAll();
    },

    clip() {
      if (this.fimg) {
        this.fcanvas.isDrawingMode = false;
        this.fimg.clipPath = this.frect;
        this.fcanvas.remove(this.frect);
      }
    },
    importImage() {
      this.adjustMask()
    },
    adjustMask() {
      const self = this;
      const backgroundRect = new fabric.Rect({
        fill: "black",
        width: 600,
        height: 600,
        absolutePositioned: true,
      });
      const newCanvas = new fabric.Canvas(null, {
        width: 600,
        height: 600,
        selectable: false,
        absolutePositioned: true
      });
      newCanvas.add(backgroundRect);
      newCanvas.add(this.frect);
      self.fbrushArr.forEach((brush) => {
        brush.stroke = "black"
        // console.log(brush);
        newCanvas.add(brush);
      })
      
      const alphaMask = newCanvas.toDataURL();
      Jimp.read(alphaMask, (err, mask) => {
        if(err) throw err;
        self.genPng(mask);
      })
      
    },
    genPng(mask) {
      const self = this;
      // this.fcanvas.clipPath = null;
      this.fcanvas.remove(this.frect)
      const imgIncode = this.fcanvas.toDataURL();
      Jimp.read(imgIncode, (err, image) => {
        if (err) throw err;
        image
          .mask(mask, 0, 0)
          .getBase64Async(Jimp.MIME_PNG)
          .then((data) => {
            self.brushURL = data;
            self.reset();
          });
      });
    },
  },
};
</script>

<style scoped>
.canvas {
  cursor: crosshair;
  border: 1px dashed rgb(165, 165, 165);
  background-color: rgb(245, 245, 245);
  /* background-color: black; */
  border-radius: 5px;
}
.image-uploader {
  margin-right: 10px;
  margin-bottom: 10px;
}
.top-menu {
  margin-bottom: 10px;
}

</style>