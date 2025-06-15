# 存客宝中台 - AI知识库功能后端开发文档\n\n## 1. 模块概述\n\nAI知识库功能旨在构建一个企业内部知识管理平台，支持上传多种类型的文档、图片、网页内容等，并通过AI技术进行智能解析、知识提取、语义搜索和智能问答。后端模块负责知识内容的存储、索引构建、AI模型集成与调用、搜索接口和问答接口的实现。\n\n## 2. API接口设计\n\n### 2.1 上传知识文件/内容\n\n- **接口路径**：`/api/v1/knowledge/upload`\n- **请求方法**：`POST`\n- **接口说明**：接收用户上传的知识文件（如PDF, DOCX, TXT等）、图片或提交的网页链接/文本内容。后端进行预处理、存储，并触发异步的AI解析流程。\n- **权限:** `knowledge:upload`\n- **请求参数 (Request Body):** 使用 multipart/form-data 或 application/json\n\n| 参数名      | 类型           | 是否必需 | 描述           | 示例值 |
|-------------|----------------|----------|----------------|--------|\
| file        | file           | 否       | 上传的文件     | 文件二进制流 |\
| url         | string         | 否       | 网页链接       | \"http://example.com/doc\" |\
| textContent | string         | 否       | 直接提交的文本内容 | \"这是一段文本...\" |\
| contentType | string         | 是       | 内容类型 (FILE, URL, TEXT, IMAGE) | \"FILE\" |\
| folderId    | integer        | 否       | 目标文件夹ID   | 1      |\
| tags        | array<string>  | 否       | 内容标签       | `[\"合同\", \"模板\"]` |\
\n- **响应数据 (统一格式 `data` 字段):** 返回上传/提交成功信息及处理状态。\n\n```json\n{\n  \"knowledgeId\": 8001,\n  \"fileName\": \"示例合同.pdf\", // 或其他标识\n  \"status\": \"PROCESSING\", // UPLOADED, PROCESSING, READY, FAILED\n  \"message\": \"文件已接收，正在进行AI解析\"\n}\n```\n- **可能返回状态码:** 202 (已接受处理), 400, 401, 403, 413 (文件过大), 422, 500\n\n### 2.2 查询知识内容列表\n\n- **接口路径**：`/api/v1/knowledge/list`\n- **请求方法**：`GET`\n- **接口说明**：获取知识库内容列表。支持分页、筛选、按文件夹和标签过滤、按状态过滤。\n- **权限:** `knowledge:view`\n- **请求参数 (Query Parameters):**\n\n| 参数名    | 类型    | 是否必需 | 描述           | 示例值 |
|-----------|--------|----------|----------------|--------|\
| folderId  | integer | 否       | 按文件夹ID过滤 | 1      |\
| tagIds    | array<integer> | 否 | 按标签ID过滤   | `[10, 11]` |\
| status    | string | 否       | 按状态过滤 (UPLOADED, PROCESSING, READY, FAILED) | \"READY\" |\
| keyword   | string | 否       | 按标题或内容关键字搜索 | \"合同\" |\
| page      | integer| 否       | 页码           | 1      |\
| size      | integer| 否       | 每页条数       | 10     |\n\n- **响应数据 (统一格式 `data` 字段):** 返回知识内容列表（支持分页）。\n\n```json\n{\n  \"records\": [\n    {\n      \"knowledgeId\": 8001,\n      \"title\": \"示例合同.pdf\", // 或从内容提取的标题\n      \"contentType\": \"FILE\",\n      \"status\": \"READY\",\n      \"uploadTime\": \"2023-10-26T18:00:00Z\",\n      \"tags\": [\"合同\", \"模板\"]\n    }\n    // ... 更多知识内容\n  ],\n  \"total\": 50,\n  \"size\": 10,\n  \"current\": 1,\n  \"pages\": 5\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 500\n\n### 2.3 获取知识内容详情\n\n- **接口路径**：`/api/v1/knowledge/{knowledgeId}`\n- **请求方法**：`GET`\n- **接口说明**：获取指定知识内容的详细信息，包括原始内容或解析后的摘要。\n- **权限:** `knowledge:view`\n- **请求参数 (Path Parameters):**\n\n| 参数名      | 类型    | 是否必需 | 说明       | 示例值 |
|-------------|--------|----------|------------|--------|\
| knowledgeId | integer | 是       | 知识内容ID | 8001   |\n\n- **响应数据 (统一格式 `data` 字段):** 返回知识内容详情。\n\n```json\n{\n  \"knowledgeId\": 8001,\n  \"title\": \"示例合同.pdf\",\n  \"contentType\": \"FILE\",\n  \"originalUrl\": \"/files/abc.pdf\", // 原始文件或链接\n  \"parsedContent\": \"合同主要内容摘要...\", // AI解析后的摘要或关键信息\n  \"status\": \"READY\",\n  \"uploadTime\": \"2023-10-26T18:00:00Z\",\n  \"tags\": [\"合同\", \"模板\"],\n  \"folderId\": 1,\n  \"parseResult\": { \"sections\": [...] } // 更详细的解析结果\n}\n```\n- **可能返回状态码:** 200, 401, 403, 404 (知识内容不存在), 500\n\n### 2.4 更新知识内容信息（如标签、文件夹）\n\n- **接口路径**：`/api/v1/knowledge/{knowledgeId}`\n- **请求方法**：`PUT`\n- **接口说明**：更新指定知识内容的元信息，如标题、标签、所属文件夹等。\n- **权限:** `knowledge:update`\n- **请求参数 (Path Parameters):**\n\n| 参数名      | 类型    | 是否必需 | 说明       | 示例值 |
|-------------|--------|----------|------------|--------|\
| knowledgeId | integer | 是       | 知识内容ID | 8001   |\n\n- **请求体 (Request Body):**\n\n| 参数名    | 类型           | 是否必需 | 描述       | 示例值 |
|-----------|----------------|----------|------------|--------|\
| title     | string         | 否       | 新标题     | \"更新后的合同文档\" |\
| folderId  | integer        | 否       | 新文件夹ID | 2      |\
| tags      | array<string>  | 否       | 新标签列表 | `[\"协议\"]` |\
\n- **响应数据 (统一格式 `data` 字段):** 返回更新后的知识内容信息。\n\n```json\n{\n  \"knowledgeId\": 8001,\n  \"title\": \"更新后的合同文档\",\n  \"folderId\": 2,\n  \"tags\": [\"协议\"],\n  \"updateTime\": \"2023-10-26T18:15:00Z\"\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 404, 422, 500\n\n### 2.5 删除知识内容\n\n- **接口路径**：`/api/v1/knowledge/{knowledgeId}`\n- **请求方法**：`DELETE`\n- **接口说明**：删除指定知识内容。\n- **权限:** `knowledge:delete`\n- **请求参数 (Path Parameters):**\n\n| 参数名      | 类型    | 是否必需 | 说明       | 示例值 |
|-------------|--------|----------|------------|--------|\
| knowledgeId | integer | 是       | 知识内容ID | 8001   |\n\n- **响应数据 (统一格式 `data` 字段):** 返回成功信息。\n\n```json\n{\n  \"message\": \"知识内容删除成功\"\n}\n```\n- **可能返回状态码:** 200, 401, 403, 404, 500\n\n### 2.6 创建/管理知识文件夹\n\n- **接口路径**：`/api/v1/knowledge/folders`\n- **请求方法**：`POST` (创建), `GET` (列表)\n- **接口说明**：创建新的知识文件夹或获取文件夹列表。\n- **权限:** `knowledge:folder:manage`\n- **请求参数 (POST Request Body):**\n\n| 参数名    | 类型   | 是否必需 | 描述       | 示例值 |
|-----------|--------|----------|------------|--------|\
| folderName| string | 是       | 文件夹名称 | \"公司文档\" |\
| parentId  | integer| 否       | 父文件夹ID | null   |\n\n- **请求参数 (GET Query Parameters):**\n\n| 参数名    | 类型    | 是否必需 | 描述       | 示例值 |
|-----------|--------|----------|------------|--------|\
| parentId  | integer | 否       | 父文件夹ID | null   |\n\n- **响应数据 (统一格式 `data` 字段):**\n    - POST: 返回新创建的文件夹信息\n    ```json\n    {\n      \"folderId\": 1,\n      \"folderName\": \"公司文档\",\n      \"parentId\": null,\n      \"createTime\": \"2023-10-26T18:20:00Z\"\n    }\n    ```\n    - GET: 返回文件夹列表\n    ```json\n    [\n      {\n        \"folderId\": 1,\n        \"folderName\": \"公司文档\",\n        \"parentId\": null,\n        \"itemCount\": 10, // 文件夹内知识内容数量\n        \"subFolderCount\": 2 // 子文件夹数量\n      }\n      // ... 更多文件夹\n    ]\n    ```\n- **可能返回状态码:** 200 (GET), 201 (POST), 400, 401, 403, 404 (父文件夹不存在), 422, 500\n
### 2.7 创建/管理知识标签\n\n- **接口路径**：`/api/v1/knowledge/tags`\n- **请求方法**：`POST` (创建), `GET` (列表)\n- **接口说明**：创建新的知识标签或获取标签列表。\n- **权限:** `knowledge:tag:manage`\n- **请求参数 (POST Request Body):**\n\n| 参数名  | 类型   | 是否必需 | 描述     | 示例值 |
|---------|--------|----------|----------|--------|\
| tagName | string | 是       | 标签名称 | \"财务\" |\
\n- **请求参数 (GET Query Parameters):**\n\n| 参数名    | 类型    | 是否必需 | 描述       | 示例值 |
|-----------|--------|----------|------------|--------|\
| keyword   | string | 否       | 按名称搜索 | \"财\" |\n\n- **响应数据 (统一格式 `data` 字段):**\n    - POST: 返回新创建的标签信息\n    ```json\n    {\n      \"tagId\": 10,\n      \"tagName\": \"财务\",\n      \"createTime\": \"2023-10-26T18:25:00Z\"\n    }\n    ```\n    - GET: 返回标签列表\n    ```json\n    [\n      {\n        \"tagId\": 10,\n        \"tagName\": \"财务\",\n        \"itemCount\": 5 // 关联知识内容数量\n      }\n      // ... 更多标签\n    ]\n    ```\n- **可能返回状态码:** 200 (GET), 201 (POST), 400, 401, 403, 422, 500\n\n### 2.8 智能搜索\n\n- **接口路径**：`/api/v1/knowledge/search`\n- **请求方法**：`POST`\n- **接口说明**：对知识库进行智能搜索，支持关键词、语义匹配。\n- **权限:** `knowledge:search`\n- **请求参数 (Request Body):**\n\n| 参数名  | 类型   | 是否必需 | 描述       | 示例值 |
|---------|--------|----------|------------|--------|\
| query   | string | 是       | 搜索查询   | \"什么是云阿米巴？\" |\
| folderIds| array<integer> | 否 | 在指定文件夹内搜索 | `[1, 2]` |\
| tagIds  | array<integer> | 否 | 过滤特定标签内容 | `[10]` |\
| page    | integer| 否       | 页码       | 1      |\
| size    | integer| 否       | 每页条数   | 10     |\n\n- **响应数据 (统一格式 `data` 字段):** 返回搜索结果列表（相关度排序）。\n\n```json\n{\n  \"records\": [\n    {\n      \"knowledgeId\": 8005,\n      \"title\": \"云阿米巴模式介绍\",\n      \"snippet\": \"...云阿米巴是一种创新的业务模式...\", // 匹配到的内容片段\n      \"score\": 0.95, // 相关度得分\n      \"contentType\": \"ARTICLE\",\n      \"folderName\": \"内部资料\",\n      \"tags\": [\"云阿米巴\", \"管理\"]\n    }\n    // ... 更多结果\n  ],\n  \"total\": 15,\n  \"size\": 10,\n  \"current\": 1,\n  \"pages\": 2\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 500\n\n### 2.9 智能问答\n\n- **接口路径**：`/api/v1/knowledge/ask`\n- **请求方法**：`POST`\n- **接口说明**：基于知识库内容进行智能问答。\n- **权限:** `knowledge:ask`\n- **请求参数 (Request Body):**\n\n| 参数名   | 类型   | 是否必需 | 描述       | 示例值 |
|----------|--------|----------|------------|--------|\
| question | string | 是       | 用户问题   | \"合同模板在哪里找？\" |\
| folderIds| array<integer> | 否 | 限定在指定文件夹内问答 | `[1]` |\
| tagIds   | array<integer> | 否 | 限定在特定标签内容中问答 | `[10]` |\
\n- **响应数据 (统一格式 `data` 字段):** 返回AI生成的答案及引用的知识来源。\n\n```json\n{\n  \"answer\": \"您可以在 \"公司文档\" 文件夹下的 \"合同\" 标签分类中找到合同模板。\",\n  \"sourceNodes\": [\n    {\n      \"knowledgeId\": 8001,\n      \"title\": \"示例合同.pdf\",\n      \"snippet\": \"...本文件夹包含各类合同模板...\" // 引用片段\n    }\n    // ... 更多引用来源\n  ]\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 500\n\n## 3. 数据模型设计\n\n### 3.1 知识内容表 `t_knowledge_content`\n\n存储知识库中的知识内容元信息和处理状态。\n\n| 字段名         | 类型         | 是否必需 | 说明             | 索引        |\n|---------------|--------------|----------|------------------|------------|\
| id            | BIGINT (PK)  | 是       | 主键             |            |\
| user_id       | BIGINT (FK)  | 是       | 上传用户ID       | Index      |\
| title         | VARCHAR(255) | 是       | 内容标题         | Index      |\
| content_type  | VARCHAR(20)  | 是       | 内容类型 (FILE, URL, TEXT, IMAGE) | Index |\
| original_url  | VARCHAR(500) | 否       | 原始文件路径或URL |            |\
| status        | VARCHAR(20)  | 是       | 处理状态 (UPLOADED, PROCESSING, READY, FAILED) | Index |\
| folder_id     | BIGINT (FK)  | 否       | 所属文件夹ID     | Index      |\
| upload_time   | DATETIME     | 是       | 上传时间         |            |\
| process_time  | DATETIME     | 否       | AI处理完成时间   |            |\
| error_message | TEXT         | 否       | 处理失败原因     |            |\
\n### 3.2 知识内容详情/解析表 `t_knowledge_parsed`\n\n存储AI解析后的知识内容细节，用于搜索和问答。\n\n| 字段名         | 类型         | 是否必需 | 说明             | 索引        |\n|---------------|--------------|----------|------------------|------------|\
| id            | BIGINT (PK)  | 是       | 主键             |            |\
| knowledge_id  | BIGINT (FK)  | 是       | 关联的知识内容ID | UNIQUE Index |\
| parsed_content| TEXT         | 否       | AI提取的文本内容或摘要 |            |\
| vector_embedding| BLOB / BYTEA | 否       | 向量嵌入 (用于向量搜索) |            |\
| keywords      | JSON / TEXT  | 否       | 提取的关键词列表 (JSON 数组) |            |\
| sections      | JSON / TEXT  | 否       | 内容分段及结构 (JSON) |            |\
\n### 3.3 知识文件夹表 `t_knowledge_folder`\n\n存储知识库文件夹信息。\n\n| 字段名      | 类型         | 是否必需 | 说明           | 索引        |\n|------------|--------------|----------|----------------|------------|\
| id         | BIGINT (PK)  | 是       | 主键           |            |\
| user_id    | BIGINT (FK)  | 是       | 创建用户ID     | Index      |\
| folder_name| VARCHAR(100) | 是       | 文件夹名称     |            |\
| parent_id  | BIGINT (FK)  | 否       | 父文件夹ID     | Index      |\
| create_time| DATETIME     | 是       | 创建时间       |            |\
| update_time| DATETIME     | 是       | 更新时间       |            |\
\n### 3.4 知识标签表 `t_knowledge_tag`\n\n存储知识标签信息。\n\n| 字段名     | 类型         | 是否必需 | 说明     | 索引        |\n|-----------|--------------|----------|----------|------------|\
| id        | BIGINT (PK)  | 是       | 主键     |            |\
| user_id   | BIGINT (FK)  | 是       | 创建用户ID | Index      |\
| tag_name  | VARCHAR(50)  | 是       | 标签名称 | UNIQUE Index |\
| create_time| DATETIME     | 是       | 创建时间 |            |\
\n### 3.5 知识内容-标签关联表 `t_knowledge_tag_rel`\n\n关联知识内容和标签，实现多对多关系。\n\n| 字段名      | 类型        | 是否必需 | 说明         | 索引        |\n|------------|-------------|----------|--------------|------------|\
| id         | BIGINT (PK) | 是       | 主键         |            |\
| knowledge_id| BIGINT (FK) | 是       | 知识内容ID   | Index      |\
| tag_id     | BIGINT (FK) | 是       | 标签ID       | Index      |\
\n## 4. 异常处理\n\n- `KnowledgeUploadException`: 知识内容上传失败异常\n- `KnowledgeParseException`: AI解析失败异常\n- `KnowledgeNotFoundException`: 知识内容不存在异常\n- `FolderNotFoundException`: 文件夹不存在异常\n- `TagNotFoundException`: 标签不存在异常\n- `KnowledgeSearchException`: 智能搜索异常\n- `KnowledgeAskException`: 智能问答异常\n- `InsufficientPermissionsException`: 权限不足异常\n\n## 5. 开发注意事项和实现要点\n\n1.  **AI 模型集成:**\n    - 集成文本解析、图像识别、向量化等AI能力。可以选择第三方AI服务（如腾讯云、阿里云的NLP/CV服务）或开源模型。\n    - 需要有可靠的AI服务调用接口和错误处理机制。\n2.  **知识存储和索引:**\n    - 存储原始文件和解析后的文本内容。对于向量搜索，需要专门的向量数据库或支持向量索引的数据库（如PostgreSQL + pgvector）。\n    - 构建文本搜索索引（如 Elasticsearch, Solr）和向量索引，提高搜索效率。\n3.  **异步处理:**\n    - 知识内容的上传和AI解析是耗时操作，应采用消息队列和异步任务处理，避免阻塞前端请求。上传成功后立即返回，前端通过轮询或 WebSocket 获取处理状态。\n4.  **搜索和问答逻辑:**\n    - 实现关键词搜索和基于向量相似度的语义搜索。\n    - 实现基于知识库内容的问答系统，将用户问题与知识内容匹配，提取相关信息生成答案。可能需要RAG (Retrieval-Augmented Generation) 架构。\n5.  **权限控制:**\n    - 对知识内容的上传、查看、编辑、删除、文件夹和标签管理、搜索、问答等操作进行权限控制。确保用户只能访问有权限的知识内容。\n6.  **文件安全:**\n    - 上传的文件需要进行安全扫描，防止病毒或恶意内容。\n    - 存储的文件需要有访问控制，防止未经授权的访问。\n7.  **可扩展性:**\n    - 设计易于扩展的架构，方便后续支持更多内容类型、集成更多AI模型或服务。\n8.  **成本控制:**\n    - 调用外部AI服务可能产生费用，需要考虑成本控制和优化。\n9.  **日志记录:**\n    - 记录上传、解析、搜索、问答等关键操作的日志，便于排查问题和监控。\n\n--- 