<template>
  <div class="export-json-dialog">
    <div class="preview">
      <pre>{{slides}}</pre>
    </div>

    <div class="btns">
      <Button class="btn export" type="primary" @click="exportJSON()">Export JSON</Button>
      <Button class="btn close" @click="emit('close')">Close</Button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { storeToRefs } from 'pinia'
import { useSlidesStore } from '@/store'
import useExport from '@/hooks/useExport'

import { Button } from 'ant-design-vue'

const emit = defineEmits<{
  (event: 'close'): void
}>()

const { slides } = storeToRefs(useSlidesStore())
const { exportJSON } = useExport()
</script>

<style lang="scss" scoped>
.export-json-dialog {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  position: relative;
  overflow: hidden;
}
.preview {
  width: 100%;
  height: calc(100% - 100px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: #f9f9f9;
  color: #0451a5;
}
.btns {
  width: 300px;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;

  .export {
    flex: 1;
  }
  .close {
    width: 100px;
    margin-left: 10px;
  }
}
::-webkit-scrollbar {
  width: 10px;
  height: 10px;
  background-color: #f9f9f9;
}
::-webkit-scrollbar-thumb {
  background-color: #c1c1c1;
}
</style>