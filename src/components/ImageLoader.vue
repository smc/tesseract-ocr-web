<template>
  <div
    class="drop"
    :class="classes"
    @dragover.prevent="dragOver"
    @dragleave.prevent="dragLeave"
    @drop.prevent="drop($event)"
  >
    <img :src="imageSource" v-if="imageSource" id="ocr-img" />
    <h1 v-if="wrongFile">Wrong file type</h1>
    <h1 v-if="!imageSource && !isDragging && !wrongFile">Drop, paste or upload an image</h1>
    <input
      type="file"
      id="uploadimage"
      :accept="'image/*'"
      @change="requestUploadFile"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, computed, reactive, toRefs } from "vue";

export default defineComponent({
  name: "ImageLoader",
  setup() {
    const state = reactive({
      isDragging: false,
      wrongFile: false,
      imageSource: null,
    });
    const classes = computed(() => state.isDragging);
    const requestUploadFile = () => {
      const src = document.querySelector("#uploadimage");
      drop({ dataTransfer: src });
    };
    const dragOver = () => {
      state.isDragging = true;
    };
    const dragLeave = () => {
      state.isDragging = false;
    };
    const drop = (e: any) => {
      const files: FileList | null | undefined = e.dataTransfer?.files;
      state.wrongFile = false;
      // allows only 1 file
      if (files?.length === 1) {
        let file: File = files[0];
        // allows image only
        if (file.type.indexOf("image/") >= 0) {
          var reader = new FileReader();
          reader.onload = (f) => {
            state.imageSource = f.target?.result;
            state.isDragging = false;
          };
          reader.readAsDataURL(file);
        } else {
          state.wrongFile = true;
          state.imageSource = null;
          state.isDragging = false;
        }
      }
    };

    document.onpaste = (event) => {
        var items = event.clipboardData?.items;
        for (let index in items) {
            var item = items[index];
            if (item.kind === 'file') {
                const blob = item.getAsFile();
                const reader = new FileReader();
                reader.onload = (pasteEvent) => {
                    state.imageSource = pasteEvent.target?.result;
                };
                reader.readAsDataURL(blob);
            }
        }
    };

    return {
      classes,
      ...toRefs(state),
      dragOver,
      dragLeave,
      drop,
      requestUploadFile
    };
  },
});
</script>

<style>
.drop {
  width: 100%;
  height: 100%;
  background-color: #eee;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.2s ease-in-out;
  font-family: sans-serif;
  padding: 4px;
  flex-direction: column;
}
.isDragging {
  background-color: #999;
  border-color: #fff;
}
img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}
</style>
