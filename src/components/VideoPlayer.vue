<script setup lang="ts">
import { ref, onMounted, onUnmounted, Ref } from 'vue'

const props = defineProps<{
  videoPath: string,
  format: string,
  maxWidth: string
}>()

const videoRef = ref()
const containerRef = ref()
let isPlaying:boolean = false
let playbackRate:Ref<number> = ref(1.0)
let volume:Ref<number> = ref(0)
let duration:Ref<number> = ref(0)
let currentTime:Ref<number> = ref(0)

const formatTimeNumber = (number:number) => {
  return number.toString().length === 1 ? `0${number.toString()}` : number.toString()
}

const formatTime = (time:number) => {
  let hours = 0
  let minutes = 0
  let seconds = 0

  hours = time / 3600

  if (hours > 0) {
    hours = Math.floor(hours)
  }

  minutes = (time - hours * 3600) / 60

  if (minutes > 0) {
    minutes = Math.floor(minutes)
  }

  seconds = Math.floor(time - hours * 3600 - minutes * 60)
 
  if (hours) {
    return `${formatTimeNumber(hours)}:${formatTimeNumber(minutes)}:${formatTimeNumber(seconds)}`
  } else {
    return `${formatTimeNumber(minutes)}:${formatTimeNumber(seconds)}`
  }
}

const play = () => {
  duration.value = videoRef.value.duration
  videoRef.value.play()
  isPlaying = true
}

const pause = () => {
  videoRef.value.pause()
  isPlaying = false
}

// const stop = () => {
//   videoRef.value.currentTime = 0
//   pause()
// }

const togglePlaybackRate = () => {
  if (playbackRate.value === 2) {
    playbackRate.value = 0.25
  } else {
    playbackRate.value = playbackRate.value + 0.25
  }

  videoRef.value.playbackRate = playbackRate.value
}

const updateVolume = () => {
  videoRef.value.volume = volume.value / 100
}

const updateCurrentTime = () => {
  videoRef.value.currentTime = currentTime.value
}

const toggleMute = () => {
  console.log(videoRef.value.volume)
  
  if (videoRef.value.volume > 0) {
    console.log('mute')
    videoRef.value.volume = 0
  } else {
    console.log('unmute')
    videoRef.value.volume = volume.value / 100
  }
}

const togglePlay = () => {
  if (isPlaying) {
    pause()
  } else {
    play()
  }
}

const goBackwards = (seconds:number) => {
  videoRef.value.currentTime -= seconds
}

const goForwards = (seconds:number) => {
  videoRef.value.currentTime += seconds
}

const toggleFullscreen = () => {
  if (document.fullscreenElement) {
    if (containerRef.value.exitFullscreen) {
      containerRef.value.exitFullscreen()
    } else if (containerRef.value.webkitExitFullscreen) { /* Safari */
      containerRef.value.webkitExitFullscreen()
    } else if (containerRef.value.msExitFullscreen) { /* IE11 */
      containerRef.value.msExitFullscreen()
    } else{
      document.exitFullscreen()
    }
  } else {
    if (containerRef.value.requestFullscreen) {
      containerRef.value.requestFullscreen()
    } else if (containerRef.value.webkitRequestFullscreen) { /* Safari */
      containerRef.value.webkitRequestFullscreen()
    } else if (containerRef.value.msRequestFullscreen) { /* IE11 */
      containerRef.value.msRequestFullscreen()
    }
  }
}

