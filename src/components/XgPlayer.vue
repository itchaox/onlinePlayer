<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-06-28 20:09
 * @LastAuthor : itchaox
 * @LastTime   : 2023-12-02 19:01
 * @desc       : 西瓜播放器组件
-->

<script setup lang="ts">
  import { onMounted, watch, ref, onUnmounted } from 'vue';
  import Player from 'xgplayer';
  import FlvPlugin from 'xgplayer-flv';
  import HlsPlugin from 'xgplayer-hls';

  import 'xgplayer/dist/index.min.css';

  interface Props {
    // 视频地址
    url: string;
    videoType: string;
    isAutoPlay: boolean;

    // m3u8 视频类型
    m3u8Type?: 'live' | 'replay'; // live 直播；replay 回放
    width?: number;
    height?: number;
  }

  const props = defineProps<Props>();
  const player = ref();

  function init() {
    console.log('test');

    if (player.value) {
      player.value.destroy(); // 销毁播放器
      player.value = null; // 将实例引用置空
    }

    player.value = new Player({
      id: 'tVideo',
      width: 235,
      height: 245,
      autoplay: props.isAutoPlay,
      url: props.url || 'http://vjs.zencdn.net/v/oceans.mp4',
      videoAttributes: {
        crossOrigin: 'anonymous',
      },
      pip: true,
      screenShot: true,
      lang: 'zh',
      ...getVideoConfig(),
      ignores: ['fullscreen'],
    });
  }

  onMounted(async () => {
    init();
  });

  //暴露方法

  defineExpose({
    init,
  });

  /**
   * @desc  : 获取视频配置
   * @return { Object } 视频配置
   */
  function getVideoConfig() {
    console.log('🚀  props.m3u8Type:', props.m3u8Type);
    switch (props.videoType) {
      case 'm3u8':
        return {
          plugins: [HlsPlugin],
          isLive: props.m3u8Type === 'live',
        };
      case 'flv':
        return {
          plugins: [FlvPlugin],
          isLive: true,
        };
      case 'mp4':
        return {};
    }
  }

  const container = ref(null);
  let isResizing = ref(false);

  const startResizing = () => {
    isResizing.value = true;
    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseup', stopResizing);
  };

  const stopResizing = () => {
    isResizing.value = false;
    document.removeEventListener('mousemove', handleMouseMove);
  };

  const handleMouseMove = (event) => {
    if (isResizing.value) {
      const newWidth = event.clientX - container.value.getBoundingClientRect().left;
      const newHeight = event.clientY - container.value.getBoundingClientRect().top;

      container.value.style.width = `${newWidth < 200 ? 200 : newWidth}px`;
      container.value.style.height = `${newHeight < 200 ? 200 : newHeight}px`;
    }
  };

  onMounted(() => {
    container.value = document.getElementById('tVideo');
  });

  onUnmounted(() => {
    document.removeEventListener('mousemove', handleMouseMove);
    document.removeEventListener('mouseup', stopResizing);
  });
</script>

<template>
  <!-- <div id="tVideo">
    <video
      id="resizable-video"
      controls
      src="123"
    ></video>
    <div
      id="resize-handle"
      @mousedown="startResizing"
    ></div>
  </div> -->

  <div id="tVideo">
    <div
      id="resize-handle"
      ref="resizeHandle"
      @mousedown="startResizing"
    ></div>
  </div>
</template>

<style scoped>
  #tVideo {
    position: fixed;
    width: 100%; /* 初始宽度 */
    overflow: hidden;
  }

  #tVideo video {
    width: 100%;
    height: 100%;
  }

  #resize-handle {
    z-index: 9999;
    position: absolute;
    bottom: 0;
    right: 0;
    width: 10px;
    height: 10px;
    cursor: nwse-resize;
  }
</style>
