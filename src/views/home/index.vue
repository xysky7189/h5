<template>
  <div>
    <van-nav-bar title="首页" fixed/>
    <van-tabs class="channel-tabs" v-model="activeChannelIndex">
      <div slot="nav-right" class="wap-nav" @click="isChannelShow = true">
        <van-icon name="wap-nav" />
      </div>
      <van-tab
        v-for="channelItem in channels"
        :key="channelItem.id"
        :title="channelItem.name"
      >
        <van-pull-refresh
          v-model="channelItem.pullRefreshLoading"
          @refresh="onRefresh"
          :success-text="channelItem.pullSuccessText"
          :success-duration="1000"
        >
          <van-list
            v-model="channelItem.upLoading"
            :finished="channelItem.finished"
            finished-text="没有更多了"
            @load="onLoad"
          >
            <van-cell
              v-for="item in channelItem.articles"
              :key="item.art_id"
              :title="item.title"
            />
          </van-list>
        </van-pull-refresh>
      </van-tab>
    </van-tabs>
    <van-tabbar route>
      <van-tabbar-item icon="home-o" to="/">首页</van-tabbar-item>
      <van-tabbar-item icon="search" to="/qa">问答</van-tabbar-item>
      <van-tabbar-item icon="friends-o" to="/video">视频</van-tabbar-item>
      <van-tabbar-item icon="setting-o" to="my">我的</van-tabbar-item>
    </van-tabbar>
    <home-channel
      v-model="isChannelShow"
      :user-channels="channels"
      :active-index="activeChannelIndex"
    />
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
      channels: [],
      activeChannelIndex: 0,
      list: [],
      loading: false,
      finished: false,
      pullRefreshLoading: false,
      isChannelShow: false
    }
  },
  computed: {
    activeChannel () {
      return this.channels[this.activeChannelIndex]
    }
  },
  watch: {
    async '$store.state.user' () {
      await this.loadChannels()
      this.activeChannel.upLoading = true
      this.onload()
    }
  },
  async created () {
    console.log('组件重新 created 渲染了')
    await this.loadChannels()
  },
  methods: {
    async onLoad () {
      await this.$sleep(800)
      const articles = await this.loadArticles()
      this.activeChannel.articles.push(...articles)
      this.activeChannel.upLoading = false
      // setTimeout(() => {
      //   for (let i = 0; i < 10; i++) {
      //     this.list.push(this.list.length + 1)
      //   }
      //   this.loading = false
      //   if (this.list.length >= 40) {
      //     this.finished = true
      //   }
      // }, 1000)
    },
    async onRefresh () {
      const data = await getArticles({
        channelId: this.activeChannel.id,
        timestamp: Date.now(),
        withTop: 1
      })
      if (data.results.length) {
        this.activeChannel.articles = data.results
        this.activeChannel.timestamp = data.pre_timestamp
        this.activeChannel.pullSuccessText = '更新完成'
        this.onLoad()
      }
      this.activeChannel.pullSuccessText = '暂无数据更新'
      this.activeChannel.pullRefreshLoading = false
    },
    async loadChannels () {
      let channels = []
      const { user } = this.$store.state
      if (user) {
        channels = (await getUserChannels()).channels
      } else {
        const localChannels = JSON.parse(window.localStorage.getItem('channels'))
        if (localChannels) {
          channels = localChannels
        } else {
          channels = (await getUserChannels()).channels
        }
      }
      channels.forEach(item => {
        item.articles = []
        item.timestamp = Date.now()
        item.finished = false
        item.upLoading = false
        item.pullRefreshLoading = false
        item.pullSuccessText = ''
      })
      this.channels = channels
    },

    // async loadChannels () {
    //   try {
    //     let channels = []
    //     const localChannels = window.localStorage.getItem('channels')
    //     if (localChannels) {
    //       channels = localChannels
    //     } else {
    //       channels = (await getUserChannels()).channels
    //     }
    //     channels.forEach(item => {
    //       item.articles = []
    //       item.timestamp = Date.now()
    //       item.finished = false
    //       item.upLoading = false
    //       item.pullRefreshLoading = false
    //       item.pullSuccessText = ''
    //     })
    //     this.channels = channels
    //   } catch (err) {
    //     console.log(err)
    //   }
    // },
    async loadArticles () {
      const { id: channelId, timestamp } = this.activeChannel
      try {
        const data = await getArticles({
          channelId,
          timestamp,
          withTop: 1
        })
        if (data.pre_timestamp && data.results.length === 0) {
          this.activeChannel.timestamp = data.pre_timestamp
          return this.loadArticles()
        }
        if (data.results.length) {
          this.activeChannel.timestamp = data.pre_timestamp
          return data.results
        }
      } catch (err) {
        console.log(err)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.channel-tabs {
  margin-bottom: 100px;
}
.channel-tabs /deep/ .van-tabs__wrap {
  position: fixed;
  top: 92px;
}
.channel-tabs /deep/ .van-tabs__content {
  margin-top: 92px;
}
.channel-tabs .wap-nav {
  position: sticky;
  right: 0;
  display: flex;
  align-items: center;
  background: #fff;
  opacity: .7;
}
</style>
