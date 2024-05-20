<template>
  <main id="sample">
    <img
      alt="Vue logo"
      class="logo"
      src="./assets/logo.svg"
      width="125"
      height="125"
    />
    <Editor class="editor" :init="init" />
  </main>
</template>
<script lang="ts" setup>
  import Editor from '@tinymce/tinymce-vue'
  import tinymce from 'tinymce'
  // import '../public/plugin.min.js'
  import { onMounted } from 'vue'

  const init = {
    selector: '.editor',
    external_plugins: {
      tiny_mce_wiris: `/node_modules/@wiris/mathtype-tinymce6/plugin.min.js`,
      powerpaste: '/plugin.min.js',
    },
    extended_valid_elements: '*[.*]',
    draggable_modal: true,
    language: 'zh_CN',
    promotion: false, // Upgrade 是否开启
    branding: false, // tiny 技术支持信息是否显示
    menubar: false, // 菜单栏
    statusbar: false, // 注意将 statusbar 设置为 false 来隐藏字数统计显示
    plugins: 'image media powerpaste',
    // powerpaste_allow_local_images: true,
    // powerpaste_word_import: 'prompt',
    // powerpaste_html_import: 'prompt',
    toolbar: `fontsize | lineheight fontfamily | bold italic forecolor | tiny_mce_wiris_formulaEditor tiny_mce_wiris_formulaEditorChemistry | image media`,
    fontfamily: '微软雅黑;',
    line_height_formats: '1 1.2 1.4 1.6 2', // 行高
    font_size_formats:
      '12pt 14pt 16pt 18pt 20pt 22pt 24pt 28pt 32pt 36pt 48pt 56pt 72pt', // 字体大小
    file_picker_types: 'media image',
    image_prepend_url: 'https://www.example.com/images/',

    file_picker_callback: (cb: any, value: any, meta: any) => {
      const input = document.createElement('input')
      input.setAttribute('type', 'file')
      input.setAttribute('accept', 'image/*, audio/*, video/*')
      input.click()
    },
  }

  onMounted(() => {
    tinymce.init(init)
  })
</script>
<style lang="less" scoped></style>
