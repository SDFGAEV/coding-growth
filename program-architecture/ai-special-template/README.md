
---
# Python项目规范目录与细则（AI专属版）
## 一、核心设计理念
本规范基于**通用版增量扩展**，专为 AI 推理/训练项目设计。通用版是所有项目的基础，本版在此基础上解决 AI 项目特有的环境重、依赖大、敏感信息多、模型部署难问题。
**AI专属铁律**：
1. **AI与通用业务解耦**：AI模块为插件式，禁用AI模式时项目零AI依赖负担。
2. **按需加载**：通过参数控制，避免无显卡环境加载 `torch` 等重依赖。
3. **敏感信息隔离**：密钥禁止硬编码，必须从 `.env.ai` 读取。
---
## 二、完整目录结构
**核心规则**：在通用版基础上**增量添加AI相关文件/目录**，AI模块与通用业务解耦。
```
your_project/
├── README.md               # 项目总说明（必选，补充AI相关说明）
├── requirements.txt        # 依赖清单（必选，添加AI相关依赖）
├── .gitignore              # Git忽略规则（必选，添加AI大文件忽略）
├── main.py                 # 项目统一入口（必选，支持AI推理参数）
├── config/                 # 全局配置（必选，无__init__.py，增量2个AI配置）
│   ├── path.py             # 路径配置（必选，新增AI相关路径）
│   ├── logger.py           # 日志配置（必选，通用版无改动）
│   ├── constants.py        # 常量/枚举管理（必选，新增AI相关常量）
│   ├── args.py             # 命令行参数解析（必选，新增AI推理参数）
│   ├── api_config.py       # 云端AI-API配置（可选，AI专属）
│   └── model_config.py     # 本地AI模型配置（可选，AI专属）
├── core/                   # 核心业务逻辑（必选，无__init__.py，增量AI推理核心）
│   ├── [业务主模块].py     # 通用业务主模块（必选，可调用AI推理模块）
│   └── infer/              # AI推理核心（可选，无__init__.py，AI专属）
│       ├── infer_manager.py # AI统一推理调度（必选，若有infer目录）
│       ├── api_infer.py    # 云端API推理（可选，AI专属）
│       └── local_infer.py  # 本地模型推理（可选，AI专属）
├── utils/                  # 通用工具库（必选，无__init__.py，增量1个AI工具）
│   ├── file.py             # 文件通用操作（必选，通用版无改动）
│   ├── env.py              # 环境检查/校验（必选，新增CUDA/显卡检测）
│   ├── exception.py        # 异常处理/装饰器（必选，通用版无改动）
│   └── ai_utils.py         # AI通用工具（可选，AI专属，可跨AI项目复用）
├── scripts/                # 可执行场景脚本（必选，无__init__.py，文件按需）
│   ├── [通用脚本].py       # 通用场景脚本（可选，通用版无改动）
│   └── [AI脚本].py         # AI推理脚本，如run_ai_infer.py（可选，AI专属）
├── tests/                  # 单元测试（必选，无__init__.py，文件按需）
│   ├── [通用测试].py       # 通用模块测试（可选，通用版无改动）
│   └── test_ai/            # AI模块测试（可选，无__init__.py，AI专属）
├── data/                   # 输入数据存储（必选，子目录按需，增量AI测试数据）
│   ├── raw/                # 原始未清洗数据（可选）
│   ├── test/               # 通用轻量测试数据（可选）
│   └── ai_test/            # AI推理专用测试数据（可选，AI专属）
├── models/                 # 本地AI模型权重（可选，无__init__.py，AI专属）
│   └── README.md           # 模型说明（必选，若有models目录，AI专属）
└── outputs/                # 输出结果存储（必选，增量AI结果目录）
    ├── logs/               # 日志文件（必选，通用版无改动）
    ├── results/            # 通用业务结果（可选，通用版无改动）
    └── ai_results/         # AI推理结果（可选，按api/local分类，AI专属）
```
---
## 三、各目录/文件核心要求
### 1. 根目录文件（必选4个，通用版基础上**增量补充AI内容**）
#### 1.1 README.md
- 包含通用版所有内容，**增量添加AI相关说明**：AI环境配置（CUDA版本、显卡要求、torch安装命令）、AI模型准备、AI专属运行命令。
- 明确区分「通用模式」和「AI模式」的运行方式。
#### 1.2 requirements.txt
- 包含通用版所有依赖，**增量添加AI相关依赖**：
  - 云端API推理：`requests>=2.31.0`、`pyjwt>=2.8.0`。
  - 本地模型推理：`torch>=2.0.0`、`transformers>=4.38.0`、`accelerate>=0.20.0`。
#### 1.3 .gitignore
- 包含通用版所有忽略项，**增量添加AI相关大文件/敏感文件忽略**：
  - 模型文件：`models/`、`!models/README.md`、`*.safetensors`、`*.bin`。
  - AI敏感文件：`.env.ai`（存储API密钥）。
