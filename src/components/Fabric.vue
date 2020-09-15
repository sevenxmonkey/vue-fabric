<template>
  <div>
    <div>Fabric.js</div>
    <canvas class="canvas" id="c" width="600" height="600"></canvas>
    <el-upload
      class="avatar-uploader"
      action="https://jsonplaceholder.typicode.com/posts/"
      :before-upload="beforeAvatarUpload"
    >
      <img v-if="imageUrl" :src="imageUrl" class="avatar" />
      <i v-else class="el-icon-plus avatar-uploader-icon"></i>
    </el-upload>
  </div>
</template>

<script>
import { fabric } from "fabric";

export default {
  name: "Fabric",
  data() {
    return {
      fcanvas: null,
      imageUrl: "",
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.fcanvas = new fabric.StaticCanvas("c", {
        selection: false,
      });

      // create a rectangle with angle=45
      var rect = new fabric.Rect({
        left: 100,
        top: 100,
        fill: "red",
        width: 20,
        height: 20,
      });

      this.fcanvas.add(rect);

      rect.set("fill", "red");
      rect.set({ strokeWidth: 5, stroke: "rgba(100,200,200,0.5)" });
      rect.set("angle", 15).set("flipY", true);
    },
    beforeAvatarUpload(file) {
      this.imageUrl = URL.createObjectURL(file.raw);
    },
  },
};
</script>

<style scoped>
.canvas {
  border: 1px solid firebrick;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>