onMounted(() => {
  window.addEventListener('keydown', (e: KeyboardEvent) => {
    if (e.key === " " || e.code.toLowerCase() === "space" || e.keyCode == 32) {
      togglePlay()
    }
  })

  videoRef.value.addEventListener('timeupdate', () => {
    currentTime.value = videoRef.value.currentTime
  })

  videoRef.value.addEventListener('loadstart', () => {
    volume.value = videoRef.value.volume * 100
  })

  videoRef.value.addEventListener('ended', () => {
    isPlaying = false
  })

  containerRef.value.addEventListener('contextmenu', (event: MouseEvent) => {
    event.preventDefault()
  })

  window.addEventListener("fullscreenchange", () => {
    if (document.fullscreenElement) {
      containerRef.value.style.maxWidth = 'unset'
      containerRef.value.style.width = '100vw'
      videoRef.value.style.width = '100vw'
    } else {
      containerRef.value.style.maxWidth = props.maxWidth
      containerRef.value.style.width = null
      videoRef.value.style = null
    }
  })
})

onUnmounted(() => {
  window.removeEventListener('keydown', () => {})
  videoRef.value.removeEventListener('timeupdate', () => {})
  videoRef.value.removeEventListener('loadstart', () => {})
  videoRef.value.removeEventListener('ended', () => {})
  containerRef.value.removeEventListener('contextmenu', () => {})
  window.removeEventListener('fullscreenchange', () => {})
})

</script>

