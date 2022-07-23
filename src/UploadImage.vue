<template>
  <div class="drop-area" :class="[{ highlight: draggedOver }]" ref="dropArea">
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css"
    />
    <i class="fa fa-camera" :class="[{ hide: numPreviewImages }]"></i>
    <div class="drop-area-text" :class="[{ hide: numPreviewImages }]">
      <p>Drop image here</p>
      <p>Or</p>
    </div>
    <label for="file-upload" class="file-upload-button"> Upload </label>
    <input id="file-upload" type="file" multiple @change="uploadFiles" />
    <div class="drop-area-image">
      <image-preview
        v-for="(image, index) in previewImageList"
        :imageId="image.imageId"
        :imageSrc="image.imageSrc"
        :removeImage="removeImage"
        :key="index"
        @remove="removePreviewImage(image.imageId)"
      >
      </image-preview>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

import ImagePreview from "./ImagePreview.vue";

const MAX_NUM_IMAGES = 9;

export default defineComponent({
  props: {
    processImage: {
      type: Function,
      required: true,
    },
    removeImage: {
      type: Function,
      required: true,
    },
  },
  components: {
    ImagePreview,
  },
  data() {
    return {
      draggedOver: false,
      numPreviewImages: 0,
      numProcessingImage: 0,
      previewImageList: [],
      imageUrlMap: {},
    };
  },
  methods: {
    customPreventDefault(event) {
      event.preventDefault();
      event.stopPropagation();
    },
    previewImage(image, imageId) {
      const fileReader = new FileReader();
      fileReader.readAsDataURL(image);
      fileReader.onloadend = () => {
        this.previewImageList.push({
          imageSrc: fileReader.result,
          imageId: imageId,
        });
        this.numPreviewImages += 1;
        this.numProcessingImage -= 1;
        if (this.numProcessingImage == 0) {
          // Finish processing and displaying all images
          this.emitUrlList();
        }
      };
    },
    async handleImage(image) {
      const { imageId, imageUrl } = await this.processImage(image).catch(
        (err) => {
          console.log("Error processing image: ", err);
          return {
            imageId: undefined,
            imageUrl: undefined,
          };
        }
      );
      if (!imageId || !imageUrl) {
        return;
      }
      this.imageUrlMap[imageId] = imageUrl;
      this.previewImage(image, imageId);
    },
    processImagesFromFiles(files) {
      const uploadedImages = [...files].filter((file) => {
        return file.type.startsWith("image");
      });
      this.numProcessingImage = Math.min(
        uploadedImages.length,
        MAX_NUM_IMAGES - this.numPreviewImages
      );
      const processedImages = uploadedImages.slice(0, this.numProcessingImage);
      processedImages.forEach(this.handleImage);
    },
    uploadFiles(event) {
      if (this.numPreviewImages >= MAX_NUM_IMAGES) {
        return;
      }
      const uploadedFiles = event.target.files;
      if (!uploadedFiles) {
        return;
      }
      this.processImagesFromFiles(uploadedFiles);
    },
    dropFiles(event) {
      if (this.numPreviewImages >= MAX_NUM_IMAGES) {
        return;
      }
      if (!event.dataTransfer) {
        return;
      }
      if (!event.dataTransfer.files.length) {
        return;
      }
      this.draggedOver = false;
      this.processImagesFromFiles(event.dataTransfer.files);
    },
    removePreviewImage(imageId) {
      delete this.imageUrlMap[imageId];
      this.emitUrlList();
      this.numPreviewImages -= 1;
    },
    emitUrlList() {
      const updatedUrlList = Object.keys(this.imageUrlMap).map(
        (key) => this.imageUrlMap[key]
      );
      this.$emit("update-urls", updatedUrlList);
    },
  },
  created() {
    ["dragover", "drop"].forEach((eventName) => {
      document.addEventListener(eventName, this.customPreventDefault);
    });
  },
  mounted() {
    // Prevent browser from opening dragged images in a new tab
    // and the drop event can be handled by our callback
    ["dragover", "drop"].forEach((eventName) => {
      this.$refs.dropArea.addEventListener(
        eventName,
        this.customPreventDefault
      );
    });

    this.$refs.dropArea.addEventListener("drop", this.dropFiles);
    // Show / hide highlight animation on drop area when files are dragged in / out
    this.$refs.dropArea.addEventListener("dragenter", () => {
      this.draggedOver = true;
    });
    this.$refs.dropArea.addEventListener("dragleave", (event) => {
      const dropAreaPosition = this.$refs.dropArea.getBoundingClientRect();
      // Only disable highlight animation when dragleave events occurs outside of drop area.
      // By default, dragleave event occurs when dragging into child elements (still inside drop area).
      const surplusGap = 10; //px
      if (
        event.clientX >= dropAreaPosition.left + surplusGap &&
        event.clientX <= dropAreaPosition.right - surplusGap &&
        event.clientY >= dropAreaPosition.top + surplusGap &&
        event.clientY <= dropAreaPosition.bottom - surplusGap
      ) {
        return;
      }
      this.draggedOver = false;
    });
  },
  beforeUnmount() {
    ["dragover", "drop"].forEach((eventName) => {
      document.removeEventListener(eventName, this.customPreventDefault);
    });
  },
});
</script>

<style scoped>
.drop-area {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 5px;
  height: 240px;
  background: #f7f7f7;
  border-radius: 4px;
  border: 1px dashed grey;
  max-width: 300px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
}
.drop-area :where(.fa-camera, .drop-area-text) {
  pointer-events: none;
}
.drop-area-image {
  position: absolute;
  top: 0;
  left: 0;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  width: 100%;
  padding: 15px;
  box-sizing: border-box;
  justify-items: center;
  align-items: center;
}
.drop-area > .fa-camera {
  font-size: 30px;
  color: #c2c2c2;
}
.drop-area-text {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 3px;
  position: relative;
  top: 7px;
  font-size: 15px;
  color: grey;
}
.drop-area-text > p {
  margin: 0;
}
.drop-area-text > p:nth-child(2) {
  font-size: 14px;
}
.drop-area-text > label {
  font-size: 16px;
}
.file-upload-button {
  position: absolute;
  bottom: 10px;
  display: block;
  width: 100px;
  background: white;
  cursor: pointer;
  border: 1px solid #bdbdbd;
  border-radius: 4px;
  box-sizing: border-box;
  padding: 3px;
  color: grey;
}
input[type="file"] {
  display: none;
}
.hide {
  display: none;
}
.highlight {
  box-shadow: 0 0 7px 5px lightblue;
}
</style>
