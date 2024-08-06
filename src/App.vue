<template>
  <main id="sample">
    <div class="flex items-center justify-center">
      <img
        alt="Vue logo"
        class="logo"
        src="./assets/vue.svg"
        width="125"
        height="125"
      />
    </div>

    <WordFile />
  </main>
</template>
<script lang="ts" setup>
  import WordFile from './components/WordFile.vue'
</script>
