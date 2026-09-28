# Label Studio

> 开源数据标注工具，支持文本/图像/音频等多模态标注

## 概览

| 项目 | 说明 |
|------|------|
| 官网 | https://labelstud.io/ |
| GitHub | https://github.com/HumanSignal/label-studio |
| 协议 | Apache 2.0 |
| 开发语言 | Python (Django) + React |
| 存储 | SQLite（默认）/ PostgreSQL |

## 核心能力

- **多模态标注**：文本分类、命名实体、关系抽取、图像目标检测、语音转写等
- **标注界面**：XML 配置定义标注任务形态，单选/多选/文本/跨度/关系
- **项目管理**：任务导入、标注进度、多人协作、标注一致性
- **导入导出**：JSON / JSONL / CSV / COCO / YOLO / CONLL 等格式
- **ML Backend**：接模型做预标注（predictions），人只做修正

## 经过验证的用法（2026-09，v1.23）

### 安装与启动

```sh
uv tool install label-studio

# 启动必须带 --enable-legacy-api-token，否则 API token 认证被拒
label-studio start --port 8090 --enable-legacy-api-token \
  --username admin@example.com --password admin123
```

**踩过的坑（两道开关都要开）**：

1. 启动参数 `--enable-legacy-api-token` —— 开启 legacy token 认证通道
2. Django shell 里 `org.jwt.legacy_api_tokens_enabled = True` —— 组织级开关，默认 False

只开一道仍报 `legacy token authentication has been disabled for this organization`。

### 账号

**必须用启动参数 `--username` / `--password` 建账号**。用 Django shell 手工 `set_password` 建的用户能通过 `authenticate()` 验证，但登录页 `POST /user/login/` 仍失败（两套认证逻辑不一致）。

登录页字段显示「Email」，但实际收的是 **username**（`--username` 传入的值）。

### API 操作

```python
import json, urllib.request

BASE = "http://localhost:8090"
TOKEN = "<启动时生成的用户 token>"   # Django shell: Token.objects.create(user=u)

def api(method, path, data=None):
    req = urllib.request.Request(f"{BASE}{path}", method=method)
    req.add_header("Authorization", f"Token {TOKEN}")
    if data is not None:
        req.add_header("Content-Type", "application/json")
        req.data = json.dumps(data).encode()
    resp = urllib.request.urlopen(req, timeout=30)
    body = resp.read().decode()
    return json.loads(body) if body else {"status": resp.code}
```

| 操作 | 端点 |
|------|------|
| 建项目 | `POST /api/projects/`（title + label_config） |
| 导入任务 | `POST /api/projects/{id}/import`（JSON 数组，每项 `{"id": 1, "data": {...}}`） |
| 列项目 | `GET /api/projects/`（响应是 `{"results": [...]}` 分页结构） |
| 删项目 | `DELETE /api/projects/{id}/`（返回 204 空体，勿当 JSON 解析） |

### 标注配置（XML）

```xml
<View>
  <Text name="text" value="$text"/>
  <Choices name="label" toName="text" choice="single" showInline="true">
    <Choice value="类别A"/>
    <Choice value="类别B"/>
    <Choice value="无关"/>
  </Choices>
</View>
```

`$text` 对应任务 `data.text` 字段，`$source` 等同理——展示字段与 JSON 键名一一对应。

## 架构模式

```
JSON 任务 → Import API → 任务库 (SQLite) → Web 标注界面 (React) → 标注结果 → Export API → JSON/CSV
```

- **单机部署**：`label-studio start` 起 Django dev server，够单人/小团队用
- **生产部署**：Docker + PostgreSQL + Nginx，见官方 deployment 文档
- **认证**：Session（网页）+ Token（API），legacy token 需显式开启（见上）
