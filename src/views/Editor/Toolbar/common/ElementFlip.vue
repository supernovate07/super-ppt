<template>
  <div class="element-flip">
    <CheckboxButtonGroup class="row">
      <CheckboxButton
        style="flex: 1;"
        :checked="flipV"
        @click="updateFlip({ flipV: !flipV })"
      ><IconFlipVertically /> Flip vertically</CheckboxButton>
      <CheckboxButton
        style="flex: 1;"
        :checked="flipH"
        @click="updateFlip({ flipH: !flipH })"
      ><IconFlipHorizontally />Flip horizontally</CheckboxButton>
    </CheckboxButtonGroup>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore, useSlidesStore } from '@/store'
import { ImageOrShapeFlip } from '@/types/slides'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'

import CheckboxButton from '@/components/CheckboxButton.vue'
import CheckboxButtonGroup from '@/components/CheckboxButtonGroup.vue'

const slidesStore = useSlidesStore()
const { handleElement } = storeToRefs(useMainStore())

const flipH = ref(false)
const flipV = ref(false)

watch(handleElement, () => {
  if (handleElement.value && (handleElement.value.type === 'image' || handleElement.value.type === 'shape')) {
    flipH.value = !!handleElement.value.flipH
    flipV.value = !!handleElement.value.flipV
  }
}, { deep: true, immediate: true })

const { addHistorySnapshot } = useHistorySnapshot()

const updateFlip = (flipProps: ImageOrShapeFlip) => {
  if (!handleElement.value) return
  slidesStore.updateElement({ id: handleElement.value.id, props: flipProps })
  addHistorySnapshot()
}
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
</style>