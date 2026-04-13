# wechat-api-lite

Lightweight WeChat Official Account API tool. Only the essentials: token fetch, image upload, draft creation. No bloat, no heavy dependencies.

---

## Commands

| Command | Description |
|---------|-------------|
| `token` | Get / refresh access_token |
| `upload-thumb` | Upload cover image → thumb_media_id |
| `upload-image` | Upload body image → media_id + url |
| `create-draft` | Create article draft |
| `draft-list` | Show draft count |

---

## Quick Start

**1. Configure credentials**

```bash
cp ~/.hermes/skills/wechat-api-lite/config.env.example \
   ~/.hermes/skills/wechat-api-lite/config.env

# Edit config.env with your AppID and AppSecret
# Get them at: mp.weixin.qq.com → Settings → Basic Config
```

**2. Upload cover image**

```bash
python3 ~/.hermes/skills/wechat-api-lite/scripts/wechat_api.py \
  upload-thumb ~/hermes/research/Cover.png
# Returns thumb_media_id
```

**3. Create draft**

```bash
python3 ~/.hermes/skills/wechat-api-lite/scripts/wechat_api.py \
  create-draft ~/hermes/research/draft.json
```

**4. Preview & publish**

Sign in to [mp.weixin.qq.com](https://mp.weixin.qq.com) → Drafts → Preview → Publish

---

## Draft JSON Format

```json
[
  {
    "title": "Article title (max 32 chars)",
    "author": "Author name",
    "digest": "Article summary",
    "content": "<p>HTML formatted body...</p>",
    "thumb_media_id": "ID returned by upload-thumb",
    "show_cover_pic": 1,
    "need_open_comment": 1,
    "only_fans_can_comment": 0
  }
]
```

---

## FAQ

**Getting 40001 errors?**
Token expired. Clear the cache and retry:
```bash
rm ~/.hermes/skills/wechat-api-lite/scripts/.token_cache
```

**url vs media_id — which one to use?**
- Image links in HTML body → use `url` from `upload-image`
- Cover image in draft → use `media_id` from `upload-thumb`

They are not interchangeable.

---

## Workflow with wechat-post

```
wechat-post generates Markdown
        ↓
Convert Markdown to HTML, write to draft.json
        ↓
wechat-api-lite upload-thumb (cover)
        ↓
wechat-api-lite create-draft (publish)
```
