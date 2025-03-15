---
layout: post
title:  Python工程项目管理
categories: [python]
comments: true
tags: [python]
---

> 本文主要介绍Python项目工程实现相关的库。

* TOC
{:toc}

<!--more-->

## 项目实战经验

### 初始化阶段

#### 版本控制系统(Version Control Systems)

* **git**: 一种分布式版本控制系统，用于追踪项目历史版本的变化。

#### 环境管理器(Dependency Management)

* **Anaconda**: 用于管理Python环境和包的工具，特别适合科学计算和数据分析项目。
* **Poetry**: 一个现代的依赖管理和打包工具，简化了Python项目的依赖管理。
* **Venv**: Python内置的虚拟环境管理工具，用于创建隔离的Python环境。
* **PDM**: 一个轻量级的Python包管理工具，结合了Poetry和Venv的优点。
* **Pip/Pipx**: 分别用于安装Python包及其独立版本的工具。

#### 项目模板

* **Flit**: 一个简单的Python包管理工具，用于构建和发布Python包。
* **Poetry**: 提供了项目模板生成器，帮助快速搭建项目结构。
* **PDM**: 也提供项目模板生成功能，方便项目初始化。

### 开发阶段

#### 编辑器(IDE and Code Editors)

* **PyCharm**: 一款专为Python开发设计的集成开发环境（IDE），提供了丰富的开发工具。
* **Visual Studio Code (VSCode)**: 一个轻量级但功能强大的源代码编辑器，支持多种语言和插件。
* **Vi(m)**: 一个经典的命令行文本编辑器，适合高级用户。

#### 测试(Testing)

* **unittest**: Python官方提供的单元测试框架，内置在标准库中。
* **pytest**: 一个流行的外部测试框架，支持单元测试、集成测试和功能测试。
* **tox**: 一个自动化测试工具，用于确保代码在多环境下运行一致。

#### 代码分析工具(Linting and Analysis)

* **pylint**: 一款强大的代码分析工具，用于检测潜在的错误和风格问题。
* **mypy**: 一个静态类型检查器，帮助提高代码的可读性和健壮性。
* **flake8**: 结合了多个工具的功能，提供统一的代码风格检查。

#### 版本控制和协作

* **GitHub/GitLab**: 提供代码托管、版本控制、问题跟踪和代码审查功能。
* **pre-commit**: 一个用于在提交代码前运行一系列检查和测试的工具。

### 构建与部署阶段

#### 文档

* **Sphinx**: 一个强大的文档生成工具，支持从源码生成高质量的文档。

#### 打包和发布(Build Tools)

* **pyinstaller**: 用于将Python应用程序打包成独立的可执行文件。
* **setuptools**: Python的标准打包工具，用于构建和发布Python包。
* **wheel**: 用于创建和管理Python轮子（.whl文件）的工具。

### 持续开发与集成(Continuous Integration/Continuous Deployment (CI/CD))

* **GitHub Actions/GitLab CI/CD**: 提供自动化测试、构建和部署流程的支持。

### 监控与运维阶段

#### 监控和日志(Monitoring and Logging)

* **logging**: Python标准库提供的日志模块，用于记录应用程序的日志信息。
* **loguru**: 一个更现代化的日志库，提供了更多的功能和更好的性能。
* **Sentry**: 一个错误跟踪和监控工具，帮助发现并修复生产环境中的问题。

#### 性能分析(Performance Profiling)

* **cProfile**: 内置的性能分析工具，帮助识别程序中的瓶颈。
* **line-profiler**: 一个行级性能分析工具，提供详细的代码执行统计。

## 参考pdm项目配置

```toml
[project]
name = "file_history_dev"
version = "0.1.0"
description = "file history"
authors = [
    {name = "web", email = "web.hu@canchip.cn"},
]
dependencies = [
]
requires-python = "<3.12,>=3.9"
readme = "README.md"
license = {text = "MIT"}


[project.optional-dependencies]
module_generate = [
    "flit>=3.9.0",  # simple module generate
]
code_style = [
    "pycodestyle>=2.12.0",  # Formatters(old pep8)
    "autopep8>=2.3.1",  # Formatters
    "black>=24.4.2",    # Formatters
    "pylint>=3.2.6",    # Linters
    "flake8>=7.1.0",    # Linters
    "mypy>=1.11.0", # Linters
    "ruff>=0.5.5",  # Formatters and Linters
    "isort>=5.13.2",    # Library makeup
    "autoflake>=2.3.1", # Library makeup
]
gui = [
    "pyside6>=6.7.2",
]
code_style_management = [
    "pre-commit>=3.8.0",
]
test = [
    "pytest>=8.3.2",
    "pytest-cov>=5.0.0",
    "coverage>=7.6.0",
    "tox>=4.16.0",
    "pytest-qt>=4.4.0",
]
doc = [
    "sphinx>=7.4.7",
]
build_tools = [
    "twine>=5.1.1",
    "pyinstaller>=6.9.0",
]
environment_management = [
    "pip>=24.2",
    "pipx>=1.6.0",
    "pipenv>=2024.0.1",
    "virtualenv>=20.26.3",
]
data_analysis = [
    "numpy>=2.0.1",
    "pandas>=2.2.2",
    "scipy>=1.13.1",
    "matplotlib>=3.9.1",
    "notebook>=7.2.1",
    "ipython>=8.18.1",
]
scientific_computing = [
    "mkl>=2024.2.0",
    "statsmodels>=0.14.2",
    "localreg>=0.5.0",
    "pymoo>=0.6.1.3",
    "scikit-rf>=1.1.0",
]

artificial_intelligence = [
    "scikit-learn>=1.5.1",
    "tensorflow>=2.14.0",
]
extend_utils = [
    "tqdm>=4.66.4",
    "psutil>=6.0.0",
    "crypto>=1.4.1",
]
profiler_utils = [
    "line-profiler>=4.1.3",
]


[build-system]
requires = ["pdm-backend"]
build-backend = "pdm.backend"


[tool.pdm]
distribution = true
```
