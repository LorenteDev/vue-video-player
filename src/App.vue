<script setup lang="ts">
import VideoPlayer from './components/VideoPlayer.vue'
import videoFile from '../src/assets/video.mp4'
import { Ref, onBeforeMount, ref } from 'vue';

const videoPlayerRef = ref()
let videoUrl:Ref<string> = ref('')
let manualVideoUrl:Ref<string> = ref('')
let format:Ref<string> = ref('')
const restartButtonRef = ref()

// const linkRegExpString:string = '^(?![^\n]*\.$)(?:https?:\/\/)?(?:(?:[2][1-4]\d|25[1-5]|1\d{2}|[1-9]\d|[1-9])(?:\.(?:[2][1-4]\d|25[1-5]|1\d{2}|[1-9]\d|[0-9])){3}(?::\d{4})?|[a-z\-]+(?:\.[a-z\-]+){2,})$'
// const linkRegExp:RegExp = new RegExp(linkRegExpString, 'g')

const updateVideo = () => {
  videoUrl.value = manualVideoUrl.value
  format.value = manualVideoUrl.value.split('.').slice(-1)[0]
}

const restartVideo = () => {
  videoUrl.value = videoFile
  format.value = videoFile.split('.').slice(-1)[0]
}

onBeforeMount(() => {
  restartVideo()
})

</script>

<template>
  <h1>Custom Vue Video Player</h1>
  <VideoPlayer :videoPath="videoUrl" :format="format" maxWidth="900" ref="videoPlayerRef" />
  <form @submit.prevent="">
    <input v-model="manualVideoUrl" type="text" placeholder="Link to a video file" />
    <div>
      <button @click="updateVideo">
        <svg xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
          <path d="M9 16h6v-6h4l-7-7-7 7h4zm-4 2h14v2H5z" fill="#FFF"/>
        </svg>
        <span>
          Load Link
        </span>
      </button>
      <button @click="restartVideo" ref="restartButtonRef">
        <svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" width="24" height="24">
          <path d="m20.3 13.43a1 1 0 0 0 -1.25.65 7.14 7.14 0 0 1 -6.87 4.92 7.1 7.1 0 0 1 -7.18-7 7.1 7.1 0 0 1 7.18-7 7.26 7.26 0 0 1 4.65 1.67l-2.17-.36a1 1 0 0 0 -1.15.83 1 1 0 0 0 .83 1.15l4.24.7h.17a1 1 0 0 0 .34-.06.33.33 0 0 0 .1-.06.78.78 0 0 0 .2-.11l.09-.11c0-.05.09-.09.13-.15s0-.1.05-.14a1.34 1.34 0 0 0 .07-.18l.75-4a1 1 0 0 0 -2-.38l-.27 1.45a9.21 9.21 0 0 0 -6.03-2.25 9.1 9.1 0 0 0 -9.18 9 9.1 9.1 0 0 0 9.18 9 9.12 9.12 0 0 0 8.82-6.32 1 1 0 0 0 -.7-1.25z" fill="#FFF"/>
        </svg>
        <span>Restart</span>
      </button>
    </div>
  </form>
  <a href="https://github.com/LorenteDev/vue-video-player" target="_blank" rel="noopener noreferrer">
    <svg viewBox="0 0 100 100" width="98" height="96" xmlns="http://www.w3.org/2000/svg">
      <path fill-rule="evenodd" clip-rule="evenodd" d="M48.854 0C21.839 0 0 22 0 49.217c0 21.756 13.993 40.172 33.405 46.69 2.427.49 3.316-1.059 3.316-2.362 0-1.141-.08-5.052-.08-9.127-13.59 2.934-16.42-5.867-16.42-5.867-2.184-5.704-5.42-7.17-5.42-7.17-4.448-3.015.324-3.015.324-3.015 4.934.326 7.523 5.052 7.523 5.052 4.367 7.496 11.404 5.378 14.235 4.074.404-3.178 1.699-5.378 3.074-6.6-10.839-1.141-22.243-5.378-22.243-24.283 0-5.378 1.94-9.778 5.014-13.2-.485-1.222-2.184-6.275.486-13.038 0 0 4.125-1.304 13.426 5.052a46.97 46.97 0 0 1 12.214-1.63c4.125 0 8.33.571 12.213 1.63 9.302-6.356 13.427-5.052 13.427-5.052 2.67 6.763.97 11.816.485 13.038 3.155 3.422 5.015 7.822 5.015 13.2 0 18.905-11.404 23.06-22.324 24.283 1.78 1.548 3.316 4.481 3.316 9.126 0 6.6-.08 11.897-.08 13.526 0 1.304.89 2.853 3.316 2.364 19.412-6.52 33.405-24.935 33.405-46.691C97.707 22 75.788 0 48.854 0z" fill="#FFF"/>
    </svg>
  </a>
</template>

<style lang="scss" scoped>
h1 {
  text-align: center;
}

a {

  &:hover {
    svg > path {
      fill: #acacac;
    }
  }

  svg {
    width: 50px;
    height: 50px;

    path {
      transition: all 0.2s;
    }
  }
}


form {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  width: 100%;
  
  input {
    box-sizing: border-box;
    width: 100%;
  }

  div {
    display: flex;
    gap: 8px;

    button {
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 4px;
      font-family: "Inter", sans-serif;
      padding: 4px 12px;
      border: none;
      background-color: #eb5c5c;
      color: #FFF;
      border-radius: 4px;
      transition: all 0.2s;
      width: 120px;

      svg {
        width: 20px;
        height: 20px;
        transition: all 0.2s;
      }

      &:hover {
        background-color: #eb2416;
      }

      &:first-child {
        &:hover {
          svg {
            animation-name: bounce;
            animation-duration: 0.3s;
            animation-iteration-count: 1;
            animation-timing-function: ease-in-out;
          }
        }
      }

      &:last-child {
        &:hover {
          svg {
            animation-name: spin;
            animation-duration: 0.4s;
            animation-iteration-count: 1;
            animation-timing-function: ease-in-out;
          }
        }
      }
    }
    
    :invalid {
      border: 1px solid red;
    }
  }

}

@keyframes bounce {
  0% {
    -ms-transform: translateY(0);
    -moz-transform: translateY(0);
    -webkit-transform: translateY(0);
    transform: translateY(0)
  }

  50% {
    -ms-transform: translateY(-3px);
    -moz-transform: translateY(-3px);
    -webkit-transform: translateY(-3px);
    transform: translateY(-3px);
  }

  100% {
    -ms-transform: translateY(0);
    -moz-transform: translateY(0);
    -webkit-transform: translateY(0);
    transform: translateY(0)
  }
}

@keyframes spin {
  0% {
    -ms-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -webkit-transform: rotate(0deg);
    transform: rotate(0deg);
  }

  100% {
    -ms-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -webkit-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}

</style>