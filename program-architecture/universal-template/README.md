# Python 项目规范目录与细则
## 一、核心设计理念
本规范旨在建立一套**高内聚、入口统一、工程化**的 Python 项目标准。
**核心原则**：
1. **入口统一**：所有业务场景（单样本/批量/实验）统一由 `main.py` 通过参数调度。
2. **业务域聚合**：`core/` 模块严格按业务域聚合，相关逻辑内聚于同一文件，严禁过度拆分，减少文件间互相调用。
3. **配置与逻辑分离**：配置层无逻辑，工具层无业务，核心层无硬编码，utils 禁止依赖 core，config 禁止反向依赖。
4. **包标识强制化**：所有 Python 模块目录（`config/`, `core/`, `utils/`, `tests/`）必须包含 `__init__.py`，仅作为包标识，**严禁包含业务逻辑**。
5. **绝对导入原则**：全项目禁止使用相对导入（`.` 或 `..`），必须使用从项目根目录开始的绝对导入（如 `from config.path import ...`）。
6. **根目录运行原则**：所有代码必须在项目根目录执行，确保 `sys.path` 一致性。
---
## 环境搭建与初始化
本章节提供跨平台的标准化环境配置指令。请在开始编写代码前按照以下步骤执行（根据实际情况调整）。
### 1. Python 版本下载与安装
**要求**：Python 版本需满足 `3.8 <= version < 3.12`（推荐 3.10 或 3.11）。
#### Windows 系统
1.  **下载**：访问 Python 官网下载 Windows installer (64-bit)。
2.  **安装关键步骤**：
    *   双击运行安装程序。
    *   **务必勾选** `Add Python 3.x to PATH`。
    *   选择 `Install Now`。
3.  **验证**：打开 CMD 或 PowerShell，输入 `python --version`。
#### Linux 系统
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.10 python3-pip
```
### 2. 虚拟环境配置（核心步骤）
**原则**：项目必须在独立虚拟环境中运行，严禁污染全局环境。
假设项目文件夹名为 `your_project`：
#### 场景 A：Windows - CMD
```cmd
cd path\to\your_project
python -m venv venv
venv\Scripts\activate.bat
```
#### 场景 B：Windows - PowerShell
```powershell
cd path\to\your_project
python -m venv venv
.\venv\Scripts\Activate.ps1
```
#### 场景 C：Linux / macOS - Bash
```bash
cd /path/to/your_project
python3 -m venv venv
source venv/bin/activate
```
### 3. 依赖安装
```bash
pip install --upgrade pip
pip install -r requirements.txt 或 pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

```
---
## 二、完整目录结构
```
your_project/
├── README.md               # 项目总说明（必选）
├── requirements.txt        # 依赖清单（必选）
├── .gitignore              # Git忽略规则（必选）
├── main.py                 # 统一入口（必选，融合所有运行场景）
├── config/                 # 全局配置（必选）
│   ├── __init__.py         # 包标识（必选）
│   ├── path.py             # 路径配置（必选）
│   ├── logger.py           # 日志配置（必选）
│   ├── constants.py        # 常量/枚举（必选）
│   └── args.py             # 命令行参数（必选）
├── core/                   # 核心业务（必选）
│   ├── __init__.py         # 包标识（必选）
│   └── [业务域].py         # 按业务域聚合（必选）
├── utils/                  # 通用工具（必选）
│   ├── __init__.py         # 包标识（必选）
│   ├── file.py             # 文件操作（必选）
│   ├── env.py              # 环境检查（必选）
│   └── exception.py        # 异常处理（必选）
├── tests/                  # 单元测试（必选）
│   ├── conftest.py         # pytest夹具（可选）
│   ├── test_core/          # 核心模块测试
│   └── test_utils/         # 工具模块测试
├── data/                   # 输入数据（必选）
│   ├── raw/                # 原始数据（可选）
│   ├── test/               # 测试数据（可选）
│   └── batch/              # 批量数据（可选）
└── outputs/                # 输出目录（必选）
    ├── logs/               # 日志文件（必选）
    └── results/            # 业务结果（可选）
