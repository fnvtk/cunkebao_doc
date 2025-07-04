# 存客宝工作台 - AI助手功能后端开发文档\n\n## 1. 模块概述\n\nAI助手功能为用户提供智能辅助，可能包括但不限于：内容创作建议、智能回复推荐、数据分析摘要、常见问题解答等。后端模块负责集成不同的AI模型和能力，接收用户请求，调用相应的AI服务进行处理，并返回结果。此模块将依赖于其他模块（如知识库、数据统计、客户管理）提供的数据。\n\n### AI助手功能流程图\n\n```mermaid\ngraph TD\n    A[用户在工作台发起AI助手请求] --> B{AI助手后端服务};\n    B -- 获取能力列表 --> C[AI能力配置表];\n    B -- 调用能力 --> D{AI服务集成层};\n    D -- 调用外部AI模型/服务 --> E[外部AI服务 (如LLM/NLP)];\n    E --> D;\n    D --> F[记录交互历史];\n    F --> G[AI助手交互历史表];\n    D -- 返回结果 --> B;\n    B --> H[将结果返回给前端];\n    B -- 查询历史 --> G;\n    G --> B;\n```\n\n## 2. API接口设计\n\n### 2.1 获取AI助手能力列表\n\n- **接口路径**：`/api/v1/workbench/ai-assistant/capabilities`\n- **请求方法**：`GET`\n- **接口说明**：获取AI助手当前支持的能力列表，供前端展示和引导用户使用。\n- **权限:** `workbench:ai-assistant:capabilities:view`\n- **请求参数 (Query Parameters):** 无\n\n- **响应数据 (统一格式 `data` 字段):** 返回能力列表。\n\n```json\n[\n  {\n    \"capabilityId\": \"CONTENT_SUGGESTION\",\n    \"capabilityName\": \"内容创作建议\",\n    \"description\": \"根据您的输入和热点提供内容创意和大纲\",\n    \"icon\": \"icon_url\",\n    \"parameters\": [ // 需要用户输入或选择的参数\n      {\n        \"paramName\": \"topic\",\n        \"paramType\": \"string\",\n        \"required\": true,\n        \"description\": \"请提供内容主题\"\n      },\n       {\n        \"paramName\": \"platform\",\n        \"paramType\": \"enum\",\n        \"options\": [\"抖音\", \"小红书\", \"微信公众号\"],\n        \"required\": false,\n        \"description\": \"目标发布平台\"\n      }\n    ]\n  },\n   {\n    \"capabilityId\": \"SMART_REPLY\",\n    \"capabilityName\": \"智能回复推荐\",\n    \"description\": \"根据对话上下文生成回复建议\",\n    \"icon\": \"icon_url\",\n    \"parameters\": [\n       {\n        \"paramName\": \"context\",\n        \"paramType\": \"string\",\n        \"required\": true,\n        \"description\": \"请输入对话上下文\"\n      }\n    ]\n  }\n  // ... 更多能力\n]\n```\n- **可能返回状态码:** 200, 401, 403, 500\n\n### 2.2 调用AI助手能力\n\n- **接口路径**：`/api/v1/workbench/ai-assistant/invoke/{capabilityId}`\n- **请求方法**：`POST`\n- **接口说明**：根据用户选择的能力ID和提供的参数，调用后端AI服务进行处理。\n- **权限:** `workbench:ai-assistant:invoke:{capabilityId}` (细粒度权限) 或 `workbench:ai-assistant:invoke`\n- **请求参数 (Path Parameters):**\n\n| 参数名       | 类型   | 是否必需 | 说明           | 示例值 |
|--------------|--------|----------|----------------|--------|\
| capabilityId | string | 是       | AI助手能力ID   | \"CONTENT_SUGGESTION\" |\n\n- **请求体 (Request Body):** 包含调用该能力所需的参数，结构与 `capabilities` 接口返回的 `parameters` 定义对应。\n\n```json\n{\n  \"parameters\": {\n    \"topic\": \"私域流量运营\",\n    \"platform\": \"抖音\"\n  }\n}\n```\n- **响应数据 (统一格式 `data` 字段):** 返回AI处理结果。结果结构根据 `capabilityId` 不同而异。\n\n```json\n{\n  \"resultType\": \"text\", // text, json, html, image_url, etc.\n  \"data\": \"这是一篇关于私域流量运营的抖音内容大纲：...\" // 具体的AI生成内容\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 404 (能力不存在), 422 (参数错误), 500 (AI服务调用失败)\n\n### 2.3 获取AI助手对话历史（可选）\n\n- **接口路径**：`/api/v1/workbench/ai-assistant/history`\n- **请求方法**：`GET`\n- **接口说明**：获取用户与AI助手的交互历史记录。\n- **权限:** `workbench:ai-assistant:history:view`\n- **请求参数 (Query Parameters):**\n\n| 参数名   | 类型    | 是否必需 | 描述       | 示例值 |
|----------|--------|----------|------------|--------|\
| capabilityId| string| 否       | 按能力过滤 | \"SMART_REPLY\" |\
| startTime| string | 否       | 时间开始   |        |\
| endTime  | string | 否       | 时间结束   |        |\
| page     | integer| 否       | 页码       | 1      |\
| size     | integer| 否       | 每页条数   | 10     |\n\n- **响应数据 (统一格式 `data` 字段):** 返回历史记录列表。\n\n```json\n{\n  \"records\": [\n    {\n      \"historyId\": 9001,\n      \"capabilityId\": \"CONTENT_SUGGESTION\",\n      \"request\": {\"topic\": \"私域流量运营\"},\n      \"response\": {\"resultType\": \"text\", \"data\": \"...\"},\n      \"requestTime\": \"2023-10-26T18:30:00Z\"\n    }\n    // ... 更多记录\n  ],\n  \"total\": 100,\n  \"size\": 10,\n  \"current\": 1,\n  \"pages\": 10\n}\n```\n- **可能返回状态码:** 200, 400, 401, 403, 500\n\n## 3. 数据模型设计\n\n### 3.1 AI助手能力配置表 `t_ai_capability_config`\n\n存储AI助手的各种能力配置信息。\n\n| 字段名         | 类型         | 是否必需 | 说明             | 索引        |\n|---------------|--------------|----------|------------------|------------|\
| id            | BIGINT (PK)  | 是       | 主键             |            |\
| capability_id | VARCHAR(50)  | 是       | 能力唯一标识ID   | UNIQUE Index |\
| capability_name| VARCHAR(100) | 是       | 能力名称         |            |\
| description   | VARCHAR(500) | 否       | 能力描述         |            |\
| icon_url      | VARCHAR(255) | 否       | 能力图标URL      |            |\
| parameters    | JSON / TEXT  | 否       | 参数定义 (JSON 数组) |            |\
| backend_service| VARCHAR(100) | 是       | 关联的后端服务标识 | Index      |\
| status        | VARCHAR(20)  | 是       | 状态 (ACTIVE, INACTIVE) | Index |\
| create_time   | DATETIME     | 是       | 创建时间         |            |\
| update_time   | DATETIME     | 是       | 更新时间         |            |\
\n### 3.2 AI助手交互历史表 `t_ai_assistant_history`\n\n存储用户与AI助手的每次交互记录（请求和响应）。\n\n| 字段名         | 类型         | 是否必需 | 说明             | 索引        |\n|---------------|--------------|----------|------------------|------------|\
| id            | BIGINT (PK)  | 是       | 主键             |            |\
| user_id       | BIGINT (FK)  | 是       | 用户ID           | Index      |\
| capability_id | VARCHAR(50)  | 是       | 关联的能力ID     | Index      |\
| request_data  | JSON / TEXT  | 是       | 用户请求数据 (JSON)|            |\
| response_data | JSON / TEXT  | 否       | AI响应数据 (JSON)  |            |\
| status        | VARCHAR(20)  | 是       | 处理状态 (SUCCESS, FAILED) | Index |\
| error_message | TEXT         | 否       | 失败原因         |            |\
| request_time  | DATETIME     | 是       | 请求时间         | Index      |\
| response_time | DATETIME     | 否       | 响应时间         |            |\
\n## 4. 异常处理\n\n- `CapabilityNotFoundException`: 指定的AI助手能力不存在异常\n- `InvalidCapabilityParametersException`: 调用能力时提供的参数无效异常\n- `AiServiceInvocationException`: 调用后端AI服务失败异常\n- `AiAssistantHistoryQueryException`: 查询历史记录异常\n- `InsufficientPermissionsException`: 权限不足异常\n\n## 5. 开发注意事项和实现要点\n\n1.  **AI服务集成层:**\n    - 需要建立一个AI服务集成层，负责对接不同的AI模型或第三方AI服务提供商（如OpenAI, 腾讯云、阿里云的LLM/NLP/CV服务）。\n    - 封装底层AI服务的调用细节，向上层提供统一的调用接口。\n2.  **能力路由和分发:**\n    - 根据 `capabilityId` 将用户请求路由到对应的后端AI服务或处理逻辑。\n3.  **数据传递和格式:**\n    - 确保用户输入的参数能正确传递给后端AI服务。\n    - 处理不同AI服务返回结果的格式差异，转换为统一的响应结构。\n4.  **异步处理:**\n    - 某些AI能力（如复杂内容生成）可能耗时较长，可以考虑异步处理，提供任务ID，前端通过轮询获取结果。\n5.  **错误处理和回传:**\n    - 捕获调用AI服务过程中的各种错误（连接超时、服务内部错误、内容违规等），并返回清晰的错误信息给前端。\n6.  **权限控制:**\n    - 对获取能力列表和调用具体能力进行权限控制。某些高级能力可能需要特定用户权限。\n7.  **成本控制:**\n    - 调用外部AI服务会产生费用，需要监控调用量，并考虑成本优化策略（如使用更经济的模型、缓存常见结果）。\n8.  **日志记录:**\n    - 记录AI助手的使用日志，包括用户、调用的能力、参数、结果（摘要）和耗时，用于监控和分析。\n9.  **与中台知识库集成:**\n    - 如果AI助手需要基于企业知识库回答问题或生成内容，需要调用知识库模块提供的搜索和问答接口。\n\n--- 

## 相关前端UI图片

以下是与AI助手功能可能相关的部分前端UI截图，帮助理解用户如何在前端界面使用和管理AI助手：

### 工作台 - AI助手入口与使用示例 (示意图)

![工作台](4、前端/UI/工作台.png) 