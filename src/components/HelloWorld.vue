<template>
  <input type="text" /><button @click="search">搜索</button>
  <div id="innerContainer">
    <!-- <div v-for="item in pdfPicturePath" :key="item">
      <div class="box" id="viewerContainer">
        <canvas :id="`the-canvas-${item}`"></canvas>
      </div>
    </div> -->
  </div>
  <div class="search-box" v-for="(item, index) in searchResult">
    <div>page:{{ item.page }}</div>
    <div>{{ item.str }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import * as pdfjs from "pdfjs-dist/build/pdf";
import "pdfjs-dist/build/pdf.worker.entry";
import { TextLayerBuilder } from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";
import test from "@/assets/test.pdf";
import {
  PDFLinkService,
  PDFFindController,
  PDFViewer,
} from "pdfjs-dist/web/pdf_viewer";
import Fuse from "fuse.js";
pdfjs.workerSrc = window.pdfjsWorker;

const pdfPicturePath = ref([]);
const loadingTask = ref(null);
const loading = ref(false);
const totalPage = ref(0);
const prfRef = ref(null);
const searchList = ref([]);
const searchResult = ref([]);
onMounted(async () => {
  // 解析链接，可能跨域
  const pdfUrl = "https://image.yitong.com/test.pdf";

  // 解析Buffer
  const pdfData = await fetch(new URL(pdfUrl, window.location).href);
  const arrayBufferPdf = await pdfData.arrayBuffer();

  // 解析本地文件
  loadingTask.value = pdfjs.getDocument(test || arrayBufferPdf || pdfUrl);

  renderPDF(1, 3).then(() => {
    if (loading.value) return;
    $("#innerContainer").turn({
      width: 595 * 2,
      height: 807,
      // autoCenter: true,
      page: 1,
      pages: totalPage.value,
      when: {
        turned: function (e, page) {
          console.log(page);
          if (page == 1) {
            loading.value = true;
          }
          if (page == 2) {
            renderPDF(4, 7);
          }
          if (page == 6) {
            renderPDF(8, 11);
          }
          if (page == 10) {
            renderPDF(12, 15);
          }
        },
      },
    });
    setTimeout(() => {
      getAllText(prfRef.value);
    }, 500);
  });
});

const renderPDF = (start, end) => {
  return loadingTask.value.promise.then(async function (pdf) {
    prfRef.value = pdf; // 页数
    totalPage.value = pdf.numPages; // 页数

    for (let i = start; i >= start && i <= end; i++) {
      pdfPicturePath.value.push(i); // 渲染页面用的数组
      let page = await pdf.getPage(i); // 每页的PDF对象
      let scale = 1; // 缩放倍数，1表示原始大小
      let viewport = page.getViewport({ scale }); // 尺寸

      // 创建一个 Canvas 画布
      var div = document.createElement("div");
      var canvas = document.createElement("canvas");
      div.appendChild(canvas);
      var context = canvas.getContext("2d");
      var outputScale = window.devicePixelRatio || 1;
      canvas.width = Math.floor(viewport.width * outputScale);
      canvas.height = Math.floor(viewport.height * outputScale);
      canvas.style.width = Math.floor(viewport.width) + "px";
      canvas.style.height = Math.floor(viewport.height) + "px";
      var transform =
        outputScale !== 1 ? [outputScale, 0, 0, outputScale, 0, 0] : null;
      let renderContext = {
        canvasContext: context,
        transform: transform,
        viewport: viewport,
      };

      // 渲染PDF
      const renderTask = page.render(renderContext);

      // 渲染文字图层
      await renderTask.promise
        .then(() => {
          return page.getTextContent();
        })
        .then((textContent) => {
          const textLayerDiv = document.createElement("div");
          textLayerDiv.setAttribute("class", "textLayer");
          textLayerDiv.style.width = `${canvas.width}px`;
          textLayerDiv.style.height = `${canvas.height}px`;
          const pageDom = canvas.parentNode;
          pageDom.appendChild(textLayerDiv);
          // 创建新的TextLayerBuilder实例
          const textLayer = new TextLayerBuilder({
            textLayerDiv,
            pageIndex: page.pageIndex,
            viewport,
          });
          textLayer.setTextContent(textContent);
          textLayer.render();
        });
      if (end === 3) {
        var con = document.getElementById("innerContainer");
        con.appendChild(div);
      }
      if (end > 3) {
        if ($("#innerContainer").turn("hasPage", i)) return;
        $("#innerContainer").turn("addPage", div, i);
      }
    }
  });
};

const search = () => {
  const textLayerSpans = document.querySelectorAll(".textLayer > span");
  textLayerSpans.forEach((span) => {
    // 如果 span 标签的文本包含 "自然"，则添加新的 span 标签
    if (span.textContent.includes("自然")) {
      const newSpan = document.createElement("span");
      newSpan.innerHTML = span.innerHTML.replace(
        /自然/g,
        '<span class="highlight selected">自然</span>'
      );
      span.innerHTML = newSpan.innerHTML;
    }
  });
  const options = {
    keys: ["str"],
  };
  const fuse = new Fuse(searchList.value, options);
  searchResult.value = fuse.search("自然");
  searchResult.value = searchResult.value.map((item) => {
    return {
      str: item.item.str,
      page: item.item.page,
    };
  });
  searchResult.value.sort((a, b) => a.page - b.page);
  console.log(searchResult.value);
};

const getAllText = async (pdf) => {
  for (let i = 1; i <= totalPage.value; i++) {
    let page = await pdf.getPage(i); // 每页的PDF对象
    const list = await page.getTextContent();
    list.items.forEach((item) => {
      item.page = i;
    });
    searchList.value = [...searchList.value, ...list.items];
  }
};
</script>

<style scoped>
.box {
  position: relative;
}
#innerContainer {
  overflow: hidden;
}
</style>