```
---
## 三、各目录/文件核心要求与代码模板
### 1. 根目录文件
#### 1.1 README.md
- **内容要求**：必须包含项目介绍、目录结构说明、环境安装步骤、**融合法运行命令示例**、参数说明、常见问题。
- **融合法运行命令**：需明确展示通过 `main.py --mode` 切换场景。
    ```markdown
    # 单样本调试
    python main.py --mode single --input data/test/sample.json
    # 批量执行
    python main.py --mode batch --input data/batch/
    ```
#### 1.2 requirements.txt
- **版本控制**：首行标注 Python 版本（如 `# Requires Python >=3.8`），库依赖明确「名称+版本」（如 `pandas==1.5.3`），避免版本冲突。
- **依赖管理**：仅保留项目必需依赖，无冗余包，可按需拆分「生产依赖」和「开发依赖」。
#### 1.3 .gitignore
- **Python标准**：包含 `__pycache__/`、`*.pyc`、`venv/`、`.env`、`*.log`。
- **项目特有**：忽略大文件/临时文件：`data/raw/*`、`outputs/`（保留目录结构说明）。
#### 1.4 main.py（核心重构：融合入口）
**职责**：统一入口，负责环境检查（调用`utils/env.py`）、参数解析（调用`config/args.py`）、模式分发，无任何业务逻辑代码，所有核心功能通过绝对导入调用`core/`模块实现。
支持`--help`参数，输出清晰的参数说明，异常捕获后优雅退出（调用`utils/exception.py`），输出用户友好提示；
**运行命令**：必须在项目根目录执行`python main.py [参数]`，禁止进入子目录运行。
**完整模板**：
```python
# -*- coding: utf-8 -*-
import sys
from utils.env import check_env
from config.logger import get_logger
from config.args import parse_args
from config.path import init_dirs
logger = get_logger(__name__)
def run_single(args):
    """单样本调试逻辑"""
    from core.processor import process_single  # 延迟导入，避免加载过重
    logger.info(f"运行模式: SINGLE | 输入: {args.input}")
    process_single(args.input)
def run_batch(args):
    """批量处理逻辑"""
    from core.processor import process_batch
    logger.info(f"运行模式: BATCH | 输入: {args.input}")
    process_batch(args.input)
def run_experiment(args):
    """实验对比逻辑"""
    from core.processor import process_experiment
    logger.info(f"运行模式: EXPERIMENT | 配置: {args.config}")
    process_experiment(args.config)
def main():
    # 1. 初始化目录
    init_dirs()
    
    # 2. 环境检查
    check_env()
    
    # 3. 参数解析
    args = parse_args()
    
    # 4. 模式分发
    mode_map = {
        'single': run_single,
        'batch': run_batch,
        'experiment': run_experiment
    }
    
    if args.mode not in mode_map:
        logger.error(f"未知的运行模式: {args.mode}")
        sys.exit(1)
        
    # 5. 执行并捕获全局异常
    try:
        mode_map[args.mode](args)
        logger.info("项目执行完成")
    except Exception as e:
        logger.error(f"主程序异常退出: {e}", exc_info=True)
        sys.exit(1)
if __name__ == "__main__":
    main()
```
---
### 2. config/ 全局配置目录（必选）
#### 2.1 通用要求
- 所有文件无业务逻辑，可直接复用至其他项目。
- 全量类型注解 + 标准 docstring。
- 仅依赖Python内置库或本目录**前置生成**的文件，禁止反向依赖utils/core，绝对导入
- **每个文件末尾加 `if __name__ == "__main__":`，支持独立运行测试。**
#### 2.2 path.py（必选）
**职责**：
- 统一使用`pathlib.Path`管理所有路径，自动推导项目根目录（`PROJECT_ROOT = Path(__file__).parent.parent`），无任何硬编码路径；
- 定义所有目录常量：`PROJECT_ROOT`、`DATA_DIR`、`OUTPUT_DIR`、`LOGS_DIR`、`RESULTS_DIR`；
- 实现`init_dirs()`函数，自动创建所有必要目录（跨平台兼容，`mkdir(exist_ok=True)`）；
- 独立运行时可验证路径正确性，打印所有目录路径并检查创建结果。
**完整模板**：
```python
# -*- coding: utf-8 -*-
from pathlib import Path
# 自动推导项目根目录
PROJECT_ROOT = Path(__file__).parent.parent
# 核心目录定义
DATA_DIR = PROJECT_ROOT / "data"
OUTPUT_DIR = PROJECT_ROOT / "outputs"
LOG_DIR = OUTPUT_DIR / "logs"
RESULT_DIR = OUTPUT_DIR / "results"
def init_dirs():
    """初始化所有必要目录（跨平台兼容）"""
    for path in [LOG_DIR, RESULT_DIR]:
        path.mkdir(parents=True, exist_ok=True)
if __name__ == "__main__":
    init_dirs()
    print(f"项目根目录: {PROJECT_ROOT}")
    print(f"日志目录: {LOG_DIR}")
```
#### 2.3 logger.py（必选）
**职责**：统一日志格式，双输出（控制台+文件），滚动存储。
- 日志格式固定包含：时间、级别、文件名、函数名、行号、信息（示例：`%(asctime)s - %(levelname)s - %(filename)s:%(funcName)s:%(lineno)d - %(message)s`）；
- 日志同时输出到**控制台**和**文件**（`outputs/logs/`），使用`RotatingFileHandler`实现日志滚动（避免单文件过大）；
- 提供`get_logger()`函数，返回全局可复用的logger实例，所有模块通过该函数获取日志器，禁止重复创建；
- 日志级别、文件保存路径等引用`config.constants.py`的常量，禁止硬编码。
**完整模板**：
```python
# -*- coding: utf-8 -*-
import logging
from logging.handlers import RotatingFileHandler
from config.path import LOG_DIR, init_dirs
init_dirs()
LOG_FORMAT = "%(asctime)s - %(levelname)s - %(filename)s:%(funcName)s:%(lineno)d - %(message)s"
def get_logger(name: str = "root") -> logging.Logger:
    logger = logging.getLogger(name)
    if logger.handlers: return logger  # 单例模式
    
    logger.setLevel(logging.DEBUG)
    
    # 控制台 Handler (INFO)
    ch = logging.StreamHandler()
    ch.setLevel(logging.INFO)
    ch.setFormatter(logging.Formatter(LOG_FORMAT))
    
    # 文件 Handler (DEBUG, 滚动)
    fh = RotatingFileHandler(LOG_DIR / "app.log", maxBytes=10*1024*1024, backupCount=3, encoding='utf-8')
    fh.setLevel(logging.DEBUG)
    fh.setFormatter(logging.Formatter(LOG_FORMAT))
    
    logger.addHandler(ch)
    logger.addHandler(fh)
    return logger
if __name__ == "__main__":
    log = get_logger("test")
    log.debug("Debug 测试")
    log.info("Info 测试")
```
#### 2.4 constants.py（必选）
**职责**：
- 集中管理所有魔法数字、固定字符串、枚举类（枚举用`enum.Enum`实现），禁止业务代码中散落固定值；
- 每个常量添加注释说明含义，命名使用**大写下划线**，无意义缩写；
- 生成完所有文件后，回头补全所有业务占位常量，替换项目中所有硬编码魔法值
- **常量分类**：分**通用基础常量**（无业务依赖，如`DEFAULT_ENCODING = "utf-8"`、`PYTHON_MIN_VERSION = (3,8)`、`LOG_LEVEL = "INFO"`）、**业务占位常量**（按业务需求预留名称，值设为None，后续补全）；

