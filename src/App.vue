<script setup lang="ts">
  import { nextTick, onMounted, ref } from 'vue'
  import { content } from './composables/quizzlet'
  import html2canvas from 'html2canvas'
  import JsPDF from 'jspdf'
  const dataList = ref<any>([])

  let pageNum = ref(0)
  const column = 2
  const getDeviceDPI = () => {
    var dpi = 96 // 默认值 96dpi
    if (
      window.screen &&
      window.screen.deviceXDPI &&
      window.screen.logicalXDPI
    ) {
      dpi = (window.screen.deviceXDPI / window.screen.logicalXDPI) * dpi
    }
    return dpi
  }

  onMounted(() => {
    const screenHeight = Math.round((getDeviceDPI() / 25.5) * 180)
    const screenWidth = Math.round((getDeviceDPI() / 25.5) * 118.5)
    // console.log(getDeviceDPI(), screenHeight, screenWidth);
    content.map((item) => {
      dataList.value.push({
        title: item.section,
        type: 'section',
      })
      item.list.forEach((ele, eleIndex) => {
        const template = document.createElement('template')
        template.innerHTML = ele.tigan
        const p =
          template.content.querySelector('p') || document.createElement('p')
        p.setAttribute('style', 'padding-top: 20px')
        p.textContent = eleIndex + 1 + '.' + (p.textContent ?? '')
        dataList.value.push({ title: template.innerHTML, type: 'tigan' })
        if (ele.option.length > 0) {
          const column = ele.option.some((opt) => opt.name.length > 9)
          const single = ele.option.some((opt) => opt.name.length <= 4)
          if (column) {
            ele.option.forEach((opt) => {
              dataList.value.push({
                title: opt.value + '.' + opt.name,
                type: 'option1',
              })
            })
          } else if (single) {
            dataList.value.push({
              data: ele.option,
              type: 'option2',
            })
          } else {
            dataList.value.push({
              data1: ele.option.slice(0, 2),
              data2: ele.option.slice(2, 4),
              type: 'option3',
            })
          }
        }
      })
      dataList.value = dataList.value.concat(item.list)
    })
    nextTick().then(() => {
      let height = 0
      const obj: any = { 1: [] }
      let num = 1
      const allContent = document.querySelectorAll('.quiz-item > p')
      allContent.forEach((item) => {
        height += item.clientHeight
        console.log(item, item.clientHeight, height)

        if (height > screenHeight) {
          num++
          obj[num] = []
          height = item.clientHeight
        }
        obj[num].push(item)
        // console.log(item, item.clientHeight);
      })
      pageNum.value = Math.round(Object.values(obj).length / column)
      for (let i = 1; i <= pageNum.value; i += 1) {
        const columnA = document.createElement('div')
        const columnB = document.createElement('div')
        const page = document.createElement('div')
        const slide = document.createElement('div')
        columnA.classList.add('column')
        columnB.classList.add('column')
        slide.classList.add('slide')
        slide.innerHTML = `
          <div class="divider">装订线装订线装订线</div>
          <div class="area">
            <div class="name">姓名：</div>
            <div class="number">准考证号：</div>
          </div>
          <div class="divider">密封线密封线密封线</div>
        `
        page.classList.add('page')
        page.append(slide, columnA, columnB)
        const content = document.querySelector('.content')
        content?.append(page)
        obj[2 * i - 1].forEach((item: any) => {
          columnA.append(item)
        })
        if (obj[2 * i]) {
          obj[2 * i].forEach((item: any) => {
            columnB.append(item)
          })
        }
      }
      const container = document.querySelector('.container')
      container?.setAttribute('style', 'display:none')
    })
  })

  const download = async () => {
    const doc = new JsPDF('landscape', 'pt')
    const tables = document.querySelectorAll('.page')
    const options = {
      allowTaint: false,
      taintTest: false,
      logging: false,
      useCORS: true,
      dpi: window.devicePixelRatio * 4, // 将分辨率提高到特定的 DPI 提高四倍
      scale: 4, // 按比例增加分辨率
    }
    for (let i = 0; i < tables.length; i++) {
      // 根据 pageWarp 将 html 分成多页
      const table = tables[i]
      const canvas = await html2canvas(table as HTMLElement, options)
      const _width = 841.89
      const _heigth = 595.28
      const contentWidth = canvas.width
      const contentHeight = canvas.height
      const imgWidth = _width
      const imgHeight = (_width / contentWidth) * contentHeight
      const image = canvas.toDataURL('image/jpeg', 1.0)
      if (i == 0) {
        doc.addImage(image, 'JPEG', 0, 0, imgWidth, imgHeight)
      } else {
        const page = doc.addPage()
        page.addImage(image, 'JPEG', 0, 0, imgWidth, imgHeight)
      }
    }
    doc.save(`xxx.pdf`)
  }
