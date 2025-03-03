<template>
  <div class="multi-style-panel">
    <div class="row">
      <div style="flex: 2;">Fill color:</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker :modelValue="fill" @update:modelValue="value => updateFill(value)" />
        </template>
        <ColorButton :color="fill" style="flex: 3;" />
      </Popover>
    </div>

    <Divider />

    <div class="row">
      <div style="flex: 2;">Border style:</div>
      <Select style="flex: 3;" :value="outline.style"
        @change="value => updateOutline({ style: value as 'solid' | 'dashed' })">
        <SelectOption value="solid">solid border</SelectOption>
        <SelectOption value="dashed">Dashed Border</SelectOption>
      </select>
    </div>
    <div class="row">
      <div style="flex: 2;">Border color:</div>
      <Popover trigger="click">
        <template #content>
          <ColorPicker :modelValue="outline.color" @update:modelValue="value => updateOutline({ color: value })" />
        </template>
        <ColorButton :color="outline.color || '#000'" style="flex: 3;" />
      </Popover>
    </div>
    <div class="row">
      <div style="flex: 2;">Border thickness:</div>
      <InputNumber :value="outline.width" @change="value => updateOutline({ width: value as number })" style="flex: 3;" />
    </div>

    <Divider />

    <InputGroup compact class="row">
      <Select style="flex: 3;" :value="richTextAttrs.fontname"
        @change="value => updateFontStyle('fontname', value as string)">
        <template #suffixIcon>
          <IconFontSize />
        </template>
        <SelectOptGroup label="system font">
          <SelectOption v-for="font in availableFonts" :key="font.value" :value="font.value">
            <span :style="{ fontFamily: font.value }">{{ font.label }}</span>
          </SelectOption>
        </SelectOptGroup>
        <SelectOptGroup label="Online font">
          <SelectOption v-for="font in WEB_FONTS" :key="font.value" :value="font.value">
            <span>{{ font.label }}</span>
          </SelectOption>
        </SelectOptGroup>
      </select>
      <Select style="flex: 2;" :value="richTextAttrs.fontsize"
        @change="value => updateFontStyle('fontsize', value as string)">
        <template #suffixIcon>
          <IconAddText />
        </template>
        <SelectOption v-for="fontsize in fontSizeOptions" :key="fontsize" :value="fontsize">
          {{ fontsize }}
        </SelectOption>
      </select>
    </InputGroup>
    <ButtonGroup class="row">
      <Popover trigger="click">
        <template #content>
          <ColorPicker :modelValue="richTextAttrs.color" @update:modelValue="value => updateFontStyle('color', value)" />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Text Color">
          <TextColorButton :color="richTextAttrs.color" style="flex: 3;">
            <IconText />
          </TextColorButton>
        </Tooltip>
      </Popover>
      <Popover trigger="click">
        <template #content>
          <ColorPicker :modelValue="richTextAttrs.backcolor"
            @update:modelValue="value => updateFontStyle('backcolor', value)" />
        </template>
        <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Text Highlight">
          <TextColorButton :color="richTextAttrs.backcolor" style="flex: 3;">
            <IconHighLight />
          </TextColorButton>
        </Tooltip>
      </Popover>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Increase font size">
        <Button class="font-size-btn" style="flex: 2;" @click="updateFontStyle('fontsize-add', '2')">
          <IconFontSize />+
        </Button>
      </Tooltip>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Reduce font size">
        <Button class="font-size-btn" style="flex: 2;" @click="updateFontStyle('fontsize-reduce', '2')">
          <IconFontSize />-
        </Button>
      </Tooltip>
    </ButtonGroup>
    <RadioGroup class="row" button-style="solid" :value="richTextAttrs.align"
      @change="e => updateFontStyle('align', e.target.value)">
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Left alignment">
        <RadioButton value="left" style="flex: 1;">
          <IconAlignTextLeft />
        </RadioButton>
      </Tooltip>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Center">
        <RadioButton value="center" style="flex: 1;">
          <IconAlignTextCenter />
        </RadioButton>
      </Tooltip>
      <Tooltip :mouseLeaveDelay="0" :mouseEnterDelay="0.5" title="Right Align">
        <RadioButton value="right" style="flex: 1;">
          <IconAlignTextRight />
        </RadioButton>
      </Tooltip>
    </RadioGroup>
  </div>
