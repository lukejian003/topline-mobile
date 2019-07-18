<template>
  <van-popup
    :value="value"
    @input="$emit('input',$event)"
    position="bottom"
    :style="{ height: '95%' }"
  >
    <!-- 我的频道 -->
    <div class="channels">
      <div class="head">
        <van-row type="flex" justify="space-between">
          <van-col span="15">
            <span class="my">我的频道</span>
            <span class="tips">点击进入频道</span>
          </van-col>
          <van-col span="5">
            <van-button on round plain type="danger" size="small">编辑</van-button>
          </van-col>
        </van-row>
      </div>
      <div class="content">
        <van-grid :gutter="10" clickable>
          <van-grid-item v-for="(channelItem,index) in userChannels" :key="channelItem.id">
            <span slot="text" class="text" :class="{active:index === acitveIndex}">
              {{channelItem.name}}
              <!-- <van-icon class="close-icon" name="close" /> -->
            </span>
          </van-grid-item>
        </van-grid>
      </div>
    </div>
    <!-- /我的频道 -->
    <!-- 频道推荐 -->
    <div class="channels">
      <div class="head">
        <van-row type="flex" justify="space-between">
          <van-col span="15">
            <span class="my">频道推荐</span>
            <span class="tips">点击添加频道</span>
          </van-col>
        </van-row>
      </div>
      <div class="content">
        <van-grid :gutter="10" clickable>
          <van-grid-item v-for="item in 30" :key="item">
            <span slot="text"  class="text" :class="{active:index === acitveIndex}"><van-icon name="plus" />{{item}} </span>
          </van-grid-item>
        </van-grid>
      </div>
    </div>
    <!-- /频道推荐 -->
  </van-popup>
</template>

<script>
export default {
  name: 'HomeChannel',
  props: {
    value: {
      tape: Boolean,
      default: false
    },
    userChannels: {
      type: Array,
      default: () => []
    },
    acitveIndex: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {}
  }
}
</script>

<style lang="less" scoped>
.head {
  margin-bottom: 10px;
  margin-top: 10px;
  .my {
    padding-left: 10px;
    padding-right: 10px;
    font-size: 20px;
  }
  .tips {
    font-size: 14px;
    color: #666;
  }
}
.close-icon {
  position: absolute;
  font-size: 18px;
  top: -5px;
  right: -5px;
}
.text {
  font-size: 14px;
}
.active {
  color: red;
}
</style>
