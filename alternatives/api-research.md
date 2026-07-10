# 云存储 API 调研报告：百度网盘 vs Dropbox

> 调研日期：2026-07-10

---

## 一、百度网盘开放平台

### 1.1 概览

| 项目 | 说明 |
|------|------|
| 官方名称 | 百度网盘开放平台 |
| 文档地址 | https://pan.baidu.com/union/doc/ |
| 控制台 | https://pan.baidu.com/union/console |
| 开发者类型 | 个人/企业均可注册 |
| 月活用户 | 7 亿+ |

### 1.2 开放能力分类

| 分类 | 能力说明 |
|------|---------|
| **基础网盘能力** | 文件上传/下载、文件管理、网盘信息查询 |
| **多媒体服务** | 视频流畅播放、图片智能分类与相册管理 |
| **智能设备入驻** | 硬件设备绑定与管理 |
| **智能小程序入驻** | 入驻百度网盘，基于网盘用户规模获客 |
| **Agent 能力** | GenFlow 2.0 智能体调度中枢 |
| **AI 创作能力** | 智能 PPT、AI 笔记、AI 绘本、AI 修图等 |

### 1.3 API 风格

- **RESTful API**，基于 HTTPS
- 认证方式：OAuth 2.0（access_token + refresh_token）
- 数据格式：JSON
- 基础域名：`https://pan.baidu.com/rest/2.0/`

### 1.4 核心 API 端点

| 功能 | 端点 (method) | 说明 |
|------|------|--------|
| 获取用户信息 | `xpan/nas?method=uinfo` (GET) | 查询网盘用户基本信息、空间容量 |
| 文件列表 | `xpan/file?method=list` (GET) | 获取目录下文件/文件夹列表 |
| 文件搜索 | `xpan/file?method=search` (GET) | 按文件名搜索 |
| 新建文件夹 | `xpan/file?method=create` (POST) | 创建目录 |
| 文件上传 | `xpan/file?method=upload` (POST) | 上传文件（< 2GB 直传，大文件分片） |
| 文件下载 | `xpan/file?method=download` (GET) | 获取文件下载链接 |
| 文件删除 | `xpan/file?method=delete` (POST) | 批量删除 |
| 文件移动/复制 | `xpan/file?method=move`/`copy` (POST) | 移动或复制文件 |
| 文件重命名 | `xpan/file?method=rename` (POST) | 重命名 |
| 秒传 | `xpan/file?method=rapiduplod` (POST) | 基于文件 MD5 实现秒传 |
| 分片上传 - 预创建 | `xpan/file?method=precreate` (POST) | 大文件分片上传前预创建 |
| 分片上传 - 上传分片 | `xpan/file?method=upload` (POST) | 上传指定分片 |
| 分片上传 - 合并 | `xpan/file?method=create` (POST) | 上传完成后合并分片 |

> 注意：百度网盘开放平台的文档是 SPA（单页应用），大部分文档页面需要浏览器 JavaScript 渲染才能查看，此处的端点信息基于公开文档整理。

### 1.5 认证流程

1. 在开放平台控制台创建应用，获取 `app_id`, `api_key`, `secret_key`
2. 引导用户授权，获取 `code`（授权码模式）
3. 用 `code` 换取 `access_token` 和 `refresh_token`
4. 调用 API 时在 URL 参数或 Header 中携带 `access_token`
5. `access_token` 过期后用 `refresh_token` 刷新

### 1.6 限制与注意事项

- **文件大小限制**：单文件直传上限 2GB，分片上传支持更大文件
- **流量限制**：对 API 调用频次有限制，具体阈值根据应用等级不同
- **隐私限制**：严禁未经授权下载传播用户数据
- **商用限制**：禁止用作图床、网盘迁移工具等
- **无官方 Python SDK**：需要自己封装 HTTP 请求（社区有第三方封装如 `baidupcs-python`）

### 1.7 官方 Library / SDK

**百度网盘没有提供官方 SDK**（只有 REST API 文档）。社区方案：

