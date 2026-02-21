# 样例工作流接口文档（前端对接约定）

本文档用于服务端实现“样例工作流”能力，供前端 `Tapnow-Studio` 调用。

---

## 1. 接口目标

前端页面顶部展示“样例工作流”按钮列表。  
点击按钮后：

1. 前端显示“下载中”Toast
2. 从服务端下载对应工作流 JSON
3. 自动导入到画布
4. 关闭“下载中”Toast（成功/失败都会关闭）

当服务端未提供列表或接口异常时，前端会自动使用默认列表：

```json
[
  { "id": "test", "name": "测试", "downloadUrl": "/workflow-samples/test.json" }
]
```

---

## 2. 列表接口

### 2.1 路径

`GET /workflow-samples`

### 2.2 返回格式（推荐）

```json
{
  "workflows": [
    {
      "id": "portrait-basic",
      "name": "人像基础流",
      "downloadUrl": "/workflow-samples/portrait-basic.json"
    },
    {
      "id": "video-story",
      "name": "短视频分镜流",
      "downloadUrl": "/workflow-samples/video-story.json"
    }
  ]
}
```

### 2.3 兼容格式

前端也兼容以下结构：

- 直接数组：`[ {...}, {...} ]`
- `{ "list": [ ... ] }`
- `{ "data": [ ... ] }`

### 2.4 字段说明

每个 workflow item 支持以下字段（前端会自动做兼容）：

- `id` / `key` / `slug`：唯一标识（建议提供）
- `name` / `title` / `label`：按钮显示名称（必须能推导出）
- `downloadUrl` / `url` / `download_url` / `href`：下载地址（优先使用）
- `file` / `fileName` / `filename` / `path`：当没给 `downloadUrl` 时，前端会拼成 `/workflow-samples/{file}`

> 建议明确返回 `id + name + downloadUrl`，最稳定。

---

## 3. 工作流文件下载接口

### 3.1 路径

由列表项 `downloadUrl` 决定。常见形式：

- `GET /workflow-samples/:id.json`
- 或任何你自定义的 URL（支持绝对地址）

### 3.2 响应头

建议：

- `Content-Type: application/json; charset=utf-8`

### 3.3 响应体（必须是 workflow JSON）

```json
{
  "version": "3.8.7",
  "type": "workflow",
  "nodes": [
    {
      "id": "node-1",
      "type": "text-node",
      "x": 100,
      "y": 120,
      "settings": {},
      "content": "hello"
    }
  ],
  "connections": [],
  "timestamp": "2026-02-21T11:00:00+08:00"
}
```

`type` 必须是 `"workflow"`，且 `nodes` 必须是非空数组，否则前端会提示导入失败。

---

## 4. 错误处理建议

- 列表接口失败：返回非 200 或空列表都可，前端会使用默认“测试”按钮。
- 下载接口失败（404/500）：前端显示“加载样例工作流失败”Toast。
- 若有统一错误体，可返回：

```json
{
  "success": false,
  "message": "not found"
}
```

---

## 5. 最小可用实现（建议）

1. 实现 `GET /workflow-samples` 返回 1~N 条样例项  
2. 实现对应 `downloadUrl` 的 JSON 文件下载  
3. 确保 JSON 为有效 workflow 结构  

完成以上三步即可被前端完整消费。

