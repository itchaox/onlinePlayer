<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-06-28 20:09
 * @LastAuthor : itchaox
 * @LastTime   : 2023-11-24 00:59
 * @desc       : 西瓜播放器组件
-->

<script setup lang="ts">
  import { onMounted, watchEffect } from 'vue';
  import Player from 'xgplayer';
  import FlvPlugin from 'xgplayer-flv';
  import HlsPlugin from 'xgplayer-hls';

  import 'xgplayer/dist/index.min.css';

  interface Props {
    // 视频地址
    url: string;

    // m3u8 视频类型
    m3u8Type?: 'live' | 'replay'; // live 直播；replay 回放
    width?: number;
    height?: number;
    getPlayer: Function;
  }

  const props = defineProps<Props>();

  onMounted(async () => {
    const player = new Player({
      id: 'tVideo',
      width: 235,
      height: 245,
      videoInit: false,
      // autoplay: true,
      // autoplayMuted: true,
      // url: 'https://test-streams.mux.dev/x36xhzz/x36xhzz.m3u8',
      // url: 'http://vjs.zencdn.net/v/oceans.mp4',
      // poster: '//lf3-static.bytednsdoc.com/obj/eden-cn/nupenuvpxnuvo/xgplayer_doc/poster.jpg',
      url: props.url,
      videoAttributes: {
        crossOrigin: 'anonymous',
      },
      lang: 'zh',
      ...getVideoConfig(props.url),
    });
    props.getPlayer(player);
  });

  /**
   * @desc  : 获取视频配置
   * @param  { string } url：视频地址
   * @return { Object } 视频配置
   */
  function getVideoConfig(url: string) {
    const extension = url.toLowerCase().split('.').pop();
    switch (extension) {
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
</script>

<template>
  <div id="tVideo"></div>
</template>

<style lang="scss" scoped></style>
