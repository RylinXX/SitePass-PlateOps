# 消纳通 SitePass-PlateOps · 智慧工地消纳与车牌核验平台

**SitePass-PlateOps** · 智慧工地消纳场资质管理、车牌核验、运单查询与场所消纳额度监控中台。

[![Website](https://img.shields.io/badge/website-web.etgq.com-0a66c2?logo=googlechrome&logoColor=white)](https://web.etgq.com/)
[![Python](https://img.shields.io/badge/python-3.11%2B-3776AB?logo=python&logoColor=white)](backend/requirements.txt)
[![FastAPI](https://img.shields.io/badge/backend-FastAPI-009688?logo=fastapi&logoColor=white)](backend/app/main.py)
[![Frontend](https://img.shields.io/badge/frontend-Vanilla%20HTML%2FJS%2FCSS-F7DF1E?logo=javascript&logoColor=black)](frontend/index.html)
[![OCR](https://img.shields.io/badge/OCR-Volcengine%20Visual-1664FF)](backend/app/main.py)
[![Tests](https://img.shields.io/badge/tests-pytest-0A9EDC?logo=pytest&logoColor=white)](backend/tests)
[![GitHub](https://img.shields.io/badge/GitHub-RylinXX%2FSitePass--PlateOps-181717?logo=github&logoColor=white)](https://github.com/RylinXX/SitePass-PlateOps)

线上入口：<https://web.etgq.com/>

## 项目简介

消纳通（SitePass）面向工程运输与渣土消纳场景，提供统一的消纳场车辆进出核验、运单实时查询、动态过期场所自动过滤与消纳方量矩阵统计大盘。系统由 **FastAPI 后端代理服务**与**原生 HTML/JavaScript 前端**组成：后端负责来源平台登录辅助、数据同步、车牌提取与本地备案库、运单与消纳额度代理、运单 OCR 备案；前端提供公开查询终端与完整的车辆备案管理后台，由 FastAPI 直接托管静态资源，单进程即可运行。

查询与统计结果依赖来源平台的账号权限、数据更新时间及接口状态，不构成独立的行政准入证明。

## 核心能力

| 模块 | 说明 |
| --- | --- |
| 车辆备案查询 | 按车牌号或企业名称检索本地已备案车辆，命中即显示准入结论，支持结果表格 PDF 导出 |
| 一键数据同步 | 通过来源平台登录辅助（验证码 / Token）拉取备案数据，递归遍历嵌套 JSON 提取车牌与车辆信息，写入本地备案库 |
| 车牌识别与校验 | 覆盖传统 7 位、新能源 8 位及挂 / 学 / 警 / 港 / 澳 / 变 / 超等特种车牌校验，兼容多字段与裸字符串数组 |
| 运单实时查询 | 后台按时间、企业、车牌等条件查询运单，支持表格 / 卡片视图切换与高级筛选 |
| 消纳额度矩阵 | 维护场所容量与月度方量，计算已消纳 / 剩余额度，过滤过期场所，生成年度方量矩阵统计 |
| 运单 OCR 备案 | 上传运单图片，经火山引擎视觉 OCR 或方舟（Ark）多模态模型识别车牌与企业，比对备案库后批量写入待备案列表 |
| 公开查询 API | `/api/public/vehicle-query` 供第三方系统按车牌查询备案信息，可在后台开启 / 关闭并配置独立 API Key（支持 Header 与 Query 传递） |
| 管理后台 | 二维码助手、运单查询、消纳额度、OCR 备案、接口配置与公开 API 配置，支持主题切换与 Token 凭证管理 |

## 技术栈

| 分层 | 技术 |
| --- | --- |
| 后端 | Python 3.11+、FastAPI、Uvicorn、httpx（异步代理请求）、pydantic |
| 数据与状态 | JSON 文件持久化（`data.json` 备案库、`config.json` 配置、`pending_filing.json` 待备案、`qr_data.json`），本地缓存 |
| OCR / AI | 火山引擎 `volcengine` SDK 视觉 OCR；兼容方舟（Ark）多模态模型接口 |
| 前端 | 原生 HTML5 + CSS3 + JavaScript（无构建工具），FastAPI `StaticFiles` 直接托管 |
| 测试 | pytest（登录辅助 Token 提取、车牌嗅探 / 递归提取） |
| 部署 | Nginx 反向代理 + systemd / 进程守护；后端监听 8001 端口；Windows 提供一键启动与计划任务脚本 |
| 辅助脚本 | `start.bat`（Windows 一键安装依赖并启动）、`setup_schedule.ps1`（注册每日 06:00 数据同步计划任务） |

## 目录结构

```text
SitePass-PlateOps/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI 应用：登录辅助、同步、查询、运单、消纳额度、OCR、后台配置与静态托管
│   │   └── sync.py              # 命令行同步脚本（供 Windows 计划任务每日执行）
│   ├── tests/                   # pytest：登录辅助与车牌提取测试
│   ├── config.json.example      # 来源平台凭据配置模板（复制为 config.json 后填写）
│   ├── site_capacities.json     # 场所容量配置快照（示例数据）
│   ├── absorptive_matrix_cache.json  # 消纳方量矩阵缓存快照（示例数据）
│   └── requirements.txt
├── frontend/
│   ├── index.html               # 公开查询终端（登录引导 + 车辆备案查询）
│   ├── admin.html               # 车辆备案管理后台
│   ├── app.js / app-runtime.js  # 查询端逻辑
│   ├── admin-runtime.js         # 后台逻辑
│   ├── style.css                # 主题样式
│   ├── assets/logo.png          # 品牌标识
│   └── logo.png
├── start.bat                    # Windows 一键启动脚本
├── setup_schedule.ps1           # Windows 每日同步计划任务注册脚本
├── SERVER-SYNC.md               # 生产源码核对说明
└── .gitignore                   # 排除配置、运行数据、日志与缓存
```

## 快速开始

需要 Python 3.11+（本地开发推荐 3.11 / 3.12）。

```sh
cd backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python -m uvicorn app.main:app --host 127.0.0.1 --port 8001
```

访问 `http://127.0.0.1:8001/` 进入公开查询终端；后台管理页为 `http://127.0.0.1:8001/admin`。首次使用需在后台「接口配置」中填写来源平台授权信息并执行一键同步。

配置模板：

```sh
cd backend
cp config.json.example config.json
# 编辑 config.json，填写来源平台 authtoken / id / worksitetype
# 如需 OCR，可在后台配置火山引擎 AK/SK 或 Ark API Key
```

Windows 用户可直接运行 `start.bat`：脚本会检测 Python、创建 `backend/venv`、安装依赖、以最小化窗口启动 8001 端口服务并打开浏览器。请先审阅脚本内容再执行。

### 测试

```sh
cd backend
python -m pytest tests
```

### 每日自动同步（可选）

```powershell
# 在 project 根目录以 PowerShell 运行，注册每日 06:00 执行 backend/app/sync.py 的计划任务
.\setup_schedule.ps1
```

任务名为 `PlateShieldDailySync`，日志输出到 `backend/sync.log`。

## 配置说明

| 配置 / 文件 | 说明 |
| --- | --- |
| `backend/config.json` | 运行时配置（**不提交**，模板见 `config.json.example`）：来源平台 `authtoken`、`id`、`worksitetype`，以及 OCR 凭据、公开查询开关与 API Key |
| `backend/data.json` | 同步生成的本地备案库（**不提交**） |
| `backend/pending_filing.json` | OCR / 人工录入的待备案车辆（**不提交**） |
| `backend/qr_data.json` | 二维码助手配置（**不提交**） |
| `backend/site_capacities.json` | 场所容量基线配置（可用于初始化，生产以线上数据为准） |
| `backend/absorptive_matrix_cache.json` | 消纳矩阵缓存快照，仅作示例，不代表当前线上数据 |

安全要求：

- 来源平台 `authtoken`、火山引擎 AK/SK / Ark API Key 等凭据只保存在服务器私有 `config.json` 或部署环境变量中，禁止写入源码、README 或提交到 Git。火山引擎凭据的环境变量为 `VOLC_ARK_API_KEY` / `VOLC_ARK_SECRET_KEY`，二者也可只设 Ark Key；后台「接口配置」不再回显密钥明文，只提示是否已配置，留空保存表示保持原值。
- 公开查询接口开启时建议设置 `public_api_key`，并通过 Header（`X-API-Key`）或 Query（`apikey`）传递；如无需对外提供，请在后台关闭公开查询。
- 登录、验证码与来源授权流程由用户本人完成，系统仅做登录辅助，禁止绕过来源平台限制。

## 部署

线上入口为 `web.etgq.com`，FastAPI 后端监听 `8001` 端口，前端静态页面由后端直接托管。

```sh
# 生产方式启动示例（在项目根目录）
cd backend
source .venv/bin/activate
python -m uvicorn app.main:app --host 127.0.0.1 --port 8001
```

Nginx 需把页面、查询、同步与静态资源路由一起转发到后端；不要与 `ylt.etgq.com` 的运输账目接口混用。更新代码前先备份 `backend/config.json`、`backend/data.json` 与运行缓存；发布时不得用仓库里的历史缓存 JSON 覆盖最新线上数据。

## 注意事项与边界

- 结果依赖来源平台权限、数据更新时间与接口状态，不构成独立的行政准入证明。
- 同步配置、管理写入与公开查询应区分访问权限；配置类接口与后台入口不建议直接暴露到公网。
- 仓库内既有 JSON 缓存/容量文件是历史快照，不是当前线上数据完整性的承诺；数据库、日志、运行配置与真实凭据均不纳入 Git。
- 代码归档与仓库改名不会更改线上域名、数据与访问权限，详见 [SERVER-SYNC.md](SERVER-SYNC.md)。
- 当前仓库未附带独立开源 LICENSE 文件，代码与品牌内容版权归项目所有者。

## 相关文档

- [生产源码核对说明](SERVER-SYNC.md)
- 部署环境：`web.etgq.com`（线上查询入口）
