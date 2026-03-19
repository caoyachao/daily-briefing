---
name: daily-briefing
description: Generate daily morning briefings with weather, traffic limits, and news. Provides structured data collection scripts for stable, reproducible briefing generation.
---

# Daily Briefing Skill

每日晨报生成工具，提供稳定、可复现的数据获取和简报生成能力。

## Features

- **稳定的数据获取** - 使用结构化脚本获取天气、限行、新闻，减少随机性
- **本地缓存** - 30分钟数据缓存，避免重复请求
- **容错设计** - API 失败时提供备用数据
- **多种输出格式** - 支持文本、JSON、简化版等多种格式
- **全自动化** - 新闻抓取脚本化，无需 AI 手动抓取

## Quick Start

```bash
# 生成今日晨报（完整版，含新闻）
node scripts/generate-briefing.mjs

# 生成次日晨报
node scripts/generate-briefing.mjs --tomorrow

# 生成简化版（天气+限行，无新闻）
node scripts/generate-briefing.mjs --simple

# 生成无新闻版本
node scripts/generate-briefing.mjs --no-news

# JSON 格式输出
node scripts/generate-briefing.mjs --json
```

## API Usage

```javascript
import { generateBriefing, generateBriefingData } from './scripts/generate-briefing.mjs';

// 生成完整晨报（含新闻）
const briefing = await generateBriefing({
  city: 'Beijing',
  dayOffset: 0,  // 0=今天, 1=明天
  includeNews: true
});

// 生成结构化数据
const data = await generateBriefingData({ dayOffset: 0 });
```

## Data Sources

| 数据类型 | 来源 | 更新频率 | 方式 |
|---------|------|---------|------|
| 天气 | wttr.in API | 实时 | curl |
| 限行 | 本地规则配置 | 按周期更新 | 代码计算 |
| 新闻 | 网易新闻、新浪新闻、搜狐新闻 | 实时 | curl + cheerio |

## 北京限行规则（2025.12.29-2026.03.29）

| 星期 | 限行尾号 | 时间 | 范围 |
|------|---------|------|------|
| 周一 | 3和8 | 07:00-20:00 | 五环路以内 |
| 周二 | 4和9 | 07:00-20:00 | 五环路以内 |
| 周三 | 5和0 | 07:00-20:00 | 五环路以内 |
| 周四 | 1和6 | 07:00-20:00 | 五环路以内 |
| 周五 | 2和7 | 07:00-20:00 | 五环路以内 |
| 周末 | 不限行 | - | - |

## Cron Job Integration

配合 OpenClaw Cron 使用示例：

```json
{
  "id": "daily-briefing-morning",
  "agentId": "main",
  "name": "每日晨报-晨间",
  "schedule": {
    "kind": "cron",
    "expr": "0 56 6 * * *",
    "tz": "Asia/Shanghai"
  },
  "payload": {
    "kind": "agentTurn",
    "message": "生成每日晨报：\n\n执行命令获取完整简报（含天气、限行、新闻）：\n```bash\nnode /root/.openclaw/skills/daily-briefing/scripts/generate-briefing.mjs\n```\n\n此命令会自动：\n1. 获取北京天气数据（实时）\n2. 计算当日尾号限行（代码化规则）\n3. 抓取网易新闻、新浪新闻头条\n4. 自动分类新闻（国际/国内/科技/财经/社会）\n5. 生成格式化简报\n\n**重要：不要调用 message 工具，只需直接输出简报内容，系统会自动推送**"
  }
}
```

## 优势对比

| 维度 | 旧方式（纯AI生成） | 新方式（Skill+脚本） |
|------|------------------|-------------------|
| 天气准确性 | 依赖AI调用工具，可能失败 | ✅ 专用脚本，带缓存和容错 |
| 限行准确性 | AI可能记忆错误 | ✅ 代码化规则，准确计算 |
| 新闻时效性 | AI抓取可能遗漏 | ✅ 脚本化抓取，结构化分类 |
| 执行时间 | 不稳定（10-60秒） | ✅ 快速（3-10秒，有缓存） |
| 随机性 | 高 | ✅ 低（固定代码逻辑） |
| 可维护性 | 低（改提示语） | ✅ 高（改代码即可） |

## Files

- `scripts/data-collector.mjs` - 天气、限行数据获取
- `scripts/news-collector.mjs` - 新闻抓取与分类
- `scripts/generate-briefing.mjs` - 简报生成主程序
- `.cache/` - 数据缓存目录（30分钟TTL）

## Changelog

### v1.1.0 (2026-03-19)
- ✨ 新增新闻自动抓取功能（网易、新浪）
- ✨ 新闻自动分类（国际/国内/科技/财经/社会）
- ✨ 新增 `--no-news` 参数
- 🔧 优化缓存机制

### v1.0.0 (2026-03-19)
- 🎉 初始版本
- ✨ 天气数据获取（wttr.in）
- ✨ 限行规则代码化
- ✨ 简报生成功能

## License

MIT