| 名称 | 语言 | 仓库 |
|------|------|------|
| `baidupcs` | Python | https://github.com/ly0/baidupcs |
| `BaiduPCS-Py3` | Python | https://github.com/PorterZhang/BaiduPCS-Py3 |
| `bypy` | Python | https://github.com/houtianze/bypy (命令行工具) |

---

## 二、Dropbox API

### 2.1 概览

| 项目 | 说明 |
|------|------|
| 官方名称 | Dropbox API v2 |
| 文档地址 | https://www.dropbox.com/developers |
| API 参考 | https://www.dropbox.com/developers/documentation/http/documentation |
| Python SDK 文档 | https://dropbox-sdk-python.readthedocs.io/ |
| 开发者控制台 | https://www.dropbox.com/developers/apps |
| 开发者类型 | 个人/企业均可注册 |

### 2.2 API 风格

- **RPC / Upload / Download** 三类端点
- 认证方式：OAuth 2.0（支持 PKCE、短期 access_token + refresh_token）
- 数据格式：JSON
- API 域名：`https://api.dropboxapi.com/`
- Content 域名：`https://content.dropboxapi.com/`

### 2.3 核心 API 端点（HTTP）

**文件/文件夹操作（`/files` 命名空间）**

| 功能 | 端点 | 说明 |
|------|------|------|
| 获取元数据 | `POST /2/files/get_metadata` | 文件/文件夹元信息 |
| 列表目录 | `POST /2/files/list_folder` | 列出目录内容，支持递归 |
| 列表续页 | `POST /2/files/list_folder/continue` | 分页拉取 |
| 长轮询 | `POST /2/files/list_folder/longpoll` | 监听变更，低延迟通知 |
| 搜索 | `POST /2/files/search_v2` | 全文搜索 |
| 下载 | `POST /2/files/download` | 下载文件 |
| 上传 | `POST /2/files/upload` | 上传（< 150MB 直传） |
| 分片上传 - 开始 | `POST /2/files/upload_session/start` | 创建上传会话 |
| 分片上传 - 追加 | `POST /2/files/upload_session/append_v2` | 追加数据 |
| 分片上传 - 完成 | `POST /2/files/upload_session/finish` | 完成上传并提交 |
| 复制 | `POST /2/files/copy_v2` | 复制文件/文件夹 |
| 移动 | `POST /2/files/move_v2` | 移动/重命名 |
| 删除 | `POST /2/files/delete_v2` | 删除 |
| 创建文件夹 | `POST /2/files/create_folder_v2` | 创建目录 |
| 获取临时链接 | `POST /2/files/get_temporary_link` | 4 小时临时下载链接 |
| 获取缩略图 | `POST /2/files/get_thumbnail` | 图片缩略图 |
| 导出 | `POST /2/files/export` | 导出 Google Docs 等非原生格式 |
| 标签 | `POST /2/files/tags/add` 等 | 文件标签管理 |
| 版本列表 | `POST /2/files/list_revisions` | 文件版本历史 |
| 恢复版本 | `POST /2/files/restore` | 恢复历史版本 |
| 批量复制/移动/删除 | `copy_batch` / `move_batch` / `delete_batch` | 异步批量操作 |
| 锁定/解锁 | `POST /2/files/lock_file_batch` / `unlock_file_batch` | 文件锁定 |

**共享操作（`/sharing` 命名空间）**

| 功能 | 端点 | 说明 |
|------|------|------|
| 创建共享链接 | `POST /2/sharing/create_shared_link_with_settings` | 创建分享链接 |
| 列出共享链接 | `POST /2/sharing/list_shared_links` | 查看已有分享 |
| 修改共享链接 | `POST /2/sharing/modify_shared_link_settings` | 修改权限/过期时间 |
| 撤销共享链接 | `POST /2/sharing/revoke_shared_link` | 取消分享 |
| 共享文件夹 | `POST /2/sharing/share_folder` | 共享文件夹 |
| 添加成员 | `POST /2/sharing/add_folder_member` | 邀请协作 |
| 列出成员 | `POST /2/sharing/list_folder_members` | 查看协作者 |

**用户/账户（`/users` 命名空间）**

