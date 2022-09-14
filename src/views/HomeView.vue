<script setup lang="ts">
import { Codemirror } from "vue-codemirror";
import { javascript } from "@codemirror/lang-javascript";
import { oneDark } from "@codemirror/theme-one-dark";
import { ref, shallowRef, unref } from "vue";
import { getPixels, savePixels } from "ndarray-pixels";
import ndarray from "ndarray";
const code = ref(`for (let i = 0; i < pixels.value.shape[0]; ++i) {
  for (let j = 0; j < pixels.value.shape[1]; ++j) {
    pixels.value.set(i, j, 1, 255);
  }
}`);
const extensions = [javascript(), oneDark];

// Codemirror EditorView instance ref
const view = shallowRef();
const handleReady = (payload: any) => {
  view.value = payload.view;
};

const log = console.log;

const uploadedFileInput = ref<InstanceType<typeof HTMLInputElement> | null>();
const previewSrc = ref<string>("");

const outputSrc = ref<string>("");

const uploadedBytes = ref<Uint8Array | null>(null);
const pixels = ref<ndarray.NdArray<
  ndarray.TypedArray | ndarray.GenericArray<number> | number[]
> | null>(null);
var alteredPixels = ref< ndarray.NdArray<
  ndarray.TypedArray | ndarray.GenericArray<number> | number[]
> | null>(null);
const transformedBytes = ref<Uint8Array | null>(null);



// function transpose(matrix) {
//   return matrix[0].map((col, i) => matrix.map(row => row[i]));
// }

function readUploadedImagePixels(file: File) {
  file.arrayBuffer().then(async (buffer) => {
    uploadedBytes.value = new Uint8Array(buffer);
    pixels.value = await getPixels(uploadedBytes.value, "image/png");
    // for (let i = 0; i < pixels.value.shape[0]; ++i) {
    //   for (let j = 0; j < pixels.value.shape[1]; ++j) {
    //     pixels.value.set(i, j, 1, 255);
    //     pixels.value.data
    //   }
    // }

    // var newArray = [];
    // for(var i = 0; i < array.length; i++){
    //     newArray.push([]);
    // };

    // for(var i = 0; i < array.length; i++){
    //     for(var j = 0; j < arrayLength; j++){
    //         newArray[j].push(array[i][j]);
    //     };
    // };

//pixels.value.get()
    log(pixels.value);

    alteredPixels.value = ndarray(
      new Uint8Array(pixels.value.data),
      unref(pixels.value.shape),
      unref(pixels.value.stride),
      unref(pixels.value.offset)
    );
    // write
    transformedBytes.value = await savePixels(alteredPixels.value!, "image/png");
    outputSrc.value = URL.createObjectURL(
      new Blob([transformedBytes.value], { type: "image/png" } /* (1) */)
    );
  });
}

function previewImage() {
  const file: FileList | null | undefined = uploadedFileInput.value?.files;
  if (file) {
    const loadedFile = file[0];
    previewSrc.value = URL.createObjectURL(loadedFile);
    readUploadedImagePixels(loadedFile);
  }
}


async function apply() {
  // modify
  if (pixels.value) {
    alteredPixels.value = ndarray(
      new Uint8Array(pixels.value.data),
      unref(pixels.value.shape),
      unref(pixels.value.stride),
      unref(pixels.value.offset)
    );
    window.eval.call(window,'(function (pixels) {'+code.value+'})')(alteredPixels);

    
  
    transformedBytes.value = await savePixels(alteredPixels.value!, "image/png");
    outputSrc.value = URL.createObjectURL(
      new Blob([transformedBytes.value], { type: "image/png" } /* (1) */)
    );
  }
}
</script>

<template>
  <main>
    <div class="upload-wrapper">
      <v-file-input
        ref="uploadedFileInput"
        accept="image/*"
        label="Carregue sua Imagem aqui."
        @change="previewImage"
      ></v-file-input>
    </div>
    <div class="main-wrapper">
      <div class="uploaded-image">
        <v-img
          :src="previewSrc"
          max-height="500"
          contain
          class="bg-grey-lighten-2"
        ></v-img>
      </div>
      <div class="code-wrapper">
        <codemirror
          v-model="code"
          placeholder="Aplique as transformações na imagem aqui..."
          :style="{ height: '100%', minHeight: '300px' }"
          :autofocus="true"
          :indent-with-tab="true"
          :tab-size="2"
          :extensions="extensions"
          @ready="handleReady"
          @change="log('change', $event)"
          @focus="log('focus', $event)"
          @blur="log('blur', $event)"
        />
        <v-btn class="apply-button" @click="apply">Aplicar</v-btn>
        <!-- <button @click="apply" class="apply-button">Aplicar</button> -->
      </div>
    </div>
    <div class="output-wrapper" v-if="outputSrc != ''">
      <h5>Resultado da transformação:</h5>
      <v-img
        :src="outputSrc"
        max-height="500"
        contain
        class="bg-grey-lighten-2"
      ></v-img>
    </div>
  </main>
</template>

<style scoped>
.upload-wrapper {
  display: flex;
  flex-flow: row nowrap;
  padding-top: 15px;
}
.main-wrapper {
  display: flex;
  flex-flow: row nowrap;
}
.uploaded-image {
  position: relative;
  flex: 1;
  display: flex;
  align-items: center;
  min-height: 500px;
  padding: 5px;
}
.code-wrapper {
  padding: 5px;
  position: relative;
  flex: 1;
}
.apply-button {
  position: absolute;
  bottom: 10px;
  z-index: 999;
  right: 10px;
}
</style>