<template>
  <div class="video-player-container" ref="containerRef" :style="{ 'max-width': `${maxWidth}px` }">
    <video class="video-player" ref="videoRef" @click="togglePlay" >
      <source :src="videoPath" :type="`video/${format}`" />
    </video>
    <div class="video-controls">
      <div class="video-controls__time">
        <input @change="updateCurrentTime" v-model="currentTime" type="range" min="0" :max="duration" value="0" >
      </div>
      <div class="video-controls__buttons">
        <button @click="togglePlay">
          <svg v-if="isPlaying" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
            <path d="M8 19c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2s-2 .9-2 2v10c0 1.1.9 2 2 2zm6-12v10c0 1.1.9 2 2 2s2-.9 2-2V7c0-1.1-.9-2-2-2s-2 .9-2 2z" fill="#FFF"/>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
            <path d="M0 0h24v24H0z" fill="none"/>
            <path d="M8 5v14l11-7z" fill="#FFF"/>
          </svg>
        </button>
        <button @click="goBackwards(5)">
          <svg height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
            <path d="m0 0h24v24h-24z" fill="none" opacity=".87"/>
            <path d="m17.51 3.87-1.78-1.77-9.89 9.9 9.9 9.9 1.77-1.77-8.13-8.13z" fill="#FFF"/>
          </svg>
        </button>
        <button @click="goForwards(5)">
          <svg height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" transform="scale(-1, 1)  translate(0, 0)">
            <path d="m0 0h24v24h-24z" fill="none" opacity=".87"/>
            <path d="m17.51 3.87-1.78-1.77-9.89 9.9 9.9 9.9 1.77-1.77-8.13-8.13z" fill="#FFF"/>
          </svg>
        </button>
        <!-- <button @click="stop">Stop</button> -->
        <div class="video-controls__volume">
          <button @click="toggleMute">
            <svg v-if="volume >= 40" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M0 0h24v24H0z" fill="none"/>
              <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z" fill="#FFF"/>
            </svg>
            <svg v-if="40 > volume && volume >= 5" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M0 0h24v24H0z" fill="none"/>
              <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5" fill="#FFF"/>
            </svg>
            <svg v-if="5 > volume && volume >= 0" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M0 0h24v24H0z" fill="none"/>
              <path d="M3 9v6h4l5 5V4L7 9H3zm13.5" fill="#FFF"/>
            </svg>
          </button>
          <input @change="updateVolume" @dragstart="updateVolume" v-model="volume" type="range" min="0" max="100" value="100" orient="vertical" >
        </div>
        <button @click="togglePlaybackRate">x{{ playbackRate }}</button>
        <span>{{ formatTime(currentTime) }}/{{ formatTime(duration) }}</span>
        <button @click="toggleFullscreen">FS</button>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.video-player-container {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
  box-sizing: content-box;
  width: 100vw;

  &:hover,
  &:focus {

    .video-controls {
      bottom: 0;
      opacity: 1;
    }
  }

  .video-player {
    max-height: 100%;
    width: 100%;
  }
  
  .video-controls {
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: #5f5f5f81;
    color: #FFF;
    box-sizing: border-box;
    width: 100%;
    padding: 18px 12px 12px 12px;
    position: absolute;
    bottom: -90px;
    opacity: 0.2;
    left: 0;
    z-index: 10;
    transition: all 0.2s ease-in-out;

    button {
      background: none;
      border: none;
      color: #FFF;
      font-family: "Inter", sans-serif;
      padding: 4px;
    }

    .video-controls__time {
      width: 100%;
      transition: all 0.2s ease-in-out;

      input {
        overflow: hidden;
        -webkit-appearance: none;
        appearance: none;
        color: #eb5c5c;
        background: transparent;
        cursor: pointer;
        width: 100%;
        height: 10px;

        &::-moz-range-track {
          background: #9e9e9e;
          height: 3px;
          transition: all 0.1s;
        }

        &::-webkit-slider-runnable-track {
          -webkit-appearance: none;
          transition: all 0.2s ease-in-out;
          height: 10px;
          background: linear-gradient(#9e9e9e 0 0) scroll no-repeat center / 100% 4px;
        }
        
        &::-moz-range-thumb {
          -webkit-appearance: none;
          appearance: none;
          background: #eb5c5c;
          height: 10px;
          width: 10px;
          border: none;
        }

        &::-webkit-slider-thumb {
          width: 10px;
          background: linear-gradient(currentColor 0 0) scroll no-repeat left center / 50% calc(3px + 1px);
          background-color: currentColor;
          box-shadow: calc(-100vmax - 10px) 0 0 100vmax currentColor;
          border-radius: 10px;

          clip-path: polygon(
            100% -1px,
            0.125em -1px,
            0 calc((10px - 3px) * 0.5 - 0.5px),
            -100vmax calc((10px - 3px) * 0.5 - 0.5px),
            -100vmax calc(10px - calc((10px - 3px) * 0.5 - 0.5px)),
            0 calc(10px - calc((10px - 3px) * 0.5 - 0.5px)),
            0.125em 100%,
            calc(100% + 1px) calc(100% + 1px)
          );

          -webkit-appearance: none;
          transition: all 0.2s ease-in-out;
          height: 10px;
        }

        &::-moz-range-progress {
          background-color: #eb5c5c;
          transition: all 0.2s ease-in-out;
        }
      }
    }

    .video-controls__buttons {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      width: 100%;
      gap: 8px;

      span {
        margin-left: auto;

        @media screen and (max-width: 750px) {

          font-size: 12px;
        }
      }

      .video-controls__volume {
        position: relative;
  
        button {
          position: relative;
          z-index: 5;
        }
  
        input[orient=vertical] {
          accent-color: #eb5c5c;
          position: absolute;
          bottom: 5px;
          left: 7px;
          width: 18px;
          margin: 0;
          height: 0;
          opacity: 0;
          appearance: slider-vertical;
          background: #5f5f5f;
          padding: 2px 0;
          transition: all 0.2s ease-in-out;

          &::-webkit-slider-runnable-track,
          &::-moz-range-track {
            background: none !important;
            width: 10px;
            transition: all 0.1s;
          }
          
          &::-webkit-slider-thumb,
          &::-moz-range-thumb {
            -webkit-appearance: none;
            appearance: none;
            background: #eb5c5c;
            width: 12px;
            height: 12px;
            border-radius: 50%;
            border: none;
          }

          &::-moz-range-progress {
            background-color: #eb5c5c;
            width: 8px;
            transition: all 0.2s ease-in-out;
          }
        }
  
        &:hover {
          input {
            height: 100px;
            bottom: 30px;
            opacity: 1;
          }
        }
      }
    }

    &:has(.video-controls__buttons > .video-controls__volume:hover) > .video-controls__time {
      opacity: 0;
    }
  }
}

</style>