**完整模板**：
```python
# -*- coding: utf-8 -*-
from enum import Enum
# 基础配置
DEFAULT_ENCODING = "utf-8"
PYTHON_MIN_VERSION = (3, 8)
# 运行模式枚举
class RunMode(Enum):
    SINGLE = "single"
    BATCH = "batch"
    EXPERIMENT = "experiment"
```
#### 2.5 args.py（必选）
**职责**：
- 集中定义所有命令行参数（使用`argparse`），每个参数必须添加`help`说明、指定`type`类型、设置`default`默认值；
- 实现参数校验逻辑（如可选值范围、路径合法性、参数非空校验），校验失败则抛出明确异常；
- 提供`parse_args()`函数，供`main.py`和`scripts/`所有脚本调用，禁止重复定义参数；
- 参数默认值、可选范围等引用`config.constants.py`的常量，禁止硬编码。
**完整模板**：
```python
# -*- coding: utf-8 -*-
import argparse
from config.constants import RunMode
def parse_args():
    parser = argparse.ArgumentParser(description="项目统一入口")
    parser.add_argument(
        "--mode", type=str, default=RunMode.SINGLE.value,
        choices=[m.value for m in RunMode],
        help="运行模式: single(单样本), batch(批量), experiment(实验)"
    )
    parser.add_argument("--input", type=str, help="输入数据路径")
    parser.add_argument("--config", type=str, help="实验配置文件路径")
    return parser.parse_args()
```
---
### 3. core/ 核心业务目录（必选，核心）
#### 3.1 拆分原则
- **优先聚合**：相关联的类/函数优先放在同一个文件（按“业务域”聚合），尽量减少core的子文件间的互相调用，例如 `StateManager`、`MemoryManager` 统一放在 `infer_manager.py`。
- **行数阈值放宽**：单个文件行数 ≤800 行即可（核心是逻辑内聚）。
- **禁止过度拆分**：禁止将一个完整业务流程拆分为多个细碎文件。
- **禁止过度聚合**：禁止将无关业务塞入同一文件
- **通用逻辑剥离**：仅将与业务无关的通用逻辑移至 `utils/`。
- 仅通过绝对导入调用`config/`的配置能力和`utils/`的通用工具，禁止重复编写通用逻辑，禁止反向依赖根目录/脚本文件。
- 类/函数必须写标准docstring，包含功能描述、参数说明、返回值说明、异常说明，命名清晰（类名大驼峰、函数/变量名小写下划线），禁止滥用缩写；
- 全量类型注解（函数参数、返回值、类属性），严格遵循PEP8规范（4空格缩进、行宽≤120字符、语义化命名）；
#### 3.2 异常安全
- **强制捕获**：所有 IO 操作、外部调用必须加 `try-except`。
- **降级处理**：异常场景需做降级处理（如解析失败时使用兜底数据），**不中断主流程**。
- **日志记录**：异常捕获后必须记录完整 traceback（`logger.error(exc_info=True)`）。
#### 3.3 独立调试
- 每个文件末尾加 `if __name__ == "__main__":`，编写独立测试代码，使用 `data/test/` 下的数据验证。
#### 3.4 [业务域].py 模板思路
```python
# -*- coding: utf-8 -*-
from utils.file import read_json
from config.logger import get_logger
logger = get_logger(__name__)
# 内部辅助类（不拆分）
class _DataParser:
    pass
def process_single(input_path: str):
    """供 main.py 调用的接口"""
    logger.info(f"Processing {input_path}...")
    # 业务逻辑...
if __name__ == "__main__":
    # 独立调试
    process_single("data/test/sample.json")
```
---
### 4. utils/ 通用工具目录（必选）
#### 4.1 边界要求
- 里面为需要多次服用的工具，大部分为通用工具直接复用至其他项目，少部分为项目特有工具。
- 所有文件无任何业务逻辑，仅封装通用工具能力；
- 单个工具函数只做一件事（如`read_json()`仅读取JSON文件，不包含数据处理），全量类型注解+标准docstring；
- 依赖config/其他文件，禁止依赖core/等上层，避免循环导入；
- 每个文件末尾加`if __name__ == "__main__":`，支持独立运行测试，异常处理复用自身模块能力。
#### 4.2 file.py（必选）
**职责**：
- 提供通用文件操作函数：`read_json()`、`write_json()`、`read_txt()`、`write_txt()`，支持`pathlib.Path`类型参数；
- 所有文件读写均指定`config.constants.py`的`DEFAULT_ENCODING`，禁止硬编码编码格式；
- 为所有函数添加异常处理（文件不存在、权限不足、JSON解析失败等），抛出明确的自定义异常；
- 独立运行时可测试文件读写功能，创建临时文件并验证读写结果，不修改项目实际数据。
**完整模板**：
```python
# -*- coding: utf-8 -*-
import json
from pathlib import Path
from typing import Any
from config.constants import DEFAULT_ENCODING
def read_json(path: Path | str) -> Any:
    if not Path(path).exists():
        raise FileNotFoundError(f"文件不存在: {path}")
    with open(path, 'r', encoding=DEFAULT_ENCODING) as f:
        return json.load(f)
def write_json(path: Path | str, data: Any):
    Path(path).parent.mkdir(parents=True, exist_ok=True)
    with open(path, 'w', encoding=DEFAULT_ENCODING) as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
```
#### 4.3 env.py
**职责**：
- 实现核心环境校验：Python版本检查（对比`config.constants.py`的`PYTHON_MIN_VERSION`）、虚拟环境激活检查、操作系统类型识别（返回标准化标识）、核心依赖版本校验；
- 版本/依赖不满足时，输出错误日志并退出程序；虚拟环境未激活时，给出明确提示（不强制退出）；
- 提供`get_env_info()`函数，返回当前环境信息（Python版本、系统、虚拟环境状态、依赖版本）；
- 独立运行时可直接输出当前环境信息，方便问题排查。
**完整模板**：
```python
# -*- coding: utf-8 -*-
import sys
import logging
from config.constants import PYTHON_MIN_VERSION
logger = logging.getLogger(__name__)
def check_env():
    """检查 Python 版本与虚拟环境"""
    if sys.version_info < PYTHON_MIN_VERSION:
        logger.error(f"Python 版本过低，要求 >= {PYTHON_MIN_VERSION}")
        sys.exit(1)
    
    in_venv = hasattr(sys, 'real_prefix') or (hasattr(sys, 'base_prefix') and sys.base_prefix != sys.prefix)
    if not in_venv:
        logger.warning("警告：未在虚拟环境中运行，建议激活 venv")
```
#### 4.4 exception.py（必选）
**职责**：
- 定义通用异常装饰器（如`catch_exception`），自动捕获函数异常、记录完整traceback、返回兜底值（可选）；
- 定义项目通用自定义异常类（均继承`Exception`），添加清晰的错误信息，如`FileReadError`、`ParamValidateError`；
- 提供降级处理函数（如`fallback_value()`），异常场景下返回指定兜底值，保证主流程不中断；
- 所有装饰器/函数无项目内依赖，仅使用Python内置库，可直接跨项目复用
**完整模板**：
```python
# -*- coding: utf-8 -*-
import functools
import logging
def catch_exception(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logging.error(f"函数 {func.__name__} 执行出错", exc_info=True)
            raise
    return wrapper
```
---

