<template>
  <view class="container">
    <view class="upload-section">
      <view class="upload-area" @click="takePhoto">
        <view class="upload-icon">📷</view>
        <text class="upload-text">点击拍照导入单词</text>
        <text class="upload-hint">支持自动识别一页单词</text>
      </view>
      
      <view class="upload-area alt-upload" @click="chooseImage">
        <view class="upload-icon">🖼️</view>
        <text class="upload-text">从相册选择</text>
      </view>
    </view>

    <view class="preview-section" v-if="imagePath">
      <text class="section-title">预览图片</text>
      <image :src="imagePath" mode="aspectFit" class="preview-image"></image>
      <button class="recognize-btn" @click="recognizeText">开始识别</button>
    </view>

    <view class="result-section" v-if="recognizedWords.length > 0">
      <text class="section-title">识别结果 ({{ recognizedWords.length }}个单词)</text>
      <view class="word-list">
        <view class="word-item" v-for="(word, index) in recognizedWords" :key="index">
          <input 
            class="word-input-en" 
            v-model="word.en" 
            placeholder="英文单词"
          />
          <input 
            class="word-input-cn" 
            v-model="word.cn" 
            placeholder="中文释义"
          />
          <view class="delete-btn" @click="deleteWord(index)">×</view>
        </view>
      </view>
      
      <view class="action-buttons">
        <button class="save-btn" @click="saveWords">保存单词</button>
        <button class="clear-btn" @click="clearAll">清空</button>
      </view>
    </view>

    <view class="loading-mask" v-if="isLoading">
      <view class="loading-content">
        <view class="loading-spinner"></view>
        <text class="loading-text">正在识别，请稍候...</text>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      imagePath: '',
      recognizedWords: [],
      isLoading: false
    }
  },
  methods: {
    takePhoto() {
      uni.chooseImage({
        count: 1,
        sourceType: ['camera'],
        success: (res) => {
          this.imagePath = res.tempFilePaths[0]
          this.recognizedWords = []
        }
      })
    },
    chooseImage() {
      uni.chooseImage({
        count: 1,
        sourceType: ['album'],
        success: (res) => {
          this.imagePath = res.tempFilePaths[0]
          this.recognizedWords = []
        }
      })
    },
    async recognizeText() {
      if (!this.imagePath) {
        uni.showToast({
          title: '请先选择图片',
          icon: 'none'
        })
        return
      }

      this.isLoading = true
      
      try {
        // 调用 OCR 识别接口（这里使用模拟数据，实际需接入百度 OCR 或其他 OCR 服务）
        const result = await this.performOCR(this.imagePath)
        
        // 解析识别结果，提取单词
        this.parseRecognizedText(result)
        
        uni.showToast({
          title: `识别成功 ${this.recognizedWords.length} 个单词`,
          icon: 'success'
        })
      } catch (error) {
        uni.showToast({
          title: '识别失败，请重试',
          icon: 'none'
        })
      } finally {
        this.isLoading = false
      }
    },
    performOCR(imagePath) {
      // TODO: 实际项目中需要接入 OCR SDK
      // 这里返回模拟数据用于演示
      return new Promise((resolve) => {
        setTimeout(() => {
          resolve('apple\nbanana\norange\ngrape\nstudent\nteacher\nschool\nbook')
        }, 1500)
      })
    },
    parseRecognizedText(text) {
      const lines = text.split('\n').filter(line => line.trim().length > 0)
      this.recognizedWords = lines.map(word => ({
        en: word.trim(),
        cn: ''
      }))
    },
    deleteWord(index) {
      this.recognizedWords.splice(index, 1)
    },
    clearAll() {
      this.recognizedWords = []
      this.imagePath = ''
    },
    saveWords() {
      if (this.recognizedWords.length === 0) {
        uni.showToast({
          title: '没有可保存的单词',
          icon: 'none'
        })
        return
      }

      // 验证单词完整性
      const incompleteWords = this.recognizedWords.filter(w => !w.en.trim())
      if (incompleteWords.length > 0) {
        uni.showToast({
          title: '请完善所有单词信息',
          icon: 'none'
        })
        return
      }

      // 保存到本地存储
      const existingWords = uni.getStorageSync('vocabWords') || []
      const newWords = [...existingWords, ...this.recognizedWords]
      uni.setStorageSync('vocabWords', newWords)

      // 更新统计数据
      this.updateStats(newWords.length)

      // 更新最近学习记录
      const recentWords = this.recognizedWords.slice(0, 5)
      uni.setStorageSync('recentWords', recentWords)

      uni.showToast({
        title: `成功保存 ${this.recognizedWords.length} 个单词`,
        icon: 'success'
      })

      // 延迟跳转到首页
      setTimeout(() => {
        uni.navigateBack()
      }, 1500)
    },
    updateStats(totalWords) {
      const stats = uni.getStorageSync('vocabStats') || {
        total: 0,
        learned: 0,
        mastery: 0
      }
      stats.total = totalWords
      if (totalWords > 0) {
        stats.mastery = Math.round((stats.learned / totalWords) * 100)
      }
      uni.setStorageSync('vocabStats', stats)
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

.upload-section {
  margin-bottom: 25px;
}

.upload-area {
  background: #fff;
  border-radius: 12px;
  padding: 30px;
  text-align: center;
  margin-bottom: 15px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.alt-upload {
  background: #f9f9f9;
  border: 2px dashed #ddd;
}

.upload-icon {
  font-size: 48px;
  margin-bottom: 10px;
  display: block;
}

.upload-text {
  font-size: 16px;
  color: #333;
  font-weight: 500;
  display: block;
  margin-bottom: 5px;
}

.upload-hint {
  font-size: 12px;
  color: #999;
  display: block;
}

.preview-section {
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

.preview-image {
  width: 100%;
  height: 200px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.recognize-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 12px;
  font-size: 16px;
  font-weight: 500;
}

.result-section {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.word-list {
  max-height: 400px;
  overflow-y: auto;
  margin-bottom: 15px;
}

.word-item {
  display: flex;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #f0f0f0;
}

.word-item:last-child {
  border-bottom: none;
}

.word-input-en,
.word-input-cn {
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  font-size: 14px;
  margin-right: 10px;
}

.delete-btn {
  width: 30px;
  height: 30px;
  background: #ff4d4f;
  color: #fff;
  border-radius: 50%;
  text-align: center;
  line-height: 30px;
  font-size: 18px;
  cursor: pointer;
}

.action-buttons {
  display: flex;
  gap: 15px;
}

.save-btn {
  flex: 2;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border: none;
  border-radius: 8px;
  padding: 12px;
  font-size: 16px;
  font-weight: 500;
}

.clear-btn {
  flex: 1;
  background: #f5f5f5;
  color: #666;
  border: none;
  border-radius: 8px;
  padding: 12px;
  font-size: 16px;
}

.loading-mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.loading-content {
  background: #fff;
  border-radius: 12px;
  padding: 30px;
  text-align: center;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #667eea;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 15px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-text {
  font-size: 14px;
  color: #666;
}
</style>
