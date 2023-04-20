<template>
  <input type="text" v-model="form.query" /><button class="btn" @click="search">
    搜索
  </button>
  <div id="viewerContainer" class="viewerContainer" ref="containerRef">
    <div class="pdfViewer" id="innerContainer"></div>
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

const containerRef = ref();
const pdfPicturePath = ref([]);
const form = ref({
  phraseSearch: true, // 是否短语查找
  query: "", // 查询字段
  findPrevious: true, // 是否循环查找
  highlightAll: true, // 是否高亮
});

const newViewer = ref(null);

onMounted(async () => {
  const linkService = new PDFLinkService();
  const findController = new PDFFindController({
    linkService,
  });
  newViewer.value = new PDFViewer({
    container: containerRef.value, // 显示PDF的容器dom
    linkService,
    useOnlyCssZoom: true, // 是否可以通过css控制页面的缩放,默认 false
    textLayerMode: 1, // 是否显示文字 默认0 不显示，1 显示
    // renderer:'svg', // 默认是canvas 可选svg
    findController, // 传入文字查找控制器
  });

  linkService.setViewer(newViewer.value);
  // 设置初始缩放
  newViewer.value.currentScaleValue = 1;
  // 解析链接，可能跨域
  const pdfUrl = "https://image.yitong.com/test.pdf";

  // 解析Buffer
  const pdfData = await fetch(new URL(pdfUrl, window.location).href);
  const arrayBufferPdf = await pdfData.arrayBuffer();

  // 解析本地文件
  const loadingTask = pdfjs.getDocument(test || arrayBufferPdf || pdfUrl);

  // 获取PDF对象
  loadingTask.promise.then(async function (pdf) {
    // setNumPages(nums);
    newViewer.value.setDocument(pdf);
    linkService.setDocument(pdf);
    console.log(newViewer.value);
    // setViewer(newViewer.value);
    // const interval = setInterval(() => {
    //   loadPdf();
    // }, 1000);
    // function loadPdf() {
    //   if (newViewer.value.pageViewsReady) {
    //     // ... 渲染完成操作
    //   }
    // }
    // setTimeout(() => {
    //   $("#innerContainer").turn({
    //     width: 1632,
    //     height: 1056,
    //     autoCenter: true,
    //   });
    // }, 2000);
  });
});

const search = () => {
  console.log(newViewer.value);
  console.log(form.value);
  newViewer.value.findController.executeCommand("findagain", form.value);
};

const getAllText = async (pdf) => {
  const nums = pdf.numPages;
  for (let i = 1; i <= nums; i++) {
    let page = await pdf.getPage(i); // 每页的PDF对象
    console.log(await page.getTextContent());
  }
};
</script>

<style scoped>
.box {
  position: relative;
}
</style>
