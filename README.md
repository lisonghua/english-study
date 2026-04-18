# 英文背单词助手 (English Vocabulary Master)

一款专为高中生设计的安卓背单词手机应用，基于 UniApp 开发。通过拍照导入、智能跟读、多样化听写等功能，帮助学生高效记忆单词，解决背单词困难问题。

## ✨ 核心功能

### 📸 1. 拍照导入单词
- **智能识别**：手机拍照一页单词，自动识别并导入生词本
- **批量处理**：支持一次性导入大量单词，节省手动输入时间
- **格式灵活**：自动识别常见单词书格式，智能提取单词与释义

### 🗣️ 2. 单词辅助背诵
- **智能领读**：标准发音领读，学生跟随 APP 朗读单词
- **个性化设置**：
  - 跟读次数：自由设置每个单词的跟读遍数
  - 发音风格：美式英语 / 英式英语随意切换
  - 人声选择：男声 / 女声两种音色可选
- **即时反馈**：语音识别技术提供发音评分与建议

### 📝 3. 多样化听写模式
- **双向听写**：
  - 模式一：APP 说英文 → 学生写中文
  - 模式二：APP 说中文 → 学生写英文
  - 支持自由切换，强化双向记忆
- **听写顺序**：
  - 顺序听写：按单词列表顺序进行
  - 随机听写：打乱顺序，避免死记硬背
  - 倒序听写：从后往前，挑战记忆深度
- **自定义范围**：自由选择听写单词范围（全部/未掌握/错题本等）
- **朗读控制**：自定义每次听写的朗读次数，适应不同学习节奏

### 🎨 4. 人性化界面设计
- **简洁美观**：清爽界面设计，减少视觉干扰
- **易用操作**：符合高中生使用习惯的交互逻辑
- **护眼模式**：长时间学习保护视力
- **进度可视化**：清晰展示学习进度与成就

## 🛠️ 技术栈

- **开发框架**：[UniApp](https://uniapp.dcloud.net.cn/) - 使用 Vue.js 语法开发跨平台应用
- **目标平台**：Android
- **OCR 识别**：集成百度/腾讯 OCR 接口实现拍照识字
- **语音合成**：使用系统 TTS 或第三方语音服务
- **语音识别**：集成语音识别 SDK 实现跟读评测
- **数据存储**：本地 SQLite + 云同步（可选）

## 📁 项目结构

```
english-vocab-app/
├── pages/              # 页面文件
│   ├── index/          # 首页
│   ├── import/         # 拍照导入页
│   ├── practice/       # 背诵练习页
│   ├── dictation/      # 听写页面
│   └── settings/       # 设置页面
├── components/         # 公共组件
│   ├── WordCard.vue    # 单词卡片组件
│   ├── VoicePlayer.vue # 语音播放组件
│   └── ProgressBar.vue # 进度条组件
├── static/             # 静态资源
│   ├── images/         # 图片资源
│   └── audio/          # 音频资源
├── utils/              # 工具函数
│   ├── ocr.js          # OCR 识别封装
│   ├── tts.js          # 语音合成封装
│   └── storage.js      # 数据存储封装
├── store/              # Vuex 状态管理
├── manifest.json       # 应用配置
├── pages.json          # 页面路由配置
└── README.md           # 项目说明
```

## 🚀 快速开始

### 环境准备

1. 安装 [HBuilderX](https://www.dcloud.io/hbuilderx.html)
2. 安装 Node.js (v14+)
3. 配置 Android 开发环境（可选，用于真机调试）

### 安装依赖

```bash
# 克隆项目
git clone <your-repo-url>
cd english-vocab-app

# 安装依赖（如使用 npm 管理）
npm install
```

### 运行项目

1. 用 HBuilderX 打开项目
2. 连接安卓设备或启动模拟器
3. 点击「运行」→「运行到手机或模拟器」
4. 选择「运行到安卓 App-base」

### 构建发布

1. 在 HBuilderX 中选择「发行」→「原生 App-云打包」
2. 配置证书和包名
3. 生成 APK 安装包

## ⚙️ 配置说明

### OCR 配置
在 `utils/ocr.js` 中配置你的 OCR 服务密钥：

```javascript
const OCR_CONFIG = {
  appId: 'YOUR_APP_ID',
  apiKey: 'YOUR_API_KEY',
  secretKey: 'YOUR_SECRET_KEY'
}
```

### 语音服务配置
在 `utils/tts.js` 中配置语音合成服务：

```javascript
const TTS_CONFIG = {
  provider: 'system', // 或 'baidu', 'azure' 等
  voiceType: 'female', // 'male' 或 'female'
  accent: 'us' // 'us' 或 'uk'
}
```

## 📱 使用说明

### 导入单词
1. 点击首页「导入单词」按钮
2. 选择「拍照导入」或「相册选择」
3. 对准单词书页拍照，等待识别完成
4. 检查识别结果，确认后加入生词本

### 开始背诵
1. 进入「背诵练习」页面
2. 选择要背诵的单词组
3. 设置发音风格、人声、跟读次数
4. 点击开始，跟随 APP 朗读

### 进行听写
1. 进入「听写训练」页面
2. 选择听写模式（英译中 / 中译英）
3. 设置听写顺序和范围
4. 调整朗读次数，开始听写
5. 完成后查看答案解析与正确率

## 🤝 贡献指南

欢迎贡献代码！请遵循以下步骤：

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情

## 📞 联系方式

如有问题或建议，请通过以下方式联系：
- 邮箱：your-email@example.com
- Issues: [提交 Issue](https://github.com/your-username/english-vocab-app/issues)

## 🙏 致谢

感谢所有为开源社区做出贡献的开发者！

---

**让背单词变得更简单、更高效！** 📚✨
