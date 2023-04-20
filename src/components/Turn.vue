<template>
  <input type="text" v-model="form.query" /><button @click="search">
    搜索
  </button>
  <div ref="bookRef" class="book">
    <!-- Pages will be dynamically added here -->
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as pdfjs from "pdfjs-dist/build/pdf";
import "pdfjs-dist/build/pdf.worker.entry";
import {
  PDFLinkService,
  PDFFindController,
  PDFViewer,
} from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";
import test from "@/assets/test.pdf";
pdfjs.workerSrc = window.pdfjsWorker;

const bookRef = ref(null);
const book = ref(null);
const pdfDoc = ref(null);
const form = ref({
  phraseSearch: true, // 是否短语查找
  query: "", // 查询字段
  findPrevious: true, // 是否循环查找
  highlightAll: true, // 是否高亮
});

const loadPdf = async () => {
  pdfDoc.value = await pdfjs.getDocument(test).promise;
  const book = new window.PDFJSViewer.PageView({
    container: bookRef.value,
    enhanceTextSelection: true,
    textLayerFactory: new window.PDFJSViewer.DefaultTextLayerFactory(),
    annotationLayerFactory:
      new window.PDFJSViewer.DefaultAnnotationLayerFactory(),
  });
  for (let pageNum = 0; pageNum < pdfDoc.value.numPages; pageNum++) {
    const page = await pdfDoc.value.getPage(pageNum + 1);
    const viewport = page.getViewport({ scale: 1 });
    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    canvas.width = viewport.width;
    canvas.height = viewport.height;
    const renderContext = {
      canvasContext: ctx,
      viewport: viewport,
    };
    await page.render(renderContext).promise;
    const pageElement = $("<div />", {
      class: "page",
      html: canvas,
    });
    book.addPage(pageElement[0]);
  }
};

onMounted(async () => {
  loadPdf();
});

const search = () => {
  console.log(newViewer.value);
  console.log(form.value);
  newViewer.value.findController.executeCommand("findagain", form.value);
};
</script>

<style scoped>
.box {
  position: relative;
}
</style>
