<template>
  <VideoPlayer :videoPath="videoFile" format="mp4" maxWidth="900" />
</template>

<script setup lang="ts">
import VideoPlayer from './components/VideoPlayer.vue'
import videoFile from '../src/assets/video.mp4'
</script>