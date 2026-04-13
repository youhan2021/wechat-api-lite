# wechat-api-lite

微信公众号 API 轻量化工具。只保留发布图文必需的功能：一个 token 获取、一个文件上传、一个草稿创建。无花哨功能，无依赖膨胀。

---

## 功能列表

| 命令 | 作用 |
|------|------|
| `token` | 获取 / 刷新 access_token |
| `upload-thumb` | 上传封面图 → thumb_media_id |
| `upload-image` | 上传正文图片 → media_id + url |
| `create-draft` | 创建图文草稿 |
| `draft-list` | 查看草稿箱数量 |

---

## 快速开始

**1. 安装配置**

```bash
# 复制配置示例文件
cp ~/.hermes/skills/wechat-api-lite/config.env.example \
   ~/.hermes/skills/wechat-api-lite/config.env

# 填入你的 AppID 和 AppSecret
# 获取地址：mp.weixin.qq.com → 设置与开发 → 基本配置
```

**2. 上传封面图**

```bash
python3 ~/.hermes/skills/wechat-api-lite/scripts/wechat_api.py \
  upload-thumb ~/hermes/research/Cover.png
# 返回 thumb_media_id
```

**3. 创建草稿**

```bash
python3 ~/.hermes/skills/wechat-api-lite/scripts/wechat_api.py \
  create-draft ~/hermes/research/draft.json
```

**4. 预览发布**

登录 [mp.weixin.qq.com](https://mp.weixin.qq.com) → 草稿箱 → 预览 → 发布

---

## 草稿 JSON 格式

```json
[
  {
    "title": "文章标题（不超过32字）",
    "author": "作者名",
    "digest": "摘要",
    "content": "<p>HTML 格式正文...</p>",
    "thumb_media_id": "upload-thumb 返回的 ID",
    "show_cover_pic": 1,
    "need_open_comment": 1,
    "only_fans_can_comment": 0
  }
]
```

---

## 常见问题

**API 返回 40001？**
token 过期，清除缓存后重试：
```bash
rm ~/.hermes/skills/wechat-api-lite/scripts/.token_cache
```

**图片用 url 还是 media_id？**
- 正文 HTML 图片链接 → 用 `upload-image` 返回的 `url`
- 封面图 ID → 用 `upload-thumb` 返回的 `media_id`

两者不可混用。

---

## 与 wechat-post 配合

```
wechat-post 生成 Markdown
        ↓
将 Markdown 转为 HTML 写入 draft.json
        ↓
wechat-api-lite upload-thumb（封面）
        ↓
wechat-api-lite create-draft（发布）
```