| 功能 | 端点 | 说明 |
|------|------|------|
| 获取当前用户 | `POST /2/users/get_current_account` | 当前账户信息 |
| 获取空间使用 | `POST /2/users/get_space_usage` | 存储容量及用量 |
| 获取账户信息 | `POST /2/users/get_account` | 按 account_id 查 |

### 2.4 官方 Python SDK: `dropbox`

| 项目 | 说明 |
|------|------|
| PyPI 包 | `dropbox` |
| 最新版本 | v12.1.0 (2026-07-10) |
| GitHub | https://github.com/dropbox/dropbox-sdk-python |
| 文档 | https://dropbox-sdk-python.readthedocs.io/ |
| 底层依赖 | `requests` |
| Python 版本 | 3.7+ |
| 协议 | MIT |

**安装**

```bash
pip install dropbox
```

**核心类**

| 类 | 用途 |
|------|------|
| `dropbox.Dropbox(access_token)` | 用户级客户端 |
| `dropbox.DropboxTeam(team_access_token)` | 团队级客户端 |
| `Dropbox.as_user(team_member_id)` | 以指定成员身份操作 |
| `Dropbox.with_path_root(path_root)` | 指定路径根命名空间 |

**SDK 模块内容**

| 模块 | 内容 |
|------|------|
| `dropbox.files` | 文件 CRUD、上传下载、搜索、标签、锁、版本 |
| `dropbox.sharing` | 共享链接、文件夹共享、成员管理 |
| `dropbox.users` | 用户信息、空间用量 |
| `dropbox.auth` | Token 颁发、撤销 |
| `dropbox.file_properties` | 文件自定义属性模板 |
| `dropbox.file_requests` | 文件请求管理 |
| `dropbox.paper` | Paper 文档创建/编辑/分享 |
| `dropbox.team` | 团队管理（成员、群组、设备、日志、报告） |
| `dropbox.team_log` | 团队事件日志 |
| `dropbox.contacts` | 联系人管理 |
| `dropbox.async_` | 异步支持 |
| `dropbox.oauth` | OAuth 流程 |
| `dropbox.exceptions` | 异常类型 |

**快速示例**

```python
import dropbox

# 初始化
dbx = dropbox.Dropbox(oauth2_access_token="YOUR_ACCESS_TOKEN")

# 列出根目录
for entry in dbx.files_list_folder("").entries:
    print(entry.name)

# 上传文件
with open("local_file.txt", "rb") as f:
    dbx.files_upload(f.read(), "/remote_file.txt")

# 下载文件
metadata, response = dbx.files_download("/remote_file.txt")
with open("downloaded.txt", "wb") as f:
    f.write(response.content)

# 创建共享链接
link = dbx.sharing_create_shared_link_with_settings("/remote_file.txt")
print(link.url)

# 搜索文件
results = dbx.files_search_v2("report")
for match in results.matches:
    print(match.metadata.name)
```

**常见 SDK 方法**

| 方法 | 说明 |
|------|------|
| `files_list_folder(path)` | 列出目录（返回游标可继续拉取） |
| `files_list_folder_continue(cursor)` | 续列 |
| `files_upload(f, path)` | 上传（< 150MB） |
| `files_upload_session_start(f)` | 分片上传 - 开始 |
| `files_upload_session_append_v2(f, cursor)` | 分片上传 - 追加 |
| `files_upload_session_finish(f, cursor, commit)` | 分片上传 - 完成 |
| `files_download(path)` | 下载 |
| `files_download_to_file(local_path, remote_path)` | 下载到本地文件 |
| `files_get_metadata(path)` | 获取元数据 |
| `files_delete(path)` | 删除 |
| `files_copy(from_path, to_path)` | 复制 |
| `files_move(from_path, to_path)` | 移动/重命名 |
| `files_create_folder(path)` | 创建文件夹 |
| `files_search_v2(query)` | 搜索 |
| `files_get_temporary_link(path)` | 临时下载链接（4h） |
| `sharing_create_shared_link_with_settings(path)` | 创建分享链接 |
| `sharing_list_shared_links(path)` | 查看分享链接 |
| `users_get_current_account()` | 当前用户信息 |
| `users_get_space_usage()` | 空间使用情况 |

