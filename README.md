# Scrapling

> 自适应、防检测、高性能的 **Python 网页抓取框架**：从单次请求到大规模爬虫，一套库全部搞定。

[![Tests](https://github.com/D4Vinci/Scrapling/actions/workflows/tests.yml/badge.svg)](https://github.com/D4Vinci/Scrapling/actions/workflows/tests.yml)
[![PyPI version](https://badge.fury.io/py/Scrapling.svg)](https://badge.fury.io/py/Scrapling)
[![Python](https://img.shields.io/pypi/pyversions/scrapling.svg)](https://www.python.org)
[![License](https://img.shields.io/badge/license-BSD-blue.svg)](LICENSE)
[![Docs](https://img.shields.io/badge/docs-scrapling.readthedocs.io-3050ff.svg)](https://scrapling.readthedocs.io)

> 本仓库为上游 [D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling) 的派生仓库（fork），版本 `0.4.11`。本文档在保留上游全部能力说明的基础上，按“技术栈 / 目录逐文件作用 / 业务逻辑 / 特性优点”的结构重新整理。英文原版见 [README.en.md](README.en.md)。

---

## 目录

- [1. 项目简介](#1-项目简介)
- [2. 核心特性与优点](#2-核心特性与优点)
- [3. 技术栈](#3-技术栈)
- [4. 系统架构与业务逻辑](#4-系统架构与业务逻辑)
  - [4.1 三大支柱](#41-三大支柱)
  - [4.2 一次抓取请求的完整链路](#42-一次抓取请求的完整链路)
  - [4.3 核心模块职责](#43-核心模块职责)
- [5. 目录结构与各文件作用](#5-目录结构与各文件作用)
- [6. 自适应解析器（parser）](#6-自适应解析器parser)
- [7. 抓取器（fetchers）](#7-抓取器fetchers)
- [8. 爬虫框架（spiders）](#8-爬虫框架spiders)
- [9. AI / MCP 与 CLI](#9-ai--mcp-与-cli)
- [10. 安装与配置](#10-安装与配置)
- [11. 许可证](#11-许可证)

---

## 1. 项目简介

**Scrapling** 是一个现代化的网页抓取（Web Scraping）框架，由资深爬虫开发者 Karim Shoair 打造，定位是“让网页抓取像它本该有的那样轻松”。它把三类能力收敛进**一个库**，对开发者零妥协：

1. **自适应解析器（Adaptive Parser）**：解析器会“学习”网页结构，当目标网站改版、元素位置变动时，自动重新定位你之前选中的元素——你的抓取脚本无需跟着改。
2. **防检测抓取器（Undetectable Fetchers）**：内置基于 `curl_cffi` 的指纹伪装请求器，以及基于 Chromium 的隐身（`StealthyFetcher`）/ 全功能（`DynamicFetcher`）浏览器抓取器，开箱绕过 Cloudflare Turnstile 等反爬机制。
3. **可扩展爬虫框架（Spiders）**：提供类 Scrapy 的 Spider 框架，支持并发、多会话、暂停/恢复（checkpoint）、自动代理轮换，并带实时统计与流式输出。

无论是“抓一个页面里的几段数据”，还是“爬取整站并做分布式调度”，Scrapling 都能用几行 Python 完成。

---

## 2. 核心特性与优点

| 维度 | 说明 |
| --- | --- |
| **自适应解析** | `Selector.css(..., adaptive=True)` 在网站改版后仍能找回目标元素；`auto_save=True` 自动把元素指纹存入本地库，下次改版自动重定位。 |
| **防检测抓取** | `Fetcher` 用 `curl_cffi` 模拟真实浏览器 TLS/HTTP2 指纹；`StealthyFetcher` 基于 Chromium + `browserforge` 生成真实指纹，可解 Cloudflare Turnstile/Interstitial。 |
| **多模式抓取器** | 同步/异步 HTTP 抓取（`Fetcher`/`AsyncFetcher`）、隐身浏览器（`StealthyFetcher`）、可编程浏览器（`DynamicFetcher`），按需取舍速度与隐身。 |
| **代理轮换** | `ProxyRotator` 线程安全，支持循环/自定义轮换策略，自动识别代理错误并切换；`StealthyFetcher` 还支持 DoH 防 DNS 泄漏。 |
| **爬虫框架** | `Spider` 支持并发请求、每域名限流、下载延迟、暂停/恢复（checkpoint）、robots.txt 遵守、实时统计与流式结果。 |
| **AI / MCP 支持** | 内置 MCP Server（`scrapling[ai]`），让 LLM 智能体直接调用抓取能力；并提供 `agent-skill/` 供 AI 编程助手使用。 |
| **CLI 工具** | `scrapling` 命令行：一行命令抓取网页并转为 HTML/Markdown/纯文本，或交互式 shell。 |
| **生态集成** | 提供与 Scrapy 的集成（`integrations/scrapy.py`）、多种 Spider 模板（Crawl/Sitemap/Shopify）。 |
| **类型完备** | 全量类型注解并附带 `py.typed`，兼容 mypy / pyright。 |
| **许可宽松** | 采用 **BSD 许可证**，商用友好。 |

---

## 3. 技术栈

### 核心依赖（必装）

| 组件 | 用途 |
| --- | --- |
| **lxml (≥6.1.1)** | 高性能 HTML/XML 解析，解析器底层。 |
| **cssselect** | CSS 选择器 → XPath 转换。 |
| **orjson** | 极速 JSON 序列化（存储系统/CLI 用）。 |
| **tld** | 从 URL 提取注册域名，用于自适应存储按站点隔离。 |
| **w3lib** | URL 规范化等 Web 工具函数。 |
| **typing_extensions** | 类型注解扩展。 |

### 可选依赖（按需安装）

| 组件 | 用途 | 对应 extra |
| --- | --- | --- |
| **curl_cffi** | 带浏览器指纹伪装的 HTTP 客户端（Fetcher 底层）。 | `fetchers` |
| **playwright / patchright** | 驱动 Chromium 的浏览器自动化（Dynamic/Stealthy 底层）。 | `fetchers` |
| **browserforge** | 生成真实浏览器指纹（UA、屏幕、WebGL 等）。 | `fetchers` |
| **apify-fingerprint-datapoints** | 指纹数据点增强。 | `fetchers` |
| **msgspec** | 高性能结构化序列化。 | `fetchers` |
| **anyio** | 异步并发原语（Spider 框架用）。 | `fetchers` |
| **protego** | robots.txt 解析。 | `fetchers` |
| **click** | CLI 框架。 | `fetchers` |
| **mcp** | Model Context Protocol Server（AI 集成）。 | `ai` |
| **markdownify** | HTML → Markdown 转换（AI/Shell 用）。 | `ai` / `shell` |
| **IPython** | 交互式抓取 shell。 | `shell` |

### 基础设施与工程

| 组件 | 用途 |
| --- | --- |
| **SQLite** | 自适应解析的元素指纹存储（`elements_storage.db`）。 |
| **pytest / tox** | 测试与多环境验证。 |
| **ruff / bandit / pre-commit** | Lint、安全扫描、提交钩子。 |
| **Docker** | 容器化运行（见 `Dockerfile`）。 |
| **MkDocs / ReadTheDocs** | 多语言文档站点（见 `docs/`）。 |

---

## 4. 系统架构与业务逻辑

### 4.1 三大支柱

```
            ┌─────────────────────────────────────────────┐
            │               scrapling 公共 API              │
            │  Selector / Fetcher / AsyncFetcher /         │
            │  StealthyFetcher / DynamicFetcher / Spider   │
            └─────────────────────────────────────────────┘
                   │                  │                  │
        ┌──────────┴───┐   ┌──────────┴───┐   ┌─────────┴────────┐
        │  ① 解析器     │   │  ② 抓取器     │   │  ③ 爬虫框架       │
        │ parser.py    │   │ fetchers/    │   │ spiders/         │
        │ Selector     │   │ Fetcher      │   │ Spider           │
        │ (CSS/XPath/  │   │ Stealthy     │   │ CrawlerEngine    │
        │  自适应)     │   │ Dynamic      │   │ Scheduler        │
        └──────────┬───┘   └──────┬───────┘   └─────────┬────────┘
                   │              │                    │
            ┌──────┴───────┐ ┌────┴──────────┐  ┌──────┴───────────┐
            │ core/storage │ │ engines/      │  │ engines/         │
            │ (SQLite 指纹)│ │ static(curl)  │  │ _browsers(Play)  │
            │ core/translator            │  │ toolbelt(代理/指纹) │
            └──────────────┘ └─────────────┘  └────────────────────┘
```

- **解析器**负责“从 HTML 里取数据”，并借助 SQLite 存储实现自适应重定位；
- **抓取器**负责“拿到 HTML”，底层是 `curl_cffi`（HTTP 指纹）或 Playwright/Chromium（浏览器）；
- **爬虫框架**负责“把抓取器编排成大规模并发爬取”，含调度、会话、代理轮换、检查点。

### 4.2 一次抓取请求的完整链路

以 `StealthyFetcher.fetch(url, adaptive=True)` 为例：

```
用户调用 StealthyFetcher.fetch(url, ...)
      │
      ▼
scrapling/fetchers/stealth_chrome.py
   - 合并 selector_config（类级解析参数 + 调用级参数）
   - 生成真实浏览器指纹（browserforge）
      │
      ▼
scrapling/engines/_browsers/_stealth.py  → StealthySession
   - 启动 Chromium（headless/headful、CDP、真实 Chrome 等）
   - 注入指纹、设置 DoH / 代理 / 广告拦截
   - 导航页面，等待 network_idle / wait_selector
   - 可选：solve_cloudflare 解 Turnstile 挑战
      │
      ▼
scrapling/engines/toolbelt/convertor.py → ResponseFactory
   - 把浏览器响应封装为统一的 Response 对象（继承 Selector）
      │
      ▼
scrapling/parser.py → Selector
   - 用 lxml 解析 HTML，提供 css()/xpath()/text() 选择器
   - 若 adaptive=True：用 core/storage.py 的 SQLiteStorageSystem
     比对已存元素指纹，网站改版后自动重定位
      │
      ▼
返回 Response（既是 HTTP 响应，也是可查询的解析器）
   用户：response.css('.product', adaptive=True)
```

**统一响应模型**：所有抓取器返回的都是 `engines/toolbelt/custom.py` 中的 `Response` 类——它**继承自 `Selector`**，因此“响应对象”天然就是“解析器对象”，无需二次传递 HTML。

### 4.3 核心模块职责

| 模块 | 职责 |
| --- | --- |
| `scrapling/__init__.py` | 包入口，懒加载并导出公共 API（Selector / 各 Fetcher / Spider）。 |
| `scrapling/parser.py` | **自适应解析器核心**：`Selector` / `Selectors` 类，支持 CSS、XPath、文本选择，集成自适应存储。 |
| `scrapling/fetchers/` | **抓取器**：`requests.py`（curl_cffi 同步/异步）、`chrome.py`（Playwright 动态）、`stealth_chrome.py`（隐身 Chromium）、`__init__.py`（导出 + `ProxyRotator`）。 |
| `scrapling/engines/` | **底层引擎**：`static.py`（curl_cffi 客户端 + 指纹）、`_browsers/`（Playwright/Stealth 控制器与类型）、`toolbelt/`（Response/BaseFetcher、convertor、proxy_rotation、fingerprints、navigation、ad_domains）、`constants.py`。 |
| `scrapling/core/` | **核心工具**：`storage.py`（自适应 SQLite 存储）、`translator.py`（CSS→XPath）、`ai.py`（MCP Server）、`shell.py`（Convertor + 交互式 REPL）、`mixins.py`（选择器生成混入）、`custom_types.py`（AttributesHandler/TextHandler）、`_types.py`（统一类型别名）、`utils/`（日志/工具/shell 解析）。 |
| `scrapling/spiders/` | **爬虫框架**：`spider.py`（Spider 抽象基类）、`engine.py`（CrawlerEngine 调度核心）、`scheduler.py`、`session.py`（SessionManager 多会话）、`request.py`（Request）、`result.py`（CrawlResult/CrawlStats）、`cache.py`、`checkpoint.py`（暂停/恢复）、`links.py`（LinkExtractor）、`robotstxt.py`、`templates/`（CrawlSpider/SitemapSpider/ShopifySpider/CrawlRule）。 |
| `scrapling/integrations/` | 生态集成：`scrapy.py`（与 Scrapy 互通）。 |
| `scrapling/cli.py` | **命令行入口**：`scrapling` 命令（fetch / extract / shell 等子命令）。 |
| `agent-skill/` | 面向 AI 编程助手的 Scrapling 技能包（含 `Scrapling-Skill/` 与说明）。 |
| `docs/` | MkDocs 文档站点（含 `ai/`、`fetching/`、`parsing/`、`spiders/`、`cli/`、`integrations/`、`tutorials/` 及多语言 README）。 |

---

## 5. 目录结构与各文件作用

```
Scrapling/
├── scrapling/                  # ★ 核心 Python 包
│   ├── core/                   # 核心工具与底层能力
│   │   ├── utils/              #   _utils.py（日志/通用工具）、_shell.py（Header/Cookie 解析）
│   │   ├── __init__.py
│   │   ├── _types.py           #   统一类型别名（兼容 3.10+）
│   │   ├── _shell_signatures.py#   shell 命令签名
│   │   ├── ai.py               #   MCP Server 实现（AI 集成）
│   │   ├── custom_types.py     #   AttributesHandler / TextHandler（惰性属性/文本访问）
│   │   ├── mixins.py           #   SelectorsGeneration（选择器生成混入）
│   │   ├── shell.py            #   Convertor（HTML→文件）、交互式 REPL
│   │   ├── storage.py          #   SQLiteStorageSystem（自适应元素指纹存储）
│   │   └── translator.py       #   css_to_xpath 转换器
│   ├── engines/                # 抓取底层引擎
│   │   ├── _browsers/          #   Playwright/Stealth 控制器与请求类型
│   │   ├── toolbelt/           #   custom.py(Response/BaseFetcher)、convertor.py、
│   │   │                       #   proxy_rotation.py(ProxyRotator)、fingerprints.py、
│   │   │                       #   navigation.py、ad_domains.py
│   │   ├── __init__.py
│   │   ├── constants.py        #   引擎常量
│   │   └── static.py           #   curl_cffi 客户端（Fetcher 底层）
│   ├── fetchers/               # 抓取器（公共 API）
│   │   ├── __init__.py         #   懒加载导出 + ProxyRotator
│   │   ├── requests.py         #   Fetcher / AsyncFetcher（HTTP）
│   │   ├── chrome.py           #   DynamicFetcher（Playwright 全功能）
│   │   └── stealth_chrome.py   #   StealthyFetcher（隐身 Chromium）
│   ├── spiders/                # 爬虫框架
│   │   ├── templates/          #   CrawlSpider / SitemapSpider / ShopifySpider / CrawlRule
│   │   ├── __init__.py         #   公共导出（Spider/Request/CrawlResult/...）
│   │   ├── spider.py           #   Spider 抽象基类
│   │   ├── engine.py           #   CrawlerEngine（调度核心）
│   │   ├── scheduler.py        #   请求调度器
│   │   ├── session.py          #   SessionManager（多会话/代理）
│   │   ├── request.py          #   Request 对象
│   │   ├── result.py           #   CrawlResult / CrawlStats
│   │   ├── cache.py            #   请求缓存
│   │   ├── checkpoint.py       #   暂停/恢复检查点
│   │   ├── links.py            #   LinkExtractor（链接提取）
│   │   └── robotstxt.py        #   robots.txt 处理
│   ├── integrations/           # 生态集成
│   │   ├── __init__.py
│   │   └── scrapy.py           #   与 Scrapy 互通
│   ├── __init__.py             #   包入口（懒加载公共 API）
│   ├── cli.py                  #   命令行（scrapling 命令）
│   ├── parser.py               #   自适应解析器（Selector/Selectors）
│   └── py.typed                #   类型标记（PEP 561）
│
├── agent-skill/                # AI 编程助手技能包
│   ├── Scrapling-Skill/        #   技能定义（md/py）
│   ├── README.md
│   └── Scrapling-Skill.zip
│
├── docs/                       # MkDocs 文档站点
│   ├── ai/ fetching/ parsing/ spiders/ cli/ integrations/ tutorials/  # 各模块文档
│   ├── assets/                 #   封面图等
│   ├── README_*.md             #   多语言 README（含 README_CN.md 简体中文）
│   ├── index.md overview.md benchmarks.md donate.md
│   └── requirements.txt
│
├── tests/                      # 测试套件（pytest）
├── images/                     # README 用赞助商图片
├── pyproject.toml              # 包元数据、依赖、构建配置
├── setup.cfg                   # setuptools 配置
├── MANIFEST.in                 # 打包包含文件
├── pytest.ini / tox.ini        # 测试配置
├── ruff.toml                   # Lint 配置
├── zensical.toml              # 质量门禁配置
├── .bandit.yml                 # 安全扫描配置
├── .pre-commit-config.yaml     # 提交钩子
├── Dockerfile                  # 容器镜像
├── .dockerignore
├── server.json                 # MCP/服务相关配置
├── benchmarks.py               # 性能基准脚本
├── cleanup.py                  # 清理脚本
├── ROADMAP.md                  # 路线图
├── CONTRIBUTING.md             # 贡献指南
├── CODE_OF_CONDUCT.md          # 行为准则
├── LICENSE                     # BSD 许可证
├── README.en.md               # 英文原版 README（保留）
└── README.md                  # 本中文文档
```

---

## 6. 自适应解析器（parser）

`scrapling/parser.py` 提供 `Selector`（单节点）与 `Selectors`（多节点）两个核心类，它们是对 `lxml.html.HtmlElement` 的轻量封装（不继承以保可序列化）。

**能力：**

- 选择器：`css()`、`xpath()`、纯文本匹配；支持 `::text`、`::attr()` 等伪类。
- 惰性访问：`AttributesHandler` / `TextHandler` 让你像属性一样访问节点的文本与属性。
- **自适应（核心卖点）**：
  - `p.css('.product', auto_save=True)`：首次抓取即把元素的结构指纹存入 SQLite（`core/storage.py` 的 `SQLiteStorageSystem`，按域名隔离）。
  - 之后网站改版，`p.css('.product', adaptive=True)`：解析器用 `difflib.SequenceMatcher` 比对已存指纹，在变化后的 DOM 中重新定位最相似的元素。

**为什么需要它**：传统抓取脚本依赖固定 XPath/CSS，网站一改版就失效；自适应解析让脚本在改版后“自愈”，大幅降低维护成本。

---

## 7. 抓取器（fetchers）

四个公共抓取器，覆盖从“快”到“隐身”的全部场景：

| 抓取器 | 底层 | 特点 | 典型用途 |
| --- | --- | --- | --- |
| `Fetcher` / `AsyncFetcher` | `curl_cffi` | 同步/异步 HTTP，`impersonate` 模拟浏览器 TLS 指纹 | 轻量、高速、需伪装时 |
| `StealthyFetcher` | Chromium（隐身） | 真实指纹、解 Cloudflare、DoH、广告拦截、WebRTC 防泄漏 | 强反爬站点 |
| `DynamicFetcher` | Playwright/Chromium | 完全可编程浏览器控制、页面自动化 | 需要 JS 渲染/交互 |

**关键能力（以 `StealthyFetcher.fetch` 为例）：**

- `headless` / `real_chrome` / `cdp_url`：无头、本机 Chrome、或连接已有浏览器。
- `solve_cloudflare`：自动通过 Cloudflare Turnstile / Interstitial 挑战。
- `network_idle` / `wait_selector` / `page_action` / `page_setup`：精细控制等待与自动化。
- `proxy` / `dns_over_https` / `block_webrtc`：代理与防泄漏。
- `block_ads` / `blocked_domains`：拦截约 3500 个广告/追踪域名。
- `hide_canvas` / `allow_webgl`：指纹对抗。

**统一响应**：所有抓取器返回 `Response`（`engines/toolbelt/custom.py`），它继承 `Selector`，因此可直接 `response.css(...)` 解析。

**代理轮换**：`ProxyRotator`（`engines/toolbelt/proxy_rotation.py`）线程安全，默认循环策略 `cyclic_rotation`，支持自定义策略；`is_proxy_error()` 自动识别代理故障并切换。

---

## 8. 爬虫框架（spiders）

`scrapling/spiders/` 提供类 Scrapy 的爬虫框架，适合规模化爬取：

- **`Spider`**（`spider.py`）：抽象基类，定义 `name` / `start_urls` / `allowed_domains` / 并发与延迟等；用户实现 `parse(self, response)` 产出数据或新请求。
- **`CrawlerEngine`**（`engine.py`）：调度核心，驱动并发请求与回调。
- **`Scheduler` / `SessionManager`**：请求队列与多会话（每会话可绑不同代理/指纹）。
- **`Request` / `CrawlResult` / `CrawlStats`**：请求、结果、实时统计。
- **`checkpoint.py`**：暂停/恢复（断点续爬），长时间任务不怕中断。
- **`cache.py` / `robotstxt.py`**：请求缓存与 robots.txt 遵守（`robots_txt_obey`）。
- **`links.py`**：`LinkExtractor` 自动提取后续链接。
- **`templates/`**：开箱即用的 `CrawlSpider`（规则驱动）、`SitemapSpider`（站点地图）、`ShopifySpider`（电商模板）、`CrawlRule`。

**示例：**

```python
from scrapling.spiders import Spider, Response

class MySpider(Spider):
    name = "demo"
    start_urls = ["https://example.com/"]

    async def parse(self, response: Response):
        for item in response.css('.product'):
            yield {"title": item.css('h2::text').get()}

MySpider().start()
```

---

## 9. AI / MCP 与 CLI

- **MCP Server**（`core/ai.py`，需 `scrapling[ai]`）：基于 `mcp` 实现 FastMCP 服务，向 LLM 智能体暴露“抓取并转 Markdown/HTML/文本”“管理浏览器会话”“截图”等工具，让 AI Agent 直接调用 Scrapling。
- **Agent Skill**（`agent-skill/`）：面向 AI 编程助手（如 Claude Code / Cursor）的技能包，含使用说明与示例，使智能体按最佳实践编写 Scrapling 代码。
- **CLI**（`cli.py`）：安装 `scrapling[fetchers]` 后可用 `scrapling` 命令：
  - `scrapling fetch <url> -o page.html`：抓取并保存为 HTML/Markdown/文本；
  - `scrapling extract <url> '<css>'`：直接提取选择器内容；
  - 交互式 shell：边抓边试。

---

## 10. 安装与配置

```bash
# 仅核心（解析器）
pip install scrapling

# 含抓取器（HTTP + 浏览器）
pip install "scrapling[fetchers]"

# 含 AI / MCP
pip install "scrapling[ai]"

# 含交互式 shell
pip install "scrapling[shell]"

# 全部
pip install "scrapling[all]"
```

浏览器抓取器首次使用需安装 Playwright 浏览器：

```bash
python -m playwright install chromium --with-deps
```

**最小示例：**

```python
from scrapling.fetchers import StealthyFetcher

StealthyFetcher.adaptive = True
page = StealthyFetcher.fetch('https://example.com', headless=True, network_idle=True)
products = page.css('.product', auto_save=True)   # 首次保存结构指纹
products = page.css('.product', adaptive=True)     # 改版后自动重定位
```

更多配置（代理、指纹、并发、MCP）详见官方文档：<https://scrapling.readthedocs.io>

---

## 11. 许可证

本项目基于 **BSD 许可证** 发布，商用友好。详见 [LICENSE](LICENSE)。

> 上游项目：[D4Vinci/Scrapling](https://github.com/D4Vinci/Scrapling)（作者 Karim Shoair）。本仓库为其派生版本，版本 `0.4.11`。
