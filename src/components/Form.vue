<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-09-26 15:10
 * @LastAuthor : itchaox
 * @LastTime   : 2023-11-24 01:08
 * @desc       : 
-->
<script setup>
  import { ref, watch } from 'vue';
  import XgPlayer from './XgPlayer.vue';
  import { bitable } from '@lark-base-open/js-sdk';
  import { ElMessage } from 'element-plus';

  const base = bitable.base;

  const videoUrl = ref('');

  const isM3u8 = ref(false);
  const isLive = ref(false);

  const player = ref();

  function getPlayer(player) {
    player.value = player;
    console.log('🚀  player.value:', player.value.play());
  }

  function play() {
    if (!videoUrl.value) {
      ElMessage({
        message: '请输入视频地址',
        type: 'error',
      });
      return;
    }

    console.log('🚀111  player.value:', player.value);
    player.value?.play();
  }

  watch(videoUrl, (newValue, oldValue) => {
    const extension = newValue.toLowerCase().split('.').pop();
    isM3u8.value = extension === 'm3u8';
  });

  base.onSelectionChange(async (event) => {
    // 获取点击的字段id和记录id
    const fieldId = event.data.fieldId;
    const recordId = event.data.recordId;

    const table = await base.getActiveTable();

    if (fieldId && recordId) {
      // 获取当前数据
      let data = await table.getCellValue(fieldId, recordId);
      if (data && data[0].text !== videoUrl.value) {
        videoUrl.value = data[0].text;
      }
    }
  });
</script>

<template>
  <div>
    <div class="tip">
      <div class="tip-text title">操作步骤:</div>
      <div class="tip-text">1. 选择单元格的视频地址或输入视频地址</div>
      <div class="tip-text">2. M3U8 格式需要选择直播或回放模式</div>
      <div class="tip-text">3. 点击[确认播放]按钮即可</div>
      <div class="tip-text">Tip: 视频支持格式为 MP4、FLV、M3U8</div>
    </div>
    <div class="label">
      <div>视频地址：</div>
      <el-input
        v-model="videoUrl"
        placeholder="请输入视频地址"
        size="small"
      />
    </div>
    <div
      class="label"
      v-if="isM3u8"
    >
      <div>是否直播：</div>
      <el-switch v-model="isLive" />
    </div>

    <el-button
      type="primary"
      @click="play"
      >确认播放</el-button
    >
    <div
      class="video"
      v-if="videoUrl"
    >
      <xg-player
        :url="videoUrl"
        :getPlayer="getPlayer"
      />
    </div>
  </div>
</template>

<style scoped>
  .tip {
    color: #8f959e;
    font-size: 12px;
    margin-bottom: 24px;
    .tip-text {
      margin-bottom: 4px;
    }

    .title {
      font-size: 14px;
      margin-bottom: 8px;
    }
  }

  .label {
    display: flex;
    align-items: center;
    white-space: nowrap;
    margin-bottom: 14px;
    font-size: 14px;
  }

  .video {
    margin-top: 14px;
  }
</style>
