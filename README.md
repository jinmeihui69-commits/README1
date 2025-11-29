# README1

# GitHub 项目健康度分析平台

## 1. 项目概述
GitHub 项目健康度分析平台是一个基于 Web Service API 的数据分析平台，通过自动化采集、计算和可视化 GitHub 项目的关键技术指标，为开源项目的健康状况提供量化评估和深度洞察。

### 核心目标
- **自动化**: 实现从数据采集到可视化报告的全流程自动化
- **标准化**: 建立统一的指标定义和计算标准
- **可扩展**: 支持指标、数据源和可视化组件的灵活扩展
- **用户友好**: 提供直观的仪表盘和交互式可视化

### 核心技术指标
#### 代码提交活跃度指数 (CAI)
- **定义**: 在给定时间窗内（如最近12周），综合衡量仓库代码提交的强度、稳定性与贡献者多样性的复合指数，范围0-100。

#### 平均问题解决时长 (MTTR-I)
- **定义**: 从 Issue 创建到关闭的平均耗时，反映社区或团队的响应与交付效率。
- **特点**: 使用 IQR 统计方法剔除异常值，同时计算 P50、P90 分位数，支持时间窗口筛选。

#### 外部依赖关系深度 (EDD)
- **定义**: 以项目为根节点，在其依赖关系图中，到任一外部第三方包的最长路径层级，衡量依赖链的纵深与潜在脆弱性。

## 2. 技术栈概述

### 后端框架
- **FastAPI**: 高性能 Python Web 框架，支持异步处理
- **Python 3.x**: 主要编程语言

### 数据存储
- **PostgreSQL**: 关系型数据库，存储项目元数据
- **InfluxDB**: 时序数据库，存储指标数据
- **Redis**: 内存数据库，用于缓存和会话管理
- **S3 兼容对象存储**: 存储原始 JSON 数据

### 部署与运维
- **Docker**: 容器化部署
- **Swagger UI/OpenAPI**: 自动 API 文档生成

## 3. 安装与运行指南

### 3.1 环境要求
- **操作系统**：Linux / macOS / Windows
- **Python**：3.10 及以上（建议）
- **Git**：用于拉取仓库

