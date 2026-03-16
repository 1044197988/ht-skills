# 灏天文库 Skill 客户端

客户端用于调用服务端 API，供 OpenClaw 使用。所有脚本通过 `lib/api_client.py` 发送请求，自动附带 Token（`Authorization: Bearer <token>` 或配置在 `config.json`）。

## 配置

- `config.json`：`server_base_url`、`token`
- 或环境变量：`HT_SKILL_SERVER_URL`、`HT_SKILL_TOKEN`

## 脚本与 API 对应关系

| 脚本 | HTTP 方法 | 路由地址 | 说明 |
|------|----------|----------|------|
| `create_collection.py` | POST | `/api/collections` | 新建文集 |
| `list_collections.py` | GET | `/api/collections` | 查询文集列表 |
| `get_collection.py` | GET | `/api/collections/{id}` | 查询文集详情 |
| `update_collection.py` | PATCH | `/api/collections/{id}` | 更新文集信息 |
| `set_document_parent.py` | PATCH | `/api/collections/{collection_id}/documents/{document_id}/parent` | 设置文档父级 |
| `add_document.py` | POST | `/api/documents` | 新建文档到指定文集 |
| `list_documents.py` | GET | `/api/documents` | 查询文档列表 |
| `get_document.py` | GET | `/api/documents/{id}` | 查询文档详情 |
| `update_document.py` | PATCH | `/api/documents/{id}` | 更新文档 |

---

## 脚本参数说明

### create_collection.py — 新建文集

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--name` | string | 是 | 文集名称 |
| `--description` | string | 否 | 文集简短介绍（50 字以内），默认「关于{name}的文集」 |
| `--brief` | string | 否 | 文集详细介绍 |
| `--get-if-exists` | flag | 否 | 若同名文集已存在则直接返回其 ID |

**请求体**：`{"name": "...", "description": "...", "brief": "...", "get_if_exists": bool}`

---

### list_collections.py — 查询文集列表

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--name` | string | 否 | 按名称模糊搜索 |

**Query 参数**：`name`（可选）

---

### get_collection.py — 查询文集详情

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--id` | int | 是 | 文集 ID |
| `--include-docs` | flag | 否 | 是否包含完整文档列表 |

**路径参数**：`{id}`（文集 ID）  
**Query 参数**：`include_docs`（可选，默认 false）

---

### update_collection.py — 更新文集信息

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--id` | int | 是 | 文集 ID |
| `--name` | string | 否 | 新文集名称 |
| `--description` | string | 否 | 新简短介绍（50 字以内） |
| `--brief` | string | 否 | 新详细介绍 |

至少指定 `--name`、`--description`、`--brief` 之一。

**路径参数**：`{id}`  
**请求体**：`{"name": "...", "description": "...", "brief": "..."}`（均为可选）

---

### set_document_parent.py — 设置文档父级

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--collection-id` | int | 是 | 文集 ID |
| `--document-id` | int | 是 | 文档 ID |
| `--parent` | int | 是 | 父文档 ID，0 表示根文档 |
| `--sort` | int | 否 | 同级排序（如第一节=1、第二节=2），sort 越小越靠前 |

**路径参数**：`{collection_id}`、`{document_id}`  
**请求体**：`{"parent": int, "sort": int?}`

---

### add_document.py — 新建文档

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--collection-id` | int | 是 | 文集 ID |
| `--name` | string | 是 | 文档标题 |
| `--content` | string | 否 | 文档正文 |
| `--content-file` | path | 否 | 从文件读取正文（与 `--content` 二选一） |
| `--parent` | int | 否 | 父文档 ID，默认 0 |

**请求体**：`{"collection_id": int, "name": "...", "content": "...", "parent": int}`

---

### list_documents.py — 查询文档列表

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--name` | string | 否 | 按文档名称模糊搜索 |
| `--collection-id` | int | 是 | 按文集 ID 筛选（必须提供） |

**Query 参数**：`name`（可选）、`collection_id`（必填）

---

### get_document.py — 查询文档详情

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--id` | int | 是 | 文档 ID |

**路径参数**：`{id}`（文档 ID）

---

### update_document.py — 更新文档

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `--id` | int | 是 | 文档 ID |
| `--name` | string | 否 | 新文档标题 |
| `--content` | string | 否 | 新正文 |
| `--content-file` | path | 否 | 从文件读取并更新正文 |
| `--sort` | int | 否 | 排序值 |
| `--parent` | int | 否 | 父文档 ID |

至少指定 `--name`、`--content`、`--content-file`、`--sort`、`--parent` 之一。

**路径参数**：`{id}`  
**请求体**：`{"name": "...", "content": "...", "sort": int?, "parent": int?}`（均为可选）

---

## 使用示例

```bash
# 查询文集
python scripts/list_collections.py --name "我的文集"

# 新建文集（有则用、无则建）
python scripts/create_collection.py --name "学习笔记" --get-if-exists

# 新建文档
python scripts/add_document.py --collection-id 1 --name "第一章" --content "正文内容"
python scripts/add_document.py --collection-id 1 --name "第二章" --content-file ./chapter2.md

# 更新文档
python scripts/update_document.py --id 10 --content "新内容"
python scripts/update_document.py --id 10 --content-file ./new_content.md

# 设置文档层级
python scripts/set_document_parent.py --collection-id 1 --document-id 5 --parent 3 --sort 1
```
