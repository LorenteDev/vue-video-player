<script setup lang="ts">
import { ref, onMounted, onUnmounted, Ref, watch } from 'vue'

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
  if (videoRef.value.volume > 0) {
    videoRef.value.volume = 0
    volume.value = 0
  } else {
    videoRef.value.volume = 0.5
    volume.value = 50
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

watch(() => props.videoPath, () => {
  videoRef.value.load()
  pause()
})

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
    pause()
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
  <div class="video-player-container" ref="containerRef" :style="{ 'max-width': `${maxWidth}px` }" :name="props.videoPath">
    <video class="video-player" ref="videoRef" @click="togglePlay" >
      <source :src="videoPath" :type="`video/${format}`" />
    </video>
    <div class="video-controls">
      <div class="video-controls__time">
        <input @change="updateCurrentTime" v-model="currentTime" type="range" min="0" :max="duration" value="0" aria-label="Progress bar" >
      </div>
      <div class="video-controls__buttons">
        <button @click="togglePlay" :aria-label="isPlaying ? 'Pause' : 'Play'">
          <svg v-if="isPlaying" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="30" width="30">
            <path d="M8 19c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2s-2 .9-2 2v10c0 1.1.9 2 2 2zm6-12v10c0 1.1.9 2 2 2s2-.9 2-2V7c0-1.1-.9-2-2-2s-2 .9-2 2z" fill="#FFF"/>
          </svg>
          <svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" height="30" width="30">
            <path d="M8 6.82v10.36c0 .79.87 1.27 1.54.84l8.14-5.18c.62-.39.62-1.29 0-1.69L9.54 5.98C8.87 5.55 8 6.03 8 6.82z" fill="#FFF"/>
          </svg>
        </button>
        <button @click="goBackwards(5)" aria-label="Go 5 seconds backwards">
          <svg height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
            <path d="m17.51 3.87-1.78-1.77-9.89 9.9 9.9 9.9 1.77-1.77-8.13-8.13z" fill="#FFF"/>
          </svg>
        </button>
        <button @click="goForwards(5)" aria-label="Go 5 seconds forwards">
          <svg height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg" transform="scale(-1, 1)  translate(0, 0)">
            <path d="m17.51 3.87-1.78-1.77-9.89 9.9 9.9 9.9 1.77-1.77-8.13-8.13z" fill="#FFF"/>
          </svg>
        </button>
        <div class="video-controls__volume">
          <button @click="toggleMute" aria-label="Volume">
            <svg v-if="volume >= 40" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M3 9v6h4l5 5V4L7 9H3zm13.5 3c0-1.77-1.02-3.29-2.5-4.03v8.05c1.48-.73 2.5-2.25 2.5-4.02zM14 3.23v2.06c2.89.86 5 3.54 5 6.71s-2.11 5.85-5 6.71v2.06c4.01-.91 7-4.49 7-8.77s-2.99-7.86-7-8.77z" fill="#FFF"/>
            </svg>
            <svg v-if="40 > volume && volume >= 5" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M 3 9 L 3 15 L 7 15 L 12 20 L 12 4 L 7 9 L 3 9 Z M 16.5 12 C 16.5 10.23 15.48 8.71 14 7.97 L 14 16.02 C 15.48 15.29 16.5 13.77 16.5 12 Z M 15.142 31.934 C 15.17 32.322 14.957 32.485 14.652 32.095 C 15.606 32.547 14.738 31.97 14.891 32.287 C 15.044 32.604 14.744 31.203 14.656 31.248 L 15.116 31.7 C 15.058 31.874 15.243 32.515 15.215 31.913 C 15.187 31.311 15.561 32.065 15.142 31.934 Z" fill="#FFF"></path>
            </svg>
            <svg v-if="5 > volume && volume >= 0" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24">
              <path d="M3 9v6h4l5 5V4L7 9H3z" fill="#FFF"/>
            </svg>
          </button>
          <input @change="updateVolume" @dragstart="updateVolume" v-model="volume" type="range" min="0" max="100" value="100" orient="vertical" aria-label="Volume selector" >
        </div>
        <button @click="togglePlaybackRate" aria-label="Playback rate">
          <span>
            x{{ playbackRate }}
          </span>
        </button>
        <span>{{ formatTime(currentTime) }}/{{ formatTime(duration) }}</span>
        <button @click="toggleFullscreen" aria-label="Toggle fullscreen">
          <svg fill="none" height="24" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg">
            <g fill="#FFF">
              <path d="m5 6c0-.55228.44772-1 1-1h2c.55228 0 1-.44772 1-1s-.44772-1-1-1h-2c-1.65685 0-3 1.34315-3 3v2c0 .55228.44772 1 1 1s1-.44772 1-1z"/>
              <path d="m5 18c0 .5523.44772 1 1 1h2c.55228 0 1 .4477 1 1s-.44772 1-1 1h-2c-1.65685 0-3-1.3431-3-3v-2c0-.5523.44772-1 1-1s1 .4477 1 1z"/>
              <path d="m18 5c.5523 0 1 .44772 1 1v2c0 .55228.4477 1 1 1s1-.44772 1-1v-2c0-1.65685-1.3431-3-3-3h-2c-.5523 0-1 .44772-1 1s.4477 1 1 1z"/>
              <path d="m19 18c0 .5523-.4477 1-1 1h-2c-.5523 0-1 .4477-1 1s.4477 1 1 1h2c1.6569 0 3-1.3431 3-3v-2c0-.5523-.4477-1-1-1s-1 .4477-1 1z"/>
            </g>
          </svg>
        </button>
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
  width: 100%;

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
      display: flex;
      align-items: center;
      justify-content: center;

      span {
        font-size: 16px;

        @media screen and (max-width: 750px) {
          font-size: 12px;
        }
      }

      span,
      path {
        transition: all 0.2s;
      }


      &:hover {
        color: #c2463d;

        svg {
          path {
            fill: #c2463d;
          }
        }
      }
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
          height: 2px;
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
          writing-mode: bt-lr;
          appearance: slider-vertical;
          -webkit-appearance: slider-vertical;
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

          @media screen and (max-width: 750px) {
            display: none;
          }
        }
  
        &:hover {
          input {
            height: 100px;
            bottom: 36px;
            opacity: 1;
          }

          path {
            fill: #eb2416;
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
