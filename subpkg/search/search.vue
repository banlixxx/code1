<template>
  <view>
    <view class="search-box">
      <uni-search-bar @input="input" placeholder="请输入搜索内容" radius="100" cancelButton="none" :focus="true"></uni-search-bar>
    </view>
    
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="item.goods_name" @click="gotoDetail(item)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
    <view class="history-box" v-else>
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clean"></uni-icons>
      </view>
      <view class="history-list">
        <uni-tag :text="item" v-for="(item, i) in historyList" :key="i" inverted="true" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer: null,
        kw: '',
        searchResults: [],
        historyList: []
      };
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    methods: {
      input(e) {
        clearTimeout(this.timer)
        this.timer = setTimeout(() => {
          this.kw = e.trim()
          this.getSearchList()
        }, 500)
      },
      async getSearchList() {
        if (this.kw.length === 0) {
          this.searchResults = []
          return
        }
        const {data: res} = await uni.$http.get('/api/public/v1/goods/qsearch', {query: this.kw})
        if (res.meta.status !== 200) return uni.$showMsg()
        this.searchResults = res.message
      },
      gotoDetail(item) {
        this.saveSearchHistory()
        uni.navigateTo({url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id})
      },
      saveSearchHistory() {
        if (this.historyList.includes(this.kw)) {
          const index = this.historyList.indexOf(this.kw)
          this.historyList.splice(index, 1)
        }
        this.historyList.unshift(this.kw)
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
      },
      clean() {
        this.historyList = []
        uni.setStorageSync('kw', '[]')
      },
      gotoGoodsList(kw) {
        const index = this.historyList.indexOf(kw)
        this.historyList.splice(index, 1)
        this.historyList.unshift(kw)
        uni.setStorageSync('kw', JSON.stringify(this.historyList))
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query=' + kw
        })
      }
    }
  }
</script>

<style lang="scss">
.search-box {
  position: sticky;
  top: 0;
  z-index: 999;
}

.sugg-list {
  padding: 0 5px;
  .sugg-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 12px;
    padding: 13px 0;
    border-bottom: 1px solid #efefef;
    .goods-name {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
  }
}

.history-box {
  padding: 0 5px;
  .history-title {
    display: flex;
    justify-content: space-between;
    height: 40px;
    align-items: center;
    font-size: 13px;
    border-bottom: 1px solid #efefef;
  }
  .history-list {
    display: flex;
    flex-wrap: wrap;
    .uni-tag {
      margin-top: 5px;
      margin-right: 5px;
      border: 0;
    }
  }
}
</style>
