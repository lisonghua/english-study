<template>
  <view class="container">
    <view class="settings-section">
      <text class="section-title">发音设置</text>
      
      <view class="setting-item">
        <text class="setting-label">默认发音类型</text>
        <picker :range="accentOptions" @change="onAccentChange" :value="accentIndex">
          <view class="picker-value">{{ accentOptions[accentIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">默认人声</text>
        <picker :range="voiceOptions" @change="onVoiceChange" :value="voiceIndex">
          <view class="picker-value">{{ voiceOptions[voiceIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">默认跟读次数</text>
        <input type="number" v-model="defaultRepeatCount" class="number-input" />
      </view>
    </view>

    <view class="settings-section">
      <text class="section-title">听写设置</text>
      
      <view class="setting-item">
        <text class="setting-label">默认听写模式</text>
        <picker :range="dictationModeOptions" @change="onDictationModeChange" :value="dictationModeIndex">
          <view class="picker-value">{{ dictationModeOptions[dictationModeIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">默认听写顺序</text>
        <picker :range="orderOptions" @change="onOrderChange" :value="orderIndex">
          <view class="picker-value">{{ orderOptions[orderIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">默认朗读次数</text>
        <input type="number" v-model="defaultReadCount" class="number-input" />
      </view>
    </view>

    <view class="settings-section">
      <text class="section-title">数据管理</text>
      
      <view class="data-actions">
        <button class="data-btn export-btn" @click="exportData">
          <text class="btn-icon">📤</text>
          <text class="btn-text">导出单词本</text>
        </button>
        
        <button class="data-btn import-btn" @click="importData">
          <text class="btn-icon">📥</text>
          <text class="btn-text">导入单词本</text>
        </button>
        
        <button class="data-btn clear-btn" @click="clearAllData">
          <text class="btn-icon">🗑️</text>
          <text class="btn-text">清空所有数据</text>
        </button>
      </view>
    </view>

    <view class="settings-section">
      <text class="section-title">关于</text>
      
      <view class="about-info">
        <view class="info-item">
          <text class="info-label">版本</text>
          <text class="info-value">1.0.0</text>
        </view>
        
        <view class="info-item">
          <text class="info-label">单词总数</text>
          <text class="info-value">{{ totalWords }}</text>
        </view>
        
        <view class="info-item">
          <text class="info-label">已掌握单词</text>
          <text class="info-value">{{ learnedWords }}</text>
        </view>
        
        <view class="info-item">
          <text class="info-label">掌握率</text>
          <text class="info-value">{{ masteryRate }}%</text>
        </view>
      </view>
    </view>

    <view class="save-section">
      <button class="save-btn" @click="saveSettings">保存设置</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      accentOptions: ['美式发音', '英式发音'],
      accentIndex: 0,
      voiceOptions: ['男声', '女声'],
      voiceIndex: 0,
      defaultRepeatCount: 3,
      dictationModeOptions: ['英文→中文', '中文→英文'],
      dictationModeIndex: 0,
      orderOptions: ['顺序', '随机', '倒序'],
      orderIndex: 0,
      defaultReadCount: 2,
      totalWords: 0,
      learnedWords: 0,
      masteryRate: 0
    }
  },
  onLoad() {
    this.loadSettings()
    this.loadStats()
  },
  methods: {
    loadSettings() {
      const settings = uni.getStorageSync('appSettings') || {}
      
      if (settings.accentIndex !== undefined) {
        this.accentIndex = settings.accentIndex
      }
      if (settings.voiceIndex !== undefined) {
        this.voiceIndex = settings.voiceIndex
      }
      if (settings.defaultRepeatCount !== undefined) {
        this.defaultRepeatCount = settings.defaultRepeatCount
      }
      if (settings.dictationModeIndex !== undefined) {
        this.dictationModeIndex = settings.dictationModeIndex
      }
      if (settings.orderIndex !== undefined) {
        this.orderIndex = settings.orderIndex
      }
      if (settings.defaultReadCount !== undefined) {
        this.defaultReadCount = settings.defaultReadCount
      }
    },
    loadStats() {
      const stats = uni.getStorageSync('vocabStats') || {
        total: 0,
        learned: 0,
        mastery: 0
      }
      
      this.totalWords = stats.total
      this.learnedWords = stats.learned
      this.masteryRate = stats.mastery
    },
    onAccentChange(e) {
      this.accentIndex = e.detail.value
    },
    onVoiceChange(e) {
      this.voiceIndex = e.detail.value
    },
    onDictationModeChange(e) {
      this.dictationModeIndex = e.detail.value
    },
    onOrderChange(e) {
      this.orderIndex = e.detail.value
    },
    saveSettings() {
      const settings = {
        accentIndex: this.accentIndex,
        voiceIndex: this.voiceIndex,
        defaultRepeatCount: parseInt(this.defaultRepeatCount) || 3,
        dictationModeIndex: this.dictationModeIndex,
        orderIndex: this.orderIndex,
        defaultReadCount: parseInt(this.defaultReadCount) || 2
      }
      
      uni.setStorageSync('appSettings', settings)
      
      uni.showToast({
        title: '设置已保存',
        icon: 'success'
      })
    },
    exportData() {
      const words = uni.getStorageSync('vocabWords') || []
      
      if (words.length === 0) {
        uni.showToast({
          title: '没有可导出的单词',
          icon: 'none'
        })
        return
      }
      
      // 转换为 CSV 格式
      let csvContent = '英文，中文\n'
      words.forEach(word => {
        csvContent += `${word.en},${word.cn}\n`
      })
      
      // #ifdef H5
      const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
      const link = document.createElement('a')
      link.href = URL.createObjectURL(blob)
      link.download = `单词本_${new Date().getTime()}.csv`
      link.click()
      // #endif
      
      // #ifdef APP-PLUS
      const fileName = `_doc/单词本_${new Date().getTime()}.csv`
      plus.io.convertLocalFileSystemURL(fileName, (entry) => {
        entry.file((file) => {
          const writer = plus.io.createWriter(file)
          writer.write(csvContent)
          writer.close()
          
          uni.showToast({
            title: '导出成功',
            icon: 'success'
          })
        })
      })
      // #endif
      
      uni.showToast({
        title: '导出成功',
        icon: 'success'
      })
    },
    importData() {
      uni.chooseFile({
        success: (res) => {
          const filePath = res.tempFilePaths[0]
          
          // #ifdef H5
          const file = res.files[0]
          const reader = new FileReader()
          reader.onload = (e) => {
            const content = e.target.result
            this.parseCSV(content)
          }
          reader.readAsText(file, 'UTF-8')
          // #endif
          
          // #ifdef APP-PLUS
          plus.io.resolveLocalFileSystemURL(filePath, (entry) => {
            entry.file((file) => {
              const reader = new plus.io.FileReader()
              reader.onload = (e) => {
                this.parseCSV(e.target.result)
              }
              reader.readAsText(file)
            })
          })
          // #endif
        }
      })
    },
    parseCSV(content) {
      const lines = content.split('\n')
      const newWords = []
      
      for (let i = 1; i < lines.length; i++) {
        const line = lines[i].trim()
        if (!line) continue
        
        const parts = line.split(',')
        if (parts.length >= 2) {
          newWords.push({
            en: parts[0].trim(),
            cn: parts.slice(1).join(',').trim()
          })
        }
      }
      
      if (newWords.length === 0) {
        uni.showToast({
          title: '未找到有效单词',
          icon: 'none'
        })
        return
      }
      
      const existingWords = uni.getStorageSync('vocabWords') || []
      const mergedWords = [...existingWords, ...newWords]
      
      uni.setStorageSync('vocabWords', mergedWords)
      
      // 更新统计
      const stats = uni.getStorageSync('vocabStats') || {
        total: 0,
        learned: 0,
        mastery: 0
      }
      stats.total = mergedWords.length
      if (stats.total > 0) {
        stats.mastery = Math.round((stats.learned / stats.total) * 100)
      }
      uni.setStorageSync('vocabStats', stats)
      
      uni.showToast({
        title: `成功导入 ${newWords.length} 个单词`,
        icon: 'success'
      })
      
      // 更新本地显示
      this.loadStats()
    },
    clearAllData() {
      uni.showModal({
        title: '确认清空',
        content: '确定要清空所有单词和学习记录吗？此操作不可恢复！',
        success: (res) => {
          if (res.confirm) {
            uni.removeStorageSync('vocabWords')
            uni.removeStorageSync('vocabStats')
            uni.removeStorageSync('learnedWords')
            uni.removeStorageSync('recentWords')
            
            this.totalWords = 0
            this.learnedWords = 0
            this.masteryRate = 0
            
            uni.showToast({
              title: '已清空所有数据',
              icon: 'success'
            })
          }
        }
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

.settings-section {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.section-title {
  font-size: 18px;
  font-weight: bold;
  color: #333;
  margin-bottom: 15px;
  display: block;
}

.setting-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.setting-item:last-child {
  border-bottom: none;
}

.setting-label {
  font-size: 16px;
  color: #333;
}

.picker-value {
  font-size: 14px;
  color: #666;
  padding: 8px 15px;
  background: #f5f5f5;
  border-radius: 6px;
}

.number-input {
  width: 80px;
  padding: 8px 10px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 14px;
  text-align: center;
}

.data-actions {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.data-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 15px;
  border-radius: 8px;
  border: none;
  font-size: 16px;
}

.btn-icon {
  font-size: 20px;
  margin-right: 8px;
}

.export-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.import-btn {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: #fff;
}

.clear-btn {
  background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
  color: #fff;
}

.about-info {
  padding: 10px 0;
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.info-item:last-child {
  border-bottom: none;
}

.info-label {
  font-size: 15px;
  color: #666;
}

.info-value {
  font-size: 15px;
  color: #333;
  font-weight: 500;
}

.save-section {
  margin-top: 30px;
}

.save-btn {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 18px;
  font-weight: 500;
}
</style>
