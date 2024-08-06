<template>
  <div class="flex justify-center my-10">
    <input
      type="file"
      class="w-[300px] h-[36px] rounded-md border border-gray-500"
      @change="handleFileChange"
    />
  </div>
  <div v-html="html"></div>
</template>
<script lang="ts" setup>
  import mammothPlus from 'mammoth-plus'
  import { ref } from 'vue'

  const html = ref('')

  const handleFileChange = (event: any) => {
    console.log(event)

    const fr = new FileReader()
    fr.readAsArrayBuffer(event.target.files[0])
    fr.onloadend = function (e: any) {
      console.log(e.target.result)

      mammothPlus
        .convertToHtml({ arrayBuffer: e.target.result })
        .then(function (result: any) {
          html.value = result.value
        })
    }
  }
</script>
<style lang="less" scoped></style>