### 3.2 克隆仓库
```bash
git clone <your-repo-url> github-health-metrics
cd github-health-metrics
### 3.3 创建并激活虚拟环境

Linux / macOS：
python3 -m venv .venv
source .venv/bin/activate
Windows:
python -m venv .venv
.\.venv\Scripts\Activate.ps1
3.4安装依赖
如果仓库中已有 requirements.txt:
pip install -r requirements.txt
若尚未生成，可手动安装当前模块所需依赖：
pip install "fastapi" "uvicorn[standard]" httpx numpy pytest
3.5使用 uvicorn 启动 FastAPI 应用：
uvicorn api:app --host 0.0.0.0 --port 8000 --reload
启动成功后，可以访问：
服务健康检查：http://127.0.0.1:8000/
GitHub API 探针接口：http://127.0.0.1:8000/check_github_api/
3.6 运行单元测试
pytest -vv test_github.py
pytest -vv test_core_algorithms.py
##4.命令行工具与参数参考
本项目主要通过以下命令行工具进行运行和测试：uvicorn（启动服务）与 pytest（执行测试）。
4.1 启动 FastAPI 服务：uvicorn
基本使用形式：
uvicorn api:app [OPTIONS]
| 参数          | 说明              |       示例               | 默认值  |
| -------------| ----------------  | -------------------  |-----------|
|  --host      | 监听的主机地址      |--host 0.0.0.0         | 127.0.0.1 |
|  --port      | 监听端口           | --port 8000           | 8000      |
|  --reload    | 代码变更自动重载     | --reload              | 关闭       |
|  --workers   | 启动的工作进程数    | --workers 4           |     1      |
|  --log-level | 日志级别           | --log-level debug     | info      |
如：uvicorn api:app--reload 
4.2 运行测试用例：pytest
基本使用形式：
pytest [OPTIONS] [PATHS...]
| 参数           | 说明                 | 示例                            |
| ------------ | ------------------- | -------------------------------  |
| -q           | 安静模式，仅输出结果    | pytest -q                       |
| -v / -vv     | 输出更详细的测试信息    | pytest -vv                      |
| -k <expr>    | 只运行名称匹配表达式的测试 | pytest -k "probe and success" |
| -x           | 遇到第一个失败即停止    | pytest -x                       |
如：pytest -vv -k "probe"
##5.HTTP API 接口说明
连接探针模块的行为与接口设计在《连接探针模块》文档中有详细说明。
连接探针模块
5.1 健康检查接口
URL：GET /
用途：检查探针服务本身是否正常启动。
响应示例（HTTP 200）：
{
  "message": "GitHub Connectivity Probe is running."
}
5.2 GitHub API 探针接口
URL：GET /check_github_api/
用途：检查 GitHub 公共 API（/events）的连通性和基本可用性。
5.2.1 成功响应
HTTP 状态码：200 OK
响应体示例：
{
  "status": "success",
  "message": "GitHub API 连接正常。",
  "data": {
    "id": "1234567890",
    "type": "PushEvent",
    "repo": "owner/repo-name"
  }
}
说明：
status：固定为 "success"。
message：对当前探针结果的自然语言描述。
data：从 GitHub /events 返回数据中抽取的 最小数据集：
事件 id
事件 type
仓库名 repo（如 "owner/repo"）
5.2.2 失败响应
HTTP 状态码：503 Service Unavailable
响应体结构：
{
  "detail": {
    "status": "failure",
    "error_type": "<ERROR_CODE>",
    "message": "<错误描述>",
    "extra": { "...": "..." }   // 某些错误类型可能包含
  }
}
##6.核心算法函数
核心指标基于系统设计规格说明书中对 CAI、MTTR-I、EDD 的定义实现。
6.1 calculate_cai(commit_data, observation_weeks=12)
作用：计算 代码提交活跃度指数（CAI），范围大致在 0–100 之间。
参数：
commit_data: Sequence[int]
每周提交次数序列，例如过去 52 周的每周提交数。
observation_weeks: int = 12
计算时考虑的观察窗口长度（周）。
行为：
若 len(commit_data) < observation_weeks，抛出 ValueError。
对最近 N 周数据计算：
F：标准化提交频次（以 50 次/周作为参考强度）。
A：观察窗口内有提交的周占比。
D：基于不同提交量数值的“多样性”近似贡献者多样性。
按一定权重组合三者得到 CAI。
返回值：
float：CAI 指数。
6.2 calculate_mttr_issues(issues_data)
作用：计算 平均问题解决时长 MTTR-I（单位：小时）。
参数：
issues_data: Sequence[dict]
每条 Issue 需包含以下键：
"created_at"：创建时间，ISO 格式字符串，如 "2023-01-01T10:00:00Z"。
"closed_at"：关闭时间，ISO 格式字符串。
行为：
若 issues_data 为空，抛出 ValueError。
对每个 Issue 计算 closed_at - created_at 的时长（小时）。
使用 IQR（四分位距） 方法去除离群值（小于 Q1 - 1.5*IQR 或大于 Q3 + 1.5*IQR 的样本）。
计算过滤后样本的平均值（MTTR）以及 P50、P90 分位数，并在控制台打印。
返回 MTTR 数值。
返回值：
float：平均问题解决时长（小时）。
6.3 calculate_edd(dependency_graph)
作用：计算 外部依赖关系深度 EDD，即从项目根到任一外部依赖的最长路径层级，用以衡量依赖链纵深与潜在脆弱性。
项目技术栈确定与《系统设计规格说明书》
参数：
dependency_graph: Sequence[dict]
行为：
遍历依赖图中所有节点。
对 type == "external" 的节点调用 calculate_depth(node) 计算层级。
返回所有外部节点中的最大层级值。
返回值：
int：外部依赖关系深度（层级数）。
7. 测试与质量保证
7.1 探针模块测试
test_github.py 中的测试用例覆盖：
探针函数 run_github_probe()：
test_probe_success：模拟 GitHub 返回 200 + 有效事件列表。
test_probe_timeout：模拟网络超时，断言返回 error_type == "TIMEOUT"。
test_probe_rate_limit_403：模拟 403 限速或禁止访问。
test_probe_invalid_url：模拟 URL 配置错误。
连接探针模块
HTTP 接口：
test_http_endpoint_success：成功时返回 200 且响应 status == "success"。
test_http_endpoint_failure：探针失败时返回 503，并且 detail 中携带错误信息。
7.2 指标算法测试
test_core_algorithms.py 中的测试用例覆盖：
CAI：高活跃度与低活跃度提交数据，断言计算结果显著不同。
MTTR-I：普通数据与包含异常值的数据，验证 IQR 去噪后平均时间在合理范围内。
EDD：简单依赖图的最大深度计算。
通过 pytest -vv 可以一次性运行全部测试。













