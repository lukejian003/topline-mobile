<template>
  <div>
    <div class="home">
      <van-nav-bar title="首页" fixed />
      <van-tabs class="channel-tabs" v-model="activeChannelIndex">
        <van-tab v-for="channelItem in channels" :key="channelItem.id" :title="channelItem.name">
          <van-pull-refresh v-model="isLoading" @refresh="onRefresh">
            <van-list v-model="loading" :finished="finished" finished-text="没有更多了" @load="onLoad">
              <van-cell v-for="item in 40" :key="item" :title="item" />
            </van-list>
          </van-pull-refresh>
        </van-tab>
      </van-tabs>
    </div>
  </div>
</template>

<script>
import { getUserChannels } from '@/api/channel'
export default {
  name: 'HomeIndex',
  data () {
    return {
      activeChannelIndex: 0,
      list: [],
      loading: false,
      finished: true,
      isLoading: false,
      channels: [] // 频道列表数据
    }
  },
  created () {
    this.loadChannels()
  },
  methods: {
    async loadChannels () {
      const { user } = this.$store.state
      let channels = []
      if (user) {
        const data = await getUserChannels()
        channels = data.channels
        console.log(data)
      } else {
        const loadChannels = window.localStorage.getItem('channels')
        if (loadChannels) {
          channels = loadChannels
        } else {
          const data = await getUserChannels()
          channels = data.channels
        }
      }
      this.channels = channels
    },
    onLoad () {
      // 异步更新数据
      setTimeout(() => {
        for (let i = 0; i < 10; i++) {
          this.list.push(this.list.length + 1)
        }
        // 加载状态结束
        this.loading = false

        // 数据全部加载完成
        if (this.list.length >= 40) {
          this.finished = true
        }
      }, 500)
    },
    onRefresh () {
      console.log('123')
    }
  }
}
</script>

<style lang="less" scoped>
.channel-tabs {
  margin-bottom: 50px;
}
.channel-tabs /deep/ .van-tabs__wrap {
  position: fixed;
  top: 46px;
}
.channel-tabs /deep/ .van-tabs__content {
  margin-top: 50px
}
</style>
