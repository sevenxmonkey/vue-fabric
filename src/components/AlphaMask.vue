<template>
  <div class="background">
    <!-- origin image -->
    <div>
      <el-upload class="image-uploader" action="/test" :before-upload="fileUpload">
        <el-button type="primary" size="mini">Add image</el-button>
      </el-upload>
      <img v-if="imageURL" :src="imageURL" height="600" />
    </div>

    <!-- mask image -->
    <div>
      <el-upload class="image-uploader" action="/test" :before-upload="maskUpload">
        <el-button type="primary" size="mini">Add mask</el-button>
      </el-upload>
      <img v-if="maskURL" :src="maskURL" height="600" />
    </div>
  </div>
</template>

<script>
import Jimp from "jimp";

export default {
  name: "AlphaMask",
  data() {
    return {
      file: null,
      imageURL: null,
      maskURL: null,
    };
  },
  mounted() {},
  methods: {
    fileUpload(file) {
      const self = this;
      this.file = file;
      var img = new Image();
      var reader = new FileReader();
      reader.onload = (e) => {
        img.src = e.target.result;
      };
      img.onload = () => {
        self.imageURL = img.src;
      };
      reader.readAsDataURL(file);
    },
    maskUpload(file) {
      const self = this;
      this.file = file;
      var img = new Image();
      var reader = new FileReader();
      reader.onload = (e) => {
        img.src = e.target.result;
      };
      img.onload = () => {
        self.maskURL = img.src;
        self.test();
      };
      reader.readAsDataURL(file);
    },
    test() {
      // console.log(data);
      const self = this;
      Jimp.read(self.imageURL, (err, image1) => {
        if (err) throw err;
        Jimp.read(self.maskURL, (err, image2) => {
          if (err) throw err;
          image1
            .mask(image2, 0, 0)
            // .crop(30,30,300,300)
            .getBase64Async(Jimp.MIME_PNG)
            .then((data) => {
              // console.log(data)
              self.imageURL = data;
            });
        });
      });
    },
  },
};
</script>

<style scoped>
.background {
  background-color: aqua;
}
</style>