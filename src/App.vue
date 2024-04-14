<script setup lang="ts">
import { nextTick, onMounted, ref } from 'vue';
import { content } from './composables/quizzlet';
import html2canvas from 'html2canvas';
import JsPDF from 'jspdf';
const dataList = ref<any>([]);

let pageNum = ref(0);
const column = 2;
const getDeviceDPI = () => {
  var dpi = 96; // 默认值 96dpi
  if (window.screen && window.screen.deviceXDPI && window.screen.logicalXDPI) {
    dpi = (window.screen.deviceXDPI / window.screen.logicalXDPI) * dpi;
  }
  return dpi;
};

onMounted(() => {
  const screenHeight = Math.round((getDeviceDPI() / 25.5) * 180);
  const screenWidth = Math.round((getDeviceDPI() / 25.5) * 118.5);
  // console.log(getDeviceDPI(), screenHeight, screenWidth);
  content.map((item) => {
    dataList.value.push({
      title: item.section,
      type: 'section',
    });
    item.list.forEach((ele, eleIndex) => {
      dataList.value.push({ title: `<p class='gap'></p>`+ele.tigan, type: 'tigan' });
      if (ele.option.length > 0) {
        const column = ele.option.some((opt) => opt.name.length > 9);
        const single = ele.option.some((opt) => opt.name.length <= 4);
        if (column) {
          ele.option.forEach((opt) => {
            dataList.value.push({
              title: opt.value + '.' + opt.name,
              type: 'option1',
            });
          });
        } else if (single) {
          dataList.value.push({
            data: ele.option,
            type: 'option2',
          });
        } else {
          dataList.value.push({
            data1: ele.option.slice(0,2),
            data2: ele.option.slice(2,4),
            type: 'option3',
          });
          
        }
      }
    });
    dataList.value = dataList.value.concat(item.list);
  });
  nextTick().then(() => {
    let height = 0;
    const obj: any = { 1: [] };
    let num = 1;
    const allContent = document.querySelectorAll('.quiz-item > p');
    allContent.forEach((item) => {
      height += item.clientHeight;
      if (height > screenHeight) {
        num++;
        obj[num] = [];
        height = item.clientHeight;
      }
      obj[num].push(item);
      console.log(item, item.clientHeight);
      
    });
    pageNum.value = Math.round(Object.values(obj).length / column);
    for (let i = 1; i <= pageNum.value; i += 1) {
      const columnA = document.createElement('div');
      const columnB = document.createElement('div');
      const page = document.createElement('div');
      columnA.classList.add('column');
      columnB.classList.add('column');
      page.classList.add('page');
      page.append(columnA, columnB);
      const content = document.querySelector('.content');
      content?.append(page);
      obj[2 * i - 1].forEach((item: any) => {
        columnA.append(item);
      });
      if (obj[2 * i]) {
        obj[2 * i].forEach((item: any) => {
          columnB.append(item);
        });
      }
    }
    const container = document.querySelector('.container');
    container?.setAttribute('style', 'display:none');
  });
});

const download = async () => {
  const doc = new JsPDF('landscape', 'pt');
  const tables = document.querySelectorAll('.page');
  const options = {
    allowTaint: false,
    taintTest: false,
    logging: false,
    useCORS: true,
    dpi: window.devicePixelRatio * 4, // 将分辨率提高到特定的DPI 提高四倍
    scale: 4, // 按比例增加分辨率
  };
  for (let i = 0; i < tables.length; i++) {
    // 根据pageWarp将html分成多页
    const table = tables[i];
    const canvas = await html2canvas(table as HTMLElement, options);
    const _width = 841.89;
    const _heigth = 595.28;
    const contentWidth = canvas.width;
    const contentHeight = canvas.height;
    const imgWidth = _width;
    const imgHeight = (_width / contentWidth) * contentHeight;
    const image = canvas.toDataURL('image/jpeg', 1.0);
    if (i == 0) {
      doc.addImage(image, 'JPEG', 0, 0, imgWidth, imgHeight);
    } else {
      const page = doc.addPage();
      page.addImage(image, 'JPEG', 0, 0, imgWidth, imgHeight);
    }
  }
  doc.save(`xxx.pdf`);
};
</script>

<template>
  <div>
    <div style="height: 40px" @click="download">导出</div>
    <div>
      <div class="w-[148.5mm] container">
        <div v-for="item in dataList">
          <template v-if="item.type === 'section'">
            <div class="quiz-item">
              <p>{{ item.title }}</p>
            </div>
          </template>
          <template v-if="item.type === 'tigan'">
            <div class="tigan quiz-item" v-html="item.title">
            </div>
          </template>
          <template v-if="item.type === 'option1'">
            <div class="quiz-item">
              <p>{{ item.title }}</p>
            </div>
          </template>
          <div
            v-if="item.type === 'option2'"
            class="quiz-item"
          >
            <p
            style="display: flex; justify-content: space-between"
            >
            <div v-for="(ele, eleIndex) in item.data"
              :key="eleIndex">{{ ele.value }}. {{ ele.name }}</div>
              
            </p>
          </div>
          <div
            v-if="item.type === 'option3'"
            class="quiz-item"
          >
            <p
            style="display: flex; justify-content: space-between"
            >
            <div v-for="(ele, eleIndex) in item.data1"
              :key="eleIndex">{{ ele.value }}. {{ ele.name }}</div>
              
            </p>
            <p
            style="display: flex; justify-content: space-between"
            >
            <div v-for="(ele, eleIndex) in item.data2"
              :key="eleIndex">{{ ele.value }}. {{ ele.name }}</div>
              
            </p>
          </div>
        </div>
      </div>
    </div>
    <div class="content"></div>
  </div>
</template>

<style lang="less" scoped>
.common {
  padding: 15mm 0;
  height: 210mm;
  width: 297mm;
  border: 1px solid #000;
}
:deep(.page) {
  .common;
  display: flex;
  .column {
    width: 50%;
    padding: 0 15mm;
  }
}

:deep(.page img) {
  width: 600px !important;
  height: auto !important;
}
.container {
  padding: 15mm;
  border: 1px solid #000;
}
.content{
  counter-reset: section;
}
:deep(.tigan img) {
  width: 600px !important;
  height: auto !important;
}
:deep(.title-inline > p) {
  display: inline;
}
:deep(.gap) {
  padding-top: 20px;
  display: inline-block;
}
:deep(.gap + p):before{
  counter-increment: section;
  content: counter(section) '.';
}
</style>
