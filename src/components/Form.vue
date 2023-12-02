<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-09-26 15:10
 * @LastAuthor : itchaox
 * @LastTime   : 2023-12-03 07:22
 * @desc       : 
-->
<script setup>
  import { ref, watch } from 'vue';
  import XgPlayer from './XgPlayer.vue';
  import { bitable } from '@lark-base-open/js-sdk';
  import { ElMessage } from 'element-plus';

  const base = bitable.base;

  const videoUrl = ref('');

  const isLive = ref(false);

  const videoType = ref('mp4');

  const player = ref();

  const isAutoPlay = ref(false);

  watch(videoType, (newValue, oldValue) => {
    isLive.value = false;
  });

  function play() {
    if (!videoUrl.value) {
      ElMessage({
        message: '请输入视频地址',
        type: 'error',
      });
      return;
    }

    player.value?.init();
    historyVideoList.value?.unshift(videoUrl.value);
  }

  base.onSelectionChange(async (event) => {
    // 获取点击的字段id和记录id
    const fieldId = event.data.fieldId;
    const recordId = event.data.recordId;

    const table = await base.getActiveTable();

    if (fieldId && recordId) {
      // 获取当前数据
      let data = await table.getCellValue(fieldId, recordId);
      if (data && data[0].text !== videoUrl.value) {
        if (isValidVideoUrlSuffix(data[0].text)) {
          videoUrl.value = data[0].text;
          // 点击历史记录元素时, 自动切换视频格式
          const extension = data[0].text.toLowerCase().split('.').pop();
          videoType.value = extension;

          ElMessage({
            message: '成功获取选中单元格的视频地址~',
            type: 'success',
            duration: 1500,
          });
        } else {
          ElMessage({
            message: '选中单元格的视频地址格式有误!',
            type: 'error',
            duration: 1500,
          });
        }
      }
    }
  });

  function isValidVideoUrlSuffix(url) {
    // 正则表达式用于匹配MP4、FLV和M3U8格式的视频链接后缀
    const validSuffixes = /\.(mp4|flv|m3u8)$/i;
    return validSuffixes.test(url);
  }

  const activeNames = ref();
  const historyVideoList = ref([]);

  const changeUrl = (url) => {
    videoUrl.value = url;

    // 点击历史记录元素时, 自动切换视频格式
    const extension = url.toLowerCase().split('.').pop();
    videoType.value = extension;
  };
</script>

<template>
  <div class="player">
    <div class="tip">
      <div class="tip-text title">操作步骤:</div>
      <div class="tip-text">1. 选择视频格式</div>
      <div class="tip-text">2. 视频地址获取方式: 选择单元格后自动获取或手动输入</div>
      <div class="tip-text">3. 自行配置是否直播和自动播放</div>
      <div class="tip-text">4. 点击【确认播放】按钮即可</div>
    </div>
    <div class="label">
      <div class="text">视频格式：</div>
      <el-radio-group v-model="videoType">
        <el-radio-button label="mp4">MP4</el-radio-button>
        <el-radio-button label="flv">FLV</el-radio-button>
        <el-radio-button label="m3u8">M3U8</el-radio-button>
      </el-radio-group>
    </div>

    <div class="label">
      <div>视频地址：</div>
      <el-input
        :title="videoUrl"
        v-model="videoUrl"
        placeholder="请输入视频地址"
      />
    </div>
    <el-collapse v-model="activeNames">
      <el-collapse-item
        title="历史记录"
        name="1"
      >
        <div class="collapse-list">
          <template v-if="historyVideoList?.length > 0">
            <div
              v-for="(item, index) in historyVideoList"
              :key="index"
              :title="item"
              class="collapse-item"
              @click="changeUrl(item)"
            >
              {{ item }}
            </div>
          </template>
          <template v-else>
            <el-empty
              description="暂无历史记录"
              :image-size="100"
            />
          </template>
        </div>
      </el-collapse-item>
    </el-collapse>

    <div
      class="label"
      v-if="videoType !== 'mp4'"
    >
      <div>是否直播：</div>
      <el-switch v-model="isLive" />
    </div>

    <div class="label">
      <div>自动播放：</div>
      <el-switch v-model="isAutoPlay" />
    </div>

    <el-button
      type="primary"
      @click="play"
      >确认播放</el-button
    >

    <div class="video">
      <xg-player
        ref="player"
        :video-type="videoType"
        :url="videoUrl"
        :isAutoPlay="isAutoPlay"
        :m3u8Type="isLive ? 'live' : 'replay'"
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

  .collapse-list {
    /* max-height: 150px;
    overflow: auto; */
  }

  .collapse-item {
    cursor: pointer;
    white-space: nowrap;
    &:hover {
      color: #1456f0;
    }
  }
</style>
