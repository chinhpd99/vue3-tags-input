<script>
import { defineComponent } from 'vue';
import Vue3TagsInput from '@/vue3-tags-input.vue';

export default defineComponent({
  name: 'ServeDev',
  components: {
    Vue3TagsInput
  }
});
</script>

<template>
  <div id="app">
    <vue3-tags-input />
  </div>
</template>
