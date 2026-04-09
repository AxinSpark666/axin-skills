# axin-skills

> Hermes Agent 技能仓库 —— 为 AI 助手扩展推特查询、Web3 情报等能力

![Hermes Agent](https://img.shields.io/badge/Hermes%20Agent-Skills-blue)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 📋 仓库简介

本仓库托管为 [Hermes Agent](https://github.com) 开发的第三方技能集，涵盖社交媒体监控、Web3 数据查询等场景。所有技能均以独立 SKILL.md 文件交付，复制到 `~/.hermes/skills/` 即可使用。

---

## 🛠 技能列表

### 🐦 social-media / twitter-query

**推特/X 数据查询**

通过 RapidAPI `twitter-api45` 接口查询推特数据，支持：

| 功能 | 说明 |
|------|------|
| 用户资料查询 | 账号信息、粉丝数、认证状态 |
| 推文 Timeline | 指定账号最新推文列表 |
| 粉丝/关注列表 | 账号的关注与被关注关系 |
| 推文搜索 | 按用户、话题、推文ID搜索 |
| 转发列表 | 查看某条推文的转发情况 |

**输出格式**：每次汇报均使用统一的「卡片式」排版，包含中文翻译与外部链接提取。

**配置要求**：
- `TWITTER_RAPIDAPI_KEY` — RapidAPI Key
- `TWITTER_RAPIDAPI_HOST` — 默认为 `twitter-api45.p.rapidapi.com`

> ⚠️ 不支持：点赞历史、书签、Quote Tweets

---

### 🔬 research / axinrootdata

**Web3 项目与融资情报**

查询加密项目详细信息、投资机构、融资历史、热门趋势与人员变动。

| 功能 | 说明 |
|------|------|
| 项目搜索 | 按关键词搜索项目/机构/人物 |
| 项目详情 | 融资金额、投资者、合约地址、社交媒体 |
| 融资轮次 | 按时间/金额范围筛选，历史最远365天 |
| 热门项目 | 今日/本周趋势榜单 |
| 人员变动 | 最新入职/离职情报 |

**配置要求**：
- `ROOTDATA_SKILL_KEY` — RootData API Key（通过 `/skill/init` 获取，或使用用户提供的 `rdsk_xxxx` 格式 Key）

**融资汇报格式**：统一使用卡片式排版，不使用 Markdown 表格。

---

## 🚀 快速安装

### 安装全部技能

```bash
# 克隆仓库
git clone https://github.com/AxinSpark666/axin-skills.git ~/.hermes/skills
```

### 安装单个技能

```bash
# 仅安装 twitter-query
mkdir -p ~/.hermes/skills/social-media
cp -r axin-skills/social-media/twitter-query ~/.hermes/skills/social-media/

# 仅安装 axinrootdata
mkdir -p ~/.hermes/skills/research
cp -r axin-skills/research/axinrootdata ~/.hermes/skills/research/
```

### 安装到指定 profile

```bash
HERMES_HOME=~/.hermes-myprofile
mkdir -p $HERMES_HOME/skills/social-media $HERMES_HOME/skills/research
cp -r social-media/twitter-query $HERMES_HOME/skills/social-media/
cp -r research/axinrootdata $HERMES_HOME/skills/research/
```

---

## ⚙️ 技能配置

### twitter-query

```bash
# 1. 创建配置目录
mkdir -p ~/.config/twitter-query

# 2. 写入默认配置
cat > ~/.config/twitter-query/.env << 'EOF'
TWITTER_RAPIDAPI_HOST=twitter-api45.p.rapidapi.com
TWITTER_RAPIDAPI_KEY=你的RapidAPIKey
EOF
chmod 600 ~/.config/twitter-query/.env
```

**获取 RapidAPI Key**：
1. 访问 [RapidAPI](https://rapidapi.com) 注册账号
2. 订阅 `twitter-api45` 包（免费层可用）
3. 复制 Key 并填入上方配置

### axinrootdata

```bash
# Key 由技能自动通过 /skill/init 接口获取
# 或手动设置：
export ROOTDATA_SKILL_KEY=rdsk_xxxx
```

> 💡 用户可主动提供 `rdsk_xxxx` 格式的 Key，技能会优先使用用户提供的 Key。

---

## 📁 目录结构

```
axin-skills/
├── README.md
├── social-media/
│   └── twitter-query/
│       └── SKILL.md          # 推特查询技能
└── research/
    └── axinrootdata/
        └── SKILL.md          # Web3 情报技能
```

---

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

**提交规范**：
- 每个技能独占一个目录（如 `category/skill-name/`）
- 技能主文件统一命名为 `SKILL.md`
- 更新技能时同步更新本文档的技能列表

---

## 📄 License

MIT License
