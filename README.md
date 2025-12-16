# 大模型评测系统

## 📋 项目简介

基于FastAPI的大模型评测系统，采用**双API架构**设计：
- **公司Chat API** - 调用被评测的各种模型获取答案
- **火山引擎API** - 专门用于评测，比较标准答案和生成答案

## 🏗️ 系统架构

### 核心架构
```
标准问题 → 公司Chat API（被评测模型） → 生成答案 → 火山引擎API（评测模型） → 评测结果
```

### 技术栈
- **后端框架**: FastAPI 0.104.0+
- **数据库**: MySQL + aiomysql (异步连接池)
- **HTTP客户端**: httpx (异步)
- **数据验证**: Pydantic 2.4.0+
- **日志系统**: Python logging + RotatingFileHandler
- **任务队列**: FastAPI BackgroundTasks
- **配置管理**: pydantic-settings + python-dotenv

## 🎯 核心功能

### 1. 双API服务
- **CompanyChatService**: 调用公司各种模型获取答案
- **LLMService**: 调用火山引擎进行评测

### 2. 以模型为核心的设计
- 每个模型预配置标准问题和标准答案
- 模型-问题强关联，避免数据不匹配
- 简化的API接口，专注核心功能

### 3. 完整评测流程
- 获取模型的标准问题集
- 调用公司模型获取答案
- 使用火山引擎进行评测打分
- 生成详细评测报告

### 4. 增强功能
- 模型统计和性能分析
- 批量聊天任务执行
- 评测结果管理和查询
- 健康检查和系统监控

## 📊 数据库设计

### 核心表结构
```sql
llm_models (模型表)
├── model_name (显示名称)
├── api_model_name (API调用名称)
├── model_type (模型类型)
├── description (模型描述)
└── is_active (是否激活)

model_questions (模型问题关联表)
├── model_id (模型ID)
├── question_id (问题ID)
├── question_order (问题顺序)
└── is_active (是否激活)

questions (问题表)
├── title (问题标题)
├── content (问题内容)
├── standard_answer (标准答案)
└── is_active (是否激活)

evaluation_records (评测记录表)
├── task_id (任务ID)
├── question_id (问题ID)
├── target_model_name (目标模型名称)
├── evaluator_model_name (评测模型名称)
├── model_answer (模型回答)
├── reference_answer (标准答案)
├── score (评分)
├── evaluation_result (评测结果)
├── evaluation_metrics (评测指标)
├── response_time (响应时间)
└── created_at (创建时间)
```

## 🚀 快速开始

### 1. 环境准备
```bash
# 克隆项目
git clone <repository>
cd llm_evaluation

# 创建虚拟环境
python -m venv venv
source venv/bin/activate  # Linux/Mac
# 或
venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt
```

### 2. 配置环境
```bash
# 复制环境变量模板
cp env_example.txt .env

# 编辑配置文件
vim .env
```

#### 必需配置项
```bash
# 数据库配置
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_password
DB_NAME=llm_evaluation

# 公司Chat API配置（被评测的模型）
COMPANY_CHAT_API_BASE=https://taicenterapi.hundun.cn/v1/chat
COMPANY_CHAT_API_TOKEN=your_bearer_token_here
COMPANY_USER_ID=your_user_id_here

# 火山引擎API配置（评测模型）
VOLCANO_API_BASE=https://ark.cn-beijing.volces.com/api/v3
VOLCANO_API_KEY=your_volcano_api_key_here
VOLCANO_MODEL=doubao-seed-1-6-thinking-250715
```

### 3. 数据库初始化
```bash
# 创建数据库
mysql -u root -p -e "CREATE DATABASE llm_evaluation CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 执行表结构脚本
mysql -u root -p llm_evaluation < app/sql/create_tables.sql

# 插入初始数据
mysql -u root -p llm_evaluation < app/sql/init_data.sql
```

### 4. 启动系统
```bash
# 开发模式启动
python run.py

# 或使用uvicorn直接启动
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 5. 验证系统
```bash
# 访问API文档
# http://127.0.0.1:8000/docs

# 健康检查
curl http://127.0.0.1:8000/api/v1/health
```

## 🔧 系统测试

### 健康检查
```bash
# 基本健康检查
curl http://127.0.0.1:8000/api/v1/health

