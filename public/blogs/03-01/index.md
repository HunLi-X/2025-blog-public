# 寒假在家没事做？我用CNB搭建了一个自动科技资讯推送系统

> 趁着寒假在家，我动手实现了一个每天自动推送科技资讯的系统，从此每天早上8点准时收到最新科技动态。

## 🎯 起因

寒假期间，作为一名对科技充满兴趣的大学生，我发现自己每天需要打开多个网站才能获取最新的科技资讯：

> 36氪 · 虎嗅 · GeekPark · TechCrunch · The Verge...

翻来覆去，既浪费时间又容易漏掉重要新闻。

**为什么不让机器来做这件事呢？**

---

## 📖 项目介绍

**24hNews** 是一个基于 CNB 云原生构建平台打造的自动化科技资讯推送系统。

### ✨ 核心特性

| 特性 | 说明 |
|------|------|
| 🌐 **多源聚合** | 自动抓取12个国内外顶级科技媒体的最新资讯 |
| 🔄 **智能去重** | 基于文章内容哈希，确保不会重复推送 |
| ⏰ **时间过滤** | 只抓取过去24小时内的新文章 |
| 🌍 **智能翻译** | 自动将英文资讯翻译成中文 |
| 📝 **自动总结** | 生成每日摘要，快速了解热点 |
| 🚀 **定时推送** | 每天早上8点自动推送到微信 |

---

## 🏗️ 技术架构

```
┌─────────────────────────────────────────┐
│     CNB 触发器 (每天 08:00)            │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│        Pull 代码仓库                   │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│  Docker 容器 (python:3.11-slim)        │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│    阶段1: 安装依赖                     │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│    阶段2: 抓取资讯                     │
│  ├─ RSS源解析 (feedparser)            │
│  ├─ 时间过滤 (24小时内)               │
│  ├─ 去重处理 (MD5哈希)                │
│  └─ 保存JSON文件                      │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│    阶段3: 推送到微信                   │
│  ├─ 读取文章数据                      │
│  ├─ 智能翻译 (腾讯云API)              │
│  ├─ 生成每日摘要                      │
│  ├─ 格式化Markdown                    │
│  └─ Server酱推送到微信                │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│           工作流完成 ✅                 │
└─────────────────────────────────────────┘
```

---

## 💻 核心功能实现

### 1️⃣ RSS资讯抓取

使用 `feedparser` 库解析RSS订阅源：

```python
def parse_rss_feed(feed_url: str) -> List[Dict]:
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
    }
    feed = feedparser.parse(feed_url, request_headers=headers)
    return feed.entries[:MAX_ARTICLES_PER_FEED]
```

### 2️⃣ 智能去重机制

基于标题和链接生成MD5哈希，避免重复推送：

```python
def get_article_hash(entry: Dict) -> str:
    title = entry.get('title', '')
    link = entry.get('link', '')
    content = f"{title}{link}"
    return hashlib.md5(content.encode('utf-8')).hexdigest()
```

### 3️⃣ 自动翻译功能

检测文章语言，自动翻译非中文内容：

```python
def detect_language(text: str) -> str:
    chinese_pattern = re.compile(r'[\u4e00-\u9fff]')
    if chinese_pattern.search(text):
        return 'zh'
    return 'en'

def translate_text(text: str, target_lang: str = 'zh') -> str:
    # 调用腾讯云翻译API
    # ...
```

### 4️⃣ 每日摘要生成

提取关键词、统计来源、生成热点总结：

```python
def generate_daily_summary(articles: list) -> str:
    # 提取热门关键词
    # 统计主要来源
    # 生成总结文案
    # ...
```

---

## 📱 推送效果示例