</script>

<template>
  <div>
    <div style="height: 40px" @click="download">导出</div>
    <div class="page container">
      <div class="slide"></div>
      <div class="column">
        <div class="quiz-item">
          <p style="text-align: center; padding: 10px 0">青岛市学业水平考试</p>
        </div>
        <div v-for="item in dataList">
          <template v-if="item.type === 'section'">
            <div class="quiz-item">
              <p>{{ item.title }}</p>
            </div>
          </template>
          <template v-if="item.type === 'tigan'">
            <div class="tigan quiz-item" v-html="item.title"></div>
          </template>
          <template v-if="item.type === 'option1'">
            <div class="quiz-item">
              <p>{{ item.title }}</p>
            </div>
          </template>
          <div v-if="item.type === 'option2'" class="quiz-item">
            <p style="display: flex; justify-content: space-between">
              <span v-for="(ele, eleIndex) in item.data" :key="eleIndex">
                {{ ele.value }}. {{ ele.name }}
              </span>
            </p>
          </div>
          <div v-if="item.type === 'option3'" class="quiz-item">
            <p style="display: flex; justify-content: space-between">
              <span v-for="(ele, eleIndex) in item.data1" :key="eleIndex"
                >{{ ele.value }}. {{ ele.name }}</span
              >
            </p>
            <p style="display: flex; justify-content: space-between">
              <span v-for="(ele, eleIndex) in item.data2" :key="eleIndex"
                >{{ ele.value }}. {{ ele.name }}</span
              >
            </p>
          </div>
        </div>
      </div>
      <div class="column"></div>
    </div>
    <div class="content"></div>
  </div>
</template>

<style lang="less" scoped>
  .common {
    padding: 15mm;
    height: 210mm;
    width: 297mm;
    border: 1px solid #000;
  }
  :deep(.page) {
    .common;
    display: flex;
    .column {
      flex: 1;
      padding: 0 40px 0 16px;
    }
    .slide {
      width: 90px;
      height: 100%;
    }
  }

  :deep(.page img) {
    width: 600px !important;
    height: auto !important;
  }

  :deep(.tigan img) {
    width: 600px !important;
    height: auto !important;
  }
  // :deep(.tigan) > p:first-child {
  //   padding-top: 20px;
  // }
  // :deep(.title-inline > p) {
  //   display: inline;
  // }
  // :deep(.gap) {
  //   padding-top: 20px;
  //   display: inline-block;
  // }
  // :deep(.gap + p):before {
  //   counter-increment: section;
  //   content: counter(section) '.';
  // }

  :deep(.slide) {
    display: flex;
    justify-content: space-between;
    .divider {
      border-left: 1px dotted black;
      text-orientation: sideways;
      writing-mode: vertical-lr;
      transform: rotate(180deg);
      padding: 40px 0;
      text-align-last: justify;
    }

    .area {
      display: flex;
      justify-content: center;
      text-orientation: sideways;
      writing-mode: vertical-lr;
      transform: rotate(180deg);
      gap: 50px;
      .name::after {
        content: '';
        display: inline-block;
        height: 150px;
        border-left: 1px dotted black;
      }
      .number::after {
        content: '';
        display: inline-block;
        height: 150px;
        border-left: 1px dotted black;
      }
    }
  }
</style>