# 数据库检查
curl http://127.0.0.1:8000/api/v1/health/database

# 双API服务检查
curl http://127.0.0.1:8000/api/v1/health/llm

# 详细健康检查
curl http://127.0.0.1:8000/api/v1/health/detailed
```

### 核心API测试
```bash
# 获取模型列表
curl http://127.0.0.1:8000/api/v1/models

# 获取模型统计
curl http://127.0.0.1:8000/api/v1/models/stats

# 获取模型详情
curl http://127.0.0.1:8000/api/v1/models/1

# 获取模型的评测问题集
curl http://127.0.0.1:8000/api/v1/models/1/questions

# 执行公司聊天对话
curl -X POST http://127.0.0.1:8000/api/v1/models/1/chat \
  -H "Content-Type: application/json" \
  -d '{
    "execution_count": 1,
    "max_workers": 2
  }'
```

## 📚 API接口文档

### 健康检查
- `GET /api/v1/health` - 基本健康检查
- `GET /api/v1/health/database` - 数据库健康检查
- `GET /api/v1/health/llm` - 双API服务检查
- `GET /api/v1/health/detailed` - 详细健康检查

### 模型管理
- `GET /api/v1/models` - 获取模型列表
- `GET /api/v1/models/stats` - 获取模型统计
- `GET /api/v1/models/{id}` - 获取模型详情
- `GET /api/v1/models/{id}/questions` - 获取模型评测问题集
- `POST /api/v1/models/{id}/chat` - 执行公司聊天对话

### 评测功能
- `POST /api/v1/evaluate/model` - 模型评测
- `GET /api/v1/models/{model_id}/tasks` - 获取模型任务列表
- `GET /api/v1/models/{model_id}/tasks/{task_id}/conversations` - 获取对话执行结果
- `GET /api/v1/evaluate/statistics` - 评测统计
- `GET /api/v1/evaluate/results/{model_id}/{task_id}` - 获取评测结果

## 🎯 评测流程详解

### 1. 标准问题配置
每个模型预配置标准问题，包含：
- 基础数学计算
- 语言理解能力
- 常识推理判断
- 创意表达能力
- 逻辑分析推理

### 2. 模型专属评测提示词
每个模型都有自己的评测提示词模板，支持：
- 针对模型特性的专项评测标准
- 统一的评分体系
- 标准化JSON输出格式

### 3. 评测流程
```python
# 1. 获取模型的标准问题和专属评测提示词
model_info = await LLMModel.get_model_with_prompt(model_id)

# 2. 使用公司Chat API获取模型答案
chat_service = await get_company_chat_service()
model_answer = await chat_service.call_model(model_name, question)

# 3. 调用火山引擎进行评测
evaluation_service = await get_evaluation_service()
eval_result = await evaluation_service.evaluate_with_parallel_calls(
    standard_question=question,
    actual_answer=model_answer,
    standard_answer=standard_answer,
    target_model_name=model_name
)

