<script setup lang="ts">
import * as monaco from "monaco-editor";
import editorWorker from "monaco-editor/esm/vs/editor/editor.worker?worker";
import jsonWorker from "monaco-editor/esm/vs/language/json/json.worker?worker";
import cssWorker from "monaco-editor/esm/vs/language/css/css.worker?worker";
import htmlWorker from "monaco-editor/esm/vs/language/html/html.worker?worker";
import tsWorker from "monaco-editor/esm/vs/language/typescript/ts.worker?worker";
import { convertCodeToSvg } from "js2flowchart";
import { ElImage, ElMessage } from "element-plus";
import { ref, onMounted, computed, nextTick } from "vue";
const image = ref("");
const previews = computed(() => [image.value]);
const editorRf = ref<HTMLElement | null>(null);
onMounted(() => {
  // 创建一个js编辑器
  nextTick(() => {
    self.MonacoEnvironment = {
      getWorker(_, label) {
        if (label === "json") {
          return new jsonWorker();
        }
        if (label === "css" || label === "scss" || label === "less") {
          return new cssWorker();
        }
        if (label === "html" || label === "handlebars" || label === "razor") {
          return new htmlWorker();
        }
        if (label === "typescript" || label === "javascript") {
          return new tsWorker();
        }
        return new editorWorker();
      },
    };
    const editor = monaco.editor.create(editorRf.value!, {
      value: `var quickSort = function(arr) {

　　if (arr.length <= 1) { return arr; }

　　var pivotIndex = Math.floor(arr.length / 2);

　　var pivot = arr.splice(pivotIndex, 1)[0];

　　var left = [];

　　var right = [];

　　for (var i = 0; i < arr.length; i++){

　　　　if (arr[i] < pivot) {

　　　　　　left.push(arr[i]);

　　　　} else {

　　　　　　right.push(arr[i]);

　　　　}

　　}

　　return quickSort(left).concat([pivot], quickSort(right));

}; `,
      language: "javascript",
      theme: "vs-dark",
      automaticLayout: true,
    });
    var KM = monaco.KeyMod;
    var KC = monaco.KeyCode;
    editor.addCommand(KM.chord(KM.CtrlCmd | KC.KeyS, 0), async () => {
      try {
        const code = editor.getValue();
        const svg = convertCodeToSvg(code);
        const outFilePath = await window.preload?.saveFile(svg);
        image.value = `file://${
          outFilePath + "?t=" + new Date().getUTCMilliseconds()
        }`;
      } catch (e) {
        ElMessage({
          message: "生成失败",
          type: "error",
        });
      }
    });
  });
});
</script>
<template>
  <div style="height: 100%" class="container">
    <div ref="editorRf" class="grid-content"></div>
    <div class="grid-content">
      <el-image
        fit="contain"
        :src="image"
        :preview-src-list="previews"
      ></el-image>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
}
.grid-title {
  text-align: center;
}
.grid-content {
  border-style: solid;
  display: flex;
  flex: 1;
  height: 100%;
  justify-content: center;
}
.el-image-viewer__img {
  background-color: white;
}
</style>
