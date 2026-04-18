<template>
  <view class="container">
    <view class="settings-section">
      <text class="section-title">听写设置</text>
      
      <view class="setting-item">
        <text class="setting-label">听写模式</text>
        <picker :range="modeOptions" @change="onModeChange" :value="modeIndex">
          <view class="picker-value">{{ modeOptions[modeIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">听写顺序</text>
        <picker :range="orderOptions" @change="onOrderChange" :value="orderIndex">
          <view class="picker-value">{{ orderOptions[orderIndex] }}</view>
        </picker>
      </view>
      
      <view class="setting-item">
        <text class="setting-label">朗读次数</text>
        <input type="number" v-model="readCount" class="number-input" />
      </view>
      
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
    </view>

    <view class="range-section">
      <text class="section-title">听写范围</text>
      <view class="range-inputs">
        <view class="range-item">
          <text class="range-label">从第</text>
          <input type="number" v-model="startRange" class="range-input" />
          <text class="range-label">个单词</text>
        </view>
        <view class="range-item">
          <text class="range-label">到第</text>
          <input type="number" v-model="endRange" class="range-input" />
          <text class="range-label">个单词</text>
        </view>
      </view>
      <text class="range-hint">共 {{ totalWords }} 个单词，留空表示全部</text>
    </view>

    <view class="dictation-area" v-if="isDictating && currentWord">
      <view class="word-display">
        <view class="word-prompt">
          <text class="prompt-label">{{ modeIndex === 0 ? '请写出中文释义' : '请写出英文单词' }}</text>
        </view>
        
        <view class="answer-area">
          <textarea 
            v-model="userAnswer" 
            class="answer-input" 
            placeholder="请输入答案"
            maxlength="100"
          />
        </view>
        
        <view class="action-buttons">
          <button class="action-btn submit-btn" @click="submitAnswer">提交</button>
          <button class="action-btn skip-btn" @click="skipWord">跳过</button>
        </view>
      </view>
      
      <view class="feedback" v-if="showFeedback">
        <view class="feedback-content" :class="isCorrect ? 'correct' : 'incorrect'">
          <text class="feedback-icon">{{ isCorrect ? '✓' : '✗' }}</text>
          <view class="feedback-text">
            <text class="correct-answer">正确答案：{{ correctAnswer }}</text>
            <text class="your-answer" v-if="!isCorrect">你的答案：{{ userAnswer }}</text>
          </view>
        </view>
        <button class="next-btn" @click="nextWord">下一个</button>
      </view>
    </view>

    <view class="start-section" v-if="!isDictating">
      <button class="start-btn" @click="startDictation">开始听写</button>
    </view>

    <view class="result-section" v-if="showResult">
      <text class="section-title">听写结果</text>
      <view class="result-stats">
        <view class="stat-item">
          <text class="stat-number">{{ totalQuestions }}</text>
          <text class="stat-label">总题数</text>
        </view>
        <view class="stat-item">
          <text class="stat-number">{{ correctCount }}</text>
          <text class="stat-label">正确数</text>
        </view>
        <view class="stat-item">
          <text class="stat-number">{{ accuracy }}%</text>
          <text class="stat-label">正确率</text>
        </view>
      </view>
      <button class="restart-btn" @click="resetDictation">重新听写</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      allWords: [],
      totalWords: 0,
      dictationWords: [],
      modeOptions: ['英文→中文', '中文→英文'],
      modeIndex: 0,
      orderOptions: ['顺序', '随机', '倒序'],
      orderIndex: 0,
      readCount: 2,
      accentOptions: ['美式发音', '英式发音'],
      accentIndex: 0,
      voiceOptions: ['男声', '女声'],
      voiceIndex: 0,
      startRange: '',
      endRange: '',
      isDictating: false,
      currentIndex: 0,
      currentWord: null,
      userAnswer: '',
      showFeedback: false,
      isCorrect: false,
      correctAnswer: '',
      totalQuestions: 0,
      correctCount: 0,
      showResult: false
    }
  },
  onLoad() {
    this.loadWords()
  },
  methods: {
    loadWords() {
      this.allWords = uni.getStorageSync('vocabWords') || []
      this.totalWords = this.allWords.length
      
      if (this.totalWords > 0) {
        this.endRange = this.totalWords
      }
    },
    onModeChange(e) {
      this.modeIndex = e.detail.value
    },
    onOrderChange(e) {
      this.orderIndex = e.detail.value
    },
    onAccentChange(e) {
      this.accentIndex = e.detail.value
    },
    onVoiceChange(e) {
      this.voiceIndex = e.detail.value
    },
    prepareDictationWords() {
      let start = this.startRange ? parseInt(this.startRange) - 1 : 0
      let end = this.endRange ? parseInt(this.endRange) : this.totalWords
      
      start = Math.max(0, Math.min(start, this.totalWords - 1))
      end = Math.max(start + 1, Math.min(end, this.totalWords))
      
      let words = this.allWords.slice(start, end)
      
      // 根据顺序调整
      if (this.orderIndex === 2) {
        words = words.reverse()
      } else if (this.orderIndex === 1) {
        words = this.shuffleArray(words)
      }
      
      return words
    },
    shuffleArray(array) {
      const shuffled = [...array]
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
      }
      return shuffled
    },
    startDictation() {
      if (this.totalWords === 0) {
        uni.showToast({
          title: '请先导入单词',
          icon: 'none'
        })
        return
      }
      
      this.dictationWords = this.prepareDictationWords()
      
      if (this.dictationWords.length === 0) {
        uni.showToast({
          title: '没有可听写的单词',
          icon: 'none'
        })
        return
      }
      
      this.isDictating = true
      this.showResult = false
      this.currentIndex = 0
      this.correctCount = 0
      this.totalQuestions = this.dictationWords.length
      
      this.loadCurrentWord()
    },
    loadCurrentWord() {
      if (this.currentIndex >= this.dictationWords.length) {
        this.finishDictation()
        return
      }
      
      this.currentWord = this.dictationWords[this.currentIndex]
      this.userAnswer = ''
      this.showFeedback = false
      
      // 播放读音
      this.playAudio()
    },
    playAudio() {
      if (!this.currentWord) return
      
      const text = this.modeIndex === 0 ? this.currentWord.en : this.currentWord.cn
      const accent = this.accentIndex === 0 ? 'en-US' : 'en-GB'
      
      // 朗读指定次数
      for (let i = 0; i < this.readCount; i++) {
        setTimeout(() => {
          this.speakText(text, accent)
        }, i * 1500)
      }
    },
    speakText(text, accent) {
      // #ifdef H5
      const utterance = new SpeechSynthesisUtterance(text)
      utterance.lang = accent
      utterance.rate = 0.7
      window.speechSynthesis.speak(utterance)
      // #endif
      
      // #ifdef APP-PLUS
      plus.audio.player.play({
        text: text,
        lang: accent
      })
      // #endif
    },
    submitAnswer() {
      if (!this.userAnswer.trim()) {
        uni.showToast({
          title: '请输入答案',
          icon: 'none'
        })
        return
      }
      
      this.checkAnswer()
    },
    checkAnswer() {
      const userAns = this.userAnswer.trim().toLowerCase()
      
      if (this.modeIndex === 0) {
        // 英文→中文
        const correctAns = this.currentWord.cn.trim()
        this.isCorrect = userAns.includes(correctAns) || correctAns.includes(userAns)
        this.correctAnswer = `${this.currentWord.en} - ${this.currentWord.cn}`
      } else {
        // 中文→英文
        const correctAns = this.currentWord.en.trim().toLowerCase()
        this.isCorrect = userAns === correctAns
        this.correctAnswer = this.currentWord.en
      }
      
      if (this.isCorrect) {
        this.correctCount++
      }
      
      this.showFeedback = true
      
      uni.vibrateShort()
    },
    skipWord() {
      this.showFeedback = true
      this.isCorrect = false
      
      if (this.modeIndex === 0) {
        this.correctAnswer = `${this.currentWord.en} - ${this.currentWord.cn}`
      } else {
        this.correctAnswer = this.currentWord.en
      }
    },
    nextWord() {
      this.currentIndex++
      this.loadCurrentWord()
    },
    finishDictation() {
      this.isDictating = false
      this.showResult = true
    },
    resetDictation() {
      this.showResult = false
      this.isDictating = false
      this.userAnswer = ''
      this.showFeedback = false
    }
  },
  computed: {
    accuracy() {
      if (this.totalQuestions === 0) return 0
      return Math.round((this.correctCount / this.totalQuestions) * 100)
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

.settings-section,
.range-section {
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

.range-inputs {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
}

.range-item {
  display: flex;
  align-items: center;
  flex: 1;
}

.range-label {
  font-size: 14px;
  color: #666;
  margin: 0 5px;
}

.range-input {
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 14px;
  text-align: center;
}

.range-hint {
  font-size: 12px;
  color: #999;
  display: block;
}

.dictation-area {
  background: #fff;
  border-radius: 12px;
  padding: 25px;
  margin-bottom: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.word-display {
  text-align: center;
}

.word-prompt {
  margin-bottom: 20px;
}

.prompt-label {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.answer-area {
  margin-bottom: 20px;
}

.answer-input {
  width: 100%;
  min-height: 100px;
  padding: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  font-size: 16px;
  line-height: 1.6;
}

.action-buttons {
  display: flex;
  gap: 15px;
}

.action-btn {
  flex: 1;
  padding: 12px;
  border-radius: 8px;
  font-size: 16px;
  border: none;
}

.submit-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
}

.skip-btn {
  background: #f5f5f5;
  color: #666;
}

.feedback {
  margin-top: 20px;
  padding-top: 20px;
  border-top: 1px solid #f0f0f0;
}

.feedback-content {
  display: flex;
  align-items: center;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.feedback-content.correct {
  background: #f0f9ff;
  border: 1px solid #bae6fd;
}

.feedback-content.incorrect {
  background: #fff1f0;
  border: 1px solid #ffa39e;
}

.feedback-icon {
  font-size: 24px;
  font-weight: bold;
  margin-right: 15px;
}

.correct .feedback-icon {
  color: #52c41a;
}

.incorrect .feedback-icon {
  color: #f5222d;
}

.feedback-text {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.correct-answer {
  font-size: 15px;
  color: #333;
  font-weight: 500;
}

.your-answer {
  font-size: 13px;
  color: #666;
}

.next-btn {
  width: 100%;
  padding: 12px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 16px;
}

.start-section {
  margin-top: 30px;
}

.start-btn {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 18px;
  font-weight: 500;
}

.result-section {
  background: #fff;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.result-stats {
  display: flex;
  justify-content: space-around;
  padding: 20px 0;
  margin-bottom: 20px;
}

.stat-item {
  text-align: center;
}

.stat-number {
  font-size: 32px;
  font-weight: bold;
  color: #667eea;
  display: block;
}

.stat-label {
  font-size: 13px;
  color: #666;
  margin-top: 5px;
  display: block;
}

.restart-btn {
  width: 100%;
  padding: 15px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
  border-radius: 10px;
  font-size: 16px;
  font-weight: 500;
}
</style>