</template>
<script lang="ts" setup>
import { ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useMainStore, useSlidesStore } from '@/store'
import { PPTElement, PPTElementOutline, TableCell } from '@/types/slides'
import emitter, { EmitterEvents } from '@/utils/emitter'
import { WEB_FONTS } from '@/configs/font'
import useHistorySnapshot from '@/hooks/useHistorySnapshot'

import ColorButton from '../common/ColorButton.vue'
import TextColorButton from '../common/TextColorButton.vue'
import ColorPicker from '@/components/ColorPicker/index.vue'
import {
  InputNumber,
  Divider,
  Button,
  Tooltip,
  Popover,
  Select,
  Radio,
  Input,
} from 'ant-design-vue'
const { Button: RadioButton, Group: RadioGroup } = Radio
const { OptGroup: SelectOptGroup, Option: SelectOption } = Select
const InputGroup = Input.Group
const ButtonGroup = Button.Group

const slidesStore = useSlidesStore()
const { richTextAttrs, availableFonts, activeElementList } = storeToRefs(useMainStore())

const { addHistorySnapshot } = useHistorySnapshot()

const updateElement = (id: string, props: Partial<PPTElement>) => {
  slidesStore.updateElement({ id, props })
  addHistorySnapshot()
}

const fontSizeOptions = [
  '12px', '14px', '16px', '18px', '20px', '22px', '24px', '28px', '32px',
  '36px', '40px', '44px', '48px', '54px', '60px', '66px', '72px', '76px',
  '80px', '88px', '96px', '104px', '112px', '120px',
]

const fill = ref('#fff')
const outline = ref<PPTElementOutline>({
  width: 0,
  color: '#fff',
  style: 'solid',
})

// Modify the fill color in batches (table elements are cell fills, audio elements are icon colors)
const updateFill = (value: string) => {
  for (const el of activeElementList.value) {
    if (
      el.type === 'text' ||
      el.type === 'shape' ||
      el.type === 'chart'
    ) updateElement(el.id, { fill: value })

    if (el.type === 'table') {
      const data: TableCell[][] = JSON.parse(JSON.stringify(el.data))
      for (let i = 0; i < data.length; i++) {
        for (let j = 0; j < data[i].length; j++) {
          const style = data[i][j].style || {}
          data[i][j].style = { ...style, backcolor: value }
        }
      }
      updateElement(el.id, { data })
    }

    if (el.type === 'audio') updateElement(el.id, { color: value })
  }
  fill.value = value
}

// modify border/line style
const updateOutline = (outlineProps: Partial<PPTElementOutline>) => {

  for (const el of activeElementList.value) {
    if (
      el.type === 'text' ||
      el.type === 'image' ||
      el.type === 'shape' ||
      el.type === 'table' ||
      el.type === 'chart'
    ) {
      const outline = el.outline || { width: 2, color: '#000', style: 'solid' }
      const props = { outline: { ...outline, ...outlineProps } }
      updateElement(el.id, props)
    }

    if (el.type === 'line') updateElement(el.id, outlineProps)
  }
  outline.value = { ...outline.value, ...outlineProps }
}

// Modify the text style
const updateFontStyle = (command: string, value: string) => {
  for (const el of activeElementList.value) {
    if (el.type === 'text' || (el.type === 'shape' && el.text?.content)) {
      emitter.emit(EmitterEvents.RICH_TEXT_COMMAND, { target: el.id, action: { command, value } })
    }
    if (el.type === 'table') {
      const data: TableCell[][] = JSON.parse(JSON.stringify(el.data))
      for (let i = 0; i < data.length; i++) {
        for (let j = 0; j < data[i].length; j++) {
          const style = data[i][j].style || {}
          data[i][j].style = { ...style, [command]: value }
        }
      }
      updateElement(el.id, { data })
    }
    if (el.type === 'latex' && command === 'color') {
      updateElement(el.id, { color: value })
    }
  }
}
</script>

<style lang="scss" scoped>
.row {
  width: 100%;
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.font-size-btn {
  padding: 0;
}
</style>