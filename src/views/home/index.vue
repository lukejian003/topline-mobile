<template>
  <div>
    <div class="home">
      <van-nav-bar title="首页" fixed />
      <van-tabs class="channel-tabs" v-model="activeChannelIndex">
        <van-tab v-for="channelItem in channels" :key="channelItem.id" :title="channelItem.name">
          <van-pull-refresh v-model="isLoading" @refresh="onRefresh">
            <van-list v-model="channelItem.upPullLoading" :finished="channelItem.upPullFinished" finished-text="没有更多了" @load="onLoad">
              <van-cell v-for="articleItem in channelItem.articles" :key="articleItem.art_id" :title="articleItem.title" />
            </van-list>
          </van-pull-refresh>
        </van-tab>
      </van-tabs>
    </div>
  </div>
</template>

<script>
import { getUserChannels } from '@/api/channel'
import { getArticles } from '@/api/article'

export default {
  name: 'HomeIndex',
  data () {
    return {
      activeChannelIndex: 0,
      list: [],
      loading: false,
      finished: false,
      isLoading: false,
      channels: [] // 频道列表数据
    }
  },
  computed: {
    // 当前激活的频道
    activeChannel () {
      return this.channels[this.activeChannelIndex]
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
      } else {
        const loadChannels = window.localStorage.getItem('channels')
        if (loadChannels) {
          channels = loadChannels
        } else {
          const data = await getUserChannels()
          channels = data.channels
        }
      }
      channels.forEach(item => {
        item.articles = [] // 用来存储当前文章的列表
        item.timestamp = Date.now() // 存储下一页数据的时间戳
        item.downPullLoading = false // 控制当前频道的下拉刷新loading状态
        item.upPullLoading = false // 控制当前频道的上拉加载更多的loading状态
        item.upPullFinished = false // 控制当前频道数据是否加载完毕
      })
      this.channels = channels
    },
    // 上拉加载更多
    async onLoad () {
      await this.$sleep(800)
      console.log('onLoad')
      let data = []
      data = await this.loadAritcles()
      // 如果没有pre_timestamp 并且数组是空的，则意味着没有数据了
      if (!data.pre_timestamp && !data.results.length) {
        // 设置该频道数据已经加载完毕，组件会自动给出提示，并且不再onLoading
        this.activeChannel.upPullFinished = true
        // 取消loading
        this.activeChannel.ssupPullLoading = false
        // 代码不要往后继续执行
        return
      }

      if (data.pre_timestamp && !data.results.length) {
        this.activeChannel.timestamp = data.pre_timestamp
        data = await this.loadAritcles()
      }
      this.activeChannel.timestamp = data.pre_timestamp
      // this.activeChannel.articles = data.results
      this.activeChannel.articles.push(...data.results)
      this.activeChannel.upPullLoading = false
    },
    // 下拉刷新
    onRefresh () {
      console.log('onRefresh')
    },
    async loadAritcles () {
      const { id: channelId, timestamp } = this.activeChannel
      const data = await getArticles({
        channelId, // 当前激活频道id
        timestamp, // 用于请求分页数据的时间戳
        withTop: 1 // 是否包含置顶数据
      })
      return data
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
  margin-top: 50px;
}
</style>