### 5. tests/ 单元测试目录（必选）
#### 5.1 规范要求
- 遵循pytest风格，测试文件命名为`test_*.py`，测试函数命名为`test_*`，按`core/`/`utils/`的目录结构对应拆分测试用例；
- 仅验证核心代码正确性，不修改`data/`和`outputs/`的实际数据，测试用例不依赖外部服务/网络；
- 所有测试通过绝对导入调用待测试模块，异常处理、日志输出复用项目内的`utils/`/`config/`模块；
- 测试覆盖核心逻辑、边界条件、异常场景（如空数据、非法参数、文件不存在），断言清晰，失败时输出明确信息。
- **测试隔离**：禁止在测试代码中修改 `data/` 和 `outputs/` 的核心数据。
#### 5.2 执行要求
- 可通过 `pytest tests/ -v` 一键运行。
- 测试用例不依赖外部服务（使用本地测试数据）。
---
### 6. data/ 输入数据目录（必选）
#### 6.1 通用规则
- **只读限制**：所有数据文件只读，业务代码禁止修改 `data/` 内文件。
- **路径管理**：路径统一由`config/path.py`管理，禁止硬编码数据路径
- **大文件处理**：大文件（>100MB）通过 `.gitignore` 忽略，仅保留数据说明文档。
- 仅存储项目**输入数据**，禁止存储输出数据/中间结果（输出统一放`outputs/`）；
- 数据文件命名规范：含时间戳/用途（如`20250320_raw_data.json`），避免乱码，使用英文/数字/下划线。
#### 6.2 子目录要求（可选）
- `raw/`：存储原始未清洗数据，命名含时间戳（如 `202403_raw.json`），配套 README。
- `test/`：存储轻量测试数据（≤10条），供单样本调试用。
- `batch/`：存储批量推理数据，按批次命名。
---
### 7. outputs/ 输出目录（必选）
#### 7.1 通用规则
- 所有输出统一放入此目录，由 `config/path.py` 自动创建。
- 无本地路径依赖，支持 Linux 服务器部署。
#### 7.2 子目录要求
- `logs/`（必选）：存储日志文件（滚动存储）。
- `results/`（可选）：按业务场景分类（如 `single/`、`batch/`），结果文件命名含时间戳。
---
## 四、全局通用规则
### 1. 模块化与解耦
- **禁止循环导入**：`core/a.py` 不得导入 `core/b.py`，同时 `core/b.py` 导入 `core/a.py`。
- **接口隔离**：模块间通过函数接口调用，禁止直接修改其他模块的类属性或变量。
### 2. 类型与命名
- **全量 Type Hint**：所有函数参数、返回值必须添加类型注解。
- **命名规范**：变量 `snake_case`，类名 `PascalCase`，常量 `UPPER_SNAKE_CASE`。
- **语义化**：禁止无意义缩写（如 `sm` -> `state_manager`）。
### 3. 可部署性
- **跨平台路径**：所有路径必须使用 `pathlib.Path`，禁止硬编码分隔符。
- **日志规范**：生产环境仅输出 INFO 及以上级别，后台运行（nohup）兼容正常。
### 4. 生成顺序
- 按模块组优先级生成，避免循环依赖：
- config/constants.py → config/path.py
- utils/exception.py
- config/logger.py → config/args.py
- utils/file.py → utils/env.py
- core/[业务主模块].py
- main.py 及 tests/
- 遵循「**底层→中层→上层→入口层**」的单向依赖体系，禁止跨层/反向依赖，禁止循环导入；
  - 底层：`config/constants.py`、`config/path.py`、`utils/exception.py`（仅依赖内置库）；
  - 中层：`config/logger.py`、`config/args.py`、`utils/file.py`、`utils/env.py`（仅依赖底层）；
  - 上层：`core/[业务主模块].py`（仅依赖底层+中层）；
  - 入口层：`main.py`、`tests/*`（依赖底层+中层+上层）；
- 模块间通过接口调用，禁止直接修改其他模块的变量，每个模块仅暴露必要的接口，禁止使用`import *`批量导入。
graph TD
    A[底层: config/constants.py, config/path.py] --> B[中层: config/logger.py, config/args.py]
    A --> C[中层: utils/file.py, utils/env.py]
    C --> D[上层: core/业务域.py]
    B --> D
    D --> E[入口层: main.py, tests/]
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style C fill:#ccf,stroke:#333,stroke-width:2px
    style D fill:#9ff,stroke:#333,stroke-width:2px
### 5. 自检清单
- [ ] 目录中是否无 __init__.py？
- [ ] 所有路径是否由 path.py 管理，无硬编码？
- [ ] 业务代码中是否无魔法数字？
- [ ] 是否能在根目录通过 `python main.p





