<template>
  <div class="media-input">
    <div class="tabs">
      <div
        class="tab"
        :class="{ 'active': type === tab.key }"
        v-for="tab in tabs"
        :key="tab.key"
        @click="type = tab.key"
      >{{tab.label}}</div>
    </div>

    <template v-if="type === 'video'">
      <Input v-model:value="videoSrc" placeholder="Please enter the video address, e.g. https://xxx.mp4"></Input>
      <div class="btns">
        <Button @click="emit('close')" style="margin-right: 10px;">Cancel</Button>
        <Button type="primary" @click="insertVideo()">Confirm</Button>
      </div>
    </template>

    <template v-if="type === 'audio'">
      <Input v-model:value="audioSrc" placeholder="Please enter the audio address, e.g. https://xxx.mp3"></Input>
      <div class="btns">
        <Button @click="emit('close')" style="margin-right: 10px;">Cancel</Button>
        <Button type="primary" @click="insertAudio()">Confirm</Button>
      </div>
    </template>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import {
  Button,
  Input,
  message,
} from 'ant-design-vue'

type TypeKey = 'video' | 'audio'
interface TabItem {
  key: TypeKey
  label: string
}

const emit = defineEmits<{
  (event: 'insertVideo', payload: string): void
  (event: 'insertAudio', payload: string): void
  (event: 'close'): void
}>()

const type = ref<TypeKey>('video')

const videoSrc = ref('https://mazwai.com/video_files/video/free/2019-01/small_watermarked/181004_04_Dolphins-Whale_06_preview.webm')
const audioSrc = ref('https://freesound.org/data/previews/614/614107_11861866-lq.mp3')

const tabs: TabItem[] = [
  { key: 'video', label: 'video' },
  { key: 'audio', label: 'audio' },
]

const insertVideo = () => {
  if (!videoSrc.value) return message.error('Please enter the correct video address first')
  emit('insertVideo', videoSrc. value)
}

const insertAudio = () => {
  if (!audioSrc.value) return message.error('Please enter the correct audio address first')
  emit('insertAudio', audioSrc. value)
}
</script>

<style lang="scss" scoped>
.media-input {
  width: 480px;
}
.tabs {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  border-bottom: 1px solid $borderColor;
  margin-bottom: 20px;
}
.tab {
  padding: 0 10px 8px;
  border-bottom: 2px solid transparent;
  cursor: pointer;

  &.active {
    border-bottom: 2px solid $themeColor;
  }
}
.btns {
  margin-top: 10px;
  text-align: right;
}
</style>