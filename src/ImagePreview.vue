<template>
  <img :src="imageSrc" alt="Preview" ref="image" :class="[{ hide: removed }]" />
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  props: {
    imageId: {
      type: String,
      required: true,
    },
    imageSrc: {
      type: String,
      required: true,
    },
    removeImage: {
      type: Function,
      required: true,
    },
  },
  data() {
    return {
      removed: false,
    };
  },
  mounted() {
    this.$refs.image.addEventListener("dblclick", async () => {
      await this.removeImage(this.imageId)
        .then(() => {
          this.$emit("remove");
          this.removed = true;
        })
        .catch((err) => {
          console.log("Error removing image: ", err);
        });
    });
  },
});
</script>

<style scoped>
img {
  width: 100%;
  height: 50px;
  box-sizing: border-box;
  border: 1px solid grey;
  border-radius: 3px;
  cursor: pointer;
}
.hide {
  display: none;
}
</style>
