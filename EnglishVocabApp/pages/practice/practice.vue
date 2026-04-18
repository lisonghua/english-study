<template>
  <view class="container">
    <view class="word-card" v-if="currentWord">
      <view class="word-en">{{ currentWord.en }}</view>
      <view class="word-cn" v-if="showTranslation">{{ currentWord.cn }}</view>
      
      <view class="audio-controls">
        <button class="control-btn" @click="playAudio">🔊 播放</button>
        <button class="control-btn" @click="toggleTranslation">
          {{ showTranslation ? '隐藏释义' : '显示释义' }}
        </button>
      </view>
    </view>

    <view class="practice-settings">
      <view class="setting-item">
        <text class="setting-label">发音类型</text>
        <picker :range="accentOptions" @change="onAccentChange" :value="accentIndex">
          <view class="picker-value">{{ accentOptions[accentIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">人声</text>
        <picker :range="voiceOptions" @change="onVoiceChange" :value="voiceIndex">
          <view class="picker-value">{{ voiceOptions[voiceIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">跟读次数</text>
        <input type="number" v-model="repeatCount" class="number-input" />
      </view>
    </view>

    <view class="progress-info">
      <text>进度：{{ currentIndex + 1 }} / {{ totalWords }}</text>
    </view>

    <view class="navigation-buttons">
      <button class="nav-btn prev-btn" @click="prevWord" :disabled="currentIndex === 0">上一个</button>
      <button class="nav-btn next-btn" @click="nextWord">下一个</button>
      <button class="nav-btn mark-btn" @click="markAsLearned">
        {{ isLearned ? '取消掌握' : '标记为已掌握' }}
      </button>
    </view>

    <view class="action-buttons">
      <button class="action-btn start-btn" @click="startPractice" v-if="!isPracticing">开始跟读</button>
      <button class="action-btn stop-btn" @click="stopPractice" v-else>停止跟读</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      allWords: [],
      currentWord: null,
      currentIndex: 0,
      totalWords: 0,
      showTranslation: false,
      accentOptions: ['美式发音', '英式发音'],
      accentIndex: 0,
      voiceOptions: ['男声', '女声'],
      voiceIndex: 0,
      repeatCount: 3,
      isPracticing: false,
      isLearned: false,
      audioContext: null
    }
  },
  onLoad() {
    this.loadWords()
    this.initAudio()
  },
  methods: {
    loadWords() {
      this.allWords = uni.getStorageSync('vocabWords') || []
      this.totalWords = this.allWords.length
      
      if (this.totalWords > 0) {
        this.currentWord = this.allWords[0]
        this.checkIfLearned(0)
      }
    },
    initAudio() {
      // 初始化音频上下文
      this.audioContext = uni.createInnerAudioContext()
    },
    playAudio() {
      if (!this.currentWord) return
      
      const accent = this.accentIndex === 0 ? 'en-US' : 'en-GB'
      const gender = this.voiceIndex === 0 ? 'male' : 'female'
      
      // 使用 TTS 朗读单词
      this.speakWord(this.currentWord.en, accent, gender)
    },
    speakWord(text, accent, gender) {
      // 使用 uni-app 的语音合成
      uni.createInnerAudioContext()
      
      // 注意：实际项目中需要接入 TTS 服务
      // 这里使用浏览器自带的 SpeechSynthesis API（H5 端）
      // #ifdef H5
      const utterance = new SpeechSynthesisUtterance(text)
      utterance.lang = accent
      utterance.rate = 0.8
      utterance.pitch = gender === 'male' ? 0.8 : 1.2
      
      for (let i = 0; i < this.repeatCount; i++) {
        window.speechSynthesis.speak(utterance)
      }
      // #endif
      
      // #ifdef APP-PLUS
      // Android/iOS 原生 TTS
      plus.audio.player.play({
        text: text,
        lang: accent
      })
      // #endif
    },
    toggleTranslation() {
      this.showTranslation = !this.showTranslation
    },
    onAccentChange(e) {
      this.accentIndex = e.detail.value
    },
    onVoiceChange(e) {
      this.voiceIndex = e.detail.value
    },
    prevWord() {
      if (this.currentIndex > 0) {
        this.currentIndex--
        this.currentWord = this.allWords[this.currentIndex]
        this.showTranslation = false
        this.checkIfLearned(this.currentIndex)
      }
    },
    nextWord() {
      if (this.currentIndex < this.totalWords - 1) {
        this.currentIndex++
        this.currentWord = this.allWords[this.currentIndex]
        this.showTranslation = false
        this.checkIfLearned(this.currentIndex)
      } else {
        uni.showToast({
          title: '已经是最后一个单词',
          icon: 'none'
        })
      }
    },
    checkIfLearned(index) {
      const learnedWords = uni.getStorageSync('learnedWords') || []
      this.isLearned = learnedWords.includes(this.allWords[index].en)
    },
    markAsLearned() {
      if (!this.currentWord) return
      
      let learnedWords = uni.getStorageSync('learnedWords') || []
      
      if (this.isLearned) {
        // 取消掌握
        learnedWords = learnedWords.filter(w => w !== this.currentWord.en)
        this.isLearned = false
      } else {
        // 标记为掌握
        learnedWords.push(this.currentWord.en)
        this.isLearned = true
        
        // 更新统计数据
        this.updateStats()
      }
      
      uni.setStorageSync('learnedWords', learnedWords)
      
      uni.showToast({
        title: this.isLearned ? '已标记为掌握' : '已取消掌握',
        icon: 'success'
      })
    },
    updateStats() {
      const stats = uni.getStorageSync('vocabStats') || {
        total: this.totalWords,
        learned: 0,
        mastery: 0
      }
      
      const learnedWords = uni.getStorageSync('learnedWords') || []
      stats.learned = learnedWords.length
      stats.total = this.totalWords
      
      if (stats.total > 0) {
        stats.mastery = Math.round((stats.learned / stats.total) * 100)
      }
      
      uni.setStorageSync('vocabStats', stats)
    },
    startPractice() {
      this.isPracticing = true
      this.autoPlaySequence()
    },
    stopPractice() {
      this.isPracticing = false
      // 停止所有正在播放的音频
      // #ifdef H5
      window.speechSynthesis.cancel()
      // #endif
    },
    autoPlaySequence() {
      if (!this.isPracticing || !this.currentWord) return
      
      this.playAudio()
      
      // 根据跟读次数和间隔自动播放
      const interval = setInterval(() => {
        if (!this.isPracticing) {
          clearInterval(interval)
          return
        }
        
        this.playAudio()
      }, 2000)
    }
  },
  beforeDestroy() {
    this.stopPractice()
    if (this.audioContext) {
      this.audioContext.destroy()
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

.word-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px;
  padding: 40px 20px;
  text-align: center;
  margin-bottom: 25px;
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.3);
}

.word-en {
  font-size: 36px;
  font-weight: bold;
  color: #fff;
  margin-bottom: 15px;
}

.word-cn {
  font-size: 20px;
  color: rgba(255, 255, 255, 0.9);
  margin-top: 10px;
}

.audio-controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 25px;
}

.control-btn {
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
  border: none;
  border-radius: 25px;
  padding: 10px 20px;
  font-size: 14px;
}

.practice-settings {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
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

.progress-info {
  text-align: center;
  margin-bottom: 20px;
  color: #666;
  font-size: 14px;
}

.navigation-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.nav-btn {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  font-size: 14px;
  border: none;
}

.prev-btn,
.next-btn {
  background: #fff;
  color: #333;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.mark-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.action-buttons {
  margin-top: 20px;
}

.action-btn {
  width: 100%;
  padding: 15px;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 500;
  border: none;
}

.start-btn {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: #fff;
}

.stop-btn {
  background: linear-gradient(135deg, #eb3349 0%, #f45c43 100%);
  color: #fff;
}
</style>
