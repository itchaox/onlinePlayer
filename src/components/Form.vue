<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-09-26 15:10
 * @LastAuthor : itchaox
 * @LastTime   : 2023-12-03 21:56
 * @desc       : 
-->
<script setup>
  import { ref, watch, watchEffect } from 'vue';
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

  const volume = ref(60);

  const clear = () => {
    historyVideoList.value = [];
  };
</script>

<template>
  <div class="player">
    <div class="tip">
      <div class="tip-text title">操作步骤:</div>
      <div class="tip-text">1. 选择视频格式</div>
      <div class="tip-text">2. 视频地址: 选择单元格自动获取或手动输入</div>
      <div class="tip-text">3. 自行配置播放器功能</div>
      <div class="tip-text">4. 点击【运行】按钮即可</div>
    </div>
    <div class="label">
      <div class="text">视频格式：</div>
      <el-radio-group
        v-model="videoType"
        size="small"
      >
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
        size="small"
      />
    </div>
    <el-collapse v-model="activeNames">
      <el-collapse-item
        title="播放记录"
        name="1"
      >
        <div class="collapse-list">
          <template v-if="historyVideoList?.length > 0">
            <span
              @click="clear"
              class="clear"
            >
              <el-icon><DeleteFilled /></el-icon>
              <span class="clear-text">清空播放记录</span>
            </span>
            <div
              v-for="(item, index) in historyVideoList"
              :key="index"
              :title="item"
              class="collapse-item"
              @click="changeUrl(item)"
            >
              <div>
                <el-icon><VideoCameraFilled /></el-icon>
              </div>
              <div class="collapse-item-text">
                {{ item }}
              </div>
            </div>
          </template>
          <template v-else>
            <el-empty
              description="暂无播放记录"
              :image-size="50"
            />
          </template>
        </div>
      </el-collapse-item>
    </el-collapse>

    <div class="switch-list">
      <div class="switch">
        <div>预设音量：</div>
        <el-input-number
          v-model="volume"
          :min="0"
          :max="100"
          size="small"
          :step="10"
          controls-position="right"
        />
      </div>
      <div class="switch-second">
        <div class="switch">
          <div>自动播放：</div>
          <el-switch v-model="isAutoPlay" />
        </div>
        <div
          class="switch"
          v-if="videoType !== 'mp4'"
        >
          <div>是否直播：</div>
          <el-switch v-model="isLive" />
        </div>
      </div>
    </div>

    <el-button
      class="button"
      type="primary"
      @click="play"
      size="small"
      ><el-icon :size="20"><CaretRight /></el-icon>运行</el-button
    >

    <div class="video">
      <xg-player
        ref="player"
        :video-type="videoType"
        :url="videoUrl"
        :isAutoPlay="isAutoPlay"
        :volume="volume"
        :m3u8Type="isLive ? 'live' : 'replay'"
      />
    </div>
  </div>
</template>

<style scoped>
  .player {
    font-family: LarkHackSafariFont, LarkEmojiFont, LarkChineseQuote, -apple-system, BlinkMacSystemFont, Helvetica Neue,
      Tahoma, PingFang SC, Microsoft Yahei, Arial, Hiragino Sans GB, sans-serif, Apple Color Emoji, Segoe UI Emoji,
      Segoe UI Symbol, Noto Color Emoji;
    font-weight: 300;

    :deep(.el-collapse-item__header) {
      font-weight: 300;
    }
  }

  .tip {
    color: #8f959e;
    font-size: 12px;
    margin-bottom: 14px;
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
    margin-bottom: 12px;
    font-size: 14px;
  }

  .switch-list {
    margin: 10px 0;
  }

  .switch-second {
    display: flex;
  }

  .switch {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    white-space: nowrap;
    margin-bottom: 4px;
    margin-right: 6px;
    font-size: 14px;
  }

  .video {
    margin-top: 14px;
  }

  .collapse-list {
    text-align: left;
  }

  .clear {
    display: inline-flex;
    align-items: center;
    color: #f56c6c;
    &:hover {
      cursor: pointer;
      opacity: 0.6;
    }
  }

  .clear-text {
    margin-left: 4px;
  }

  .collapse-item {
    display: flex;
    align-items: center;
  }

  .collapse-item-text {
    margin-left: 4px;
    font-size: 12px;
    height: 12px;
    line-height: 12px;
    cursor: pointer;
    white-space: nowrap;

    &:hover {
      color: #1456f0;
      border-bottom: 1px solid #1456f0;
    }
  }
</style>
