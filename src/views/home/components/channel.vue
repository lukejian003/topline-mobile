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
            <van-button
              on
              round
              plain
              type="danger"
              size="small"
              @click="isEdit = !isEdit"
            >{{isEdit ? '完成':'编辑'}}</van-button>
          </van-col>
        </van-row>
      </div>
      <div class="content">
        <van-grid :gutter="10" clickable>
          <van-grid-item
            v-for="(channelItem,index) in userChannels"
            :key="channelItem.id"
            @click="handleUserChannelClick(channelItem,index)"
          >
            <span slot="text" class="text" :class="{active:index === acitveIndex && !isEdit}">
              {{channelItem.name}}
              <van-icon class="close-icon" v-show="isEdit && !aliveChannels.includes(channelItem.name)" name="close" />
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
          <van-grid-item
            v-for="item in recommeChannels"
            :key="item.id"
            @click="hanleAddChannel(item)"
          >
            <span slot="text" class="text">{{item.name}}</span>
          </van-grid-item>
        </van-grid>
      </div>
    </div>
    <!-- /频道推荐 -->
  </van-popup>
</template>

<script>
import {
  getAllChannels,
  deleteUserChannels,
  resetUserChannels
} from '@/api/channel'
import { mapState } from 'vuex'

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
    return {
      // getAllChannels: [], //存储所有频道列表
      allChannels: [], // 所有的频道列表
      isEdit: false,
      aliveChannels: ['推荐']
    }
  },
  computed: {
    // 过滤出不包含用户频道的列表数据
    recommeChannels () {
      const duplicates = this.userChannels.map(item => item.id)
      return this.allChannels.filter(item => !duplicates.includes(item.id))
    },
    // Vuex的辅助方法，用来讲state中的数据映射到本地计算属性
    // 就是user = this.$store.state.user
    ...mapState(['user'])
  },
  created () {
    this.loadAllChannels()
  },
  methods: {
    async loadAllChannels () {
      const data = await getAllChannels()
      this.allChannels = data.channels
    },
    async hanleAddChannel (item) {
      // 将点击添加的频道添加到用户中
      this.userChannels.push(item)
      if (this.user) {
        // 如果用户已登录，则将数据请求添加到后端
        const data = this.userChannels.slice(1).map((item, index) => {
          return {
            id: item.id, // 频道id
            seq: index + 2
          }
        })
        await resetUserChannels(data)
        return
      }
      // 如果未登录，则将数据持久化到本地存储
      window.localStorage.setItem(
        'channels',
        JSON.stringify(this.userChannels)
      )
    },
    // 编辑频道
    changeChannel (channelItem, index) {
      this.$emit('update:active-index', index)
      this.$emit('input', false)
    },
    // 删除频道
    deleteChannel (channelItem, index) {
      this.userChannels.splice(index, 1)
      if (this.user) {
        // 登录：发请求删除
        deleteUserChannels(channelItem.id)
        console.log(channelItem.id)
        return
      }
      // 未登陆删除本地存储的数据
      window.localStorage.setItem(
        'channels',
        JSON.stringify(this.userChannels)
      )
    },
    handleUserChannelClick (channelItem, index) {
      if (!this.isEdit) {
        // 非编辑状态：切换频道
        this.changeChannel(channelItem, index)
      } else {
        if (!this.aliveChannels.includes(channelItem.name)) {
          // 编辑状态：删除频道
          this.deleteChannel(channelItem, index)
        }
      }
    }
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
    font-size: 18px;
  }
  .tips {
    font-size: 12px;
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
  font-size: 12px;
}
.active {
  color: red;
}
</style>
