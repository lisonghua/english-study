<template>
  <view class="container">
    <view class="header">
      <text class="title">欢迎使用单词记忆助手</text>
      <text class="subtitle">帮助高中生轻松背单词</text>
    </view>

    <view class="stats-card">
      <view class="stat-item">
        <text class="stat-number">{{ totalWords }}</text>
        <text class="stat-label">总单词数</text>
      </view>
      <view class="stat-item">
        <text class="stat-number">{{ learnedWords }}</text>
        <text class="stat-label">已学习</text>
      </view>
      <view class="stat-item">
        <text class="stat-number">{{ masteryRate }}%</text>
        <text class="stat-label">掌握率</text>
      </view>
    </view>

    <view class="quick-actions">
      <view class="action-card" @click="goToImport">
        <view class="action-icon">📷</view>
        <text class="action-text">拍照导入</text>
      </view>
      <view class="action-card" @click="goToPractice">
        <view class="action-icon">📖</view>
        <text class="action-text">开始背诵</text>
      </view>
      <view class="action-card" @click="goToDictation">
        <view class="action-icon">✍️</view>
        <text class="action-text">单词听写</text>
      </view>
      <view class="action-card" @click="goToSettings">
        <view class="action-icon">⚙️</view>
        <text class="action-text">设置</text>
      </view>
    </view>

    <view class="recent-words">
      <text class="section-title">最近学习的单词</text>
      <view class="word-list">
        <view class="word-item" v-for="(word, index) in recentWords" :key="index">
          <text class="word-en">{{ word.en }}</text>
          <text class="word-cn">{{ word.cn }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      totalWords: 0,
      learnedWords: 0,
      masteryRate: 0,
      recentWords: []
    }
  },
  onLoad() {
    this.loadStats()
    this.loadRecentWords()
  },
  methods: {
    loadStats() {
      // 从本地存储加载统计数据
      const stats = uni.getStorageSync('vocabStats') || {
        total: 0,
        learned: 0,
        mastery: 0
      }
      this.totalWords = stats.total
      this.learnedWords = stats.learned
      this.masteryRate = stats.mastery
    },
    loadRecentWords() {
      // 从本地存储加载最近学习的单词
      const words = uni.getStorageSync('recentWords') || []
      this.recentWords = words.slice(0, 5)
    },
    goToImport() {
      uni.navigateTo({
        url: '/pages/import/import'
      })
    },
    goToPractice() {
      uni.navigateTo({
        url: '/pages/practice/practice'
      })
    },
    goToDictation() {
      uni.navigateTo({
        url: '/pages/dictation/dictation'
      })
    },
    goToSettings() {
      uni.navigateTo({
        url: '/pages/settings/settings'
      })
    }
  }
}
</script>

<style scoped>
.container {
  padding: 20px;
  background-color: #F5F5F5;
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 8px;
}

.subtitle {
  font-size: 14px;
  color: #666;
}

.stats-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 15px;
  padding: 20px;
  display: flex;
  justify-content: space-around;
  margin-bottom: 25px;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.stat-item {
  text-align: center;
}

.stat-number {
  font-size: 28px;
  font-weight: bold;
  color: #fff;
  display: block;
}

.stat-label {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.8);
  margin-top: 5px;
  display: block;
}

.quick-actions {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 15px;
  margin-bottom: 25px;
}

.action-card {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  text-align: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.action-icon {
  font-size: 32px;
  margin-bottom: 10px;
  display: block;
}

.action-text {
  font-size: 14px;
  color: #333;
  font-weight: 500;
}

.recent-words {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
  display: block;
}

.word-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.word-item:last-child {
  border-bottom: none;
}

.word-en {
  font-size: 16px;
  color: #333;
  font-weight: 500;
}

.word-cn {
  font-size: 14px;
  color: #666;
}
</style>
