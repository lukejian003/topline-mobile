<template>
  <div class="home">
    <van-nav-bar title="首页" fixed />
    <van-tabs class="channel-tabs" v-model="activeChannelIndex">
      <div slot="nav-right">
        <van-icon name="wap-nav" class="wap-nav" @click="isChannelShow = true" />
      </div>
      <van-tab v-for="channelItem in channels" :key="channelItem.id" :title="channelItem.name">
        <van-pull-refresh
          v-model="channelItem.downPullLoading"
          @refresh="onRefresh"
          :success-text="channelItem.downPullSuccessText"
          :success-duration="1000"
        >
          <van-list
            v-model="channelItem.upPullLoading"
            :finished="channelItem.upPullFinished"
            finished-text="没有更多了"
            @load="onLoad"
          >
            <van-cell
              v-for="articleItem in channelItem.articles"
              :key="articleItem.art_id"
              :title="articleItem.title"
            />
          </van-list>
        </van-pull-refresh>
      </van-tab>
    </van-tabs>
    <!-- 频道管理 -->
    <HomeChannel
      v-model="isChannelShow"
      :user-channels="channels"
      :acitve-index.sync="activeChannelIndex"
    />
    <!-- 频道管理 -->
  </div>
</template>

<script>
import { getUserChannels } from '@/api/channel'
import { getArticles } from '@/api/article'
import HomeChannel from './components/channel'
export default {
  name: 'HomeIndex',
  components: {
    HomeChannel
  },
  data () {
    return {
      activeChannelIndex: 0,
      channels: [], // 频道列表数据
      isChannelShow: false // 控制频道
    }
  },
  computed: {
    // 当前激活的频道
    activeChannel () {
      return this.channels[this.activeChannelIndex]
    }
  },
  watch: {
    // 监视容器中的user用户
    // 由于路由缓存了，这里监视用户的登录状态，如果登陆了，则重新加载用户的频道列表
    async '$store.state.user' () {
      this.loadChannels()
      // 频道数据改变，重新加载当前激活频道的数据
      this.activeChannel.upPullLoading = true
      await this.onLoad()
      // this.activeChannel.upPullLoading = true
      // await this.onLoad()
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
        // 获取本地存储中的频道列表
        const loadChannels = window.localStorage.getItem('channels')
        if (loadChannels) {
          channels = JSON.parse(loadChannels)
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
    async onRefresh () {
      const { activeChannel } = this
      // 备份加载下一页数据的时间戳
      // const timestamp = activeChannel.timestamp
      activeChannel.timestamp = Date.now()
      const data = await this.loadAritcles()
      if (data.results.length) {
        // 重置最新数据列表
        activeChannel.articles = data.results
        // 重置后，数据中的pre_timestamp就是下一页的时间戳
        activeChannel.timestamp = data.pre_timestamp
        activeChannel.downPullSuccessText = '更新成功'
        // 当下拉刷新数据无法满足一屏，使用onLoading
        this.onLoad()
      } else {
        // 没有最新数据，提示已是最新数据
        activeChannel.downPullSuccessText = '已是最新数据'
      }
      // 下拉刷新结束，取消loading状态
      activeChannel.downPullLoading = false
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
.wap-nav {
  position: fixed;
  right: 0;
  padding: 10px;
  background-color: #fff;
}

.channel-tabs /deep/ .van-tabs__wrap {
  position: fixed;
  top: 46px;
  margin-right: 44px;
  box-shadow: 0 -0.5px 0.1px #000;
}
.channel-tabs /deep/ .van-tabs__content {
  margin-top: 50px;
}
</style>
