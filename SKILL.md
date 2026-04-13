---
name: wechat-api
description: 微信公众号 API 集成 — access_token、素材上传、图文草稿创建与管理
triggers:
  - "公众号 API"
  - "微信发布"
  - "创建草稿"
  - "上传素材"
  - "wechat api"
---

# WeChat API — 微信公众号接口集成

通过微信公众平台 API 直接操作：获取凭证、上传素材（图片/封面）、创建图文草稿等。

---

## 凭证配置

**config.env**（已加入 .gitignore，不会提交到 Git）：

```
WECHAT_APP_ID=your_app_id_here
WECHAT_APP_SECRET=your_app_secret_here
```

首次使用前，复制示例文件并填入真实值：

```bash
SKILL_DIR="$HOME/.hermes/skills/wechat-api"
cp "$SKILL_DIR/config.env.example" "$SKILL_DIR/config.env"
# 编辑 config.env 填入真实 AppID 和 AppSecret
```

> AppID 和 AppSecret 需在 [微信公众平台](https://mp.weixin.qq.com) → 设置与开发 → 基本配置 中获取。

---

## 核心 API

### access_token

微信 API 的全局唯一凭证，有效期 7200 秒。脚本内自动缓存 + 提前刷新。

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py token
```

---

### 上传图片素材（永久）

上传后获得 `media_id`，可用于正文中引用图片。

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py upload-image <文件路径>
# 示例
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py upload-image ~/hermes/research/cover.png
```

返回示例：
```json
{"media_id": "xxx", "url": "https://mmbiz.qpic.cn/..."}
```

---

### 上传封面缩略图（永久）

封面图需先上传获得 `thumb_media_id`，再用于创建草稿。

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py upload-thumb <文件路径>
# 示例
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py upload-thumb ~/hermes/research/Cover.png
```

返回示例：
```json
{"media_id": "THUMB_MEDIA_ID"}
```

> 封面图推荐尺寸：900 × 383 px（2.35:1 宽屏），格式 PNG/JPG，不超过 2MB。

---

### 创建图文草稿

草稿内容为 **HTML 格式正文**，由 `wechat-post` skill 生成的 HTML 转换而来。

**步骤 1：准备 JSON 草稿文件**

```json
[
  {
    "title": "文章标题（必填，不超过32字）",
    "author": "作者名（可选）",
    "digest": "摘要（可选，默认取正文前54字）",
    "content": "<p>HTML 格式正文内容...</p>",
    "thumb_media_id": "封面图 thumb_media_id（必填）",
    "show_cover_pic": 1,
    "need_open_comment": 1,
    "only_fans_can_comment": 0
  }
]
```

**步骤 2：上传封面获取 thumb_media_id（如果还没有）**

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py upload-thumb ~/hermes/research/Cover.png
```

**步骤 3：创建草稿**

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py create-draft ~/hermes/research/draft.json
```

成功输出：
```
✅ 草稿创建成功: media_id=XXXXXXXXXXXXXXXXXXXXXXXX
{"media_id": "XXXXXXXXXXXXXXXXXXXXXXXX"}
```

---

### 查看草稿数量

```bash
python3 $HOME/.hermes/skills/wechat-api/scripts/wechat_api.py draft-list
```

---

## 完整发布流程（wechat-post + wechat-api 配合）

```
1. wechat-post 生成正文（Markdown）
      ↓
2. wechat-post 渲染为 .docx（人工排版校对）
      ↓
3. 将 HTML 格式正文写入 draft.json
      ↓
4. wechat-api upload-thumb（上传封面 → thumb_media_id）
      ↓
5. wechat-api create-draft（创建草稿 → media_id）
      ↓
6. 登录 mp.weixin.qq.com 预览草稿 → 确认后发布
```

---

## 草稿 JSON 结构说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `title` | ✅ | 标题，不超过 32 个字符 |
| `author` | 否 | 作者名，不超过 16 字 |
| `digest` | 否 | 摘要，默认取正文前 54 字 |
| `content` | ✅ | HTML 格式正文（CDATA 或普通字符串） |
| `thumb_media_id` | ✅ | 封面图 media_id（需先调用 upload-thumb） |
| `show_cover_pic` | 否 | 是否在正文显示封面图（0/1），默认 0 |
| `need_open_comment` | 否 | 是否打开评论（0/1），默认 0 |
| `only_fans_can_comment` | 否 | 是否仅粉丝可评论（0/1），默认 0 |

---

## 注意事项

- `access_token` 有效期 2 小时，脚本自动维护缓存，勿频繁手动刷新
- 永久素材有总量限制（公众平台限制），缩略图不超过 2MB
- 封面图尺寸推荐 900 × 383 px（2.35:1），正文图片建议宽度 ≤ 1080px
- 草稿创建后可在 [mp.weixin.qq.com](https://mp.weixin.qq.com) 草稿箱中查看和编辑