# 4. 自动计算总分和维度平均分
total_score = eval_result['total_score']    # 100分制
dimension_scores = eval_result['scores']    # 维度平均分
```

### 4. 评测体系
系统采用标准化的评测体系：

| 维度 | 权重 | 说明 |
|------|------|------|
| **完整性** | 25% | 答案覆盖核心信息的完整程度 |
| **专业性** | 25% | 专业水准和理论应用准确性 |
| **逻辑性** | 20% | 逻辑结构和推理合理性 |
| **实用性** | 15% | 可操作性和实际应用价值 |
| **创新性** | 10% | 独特见解和创新思维 |
| **表达清晰度** | 5% | 语言表达和结构组织 |

**总分计算公式**: `总分 = Σ(维度分数 × 维度权重) × 20`（转换为100分制）

### 5. 评测保障机制
- **并发调用**: 同时发起多次评测请求，提高准确性
- **自动重试**: 每次调用失败时自动重试
- **保护机制**: 成功调用少于预期时自动补充调用
- **降级处理**: 所有评测失败时使用默认分数
- **智能修复**: 自动修复AI返回的JSON格式错误

## 📁 项目结构

```
llm_evaluation/
├── app/                          # 应用主目录
│   ├── main.py                  # FastAPI应用入口 ⭐
│   ├── database.py              # 数据库连接管理 ⭐
│   ├── startup.py               # 应用启动管理
│   ├── api/                     # API路由
│   │   ├── health.py           # 健康检查
│   │   ├── models.py           # 模型管理
│   │   └── evaluation.py       # 评测接口
│   ├── core/                    # 核心配置
│   │   ├── config.py           # 配置管理
│   │   ├── logging.py          # 日志配置
│   │   ├── dependencies.py     # 依赖注入
│   │   └── error_codes.py      # 错误码定义
│   ├── models/                  # 数据模型
│   │   ├── base.py             # 基础模型
│   │   ├── llm_model.py        # 模型管理
│   │   ├── question.py         # 问题管理
│   │   ├── evaluation.py       # 评测记录
│   │   └── model_evaluation_prompt.py  # 模型专属评测提示词
│   ├── schemas/                 # Pydantic数据模式
│   │   ├── evaluation.py       # 评测相关模式
│   │   ├── question.py         # 问题相关模式
│   │   └── enums.py            # 枚举定义
│   ├── services/                # 业务服务层
│   │   ├── company_chat_service.py  # 公司Chat API
│   │   ├── llm_service.py          # 火山引擎API
│   │   └── evaluation_service.py   # 评测服务
│   ├── sql/                     # SQL脚本
│   │   ├── create_tables.sql   # 表结构
│   │   └── init_data.sql       # 初始数据
│   ├── utils/                   # 工具函数
│   │   ├── helpers.py          # 通用工具
│   │   └── db_init.py          # 数据库检查
│   └── data/                    # 数据文件
│       └── phone_numbers.json  # 手机号数据
├── tests/                       # 测试目录 🧪
│   ├── __init__.py
│   └── test_system.py          # 综合系统测试
├── logs/                        # 日志文件
├── run.py                      # 启动脚本
├── requirements.txt            # 依赖文件
├── .env                        # 环境变量
├── .gitignore                 # Git忽略文件
└── README.md                   # 项目文档
```

### 🏗️ 核心架构文件说明

- ⭐ **`app/main.py`** - FastAPI应用入口，定义路由、中间件、异常处理
- ⭐ **`app/database.py`** - 数据库连接池管理，提供aiomysql操作封装
- 🧪 **`tests/test_system.py`** - 综合系统测试，整合所有测试功能

## 🔍 故障排除

### 常见问题

1. **数据库连接失败**
   ```bash
   # 检查数据库配置
   curl http://127.0.0.1:8000/api/v1/health/database
   
   # 确认数据库服务启动
   systemctl status mysql
   ```

2. **公司Chat API连接失败**
   ```bash
   # 检查API配置
   curl http://127.0.0.1:8000/api/v1/health/llm
   
   # 确认Token和用户ID正确
   ```

3. **火山引擎API连接失败**
   ```bash
   # 检查API密钥和模型配置
   # 确认网络可访问火山引擎API
   ```

4. **模型没有关联问题**
   ```bash
   # 检查model_questions关联表
   mysql -u root -p llm_evaluation -e "SELECT * FROM model_questions;"
   
   # 重新执行初始数据脚本
   mysql -u root -p llm_evaluation < app/sql/init_data.sql
   ```

### 日志查看
```bash
# 查看应用日志
tail -f logs/app.log

# 查看错误日志
tail -f logs/error.log

# 查看API日志
tail -f logs/api.log

# 查看数据库日志
tail -f logs/database.log

# 查看LLM调用日志
tail -f logs/llm.log
```

## 🚀 部署建议

### 生产环境配置
1. **使用HTTPS**
2. **配置反向代理** (Nginx/Apache)
3. **设置防火墙规则**
4. **配置数据库连接池**
5. **启用日志轮转**
6. **配置监控告警**

### Docker部署
```dockerfile
# 可以创建Dockerfile进行容器化部署
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "run.py"]
```

## 📞 技术支持

- **文档**: 查看API文档 http://127.0.0.1:8000/docs
- **健康检查**: http://127.0.0.1:8000/api/v1/health/detailed
- **应用信息**: http://127.0.0.1:8000/info

## 📈 版本信息

- **版本**: 1.0.0
- **架构**: 双API设计（公司Chat API + 火山引擎API）
- **特性**: 以模型为核心，标准问题预配置，完整评测流程，批量聊天任务