### 2.5 其他语言官方 SDK

| 语言 | 链接 |
|------|------|
| Python | `pip install dropbox` |
| JavaScript | `npm install dropbox` |
| Java | Maven: `com.dropbox.core:dropbox-core-sdk` |
| Swift | CocoaPods: `DropboxClient` |
| .NET | NuGet: `Dropbox.Api` |
| Ruby | `gem install dropbox-sdk` |
| Go | `github.com/dropbox/dropbox-sdk-go-unofficial` |
| Rust | `crates.io/crates/dropbox-sdk` (社区) |

### 2.6 限制与注意事项

- **单文件上传上限**：直传 150MB / 分片 350GB
- **分片大小**：必须为 4MB 的倍数（除最后一片）
- **上传会话有效期**：7 天
- **临时链接有效期**：4 小时
- **搜索上限**：10,000 条结果
- **文件夹打包下载限制**：< 20GB / < 10,000 条目
- **免费版 API 额度**：取决于 App 类型（"Full Dropbox" / "App Folder"）
- **支持 Webhook**：文件变更推送
- **兼容性**：2026 年 1 月起，旧版 SDK 因证书变更停止工作，需使用 v12.0.2+

---

## 三、对比总结

| 维度 | 百度网盘 | Dropbox |
|------|----------|---------|
| **API 风格** | REST | RPC-style REST |
| **认证** | OAuth 2.0 | OAuth 2.0 (支持 PKCE) |
| **官方 SDK** | ❌ 无 | ✅ Python / JS / Java / Swift / .NET / Ruby |
| **社区 SDK** | ✅ 非官方 Python 封装 | - |
| **Python 包** | 社区 `baidupcs` | 官方 `dropbox` (pip 安装) |
| **文件上传上限** | 2GB（直传）/ 分片支持更大 | 150MB（直传）/ 350GB（分片） |
| **文件下载** | 获取下载链接后下载 | 直接流式下载 / 临时链接 |
| **搜索** | 文件名搜索 | 文件名 + 内容搜索（Business） |
| **批量操作** | 支持 | 支持（异步 job） |
| **版本管理** | 支持 | 支持（`list_revisions` / `restore`） |
| **分享链接** | 支持 | 支持（权限/过期/密码）|
| **文件锁定** | ❌ | ✅ |
| **标签/属性** | ❌ | ✅ 自定义属性模板 + 标签 |
| **缩略图** | ❌ | ✅ |
| **Webhook** | ❌ | ✅ 文件变更通知 |
| **长轮询** | ❌ | ✅ `list_folder/longpoll` |
| **团队管理** | ❌ | ✅ 完善的 Team API |
| **文档质量** | SPA 需 JS 渲染，不易抓取 | ✅ 结构清晰，Mock 可读 |
| **免费额度** | 依据开发者等级 | 依据 App 类型 |
| **商用限制** | 禁止图床/迁移工具 | 较宽松 |

### 选择建议

- **面向国内用户**：百度网盘是唯一选择，适合备份/存储/多媒体场景
- **面向国际用户 / 需要完善 SDK**：Dropbox 开发者体验远超百度网盘，官方 SDK 覆盖多语言、API 设计更现代
- **需要文件协作/团队管理**：Dropbox Team API 成熟
- **AI 能力集成**：百度网盘提供 AI 创作（智能 PPT、笔记），Dropbox 暂无
- **需要文件锁定/标签属性**：Dropbox 原生支持

### 备注

- 百度网盘文档为 SPA 架构，需要浏览器 JavaScript 运行，因此自动化工具难以直接爬取完整的接口文档。建议通过人工浏览（https://pan.baidu.com/union/doc/）或抓取 HAR 文件来补充端点信息。
- Dropbox 的 Python SDK 底层使用 `requests` 库，支持连接池、自动重试、令牌刷新，可直接在项目中使用。
- Dropbox API 支持 `scope` 粒度权限控制，适合最小权限原则。
