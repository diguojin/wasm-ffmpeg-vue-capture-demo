<template>
  <video :src="video" controls />
  <br />
  <img :src="img" alt="" />
  <br />
  <button @click="transcode">Start</button>
  <input type="text" v-model="ss" />
  <button @click="getImage">getImage</button>
  <p>{{ message }}</p>
</template>

<script lang="ts">
import { FFmpeg } from '@ffmpeg/ffmpeg'
import type { LogEvent } from '@ffmpeg/ffmpeg/dist/esm/types'
import { fetchFile, toBlobURL } from '@ffmpeg/util'
import { defineComponent, ref } from 'vue'

// const baseURL = 'https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm'
const baseURL = 'https://unpkg.com/@ffmpeg/core@0.12.6/dist/esm'
// const baseURL = 'http://localhost:5173/src/assets'
const videoURL = '/src/assets/pi.mp4'

export default defineComponent({
  name: 'App',
  setup() {
    const ffmpeg = new FFmpeg()
    const message = ref('Click Start to Transcode')
    const ss = ref('3')
    const loading = ref(false)
    let video = ref('')
    let img = ref('')
    initFF()
    async function transcode() {
      message.value = 'Loading ffmpeg-core.js'
      ffmpeg.on('log', ({ message: msg }: LogEvent) => {
        console.log(msg)
        message.value = msg
      })
      await ffmpeg.load({
        coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
        wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm')
        // workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
      })
      message.value = 'Start transcoding'
      await ffmpeg.writeFile('pi.mp4', await fetchFile(videoURL))
      await ffmpeg.exec(['-i', 'pi.mp4', 'test.mp4'])
      message.value = 'Complete transcoding'
      const data = await ffmpeg.readFile('test.mp4')
      video.value = URL.createObjectURL(
        new Blob([(data as Uint8Array).buffer], { type: 'video/mp4' })
      )
    }

    async function getImage() {
      if (!loading.value) {
        try {
          loading.value = true
          console.log('----------1')
          await ffmpeg.exec(['-ss', ss.value, '-i', 'pi.mp4', '-vframes', '1', 'test.jpg'])
          const data = await ffmpeg.readFile('test.jpg')
          message.value = 'Complete transcoding'
          console.log('Complete transcoding')
          console.log('----------2')
          img.value = URL.createObjectURL(
            new Blob([(data as Uint8Array).buffer], { type: 'image/jpeg' })
          )
          console.log(img.value)
          loading.value = false
        } catch (error) {
          console.log('++++++++++++')
          console.log(error)
          console.log('++++++++++++')
          await ffmpeg.terminate()
          initFF()
          loading.value = false
        }
      }else{
        console.log('task is loading....')
      }
    }

    async function initFF() {
      message.value = 'Loading-getImage ffmpeg-core.js'
      ffmpeg.on('log', ({ message: msg }: LogEvent) => {
        console.log(msg)
        message.value = msg
      })
      await ffmpeg.load({
        coreURL: 'http://localhost:5173/src/assets/ffmpeg-core.js',
        // wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
        wasmURL: 'http://localhost:5173/src/assets/a.wasm'
        // workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
      })
      await ffmpeg.writeFile('pi.mp4', await fetchFile(videoURL))
      message.value = 'file ready'
      console.log('ff is ready')
    }

    return {
      video,
      message,
      getImage,
      img,
      ss,
      transcode
    }
  }
})
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