#### 1.4 main.py
- 保留通用版所有核心职责，**增量添加AI专属可选参数**：`--infer-type`（api/local/none，默认none）。
- 根据`--infer-type`参数**动态加载AI模块**，非AI场景不加载任何AI相关代码。
### 2. config/ 全局配置目录（通用版4个必选+AI2个可选）
#### 2.1 通用版必选文件（4个，**仅增量补充AI内容**）
- `path.py`：新增AI相关路径常量：`MODELS_DIR`、`AI_TEST_DATA_DIR`、`AI_RESULTS_DIR`。
- `constants.py`：新增AI相关常量：`ModelType`（模型枚举）、`DEFAULT_TEMPERATURE`、`API_TIMEOUT`。
- `args.py`：新增AI专属命令行参数。
- `logger.py`：通用版无任何改动。
#### 2.2 AI专属可选文件（2个）
- **api_config.py**：管理云端AI API配置。**密钥优先从`.env.ai`文件读取，禁止硬编码**。
- **model_config.py**：管理本地AI模型配置。实现设备自动检测（调用`utils/env.py`），无CUDA则自动切换为CPU。
### 3. core/ 核心业务目录（通用版1个必选+AI infer子目录）
#### 3.1 通用版[业务主模块].py（必选，无任何改动）
- 保留通用版所有核心业务逻辑，如需融合AI能力，仅通过调用`core/infer/infer_manager.py`的统一接口实现。
#### 3.2 AI专属infer/子目录（可选，AI推理核心）
- **infer_manager.py（必选）**：对外暴露统一的AI推理接口`run_ai_infer(input_data, **kwargs)`，屏蔽底层差异；根据`infer_type`调度子模块。
- **api_infer.py（可选）**：纯HTTP请求逻辑，**无任何模型加载代码**。
- **local_infer.py（可选）**：纯本地模型逻辑，**无任何网络请求代码**，支持CUDA/CPU自动切换、量化加载。
### 4. utils/ 通用工具目录（通用版3个必选+AI1个可选）
#### 4.1 通用版必选文件（3个，**局部补充AI能力**）
- `env.py`：新增AI环境专属检测：CUDA版本/显卡可用性检测、AI核心依赖版本校验。
- `file.py`/`exception.py`：通用版无任何改动。
#### 4.2 AI专属可选文件：ai_utils.py
- 封装无业务逻辑的AI通用工具函数：prompt模板渲染、文本按token截断、模型输入格式转换。
### 5. scripts/ / tests/ / data/ / models/ / outputs/（AI专属增量）
- **scripts/**：AI专属脚本支持`--infer-type`/`--model-type`等参数。
- **tests/**：`test_ai/`子目录，AI测试用例使用`data/ai_test/`专用数据，禁止调用真实云端API。
- **models/**：仅存储本地AI模型权重，必须包含README.md说明下载地址，权重文件通过`.gitignore`忽略。
- **outputs/**：新增`ai_results/`存储推理结果。
---
## 四、全局通用规则
### 1. 继承通用版所有规则
AI专属版完全继承通用版的**模块化与解耦、导入与运行、类型与命名、柔性生成顺序、生成后自检、可部署性**规则。
### 2. AI专属核心规则
#### 2.1 AI与通用业务完全解耦
- AI模块为增量扩展，删除所有AI专属文件后，项目可恢复为纯通用版正常运行。
- 通用业务代码不包含任何AI相关代码，仅可通过接口调用。
#### 2.2 AI模块按需加载（关键）
- **延迟导入机制**：在 `main.py` 或业务代码中，必须在函数内部 `import core.infer`，而非文件头部，以确保 `--infer-type none` 时不加载 torch 等重依赖。
#### 2.3 敏感信息禁止硬编码
- 云端API密钥、模型密码必须从`.env.ai`文件读取。
#### 2.4 本地模型轻量化加载
- 默认使用量化加载，自动检测CUDA并降级CPU。
### 3. AI柔性生成顺序
通用版模块组顺序不变，在**core业务组**后增量AI专属模块组：
1. 通用版所有模块组。
2. AI核心组：`core/infer/infer_manager.py` → `api_infer.py`/`local_infer.py`。
3. AI工具组：`utils/ai_utils.py`。
4. AI配置组：`config/api_config.py` → `model_config.py`。
5. 通用版入口层/汇总组。
6. AI专属层：`scripts/[AI脚本]` → `tests/test_ai/`。
### 4. AI专属生成后自检规则
完成通用版所有自检后，增量AI专属自检：
1. AI解耦校验：删除AI文件后，通用版代码无导入错误。
2. AI参数校验：`--infer-type none`时不加载任何AI代码（检查内存占用）。
3. 敏感信息校验：代码中无硬编码密钥。
4. 模型路径校验：无硬编码路径。
5. AI环境校验：CUDA/CPU自动检测生效。