> 📰 科技资讯日报
>
> 📅 2026年01月23日 星期五
> ⏰ 08:00
> 📊 精选 20 篇文章来自 8 个科技媒体
>
> ---
>
> 📊 今日亮点
>
> 🔥 **热门话题**: AI · ChatGPT · 苹果 · 腾讯 · 智能驾驶
>
> 📰 **主要来源**: 36kr.com · techcrunch.com · theverge.com
>
> 💡 **今日共收录**: 20 篇前沿科技资讯
>
> ---
>
> ### 🔹 OpenAI发布GPT-5预览版
> *🇨🇳 OpenAI发布GPT-5预览版*
>
> > 📎 openai.com
> > ⏰ 10:00
>
> 📝 **摘要**:
> OpenAI今天发布了GPT-5的预览版本，在推理能力和多模态理解方面有显著提升...
>
> [📖 阅读全文](https://openai.com/...)
>
> ---
>
> *...更多文章...*
>
> > 💡 完整来源
> > • 36kr.com: 5篇
> > • techcrunch.com: 4篇
> > • theverge.com: 3篇
>
> ---
>
> 🤖 由 CNB 云原生构建平台 自动生成
> ⚡ 每天08:00准时推送

---

## 🆚 为什么选择CNB？

对比 GitHub Actions，CNB 有以下优势：

| 特性 | CNB | GitHub Actions |
|:----:|:---:|:--------------:|
| 配置文件 | 单文件 `.cnb.yml` | 多文件 workflows |
| 持久化缓存 | `volumes` 原生支持 | 需要 actions/cache |
| 密钥管理 | 独立密钥仓库 | Repository Secrets |
| 资源限制 | 可自定义 | 固定规格 |

> 💡 特别是 `volumes` 功能，让我可以轻松实现去重机制的持久化存储，不需要额外的数据库。

---

## 🚀 部署步骤

### 步骤 1️⃣

获取 Server 酱密钥

访问 [sct.ftqq.com](https://sct.ftqq.com/) 扫码登录，获取你的 SendKey

---

### 步骤 2️⃣

配置 CNB 工作流

```yaml
stages:
  - name: 抓取科技资讯
    script: python scripts/fetch_tech_news.py

  - name: 推送到微信
    script: python scripts/push_to_wechat.py
```

---

### 步骤 3️⃣

推送到代码仓库

```bash
git add .
git commit -m "Add tech news push system"
git push origin main
```

就这么简单！CNB 会自动识别 `.cnb.yml` 并执行工作流 ✨

---

## ⚠️ 遇到的坑

### 1️⃣ RSS源不稳定

**问题**：有些 RSS 源偶尔会返回空数据或格式错误

**解决**：
```python
try:
    feed = feedparser.parse(feed_url)
    if feed.bozo:
        print(f"⚠️ RSS解析警告: {feed.bozo_exception}")
except Exception as e:
    print(f"❌ RSS解析异常: {e}")
```

---

### 2️⃣ 时区问题

**问题**：不同 RSS 源的发布时间格式不统一

**解决**：
```python
def is_within_last_24_hours(entry: Dict) -> bool:
    try:
        if hasattr(entry, 'published_parsed') and entry.published_parsed:
            pub_time = datetime(*entry.published_parsed[:6])
        time_threshold = datetime.now() - timedelta(hours=HOURS_TO_FETCH)
        return pub_time >= time_threshold
    except:
        return True  # 解析失败时不过滤
```

---

### 3️⃣ 翻译API调用次数限制

**问题**：腾讯云翻译 API 有调用次数限制

**解决**：
- ✅ 只翻译标题和摘要
- ✅ 缓存已翻译内容
- ✅ 设置超时时间

---

## 🔮 后续优化方向

- [ ] **多平台推送** - 支持钉钉、企业微信、Telegram
- [ ] **个性化推荐** - 根据阅读历史推荐相关文章
- [ ] **智能分类** - 自动将文章分类（AI、芯片、云计算等）
- [ ] **图文摘要** - 使用 AI 生成更详细的摘要
- [ ] **Web界面** - 开发一个 Web 端，支持历史文章查询

---

## 🎓 总结

这个项目从构思到完成大概用了一周时间。虽然不算复杂，但让我深入理解了：

```
✅ CNB云原生构建平台的使用
✅ RSS数据抓取和处理
✅ 异常处理和容错设计
✅ 持久化存储机制
✅ 第三方API集成（翻译、推送）
```

最重要的是，**每天早上8点收到最新科技资讯的感觉真的很爽！** 🎉

---

## 🌟 开源地址

欢迎 Star 和 Fork！

> [📦 查看项目](https://cnb.cool/1255027942/XingTeam-2025/24hNews)

---

## 💭 最后的话

如果这个项目对你有帮助，欢迎点个 Star！有改进建议也可以提 Issue。

> 寒假在家没事做，不如动手造点什么？说不定会有意外收获！😊

---

<div align="center">

**Made with ❤️ by HunLi**

2026年寒假

</div>
