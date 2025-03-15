---
layout: post
title:  【Python Testing with pytest Simple, Rapid, Effective, and Scalable】笔记
categories: [python]
comments: true
tags: [python, pytest]
---

> 本文主要介绍使用pytest进行Python单元测试的指南。

* TOC
{:toc}

<!--more-->

下面是这本书的架构及内容概览：

> Acknowledgments 和 Preface

* **Acknowledgments**：感谢那些对本书创作做出贡献的人。
* **Preface**：介绍为什么选择pytest作为测试工具，以及本书的组织方式、读者需要的先决知识、第二版更新的理由和如何获取示例代码与在线资源。

> Part I. Primary Power

这部分介绍了pytest的基础知识和核心概念，适合pytest初学者。

1. **Getting Started with pytest**：讲解如何安装pytest，以及如何运行第一个测试。
2. **Writing Test Functions**：教授如何编写测试函数，包括使用assert语句、测试异常、以及如何组织测试函数。
3. **pytest Fixtures**：详解pytest的fixture机制，这是pytest的核心特性之一，用于设置和清理测试环境。
4. **Builtin Fixtures**：介绍pytest自带的fixtures，如tmp_path和monkeypatch。
5. **Parametrization**：学习如何使用参数化测试，可以让你的测试函数接受不同的参数值。
6. **Markers**：标记测试函数，用于控制测试的执行，如跳过、预期失败和选择性执行。

> Part II. Working with Projects
这部分深入探讨了在实际项目中使用pytest的方法。

7. **Strategy**：制定测试策略，包括测试范围、软件架构考量和创建测试案例。
8. **Configuration Files**：理解和使用pytest的配置文件，如pytest.ini和conftest.py。
9. **Coverage**：使用coverage.py和pytest-cov插件来衡量代码覆盖率。
10. **Mocking**：隔离外部依赖，使用mock对象来模拟对象的行为。
11. **tox and Continuous Integration**：介绍持续集成的概念，以及如何使用tox进行多版本Python测试。
12. **Testing Scripts and Applications**：测试脚本和应用程序的策略，包括如何组织项目目录结构。
13. **Debugging Test Failures**：当测试失败时，如何使用pytest的工具进行调试。

> Part III. Booster Rockets
这部分涵盖了高级主题和扩展功能。

14. **Third-Party Plugins**：探索第三方pytest插件，包括并行测试和随机测试顺序。
15. **Building Plugins**：指导读者如何自己创建pytest插件，包括本地开发、发布和测试。
16. **Advanced Parametrization**：高级参数化技巧，如动态参数和间接参数化。

> Appendices

* **Virtual Environments**：解释虚拟环境的重要性，以及如何使用它们。
* **pip**：介绍pip，Python的包管理器，用于安装和管理软件包。

整本书从基础到进阶，不仅提供了理论知识，还包含了大量的练习题和实战案例，旨在让读者能够熟练运用pytest进行高效、高质量的测试工作。

## 第1章. pytest入门指南

### 安装pytest

要在虚拟环境中安装pytest，请按照以下步骤操作：

```shell
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $ pip install pytest
```

### 运行pytest

pytest支持多种运行方式：

* `pytest`：无参数时，pytest会在当前目录及其子目录中搜索测试。
* `pytest <filename>`：运行指定文件中的测试。
* `pytest <filename> <filename> ...`：运行多个指定文件中的测试。
* `pytest <dirname>`：从特定目录（或多个目录）开始，递归搜索并运行测试。

示例测试文件如下：

```python
# test_one.py
def test_passing():
    assert (1, 2, 3) == (1, 2, 3)

# test_two.py
def test_failing():
    assert (1, 2, 3) == (3, 2, 1)
```

使用`-v`或`--verbose`命令行标志可以显示更详细的输出。

使用`--tb=no`命令行标志可以关闭追踪回溯信息。

示例命令如下：

```shell
pytest test_one.py
pytest test_two.py
pytest -v test_two.py
pytest --tb=no
pytest --tb=no test_one.py test_two.py
pytest -v ch1/test_one.py::test_passing
```

### 测试发现机制

测试发现机制指的是pytest如何找到你的测试代码，这取决于命名规则：

* 测试文件应当命名为`test_<something>.py`或`<something>_test.py`。
* 测试方法和函数应当命名为`test_<something>`。
* 测试类应当命名为`Test<Something>`。

### 测试函数的结果

测试函数的结果可能有以下几种情况：

* 成功：当所有断言都通过时，测试被视为成功。
* 失败：当断言失败或测试中抛出异常时，测试被视为失败。
* 跳过：当测试被显式跳过（使用`pytest.mark.skip`）时，测试被视为跳过。
* 错误：当测试在执行前就遇到错误（如导入错误）时，测试被视为错误。
* 预期失败：当测试预计会失败但实际通过（或反之）时，测试被视为预期失败。
* 挂起：当测试因外部条件未满足而无法执行时，测试被视为挂起。例如，网络连接不可用。

## 第2章. 编写测试函数

### Cards 应用程序

要在虚拟环境中安装 Cards 应用程序及 pytest，按照以下步骤操作：

* 进入代码目录：`cd /path/to/code`
* 创建虚拟环境：`python -m venv venv --prompt cards`
* 激活虚拟环境：`source venv/bin/activate`（Windows上使用`venv\Scripts\activate.bat`）
* 安装 Cards 应用：`pip install ./cards_proj`
* 安装 pytest：`pip install pytest`

### 使用 assert 语句

pytest 支持 assert 重写，允许我们使用标准的 Python assert 表达式。

| pytest | unittest |
| --- | --- |
| assert something | assertTrue(something) |
| assert not something | assertFalse(something) |
| assert a == b | assertEqual(a, b) |
| assert a != b | assertNotEqual(a, b) |
| assert a is None | assertIsNone(a) |
| assert a is not None | assertIsNotNone(a) |
| assert a <= b | assertLessEqual(a, b) |

### 通过 pytest.fail() 和异常失败

测试可以从断言失败、调用 fail() 或任何未捕获的异常中失败。

如果出现任何未捕获的异常，测试将失败。这可能发生在：

* 断言语句失败，将引发 AssertionError 异常，
* 测试代码调用 pytest.fail()，将引发异常，
* 其他任何异常被抛出。

### 编写断言辅助函数

使用 pytest.raises() 来检测预期的异常。

`with pytest.raises(TypeError):` 语句表示接下来的代码块应该抛出 TypeError 异常。如果没有抛出异常，测试失败；如果抛出了不同的异常，测试同样失败。

### 结构化测试函数

一种优秀的测试结构方法被称为 Given-When-Then 或 Arrange-Act-Assert。

Given-Act-Assert 模式最初由 Bill Wake 在 2001 年命名。Kent Beck 后来将其作为测试驱动开发 (TDD) 的一部分加以推广。

行为驱动开发 (BDD) 使用 Given-When-Then 术语，这是一种来自 Ivan Moore 的模式，由 Dan North 推广。

不论步骤的命名如何，目的始终相同：将测试分阶段。

一种常见的反模式是采用“Arrange-Assert-Act-Assert-Act-Assert...”的模式，其中大量动作后跟着状态或行为检查，以验证工作流。

* Given/Arrange —— 初始状态。在这里，你设置数据或环境以准备进行动作。
* When/Act —— 执行某个动作。这是测试的重点——我们试图确保正确运行的行为。
* Then/Assert —— 应该发生一些预期的结果或最终状态。测试末尾，我们确保动作产生了预期的行为。

### 通过类分组测试

类可以用于分组测试。
我建议在生产测试代码中，也要谨慎使用测试类，主要用作分组手段。

### 运行测试子集

在调试时运行测试的小子集非常方便，pytest 允许你以多种方式运行少量测试。

| 子集 | 语法 |
| --- | --- |
| 单个测试方法 | pytest path/test_module.py::TestClass::test_method |
| 类中的所有测试 | pytest path/test_module.py::TestClass |
| 单个测试函数 | pytest path/test_module.py::test_function |
| 模块中的所有测试 | pytest path/test_module.py |
| 目录中的所有测试 | pytest path |
| 匹配名称模式的测试 | pytest -k pattern |
| 按标记筛选测试 | pytest -m marker |

### 命令行标志

`-vv` 命令行标志在测试失败时显示更多信息。
`--tb=short` 命令行标志使用较短的追踪回溯格式。
`-k` 参数，结合 and、not 和 or，接收一个表达式，告诉 pytest 运行包含与表达式匹配子串的测试。

## 第3章. pytest 中的fixture

fixtures是测试辅助函数，对于几乎任何非平凡软件系统的测试代码结构至关重要。fixture是在实际测试函数之前（有时在之后）由 pytest 运行的函数。你可以使用fixture获取测试所需的数据集，让系统在运行测试前处于已知状态，或为多个测试准备好数据。

### 开始使用fixture

fixture是通过 `@pytest.fixture()` 装饰器定义的函数。

在 pytest 和本书中，测试fixture指的是 pytest 提供的一种机制，用于将“准备”和“清理”代码与测试函数本身分开。

在测试代码主体中发生的异常（或断言失败或调用 pytest.fail()）会导致测试结果为“失败”。然而，在fixture中，测试函数会被报告为“错误”。

### 使用fixture进行设置和清理

测试函数或其他fixture依赖于一个fixture，只需在它们的参数列表中包含fixture的名称即可。

fixture可以使用 `return` 或 `yield` 返回数据。

`yield` 前面的代码是设置代码。`yield` 后面的代码是清理代码。

```python
# ch3/test_count_initial.py
from pathlib import Path
from tempfile import TemporaryDirectory
import cards

def test_empty():
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db = cards.CardsDB(db_path)
        count = db.count()
        db.close()
        assert count == 0
```

pytest 会查看测试的具体参数名称，然后寻找具有相同名称的fixture。我们从不直接调用fixture函数，pytest 会为我们完成调用。

### 使用 --setup-show 追踪fixture执行

pytest 提供了命令行标志 `--setup-show`，它可以展示测试和fixture的操作顺序，包括fixture的设置和清理阶段。

`$ pytest --setup-show test_count.py`

### 指定fixture的作用域

fixture可以设置为 `function`、`class`、`module`、`package` 或 `session` 作用域。

* `scope='function'`
每个测试函数运行一次。使用fixture的每个测试之前的设置部分都会运行，使用fixture的每个测试之后的清理部分也会运行。这是没有指定作用域参数时的默认作用域。
* `scope='class'`
每个测试类运行一次，无论类中有多少测试方法。
* `scope='module'`
每个模块运行一次，无论模块中有多少测试函数、方法或其他fixture使用它。
* `scope='package'`
每个包（或测试目录）运行一次，无论包中有多少测试函数、方法或其他fixture使用它。
* `scope='session'`
每个会话运行一次。所有使用会话作用域fixture的测试方法和函数共享一个设置和清理调用。

默认作用域为函数作用域，甚至可以动态定义作用域。

### 通过 conftest.py 文件共享fixture

多个测试函数可以使用同一个fixture。

多个测试模块可以通过 `conftest.py` 文件使用同一个fixture。

不同作用域的多个fixture可以在加速测试套件的同时保持测试隔离。

测试和fixture可以使用多个fixture。

虽然 `conftest.py` 是一个 Python 模块，但它不应被测试文件导入。`conftest.py` 文件会被 pytest 自动读取，所以无需在任何地方导入 conftest。

### 查找fixture定义的位置

使用 `--fixtures`，pytest 可以向我们展示测试可使用的所有可用fixture的列表。这个列表包括内置fixture。

    `$ pytest --fixtures -v`

使用 `--fixtures-per-test` 可以查看每个测试使用的fixture以及fixture的定义位置。

    `$ pytest --fixtures-per-test test_count.py::test_empty`

### 使用多级fixture

使用多层次的fixture可以提供显著的速度优势，同时保持测试顺序的独立性。

数据库的设置最先发生，具有会话作用域（来自 S）。接着是 cards_db 的设置，在每个测试函数调用之前发生，具有函数作用域（来自 F）。此外，所有三个测试都通过了。

```python
@pytest.fixture(scope="session")
def db():
    """连接到临时数据库的 CardsDB 对象"""
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db_ = cards.CardsDB(db_path)
        yield db_
        db_.close()

@pytest.fixture(scope="function")
def cards_db(db):
    """空的 CardsDB 对象"""
    db.delete_all()
    return db
```

### 动态决定fixture作用域

代替具体的作用域，我们放入了一个函数名，db_scope。为了允许 pytest 使用这个新标志，我们需要编写一个钩子函数（`pytest_addoption`）。

```python
# conftest.py
@pytest.fixture(scope=db_scope)
def db():
    """连接到临时数据库的 CardsDB 对象"""
    with TemporaryDirectory() as db_dir:
        db_path = Path(db_dir)
        db_ = cards.CardsDB(db_path)
        yield db_
        db_.close()


def db_scope(fixture_name, config):
    if config.getoption("--func-db", None):
        return "function"
    return "session"


def pytest_addoption(parser):
    parser.addoption(
        "--func-db",
        action="store_true",
        default=False,
        help="为每个测试创建新的数据库",
    )
```

经过所有这些之后，默认行为与之前相同，即具有会话作用域的 db：

```shell
pytest --setup-show test_count.py
```

但是当我们使用新标志时，我们会得到一个函数作用域的 db fixture：

```shell
pytest --func-db --setup-show test_count.py
```

### 使用 autouse 对于总是被使用的fixture

你可以使用 `autouse=True` 让fixture始终运行。自动使用的fixture不必在测试函数中被命名。

### 重命名fixture

fixture的名称可以与fixture函数名称不同。
pytest 允许你使用 `name` 参数对 `@pytest.fixture()` 进行fixture重命名。

```python
# test_rename_fixture.py
import pytest


@pytest.fixture(name="ultimate_answer")
def ultimate_answer_fixture():
    return 42


def test_everything(ultimate_answer):
    assert ultimate_answer == 42
```

### 命令行标志

`pytest --setup-show` 用于查看执行顺序。
`pytest --fixtures` 用于列出可用的fixture及其位置。
`pytest -s` 和 `--capture=no` 允许在通过的测试中也能看到打印语句。

## 第4章. 内置fixture

### 使用 tmp_path 和 tmp_path_factory

`tmp_path` 和 `tmp_path_factory` 用于创建临时目录。

`tmp_path` 和 `tmp_path_factory` fixture用于处理临时目录，其中 `tmp_path` 具有函数作用域，而 `tmp_path_factory` 具有会话作用域。

`tmp_path` 函数作用域fixture返回一个指向在测试期间和之后一段时间内存在的临时目录的 `pathlib.Path` 实例。`tmp_path_factory` 会话作用域fixture则返回一个 `TempPathFactory` 对象。

```python
def test_tmp_path(tmp_path):
    file = tmp_path / "file.txt"
    file.write_text("Hello")
    assert file.read_text() == "Hello"

def test_tmp_path_factory(tmp_path_factory):
    path = tmp_path_factory.mktemp("sub")
    file = path / "file.txt"
    file.write_text("Hello")
    assert file.read_text() == "Hello"

# conftest.py
@pytest.fixture(scope="session")
def db(tmp_path_factory):
    """连接到临时数据库的 CardsDB 对象"""
    db_path = tmp_path_factory.mktemp("cards_db")
    db_ = cards.CardsDB(db_path)
    yield db_
    db_.close()
```

以下是两个相关的内置fixture：

* `*tmpdir` — 与 `tmp_path` 类似，但返回一个 `py.path.local` 对象。此fixture在 `tmp_path` 出现之前就在 pytest 中存在。`py.path.local` 在 Python 3.4 添加 `pathlib` 之前存在，现在正在逐步淘汰，转而支持标准库中的 `pathlib` 版本。因此，推荐使用 `tmp_path`。
* `tmpdir_factory` — 与 `tmp_path_factory` 类似，但其 `mktemp` 函数返回的是 `py.path.local` 对象而不是 `pathlib.Path` 对象。

你也可以通过 `pytest --basetemp=mydir` 指定自己的基目录。

### 使用 capsys

`capsys` — 用于捕获输出。

`capsys` 可以用来捕获 `stdout` 和 `stderr`。它还可以暂时禁用输出捕获。

测试版本号的一个方法是使用 `subprocess.run()` 实际运行命令，抓取输出，并将其与 API 版本进行比较：

```python
# test_version.py
import subprocess
def test_version_v1():
    process = subprocess.run(
        ["cards", "version"], capture_output=True, text=True
    )
    output = process.stdout.rstrip()
    assert output == cards.__version__
```

`capsys` fixture使你能够捕获对 `stdout` 和 `stderr` 的写入。我们可以直接调用实现 CLI 版本功能的方法，并使用 `capsys` 来读取输出。`capsys` 的另一个特性是能够暂时禁用 pytest 正常的输出捕获。

```python
# test_version.py
import cards

def test_version_v2(capsys):
    cards.cli.version()
    output = capsys.readouterr().out.rstrip()
    assert output == cards.__version__
```

使用 `-s` 或 `--capture=no` 标志可以看到所有输出，即使在通过的测试中也是如此。

```python
# test_print.py
def test_normal():
    print("\nnormal print")
```

`$ pytest -s test_print.py::test_normal`

另一种始终包含输出的方法是使用 `capsys.disabled()`。在 `with` 块中的输出始终会被显示，即使没有 `-s` 标志。

```python
# test_print.py
def test_disabled(capsys):
    with capsys.disabled():
        print("\ncapsys disabled print")
```

相关的fixture有 `capsysbinary`、`capfd`、`capfdbinary` 和 `caplog`。

以下是相关的内置fixture：

* `capfd` — 类似于 `capsys`，但捕获文件描述符 1 和 2，这通常等同于 `stdout` 和 `stderr`
* `capsysbinary` — `capsys` 捕获文本，而 `capsysbinary` 捕获字节。
* `capfdbinary` — 在文件描述符 1 和 2 上捕获字节
* `caplog` — 捕获使用 Python 日志记录包写入的输出

### 使用 monkeypatch

`monkeypatch` — 用于改变环境或应用程序代码，类似一种轻量级的模拟。

`monkeypatch` 可以用来改变应用程序代码或环境。在 Cards 应用程序中，我们用它将数据库位置重定向到由 `tmp_path` 创建的临时目录。

内置的 `monkeypatch` fixture允许你在单个测试的上下文中执行这些操作。它用于修改对象、字典、环境变量、Python 搜索路径或当前目录。这就像一个小型的模拟版本。

`monkeypatch` fixture提供了以下功能：

* `setattr(target, name, value, raising=True)` — 设置属性
* `delattr(target, name, raising=True)` — 删除属性
* `setitem(dic, name, value)` — 设置字典条目
* `delitem(dic, name, raising=True)` — 删除字典条目
* `setenv(name, value, prepend=None)` — 设置环境变量
* `delenv(name, raising=True)` — 删除环境变量
* `syspath_prepend(path)` — 将 `path` 插入到 `sys.path` 开头，`sys.path` 是 Python 的导入位置列表
* `chdir(path)` — 更改当前工作目录

```python
from typer.testing import CliRunner
import cards

def run_cards(*params):
    runner = CliRunner()
    result = runner.invoke(cards.app, params)
    return result.output.rstrip()

def test_run_cards():
    assert run_cards("version") == cards.__version__

def test_patch_get_path(monkeypatch, tmp_path):
    def fake_get_path():
        return tmp_path
    monkeypatch.setattr(cards.cli, "get_path", fake_get_path)
    assert run_cards("config") == str(tmp_path)

def test_patch_home(monkeypatch, tmp_path):
    full_cards_dir = tmp_path / "cards_db"

    def fake_home():
        return tmp_path

    monkeypatch.setattr(cards.cli.pathlib.Path, "home", fake_home)
    assert run_cards("config") == str(full_cards_dir)
```

### 剩余的内置fixture

以下是本版写作时 pytest 带有的剩余内置fixture列表：

* `capfd`、`capfdbinary`、`capsysbinary` — `capsys` 的变体，用于处理文件描述符和/或二进制输出
* `caplog` — 与 `capsys` 类似，用于处理 Python 日志系统生成的消息
* `cache` — 用于在 pytest 运行之间存储和检索值。这个fixture最有用的部分在于它使得 `--last-failed`、`--failed-first` 等标志成为可能。
* `doctest_namespace` — 如果你喜欢使用 pytest 运行 doctest 风格的测试，这将很有用。
* `pytestconfig` — 用于访问配置值、插件管理器和插件挂钩。
* `record_property`、`record_testsuite_property` — 用于向测试或测试套件添加额外属性。特别适用于向 XML 报告中添加数据，以便持续集成工具使用。
* `recwarn` — 用于测试警告消息。
* `request` — 用于提供有关正在执行的测试函数的信息。最常用于fixture参数化中。
* `pytester`、`testdir` — 用于提供一个临时测试目录，以帮助运行和测试 pytest 插件。`pytester` 是基于 `pathlib` 的 `testdir` 替代品。
* `tmpdir`、`tmpdir_factory` — 类似于 `tmp_path` 和 `tmp_path_factory`；用于返回 `py.path.local` 对象而不是 `pathlib.Path` 对象。

### 命令行标志

使用 `pytest -s` 或 `pytest --capture=no` 标志可以看到所有输出，即使在通过的测试中也是如此。
你可以使用 `pytest --fixtures` 来阅读这些和其他fixture的详细信息。

以下是中文重构后的代码和说明：

下面是这段文本的中文重构版本，其中代码部分保持不变：

## 第5章. 参数化

### 使用 `@pytest.mark.parametrize()` 参数化函数

为了参数化一个测试函数，需要在测试定义中添加参数，并使用 `@pytest.mark.parametrize()` 装饰器来定义要传递给测试的不同参数组合，例如：

```python
# test_func_param.py
import pytest
from cards import Card

@pytest.mark.parametrize(
    "start_summary, start_state",
    [
    ("write a book", "done"),
    ("second edition", "in prog"),
    ("create a course", "todo"),
    ],
)
def test_finish(cards_db, start_summary, start_state):
    initial_card = Card(summary=start_summary, state=start_state)
    index = cards_db.add_card(initial_card)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"
```

### 使用 `@pytest.fixture(params=())` 参数化 fixture

pytest 会根据我们提供的每一组值调用 fixture 一次。之后，依赖于该 fixture 的每个测试函数也会被调用一次，每次对应一个 fixture 值。每个 `params` 的值会被保存到 `request.param` 中以供 fixture 使用。

```python
# test_fix_param.py
@pytest.fixture(params=["done", "in prog", "todo"])
def start_state(request):
    return request.param

def test_finish(cards_db, start_state):
    c = Card("write a book", state=start_state)
    index = cards_db.add_card(c)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"
```

### 使用名为 `pytest_generate_tests` 的钩子函数

钩子函数通常被插件用来改变 pytest 的正常执行流程。但是我们也可以在测试文件和 `conftest.py` 文件中使用许多钩子函数。`metafunc` 对象包含了大量信息，但这里我们只使用它来获取参数名称并生成参数化。

使用 `pytest_generate_tests` 实现相同的流程如下：

```python
# test_gen.py
from cards import Card

def pytest_generate_tests(metafunc):
    if "start_state" in metafunc.fixturenames:
        metafunc.parametrize("start_state", ["done", "in prog", "todo"])

def test_finish(cards_db, start_state):
    c = Card("write a book", state=start_state)
    index = cards_db.add_card(c)
    cards_db.finish(index)
    card = cards_db.get_card(index)
    assert card.state == "done"
```

`pytest_generate_tests` 特别有用，如果我们在测试收集时想要以有趣的方式修改参数化列表。

* 我们可以根据命令行标志来构建我们的参数化列表，因为 `metafunc` 给我们提供了访问 `metafunc.config.getoption("--someflag")` 的能力。也许我们可以添加一个 `--excessive` 标志来测试更多的值，或者一个 `--quick` 标志来测试少量的值。
* 一个参数的参数化列表可以基于另一个参数的存在。例如，对于请求两个相关参数的测试函数，我们可以为这两个参数使用不同的值集，而不是仅请求一个参数的情况。
* 我们可以同时参数化两个相关的参数，例如使用 `metafunc.parametrize("planet, moon", [('Earth', 'Moon'), ('Mars', 'Deimos'), ('Mars', 'Phobos'), ...])`。

### 使用关键字选择测试用例

参数化技术非常适合快速创建大量测试用例。因此，能够运行测试用例的一个子集往往是非常有益的。我们探讨了如何使用 `pytest -k` 来运行参数化测试用例的子集。

我们可以运行所有 "todo" 的测试用例：

```
pytest -v -k todo
```

如果我们想要排除带有 "play" 或 "create" 的测试用例，我们可以进一步筛选：

```
pytest -v -k "todo and not (play or create)"
```

我们也可以选择运行一个特定的测试用例：

```
pytest -v "test_func_param.py::test_finish[write a book-done]"
```

### 命令行标志

我们探讨了如何使用 `pytest -k` 来运行参数化测试用例的子集。

下面是这段文本的中文重构版本，其中代码部分保持不变：

## 第6章. 标记

### 使用内置标记

pytest 的内置标记用于修改测试运行的行为。以下是 pytest 6 版本中包含的所有内置标记：

* `@pytest.mark.filterwarnings(warning)`: 此标记向给定测试添加警告过滤器。
* `@pytest.mark.skip(reason=None)`: 此标记跳过测试，并可选地给出原因。
* `@pytest.mark.skipif(condition, ..., *, reason)`: 如果任何条件为真，则此标记跳过测试。
* `@pytest.mark.xfail(condition, ..., *, reason, run=True, raises=None, strict=xfail_strict)`: 此标记告诉 pytest 我们期望测试失败。
* `@pytest.mark.parametrize(argnames, argvalues, indirect, ids, scope)`: 此标记多次调用测试函数，依次传递不同的参数。
* `@pytest.mark.usefixtures(fixturename1, fixturename2, ...)`: 此标记标记测试需要指定的所有 fixture。

### 使用 `pytest.mark.skip` 跳过测试

`@pytest.mark.skip()` 标记告诉 pytest 跳过测试。原因虽然是可选的，但在后期维护时列出原因很重要。

```python
import pytest

@pytest.mark.skip(reason="Card 尚不支持 < 比较")
def test_less_than():
    c1 = Card("a task")
    c2 = Card("b task")
    assert c1 < c2
```

当我们运行被跳过的测试时，它们会显示为 `s` 或者 `SKIPPED`（在详细模式下）：

```
pytest test_skip.py
```

或者

```
pytest -v -ra test_skip.py
```

### 条件性跳过测试 `pytest.mark.skipif`

`skipif` 标记允许你传入任意数量的条件，只要其中任何一个条件为真，就会跳过测试。在此示例中，我们使用 `packaging.version.parse` 来允许我们隔离主要版本。

```python
import cards
from packaging.version import parse

@pytest.mark.skipif(
    parse(cards.__version__).major < 2,
    reason="Card < 比较在 1.x 版本不支持",
)
def test_less_than():
    c1 = Card("a task")
    c2 = Card("b task")
    assert c1 < c2
```

### 预期测试失败 `pytest.mark.xfail`

如果我们想运行所有的测试，即使是那些我们知道会失败的测试，我们可以使用 `xfail` 标记。

这是 `xfail` 的完整签名：

```python
@pytest.mark.xfail(condition, ..., *, reason, run=True,
raises=None, strict=xfail_strict)
```

此标记的第一组参数与 `skipif` 相同。默认情况下，测试会被运行，但 `run` 参数可用于告诉 pytest 不运行测试，通过设置 `run=False`。`raises` 参数允许你提供一个异常类型或异常类型的元组，这些异常类型会导致 xfail。`strict` 告诉 pytest 如果通过的测试应该标记为 XPASS（strict=False）还是 FAIL（strict=True）。

```python
@pytest.mark.xfail(
    parse(cards.__version__).major < 2,
    reason="Card < 比较在 1.x 版本不支持",
)
def test_less_than():
    c1 = Card("a task")
    c2 = Card("b task")
    assert c1 < c2

@pytest.mark.xfail(reason="XPASS 示例")
def test_xpass():
    c1 = Card("a task")
    c2 = Card("a task")
    assert c1 == c2

@pytest.mark.xfail(reason="严格示例", strict=True)
def test_xfail_strict():
    c1 = Card("a task")
    c2 = Card("a task")
    assert c1 == c2
```

### 使用自定义标记选择测试

自定义标记是我们自己创建并应用于测试的标记。可以把它们想象成标签或标签。自定义标记可用于选择要运行或跳过的测试。

我们从“smoke”开始，并将 `@pytest.mark.smoke` 添加到 `test_start()`：

```python
def test_start(cards_db):
    """
    start 将状态从 "todo" 更改为 "in prog"
    """
    i = cards_db.add_card(Card("foo", state="todo"))
    cards_db.start(i)
    c = cards_db.get_card(i)
    assert c.state == "in prog"
```

现在我们应该能够使用 `-m smoke` 标志仅选择这个测试：

```
pytest -v -m smoke test_start.py
```

我们在 `pytest.ini` 文件中通过添加 markers 部分来注册自定义标记。每个列出的标记的形式为 `<marker_name>: <description>`，如下所示：

```python
# pytest.ini
[pytest]
markers =
    smoke: 测试的一部分
    exception: 检查预期的异常
```

### 标记文件、类和参数

* 使用文件级别的标记

```python
import pytest
from cards import Card, InvalidCardId

pytestmark = pytest.mark.finish
```

* 使用类级别的标记

```python
@pytest.mark.smoke
class TestFinish:
    def test_finish_from_todo(self, cards_db):
        i = cards_db.add_card(Card("foo", state="todo"))
        cards_db.finish(i)
        c = cards_db.get_card(i)
        assert c.state == "done"

    def test_finish_from_in_prog(self, cards_db):
        i = cards_db.add_card(Card("foo", state="in prog"))
        cards_db.finish(i)
        c = cards_db.get_card(i)
        assert c.state == "done"

    def test_finish_from_done(self, cards_db):
        i = cards_db.add_card(Card("foo", state="done"))
        cards_db.finish(i)
        c = cards_db.get_card(i)
        assert c.state == "done"
```

* 使用参数级别的标记

```python
@pytest.mark.parametrize(
    "start_state",
    [
        "todo",
        pytest.param("in prog", marks=pytest.mark.smoke),
        "done",
    ],
)
def test_finish_func(cards_db, start_state):
    i = cards_db.add_card(Card("foo", state=start_state))
    cards_db.finish(i)
    c = cards_db.get_card(i)
    assert c.state == "done"
```

* 使用参数级别的标记与 fixture 结合

```python
@pytest.fixture(
    params=[
        "todo",
        pytest.param("in prog", marks=pytest.mark.smoke),
        "done",
    ]
)
def start_state_fixture(request):
    return request.param

def test_finish_fix(cards_db, start_state_fixture):
    i = cards_db.add_card(Card("foo", state=start_state_fixture))
    cards_db.finish(i)
    c = cards_db.get_card(i)
    assert c.state == "done"
```

* 使用多个标记

```python
@pytest.mark.smoke
@pytest.mark.exception
def test_finish_non_existent(cards_db):
    i = 123  # 任何数字都行，数据库为空
    with pytest.raises(InvalidCardId):
        cards_db.finish(i)
```

### 使用 “and”，“or”，“not” 和括号与标记结合

```
pytest -v -m "finish and exception"
```

```
pytest -v -m "finish and not smoke"
```

```
pytest -v -m "(exception or smoke) and (not finish)"
```

```
pytest -v -m smoke -k "not TestFinish"
```

### 对标记严格要求

```python
@pytest.mark.smok
@pytest.mark.exception
def test_start_non_existent(cards_db):
    """
    不应能启动一个不存在的任务卡。
    """
    any_number = 123  # 任何数字都是无效的，数据库为空
    with pytest.raises(InvalidCardId):
        cards_db.start(any_number)
```

但是，如果我们希望这个警告成为错误，我们可以使用 `--strictmarkers` 标志：

```
pytest --strict-markers -m smoke
```

为了避免每次都要键入，你可以在 `pytest.ini` 文件的 addopts 部分添加 `--strict-markers`：

```python
# pytest.ini
[pytest]
markers =
    smoke: 测试的一部分
    exception: 检查预期的异常
    finish: 所有的 "cards finish" 相关测试
    num_cards: 为 cards_db fixture 预填充卡片的数量
addopts =
    --strict-markers
    -ra
xfail_strict = true
```

### 将标记与 fixture 结合

标记可以与 fixture 结合使用。它们也可以与其他插件和钩子函数结合使用。

现在我们需要修改 `cards_db` fixture：

```python
@pytest.fixture(scope="function")
def cards_db(session_cards_db, request, faker):
    db = session_cards_db
    db.delete_all()
    # 支持 `@pytest.mark.num_cards(<some number>)`
    # 随机种子
    faker.seed_instance(101)
    m = request.node.get_closest_marker("num_cards")
    if m and len(m.args) > 0:
        num_cards = m.args[0]
        for _ in range(num_cards):
            db.add_card(
                Card(summary=faker.sentence(), owner=faker.first_name())
            )
    return db
```

然后我们需要声明我们的标记：

```python
# pytest.ini
[pytest]
markers =
    smoke: 测试的一部分
    exception: 检查预期的异常
    finish: 所有的 "cards finish" 相关测试
    num_cards: 为 cards_db fixture 预填充卡片的数量
addopts =
    --strict-markers
    -ra
xfail_strict = true
```

现在所有旧的测试即使没有使用标记也会以相同的方式工作，而新的测试如果希望数据库中有初始卡片也能使用相同的 fixture：

```python
import pytest

def test_no_marker(cards_db):
    assert cards_db.count() == 0

@pytest.mark.num_cards
def test_marker_with_no_param(cards_db):
    assert cards_db.count() == 0

@pytest.mark.num_cards(3)
def test_three_cards(cards_db):
    assert cards_db.count() == 3
    # 仅仅为了好玩，让我们看看 Faker 为我们生成的卡片
    print()
    for c in cards_db.list_cards():
        print(c)

@pytest.mark.num_cards(10)
def test_ten_cards(cards_db):
    assert cards_db.count() == 10
```

让我们运行这些测试确保一切正常，并查看这个假数据的一个例子：

```
pytest -v -s test_num_cards.py
```

### 列出标记

要列出所有可用的标记，包括描述和参数，运行 `pytest --markers`：

```
pytest --markers
```

这是一个非常方便的功能，让我们可以快速查找标记，并且是一个很好的理由，在我们自己的标记中包含有用的描述。

### 命令行标志

* `--strict-markers` 标志告诉 pytest 如果看到我们使用未声明的标记则抛出错误。默认情况下，这是一个警告。
* `-ra` 标志告诉 pytest 列出任何未通过测试的原因。这包括失败、错误、跳过、xfail 和 xpass。
* 设置 `xfail_strict = true` 会使任何通过的 xfail 测试变为失败的测试，因为我们对系统行为的理解有误。如果你希望 xfail 通过的测试标记为 XPASS，则可以省略这一设置。
* `-m` 标志可以使用逻辑运算符 and、or、not 以及括号。
* pytest `--markers` 列出所有可用的标记。

## 第7章. 测试策略

### 确定测试范围

#### 范围问题

在确定我们需要做多少测试时，还有许多其他问题需要考虑：

* 安全性是否重要？如果你保存了任何机密信息，这一点尤为重要。
* 性能如何？交互操作需要快速吗？具体多快？
* 负载能力怎样？你能处理大量用户发出的大量请求吗？你是否预计将来会有这方面的需求？如果有，你应该对此进行测试。
* 输入验证是否到位？对于任何接受用户输入的系统来说，我们应在执行操作之前验证数据的有效性。

#### 测试案例范围

这里有一个合理的起点：

* 测试用户可见功能的行为。
* 推迟安全性、性能和负载测试，直到现有设计发生变化。目前的设计是将数据库存储在用户的主目录中。如果将来移动到共享位置供多位用户使用，这些问题就变得非常重要。
* 输入验证在 Cards 作为单用户应用时相对不那么重要。然而，我也不希望在使用应用程序时出现堆栈跟踪错误，所以我们至少应该在命令行界面级别测试一些奇怪的输入。

### 考虑软件架构

#### 测试策略

所有这些都会以多种方式影响我们的测试策略：

* 我们应该在哪一层级上进行测试？顶层用户界面？更低的层级？子系统？所有层级？
* 在不同层级进行测试有多容易？UI 测试通常是难度最大的，但也更容易与客户特性联系起来。针对单个函数的测试可能更容易实施，但更难与客户需求关联。
* 谁负责不同层级及其测试？如果你提供的是一个子系统，你只对该子系统负责吗？是否有人专门负责系统测试？如果是这样，选择就很简单：测试你自己的子系统。不过，最好至少参与了解在系统层面正在测试的内容。

#### 测试案例策略

`cli.py` 和 `db.py` 都尽可能地轻薄，原因有几个：

* 通过 API 进行测试可以测试大部分系统和逻辑。
* 第三方依赖被隔离在一个单独的文件中。

这与测试策略有何关系？几个方面：

* 因为 CLI 在逻辑上很薄，我们可以通过 API 测试几乎所有内容。
* 只需测试 CLI 是否正确调用了 API 入口点即可。
* 由于数据库交互被隔离在 `db.py` 中，如果觉得有必要，我们可以在这一层添加子系统测试。

以下是 Cards 的可行测试策略：

* 测试用户可以访问的功能——即 CLI 中可见的功能。
* 通过 API 而不是 CLI 来测试这些功能。
* 测试 CLI 是否足够确保它正确连接到 API。

### 评估待测试的功能

#### 优先级划分

我通常根据以下因素来优先排序待测试的功能：

* 最近变化 — 新功能、新代码区域、最近修复、重构或以其他方式修改的新功能。
* 核心 — 你的产品的独特卖点（USPs）。必须持续运作才能使产品有用的基本功能。
* 风险 — 应用程序中风险较大的区域，例如对客户很重要的区域，但开发团队不经常使用，或使用你不完全信任的第三方代码的部分。
* 问题频发 — 经常出错或经常收到缺陷报告的功能。
* 专业知识 — 仅被一小部分人理解的特性或算法。

#### 测试案例功能

因为我们把 Cards 项目视为需要测试的遗留系统，这些标准中有些比其他的更有帮助：

* 核心
  * `add`, `count`, `delete`, `finish`, `list`, `start`, 和 `update` 看起来都是核心功能。
  * `config` 和 `version` 看起来没那么重要。
* 风险
  * 第三方包是用于 CLI 的 Typer 和用于数据库的 TinyDB。围绕我们使用这些组件的一些重点测试将是明智的。我们将通过测试 CLI 来测试 Typer 的使用。我们将通过所有其他测试来测试 TinyDB 的使用，并且由于 `db.py` 隔离了我们与 TinyDB 的交互，我们可以在必要时为此层创建测试。

即使这种快速的功能分析也帮助我们形成了策略：

* 彻底测试核心功能。
* 至少为非核心功能编写一个测试案例。
* 隔离测试 CLI。

### 创建测试案例

#### 测试标准

对于生成初始的一组测试案例，以下标准将会很有帮助：

* 从一个非平凡的、“理想路径”测试案例开始。
* 然后寻找代表
  * 有趣的数据集，
  * 有趣的起始状态，
  * 有趣的结束状态，或
  * 所有可能的错误状态的测试案例。

什么是有趣的起始状态？我认为：

* 空数据库
* 一项任务
* 多项任务

#### 测试案例

因此，对于 `count` 我们有了这些测试案例：

* 从空数据库计数
* 有一项任务时计数
* 有多项任务时计数

以下是 `add` 的测试案例：

* 向空数据库添加任务，具有摘要
* 向非空数据库添加任务，具有摘要
* 添加带有摘要和所有者的任务
* 添加缺少摘要的任务
* 添加重复的任务

以下是 `delete` 的测试案例：

* 从有多项任务的数据库中删除一项
* 删除最后一项任务
* 删除不存在的任务

这给我们带来了这些新的测试案例：

* 从 “todo”, “in prog”, 和 “done” 状态开始任务
* 开始一个无效的 ID
* 从 “todo”, “in prog”, 和 “done” 状态完成任务
* 完成一个无效的 ID

以下是剩余功能的测试案例：

* 更新任务的所有者
* 更新任务的摘要
* 同时更新任务的所有者和摘要
* 更新不存在的任务
* 从空数据库列出任务
* 从非空数据库列出任务
* `config` 返回正确的数据库路径
* `version` 返回正确的版本

### 编写测试策略

即使有了这个对我们测试策略的快速总结，一旦深入测试时也很容易忘记细节。因此，我喜欢把测试策略写下来以便稍后参考。如果是在团队中工作，即使只有两个人，写下策略尤其重要。

这里是当前 Cards 的测试策略：

* 测试可以通过最终用户界面，即 CLI 访问的行为和功能。
* 尽可能通过 API 来测试这些功能。
* 测试 CLI 足够以确认所有功能都正确调用了 API。
* 彻底测试以下核心功能：`add`, `count`, `delete`, `finish`, `list`, `start`, 和 `update`。
* 包含对 `config` 和 `version` 的简略测试。
* 通过针对 `db.py` 的子系统测试来测试我们使用 TinyDB 的情况。

### 常见错误

在编写自动化测试时，这些是常见的错误：

* 只编写理想路径测试案例
* 花费太多时间思考可能出现的问题
* 忽略基于系统或组件状态的行为变化

## 第8章. 配置文件

### 了解 pytest 配置文件

#### 配置文件

让我们来看看对 pytest 重要的非测试文件：

* `pytest.ini`：这是主要的 pytest 配置文件，允许您更改 pytest 的默认行为。其位置也定义了 pytest 的根目录 (`rootdir`)。
* `conftest.py`：这个文件包含 fixture 和钩子函数。它可以在 `rootdir` 或任何子目录中存在。
* `__init__.py`：当放置在测试子目录中时，此文件允许您在多个测试目录中有相同的名字的测试文件。
* `tox.ini`、`pyproject.toml` 和 `setup.cfg`：这些文件可以替代 `pytest.ini`。如果您已经在项目中有一个这样的文件，您可以使用它来保存 pytest 设置。
  * `tox.ini` 是由 tox，一个命令行自动化测试工具所使用的。
  * `pyproject.toml` 用于打包 Python 项目，并且可以用来保存各种工具的设置，包括 pytest。
  * `setup.cfg` 同样用于打包，并且可以用来保存 pytest 设置。

#### 项目目录结构

```shell
cards_proj
├── ... 顶级项目文件、src 目录、文档等 ...
├── pytest.ini
└── tests
    ├── conftest.py
    ├── api
    │ ├── __init__.py
    │ ├── conftest.py
    │ └── ... api 的测试文件 ...
    └── cli
    ├── __init__.py
    ├── conftest.py
    └── ... cli 的测试文件 ...
```

### 在 `pytest.ini` 中保存设置和标志

* `[pytest]`
  * 文件以 `[pytest]` 开始，表示 pytest 设置的开始。包括 `[pytest]` 可以让 pytest 解析器同样地处理 `pytest.ini` 和 `tox.ini` 文件。之后是各个设置，每个设置占据一行（或多行），形式为 `<setting> = <value>`。
* `addopts = --strict-markers --strict-config -ra`
  * `addopts` 设置使我们能够列出在这个项目中始终想要运行的 pytest 标志。
  * `--strict-markers` 告诉 pytest 对测试代码中遇到的任何未注册的标记抛出错误，而不是警告。打开此选项以避免标记名称拼写错误。
  * `--strict-config` 告诉 pytest 对解析配置文件的任何困难抛出错误。默认情况下是警告。打开此选项以避免配置文件中的拼写错误被忽略。
  * `-ra` 告诉 pytest 在测试运行结束时显示额外的测试总结信息。默认情况下仅在测试失败和错误时显示额外信息。`-ra` 的 `a` 部分告诉 pytest 显示除了通过测试之外的所有信息。这增加了跳过、期望失败和期望通过的测试。
* `testpaths = tests`
  * `testpaths` 设置告诉 pytest 如果您没有在命令行给出文件或目录名称，那么在哪里查找测试。设置 `testpaths` 为 `tests` 告诉 pytest 查找 `tests` 目录。
  * 初看之下，将 `testpaths` 设置为 `tests` 可能显得多余，因为 pytest 无论如何都会在那里查找，而且我们的 `src` 或 `docs` 目录中并没有 `test_` 文件。但是指定 `testpaths` 目录可以节省一点启动时间，特别是如果我们的 `docs`、`src` 或其他目录相当大的时候。
* `markers = ...`
  * `markers` 设置用于声明标记。每个列出的标记都是形式 `<marker_name>: <description>`。

```INI
# pytest.ini
[pytest]
addopts =
    --strict-markers
    --strict-config
    -ra
testpaths = tests
markers =
    smoke: 测试子集
    exception: 检查预期的异常
```

### 使用 `tox.ini`、`pyproject.toml` 或 `setup.cfg` 替代 `pytest.ini`

#### `tox.ini`

它也可以包含一个 `[pytest]` 部分。由于它也是一个 `.ini` 文件，下面的 `tox.ini` 示例几乎与前面所示的 `pytest.ini` 示例相同。唯一的区别是也会有一个 `[tox]` 部分。

```INI
# tox.ini
[tox]
; tox 特定设置

[pytest]
addopts =
    --strict-markers
    --strict-config
    -ra
testpaths = tests
markers =
    smoke: 测试子集
    exception: 检查预期的异常
```

#### `pyproject.toml`

不是 `[pytest]`，而是以 `[tool.pytest.ini_options]` 开始部分。设置值需要用引号括起来，列表形式的设置值需要放在方括号内。

```INI
# pyproject.toml
[tool.pytest.ini_options]
addopts = [
    "--strict-markers",
    "--strict-config",
    "-ra"
]
testpaths = "tests"
markers = [
    "smoke: 测试子集",
    "exception: 检查预期的异常"
]
```

#### `setup.cfg`

与 `pytest.ini` 之间的唯一明显区别是部分指定符为 `[tool:pytest]`。然而，pytest 文档警告称 `.cfg` 解析器不同于 `.ini` 文件解析器，这种差异可能会导致难以追踪的问题。

```INI
# setup.cfg
[tool:pytest]
addopts =
    --strict-markers
    --strict-config
    -ra
testpaths = tests
markers =
    smoke: 测试子集
    exception: 检查预期的异常
```

### 确定根目录和配置文件

#### 根目录搜索过程

如果您已经传递了一个测试目录，pytest 从那里开始查找。
如果您传递了多个文件或目录，pytest 从它们的共同祖先目录开始。
如果您没有传递文件或目录，它从当前目录开始。

如果 pytest 在起始目录找到配置文件，那就是根目录。
如果没有找到，pytest 会沿着目录树向上查找直到找到含有 pytest 部分的配置文件。

一旦 pytest 找到了配置文件，它会标记找到该文件的目录作为根目录，也就是 `rootdir`。这个根目录也是测试节点 ID 的相对根目录。它还会告诉您在哪里找到了配置文件。

#### 配置文件

甚至在开始寻找要运行的测试文件之前，pytest 就会读取配置文件 —— `pytest.ini` 文件或包含 pytest 部分的 `tox.ini`、`setup.cfg` 或 `pyproject.toml` 文件。

即使您不需要任何配置设置，还是建议在项目的顶部放置一个空的 `pytest.ini` 文件。如果您没有任何配置文件，pytest 将一直搜索到文件系统的根目录。最好的情况，这只会造成轻微的延迟，因为 pytest 正在查找；最坏的情况，它会在途中找到一个与您的项目无关的配置文件。

一旦定位了配置文件，pytest 将在测试运行开始时打印出它正在使用的 `rootdir` 和配置文件。

### 通过 `conftest.py` 共享本地 fixture 和钩子函数

`conftest.py` 文件用于存储 fixture 和钩子函数。您可以在项目中拥有任意数量的 `conftest.py` 文件，甚至每个测试子目录一个。在 `conftest.py` 文件中定义的任何内容都适用于该目录及其所有子目录中的测试。

然而，最好尽量坚持使用一个 `conftest.py` 文件，这样您可以轻松地找到 fixture 定义。尽管您总是可以通过使用 `pytest --fixtures -v` 找到 fixture 被定义的位置，但如果知道它要么在您正在查看的测试文件中，要么在一个文件中，即 `conftest.py` 文件，那会更容易。

### 避免测试文件名冲突

`__init__.py` 文件影响 pytest 的方式只有一个：它允许您有重复的测试文件名。
如果您在每个测试子目录中都有 `__init__.py` 文件，您可以在多个目录中拥有相同的测试文件名。这就是使用 `__init__.py` 文件的唯一原因。

```shell
$ tree tests_with_init
tests_with_init
├── api
│ ├── __init__.py
│ └── test_add.py
├── cli
│ ├── __init__.py
│ └── test_add.py
└── pytest.ini
```

为了避免遇到重复文件名带来的混淆错误，养成在子目录中放置 `__init__.py` 文件的习惯是个不错的选择。

## 第9章. 代码覆盖率

测量代码覆盖率的工具会在运行测试套件时监视您的代码，并记录哪些行被执行了，哪些行未被执行。这种度量称为行覆盖率，它是通过将已执行的行数除以总行数计算得出的。代码覆盖率工具还可以告诉您控制语句中的所有路径是否都被遍历，这种度量称为分支覆盖率。

### 使用 `coverage.py` 与 `pytest-cov`

`coverage.py` 和 `pytest-cov` 都是第三方包，在使用前需要安装：

```shell
pip install coverage
pip install pytest-cov
```

使用 `pytest-cov` 运行覆盖率：

* 使用 `pytest --cov=<源代码目录> <测试路径>` 来运行并生成简单的报告
* 使用 `pytest --cov=<源代码目录> --cov-report=term-missing <测试路径>` 来显示哪些行未被执行

```shell
pytest --cov=cards ch7
pytest --cov=cards --cov-report=term-missing ch7
```

单独使用 `coverage.py` 运行覆盖率：

* 使用 `coverage run --source=<源代码目录> -m pytest <测试路径>` 来使用覆盖率运行测试套件
* 使用 `coverage report` 来显示简单的终端报告
* 使用 `coverage report --show-missing` 来显示哪些行未被执行

```shell
coverage run --source=cards -m pytest ch7
coverage report
coverage report --show-missing
```

### 使用配置文件

`coverage.py` 的配置文件设置源代码目录。

```INI
# .coveragerc
[paths]
source =
    cards_proj/src/cards
    */site-packages/cards
```

### 生成 HTML 报告

使用 `pytest-cov` 生成 HTML 报告：

* 使用 `pytest --cov=<源代码目录> --cov-report=html <测试路径>` 来生成 HTML 报告

```shell
pytest --cov=cards --cov-report=html ch7
```

使用 `coverage.py` 生成 HTML 报告：

* 使用 `coverage html` 来生成 HTML 报告

```shell
pytest --cov=cards ch7
coverage html
```

### 排除代码不计入覆盖率

这些类型的代码块经常通过简单的 `pragma` 语句从测试中排除：

```python
if __name__ == '__main__': # pragma: no cover
    main()
```

### 在测试上运行覆盖率

除了使用覆盖率来确定测试套件是否覆盖了应用程序代码的每一行，我们还可以将测试目录添加到覆盖率报告中：

```shell
pytest --cov=cards --cov=ch7 ch7
```

`--cov=cards` 命令告诉覆盖率工具监视 `cards` 包。`--cov=ch7` 命令告诉覆盖率工具监视 `ch7` 目录，其中存放着我们的测试文件。

### 在目录上运行覆盖率

在 `ch9/some_code` 目录中，有若干源代码模块和一个测试模块：

```shell
$ tree ch9/some_code
ch9/some_code
├── bar_module.py
├── foo_module.py
└── test_some_code.py
```

为了演示如何指向一个路径而不是包，我们可以留在顶层代码目录中，并从那里运行测试：

```shell
pytest --cov=ch9/some_code ch9/some_code/test_some_code.py
```

### 在单个文件上运行覆盖率

在这种情况下，即使没有导入任何东西，我们也把文件当作一个包处理，并使用 `--cov=single_file`（不含 `.py` 扩展名）：

```shell
pytest --cov=single_file single_file.py
```

## 第十章 模拟

### 隔离命令行界面

通过这个 `cards` 命名空间，`cli.py` 访问：

* `cards.__version__`（一个字符串）
* `cards.CardDB`（一个代表主要API方法的类）
* `cards.InvalidCardID`（一个异常）
* `cards.Card`（CLI和API之间主要的数据类型）

大多数API访问是通过一个上下文管理器创建 `cards.CardsDB` 对象：

```python
# cards_proj/src/cards/cli.py
@contextmanager
def cards_db():
    db_path = get_path()
    db = cards.CardsDB(db_path)
    yield db
    db.close()
```

大多数函数都通过这个对象工作。例如，`start` 命令通过 `db`（一个 `CardsDB` 实例）访问 `db.start()`：

```python
# cards_proj/src/cards/cli.py
@app.command()
def start(card_id: int):
    """将卡片状态设置为'in prog'。"""
    with cards_db() as db:
        try:
            db.start(card_id)
        except cards.InvalidCardId:
            print(f"错误: 无效的卡片ID {card_id}")
```

`add` 和 `update` 也使用了我们之前玩过的 `cards.Card` 数据结构：

```python
# cards_proj/src/cards/cli.py
db.add_card(cards.Card(summary, owner, state="todo"))
```

而 `version` 命令则查找 `cards.__version__`：

```python
# cards_proj/src/cards/cli.py
@app.command()
def version():
    """返回卡片应用的版本号。"""
    print(cards.__version__)
```

### 使用Typer进行测试

这里有一个调用 `version` 函数的例子：

```python
# test_typer_testing.py
from typer.testing import CliRunner
from cards.cli import app

runner = CliRunner()

def test_typer_runner():
    result = runner.invoke(app, ["version"])
    print()
    print(f"版本: {result.stdout}")
    result = runner.invoke(app, ["list", "-o", "brian"])
    print(f"列表:\n{result.stdout}")
```

在这个例子测试中：

* 要运行 `cards version`，我们执行 `runner.invoke(app, ["version"])`。
* 要运行 `cards list -o brian`，我们执行 `runner.invoke(app, ["list", "-o", "brian"])`。

让我们简化一下：

```python
# test_typer_testing.py
import shlex

def cards_cli(command_string):
    command_list = shlex.split(command_string)
    result = runner.invoke(app, command_list)
    output = result.stdout.rstrip()
    return output
    
def test_cards_cli():
    result = cards_cli("version")
    print()
    print(f"版本: {result}")
    result = cards_cli("list -o brian")
    print(f"列表:\n{result}")
```

### 模拟属性

我们将主要使用 `patch.object` 的上下文管理器形式。

```python
# test_mock.py
from unittest import mock
import cards
import pytest
from cards.cli import app
from typer.testing import CliRunner


runner = CliRunner()
def test_mock_version():
    with mock.patch.object(cards, "__version__", "1.2.3"):
        result = runner.invoke(app, ["version"])
        assert result.stdout.rstrip() == "1.2.3"
```

在这种情况下，`cards` 的 `__version__` 属性在 with 块的持续时间内被替换为 "1.2.3"。然后我们使用 `invoke` 来调用我们的应用并传入 `"version"` 命令。

模拟替换了系统的一部分，即模拟对象。使用模拟对象，我们可以做很多事情，比如设置属性值、可调用对象的返回值，甚至可以查看可调用对象是如何被调用的。

### 模拟类和方法

这是一个探索性的测试函数，用于确认模拟是否设置正确。这次，我们希望让 `CardsDB` 成为一个模拟对象。

```python
# test_mock.py
def test_mock_CardsDB():
    with mock.patch.object(cards, "CardsDB") as MockCardsDB:
        print()
        print(f" 类:{MockCardsDB}")
        print(f"返回值:{MockCardsDB.return_value}")
        with cards.cli.cards_db() as db:
            print(f" 对象:{db}")
```

对于 `CardsDB()` 返回的第二个模拟对象，我们可以在那里改变 `path` 属性。

```python
# test_mock.py
def test_mock_path():
    with mock.patch.object(cards, "CardsDB") as MockCardsDB:
        MockCardsDB.return_value.path.return_value = "/foo/"
        with cards.cli.cards_db() as db:
            print()
            print(f"{db.path=}")
            print(f"{db.path()=}")
```

在真正开始测试CLI之前，我们需要做的最后一件事就是将数据库的模拟推入一个fixture中——因为我们将需要它在很多测试方法中：

```python
# test_mock.py
@pytest.fixture()
def mock_cardsdb():
    with mock.patch.object(cards, "CardsDB", autospec=True) as CardsDB:
        yield CardsDB.return_value


def test_config(mock_cardsdb):
    mock_cardsdb.path.return_value = "/foo/"
    result = runner.invoke(app, ["config"])
    assert result.stdout.rstrip() == "/foo/"
```

### 通过使用 `autospec` 保持模拟与实现同步

当被模拟的接口发生变化而你的测试代码中的模拟没有变化时，就会出现模拟漂移。

这种形式的模拟漂移可以通过在创建模拟时添加 `autospec=True` 来解决，就像我们对 `CardsDB` 所做的那样。如果没有 `autospec=True`，模拟将会允许你调用任何函数并传入任何参数，即使这不符合被模拟的真实对象的行为。

```python
# test_mock.py
def test_bad_mock():
    with mock.patch.object(cards, "CardsDB", autospec=True) as CardsDB:
        db = CardsDB("/some/path")
        db.path() # 正确
        db.path(35) # 无效参数
        db.not_valid() # 无效函数
```

### 确保函数被正确调用

有一些命令，我们无法通过检查输出来测试其行为，因为这些命令不会产生输出。我们将使用模拟来确保CLI正确地调用了API方法。

```python
# test_cli.py
def test_add_with_owner(mock_cardsdb):
    cards_cli("add some task -o brian")
    expected = cards.Card("some task", owner="brian", state="todo")
    mock_cardsdb.add_card.assert_called_with(expected)
```

### 创建错误条件

现在让我们检查一下卡片CLI是否能正确处理错误条件。

为了测试CLI处理错误条件的能力，我们可以假设 `delete_card` 会引发一个异常，通过将异常赋给模拟对象的 `side_effect` 属性：

```python
# cards_proj/src/cards/cli.py
@app.command()
def delete(card_id: int):
    """删除具有给定ID的数据库中的卡片。"""
    with cards_db() as db:
        try:
            db.delete_card(card_id)
        except cards.InvalidCardId:
            print(f"错误: 无效的卡片ID {card_id}")

# test_cli.py
def test_delete_invalid(mock_cardsdb):
    mock_cardsdb.delete_card.side_effect = cards.api.InvalidCardId
    out = cards_cli("delete 25")
    assert "错误: 无效的卡片ID 25" in out
```

### 通过多层测试避免模拟

```python
# test_cli_alt.py
def test_add_with_owner(cards_db):
    """
    添加的卡片出现在列表中，内容如预期所示。
    """
    cards_cli("add some task -o brian")
    expected = cards.Card("some task", owner="brian", state="todo")
    all_cards = cards_db.list_cards()
    assert len(all_cards) == 1
    assert all_cards[0] == expected

# test_cli.py
def test_add_with_owner(mock_cardsdb):
    cards_cli("add some task -o brian")
    expected = cards.Card("some task", owner="brian", state="todo")
    mock_cardsdb.add_card.assert_called_with(expected)
```

### 使用插件辅助模拟

使用 `pytest-mock` 的一个好处是，该fixture会在结束后自动清理，因此你不必像我们的示例中那样使用with块。

还有一些特定用途的模拟库，在其关注点与你要测试的内容相符时应该考虑使用：

* 对于模拟数据库访问，尝试使用 pytest-postgresql, pytest-mongo, pytest-mysql, 和 pytest-dynamodb。
* 对于模拟HTTP服务器，尝试使用 pytest-httpserver。
* 对于模拟请求，尝试使用 responses 和 betamax。
* 还有更多工具，如 pytest-rabbitmq, pytest-solr, pytest-elasticsearch, 和 pytest-redis。

## 第十一章 tox 和持续集成

在多人协作开发同一个代码库的情况下，持续集成（CI）可以极大地提升生产力。CI 是指定期（通常是每天多次）将所有开发者的代码变更合并到共享仓库中的实践。

大多数CI工具都在服务器上运行（GitHub Actions 是一个例子）。tox 是一个自动化工具，它的作用类似于CI工具，但可以在本地运行，也可以与其他CI工具配合在服务器上使用。

### 什么是持续集成？

CI 工具可以自行构建和运行测试，通常是由合并请求触发的。由于构建和测试阶段都是自动化的，开发者可以更频繁地进行集成，甚至每天多次。这种频率使得分支间的代码变更更小，减少了合并冲突的可能性。结合Git等工具提供的自动化合并功能，我们实现了“持续”这一持续集成过程的一部分。

CI 工具传统上会自动化构建和测试的过程。最终主代码分支的实际合并有时也可以由CI系统处理。然而，更常见的是，工具在测试完成后停止，然后软件团队可以继续进行代码审查，并手动在版本控制系统中点击“合并”按钮。

### 介绍 tox

tox 是一个命令行工具，允许你在多个环境中运行完整的测试套件。学习CI时，tox 是一个很好的起点。虽然它严格意义上不是一个CI系统，但它的工作方式很类似，并且可以在本地运行。

tox 不仅限于Python版本的测试，还可以用于不同的依赖配置和不同操作系统的配置。

简而言之，tox 的工作模型如下：

tox 使用 `setup.py` 或 `pyproject.toml` 中的项目信息来创建你的包的可安装发行版。它会在 `tox.ini` 文件中查找环境列表，然后为每个环境：

1. 在 `.tox` 目录中创建虚拟环境，
2. 使用 `pip` 安装一些依赖，
3. 构建你的包，
4. 使用 `pip` 安装你的包，
5. 运行你的测试。

尽管许多项目都在使用 tox，但也存在类似的替代工具，如 `nox` 和 `invoke`。

### 设置 tox

下面是简化后的代码布局：

```
cards_proj
├── LICENSE
├── README.md
├── pyproject.toml
├── pytest.ini
├── src
│ └── cards
│ └── ...
├── tests
│ ├── api
│ │ └── ...
│ └── cli
│ └── ...
└── tox.ini
```

让我们看一下 Cards 项目的 `tox.ini` 文件的基本配置：

```INI
# tox.ini
[tox]
envlist = py310
isolated_build = True
[testenv]
deps =
    pytest
    faker
commands = pytest
```

在 `[tox]` 下，`envlist = py310` 告诉 tox 使用 Python 版本 3.10 运行测试。

对于所有 `pyproject.toml` 配置的包，需要设置 `isolated_build = True`。对于使用 `setuptools` 库配置的 `setup.py` 项目，可以省略此行。

在 `[testenv]` 下，`deps` 部分列出了 `pytest` 和 `faker`。这告诉 tox 需要安装这两个工具以进行测试。

最后，`commands` 设置告诉 tox 在每个环境中运行 pytest。

### 运行 tox

在运行 tox 之前，需要先安装它：

`$ pip install tox`

然后只需运行 tox 即可：

`$ tox`

### 测试多个 Python 版本

让我们扩展 `tox.ini` 中的 `envlist` 来添加更多的 Python 版本：

```INI
# tox_multiple_pythons.ini
[tox]
envlist = py37, py38, py39, py310
isolated_build = True
skip_missing_interpreters = True
```

设置 `skip_missing_interpreters = True` 后，tox 将在任何可用的 Python 版本上运行测试，但会跳过找不到的版本而不导致失败。

我们使用 `tox -c tox_multiple_pythons.ini` 来查看同一项目的不同 `tox.ini` 设置。

`$ tox -c tox_multiple_pythons.ini`

### 并行运行 tox 环境

也可以使用 `-p` 标志并行运行它们：

`$ tox -c tox_multiple_pythons.ini -p`

### 向 tox 添加覆盖率报告

通过在 `tox.ini` 文件中进行一些修改，tox 可以为测试运行添加覆盖率报告。

```INI
# tox_coverage.ini
[testenv]
deps =
    pytest
    faker
    pytest-cov
commands = pytest --cov=cards
```

当使用覆盖率与 tox 一起时，设置 `.coveragerc` 文件也很有用，这样覆盖率工具就可以知道哪些源路径应该被视为相同：

```python
# .coveragerc
[paths]
source =
    src
    .tox/*/site-packages
```

将覆盖率源设置为包括 `src` 和 `.tox/*/site-packages` 是一种简写方式，使之前的代码能够工作，并生成以下输出：

`$ tox -c tox_coverage.ini -e py310`

### 指定最低覆盖率级别

当从 tox 运行覆盖率时，设置一个基线覆盖率百分比以标记覆盖率下降也很有用。这是通过 `--cov-fail-under` 标志完成的：

```python
# tox_coverage_min.ini
[testenv]
deps =
    pytest
    faker
    pytest-cov
commands = pytest --cov=cards --cov=tests --cov-fail-under=100
```

这将在输出中添加一行：

`$ tox -c tox_coverage_min.ini -e py310`

我们还使用了一些其他标志。在 `tox.ini` 中，我们在 pytest 调用中添加了 `--cov=tests` 以确保所有的测试都被运行。在 tox 命令行中，我们使用了 `-e py310`。`-e` 标志允许我们运行特定的 tox 环境。

### 通过 tox 传递 pytest 参数

我们还可以通过进一步修改允许参数传递给 pytest 来聚焦于单个测试。

修改只需在 pytest 命令中添加 `{posargs}`：

```INI
# tox_posargs.ini
[testenv]
deps =
    pytest
    faker
    pytest-cov
commands =
    pytest --cov=cards --cov=tests --cov-fail-under=100 {posargs}
```

然后，要向 pytest 传递参数，请在 tox 参数和 pytest 参数之间添加 `--`：

`$ tox -c tox_posargs.ini -e py310 -- -k test_version --no-cov`

### 使用 GitHub Actions 运行 tox

要将 Actions 添加到仓库，只需要在项目顶层的 `.github/workflows/` 目录中添加一个工作流 `.yml` 文件即可。

```YML
# cards_proj/.github/workflows/main.yml
name: CI
on: [push, pull_request]
jobs:
    build:
        runs-on: ubuntu-latest
        strategy:
            matrix:
                python: ["3.7", "3.8", "3.9", "3.10"]
        steps:
            - uses: actions/checkout@v2
            - name: Setup Python
            uses: actions/setup-python@v2
            with:
                python-version: ${{ matrix.python }}
            - name: Install Tox and any other packages
            run: pip install tox
            - name: Run Tox
            run: tox -e py
```

现在让我们逐一解释这个文件所指定的内容：

* `name` 可以是任意名称。它会出现在稍后我们将看到的 GitHub Actions 用户界面中。
* `on: [push, pull_request]` 告诉 Actions 每当我们推送代码到仓库或创建拉取请求时运行测试。如果我们推送代码更改，我们的测试将会运行；如果任何人创建拉取请求，测试也会运行。在拉取请求界面上可以看到测试运行的结果。所有 Actions 运行结果都可以在 GitHub 界面的 Actions 标签页中看到。
* `runs-on`: `ubuntu-latest` 指定了运行测试的操作系统。这里我们只在 Linux 上运行，但也可以选择其他操作系统。
* `matrix`: python: `["3.7", "3.8", "3.9", "3.10"]` 指定了要运行的 Python 版本。
* `steps` 是一系列步骤的列表。每个步骤的名称可以是任意的，而且是可选的。
* `uses: actions/checkout@v2` 是 GitHub Actions 的一个工具，用于检出我们的仓库，以便后续的工作流程可以访问它。
* `uses: actions/setup-python@v2` 是 GitHub Actions 的一个工具，用于在构建环境中配置和安装 Python。
* `with: python-version: ${{ matrix.python }}` 表示为 `matrix: python` 列表中的每个 Python 版本创建环境。
* `run: pip install tox` 安装 tox。
* `run: tox -e py` 运行 tox。`-e py` 看起来有点奇怪，因为我们没有指定 `py` 环境。然而，这可以正常工作，以选择 `tox.ini` 中指定的正确 Python 版本。

## 第十二章 测试脚本和应用程序

为了明确术语，在本章中适用以下定义：

* **脚本**：是指包含 Python 代码的单个文件，旨在直接通过 Python 运行，例如 `python my_script.py`。
* **可导入脚本**：是指导入时不执行任何代码的脚本。代码只有在直接运行时才会被执行。
* **应用程序**：是指包含外部依赖项的包或脚本，这些依赖项定义在一个 `requirements.txt` 文件中。

在测试脚本和应用程序时，经常会出现以下问题：

* 如何从测试中运行脚本？
* 如何捕获脚本的输出？
* 我想在我的测试中导入我的源模块或包。如果测试和代码位于不同的目录中，我该如何实现这一点？
* 如果没有包需要构建，如何使用 tox？
* 如何让 tox 从 `requirements.txt` 文件中加载外部依赖项？

### 使用虚拟环境

首先，我们可以创建一个虚拟环境，并安装必要的测试工具：

```shell
$ cd /path/to/code/ch12
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $ pip install -U pip
(venv) $ pip install pytest
(venv) $ pip install tox
```

### 测试简单的 Python 脚本

对于 `hello.py` 脚本，我们的挑战在于（1）如何从测试中运行它，以及（2）如何捕获输出。Python 标准库中的 `subprocess` 模块有一个 `run()` 方法可以很好地解决这两个问题：

```python
# script/hello.py
print("Hello, World!")


# script/test_hello.py
from subprocess import run

def test_hello():
    result = run(["python", "hello.py"], capture_output=True, text=True)
    output = result.stdout
    assert output == "Hello, World!\n"
```

该测试启动了一个子进程，捕获输出，并将其与 `"Hello, World!\n"` 进行比较，包括 `print()` 自动添加的换行符。

```shell
pytest -v test_hello.py
```

如果我们设置一个标准的 `tox.ini` 文件，它可能不会正常工作。不过我们还是尝试一下：

```INI
# script/tox.ini
[tox]
envlist = py39, py310
skipsdist = true
[testenv]
deps = pytest
commands = pytest
[pytest]
```

运行这个配置会暴露出问题所在：

```shell
tox
```

我们使用 `pytest` 和 `tox` 测试了脚本，并使用 `subprocess.run()` 启动脚本以捕获输出。

### 测试可导入的 Python 脚本

我们可以稍微修改脚本代码使其可导入，并允许测试和代码位于不同的目录中。

```python
# script_importable/hello.py
def main():
    print("Hello, World!")

if __name__ == "__main__":
    main()
```

现在我们可以像测试任何其他函数一样测试 `main()`。在修改后的测试中，我们使用了 `capsys`：

```python
# script_importable/test_hello.py
import hello
def test_main(capsys):
    hello.main()
    output = capsys.readouterr().out
    assert output == "Hello, World!\n"
```

我们现在可以分别测试这些函数。

```python
# script_funcs/hello.py
def full_output():
    return "Hello, World!"

def main():
    print(full_output())

if __name__ == "__main__":
    main()
```

我们将输出内容放入 `full_output()` 函数中，而实际打印则放在 `main()` 函数中。现在我们可以分别测试它们：

```python
# script_funcs/test_hello.py
import hello

def test_full_output():
    assert hello.full_output() == "Hello, World!"

def test_main(capsys):
    hello.main()
    output = capsys.readouterr().out
    assert output == "Hello, World!\n"
```

### 将代码分离到 `src` 和 `tests` 目录

我们决定将脚本移动到 `src` 目录中，而测试则放在 `tests` 目录中，结构如下：

```
script_src
├── src
│   └── hello.py
├── tests
│   └── test_hello.py
└── pytest.ini
```

如果不做其他改变，`pytest` 会失败。

```shell
cd /path/to/code/script_src
pytest --tb=short -c pytest_bad.ini
```

我们需要做的就是将想要导入的源代码目录添加到 `sys.path` 中。`pytest` 提供了一个选项帮助我们做到这一点，即 `pythonpath`。这个选项是在 `pytest 7` 中引入的。如果你需要使用 `pytest 6.2`，你可以使用 `pytest-srcpaths` 插件来添加这个选项。

首先，我们需要修改 `pytest.ini` 文件以设置 `pythonpath` 为 `src`：

```INI
# script_src/pytest.ini
[pytest]
addopts = -ra
testpaths = tests
pythonpath = src
```

现在 `pytest` 可以正常运行了：

```shell
pytest tests/test_hello.py
```

### 定义 Python 的搜索路径

Python 的搜索路径仅仅是 Python 存储在 `sys.path` 变量中的一系列目录。

```python
# script_src/tests/test_sys_path.py
import sys

def test_sys_path():
    print("sys.path: ")
    for p in sys.path:
        print(p)
```

当我们运行它时，注意搜索路径：

```shell
pytest -s tests/test_sys_path.py
```

### 测试基于 `requirements.txt` 的应用程序

`requirements.txt` 文件中的依赖列表可能只是一个松散的依赖列表，例如：

```INI
# sample_requirements.txt
typer
requests
```

然而，对于应用程序来说，更常见的做法是通过定义已知可以工作的具体版本来“锁定”依赖项：

```INI
# sample_pinned_requirements.txt
typer==0.3.2
requests==2.26.0
```

`requirements.txt` 文件被用来通过 `pip install -r` 重新创建运行环境。其中的 `-r` 告诉 `pip` 读取并安装 `requirements.txt` 文件中的所有内容。

一个合理的流程可能是：

* 以某种方式获取代码。例如，`git clone <项目仓库>`。
* 创建一个虚拟环境：`python3 -m venv venv`。
* 激活虚拟环境。
* 安装依赖项：`pip install -r requirements.txt`。
* 运行应用程序。

我们将使用 Typer 来帮助我们向问候命令添加一个命令行参数。

```INI
# app/requirements.txt
typer==0.3.2
```

请注意，我还锁定了 Typer 的版本为 0.3.2。

```shell
pip install typer==0.3.2
```

或者

```shell
pip install -r requirements.txt
```

我们也需要对代码做一些改动：

```python
# app/src/hello.py
import typer
from typing import Optional

def full_output(name: str):
    return f"Hello, {name}!"

app = typer.Typer()

@app.command()
def main(name: Optional[str] = typer.Argument("World")):
    print(full_output(name))

if __name__ == "__main__":
    app()
```

Typer 使用类型提示来指定传递给命令行应用的选项和参数的类型，包括可选参数。

```shell
$ cd /path/to/code/app/src
$ python hello.py
Hello, World!
$ python hello.py Brian
Hello, Brian!
```

现在我们需要修改测试以确保 `hello.py` 无论有无名字参数都能正常工作：

```python
# app/tests/test_hello.py
import hello
from typer.testing import CliRunner

def test_full_output():
    assert hello.full_output("Foo") == "Hello, Foo!"
    
def test_hello_app_no_name():
    runner = CliRunner()
    result = runner.invoke(hello.app)
    assert result.stdout == "Hello, World!\n"

def test_hello_app_with_name():
    result = runner.invoke(hello.app, ["Brian"])
    assert result.stdout == "Hello, Brian!\n"
```

我们不再直接调用 `main()`，而是使用 Typer 内置的 `CliRunner()` 来测试应用程序。

让我们先用 `pytest` 运行它，然后再用 `tox` 运行：

```shell
cd /path/to/code/app
pytest -v
```

我们通过在 `deps` 设置中添加 `-rrequirements.txt` 来实现这一点：

```INI
# ch12/app/tox.ini
[tox]
envlist = py39, py310
skipsdist = true
[testenv]
deps = pytest
    pytest-srcpaths
    -rrequirements.txt
commands = pytest
[pytest]
addopts = -ra
testpaths = tests
pythonpath = src
```

我们有一个包含外部依赖项的应用程序，这些依赖项列在 `requirements.txt` 文件中。我们使用 `pythonpath` 指定源代码位置，并在 `tox.ini` 中添加了 `-rrequirements.txt` 以安装这些依赖项。

这很简单。我们来试一试：

```shell
tox
```

## 第 13 章. 调试测试失败

Python 包含一个内置的源代码调试器叫做 `pdb`，以及一些标志来方便地使用 `pdb` 进行调试。

### 向 Cards 项目添加新功能

为此，我们需要一个命令行接口（CLI）命令：

```python
# cards_proj/src/cards/cli.py
@app.command("done")
def list_done_cards():
    """
    列出数据库中状态为 'done' 的卡片。
    """
    with cards_db() as db:
        the_cards = db.list_done_cards()
        print_cards_list(the_cards)
```

`list_done_cards()` 方法实际上只需要调用 `list_cards()` 并预填充 `state="done"`：

```python
# cards_proj/src/cards/api.py
def list_done_cards(self):
    """返回状态为 'done' 的卡片。"""
    done_cards = self.list_cards(state="done")
```

首先，`@pytest.mark.num_cards(10)` 让 `Faker` 生成卡片的内容。API 测试如下：

```python
# cards_proj/tests/api/test_list_done.py
import pytest

@pytest.mark.num_cards(10)
def test_list_done(cards_db):
    cards_db.finish(3)
    cards_db.finish(5)
    the_list = cards_db.list_done_cards()
    assert len(the_list) == 2
    for card in the_list:
        assert card.id in (3, 5)
        assert card.state == "done"
```

现在我们添加 CLI 测试：

```python
# cards_proj/tests/cli/test_done.py
import cards

expected = """\
    ID state owner summary
    ────────────────────────────────
    1 done some task
    3 done a third"""

def test_done(cards_db, cards_cli):
    cards_db.add_card(cards.Card("some task", state="done"))
    cards_db.add_card(cards.Card("another"))
    cards_db.add_card(cards.Card("a third", state="done"))
    output = cards_cli("done")
    assert output == expected
```

### 以可编辑模式安装 Cards

让我们创建一个新的虚拟环境：

```shell
$ cd /path/to/code/ch13
$ python3 -m venv venv
$ source venv/bin/activate
(venv) $ pip install -U pip
```

我们可以以可编辑模式安装 Cards，并且一次性安装所有测试工具及其可选依赖项。

`$ pip install -e "./cards_proj/[test]"`

这能起作用是因为所有这些依赖项都在 `pyproject.toml` 文件的可选依赖项部分定义了：

```toml
# cards_proj/pyproject.toml
[project.optional-dependencies]
test = [
    "pytest",
    "faker",
    "tox",
    "coverage",
    "pytest-cov",
]
```

现在我们运行测试。我们将使用 `--tb=no` 来关闭堆栈跟踪信息：

`$ cd /path/to/code/cards_proj`
`$ pytest --tb=no`

### 使用 pytest 标志进行调试

pytest 包含许多命令行标志，对于调试非常有用。我们将使用其中的一些来调试测试失败。

用于选择要运行哪些测试、运行顺序以及何时停止的标志：

* `-lf / --last-failed`: 只运行上次失败的测试
* `-ff / --failed-first`: 按上次失败的顺序运行所有测试
* `-x / --exitfirst`: 在第一个失败后停止测试会话
* `--maxfail=num`: 在达到指定数量的失败后停止测试
* `-nf / --new-first`: 按文件修改时间顺序运行所有测试
* `--sw / --stepwise`: 在第一个失败后停止测试；下次从上一次失败的地方开始运行
* `--sw-skip / --stepwise-skip`: 与 `--sw` 相同，但跳过第一个失败

用于控制 pytest 输出的标志：

* `-v / --verbose`: 显示所有测试名称，无论是通过还是失败
* `--tb=[auto/long/short/line/native/no]`: 控制堆栈跟踪的样式
* `-l / --showlocals`: 在堆栈跟踪旁边显示局部变量

用于启动命令行调试器的标志：

* `--pdb`: 在失败点启动交互式调试会话
* `--trace`: 在运行每个测试时立即启动 pdb 源码调试器
* `--pdbcls`: 使用 pdb 的替代品，如 IPython 的调试器 `--pdbcls=IPython.terminal.debugger:TerminalPdb`

### 重新运行失败的测试

我们将使用 `--lf` 仅重新运行失败的测试，并使用 `--tb=no` 隐藏堆栈跟踪，因为我们还不准备查看它：

`$ pytest --lf --tb=no`

让我们只运行第一次失败的测试，在失败后停止并查看堆栈跟踪：

`$ pytest --lf -x`

为了确保理解问题所在，我们可以使用 `-l/--showlocals` 再次运行相同的测试。我们不需要再次看到完整的堆栈跟踪，所以可以使用 `--tb=short` 缩短它：

`$ pytest --lf -x -l --tb=short`

### 使用 pdb 进行调试

`pdb`，即“Python 调试器”，是 Python 标准库的一部分，因此我们无需额外安装即可使用它。

你可以通过以下几种方式从 pytest 中启动 pdb：

* 在测试代码或应用程序代码中添加一个 `breakpoint()` 调用。当 pytest 运行遇到 `breakpoint()` 函数调用时，它将在该处停止并启动 pdb。
* 使用 `--pdb` 标志。使用 `--pdb`，pytest 将在失败点停止。在这种情况下，将是 `assert len(the_list) == 2` 这一行。
* 使用 `--trace` 标志。使用 `--trace`，pytest 将在每个测试开始时停止。

对于我们的目的，结合 `--lf` 和 `--trace` 是完美的。这个组合将告诉 pytest 重新运行失败的测试并在 test_list_done() 开始之前停止，即在调用 list_done_cards() 之前：

`$ pytest --lf --trace`

以下是 pdb 支持的常用命令。

元命令：
• `h(elp)`: 打印命令列表
• `h(elp) command`: 打印特定命令的帮助
• `q(uit)`: 退出 pdb

查看当前位置：
• `l(ist)`: 列出当前行周围的 11 行。再次使用会列出下 11 行，依此类推
• `l(ist) .`: 同上，但使用一个点。列出当前行周围的 11 行。如果你已经使用 `l(list)` 多次并丢失了当前位置，这很有用
• `l(ist) first, last`: 列出特定行范围
• `ll`: 列出当前函数的所有源代码
• `w(here)`: 打印堆栈跟踪

查看值：
• `p(rint) expr`: 评估表达式并打印其值
• `pp expr`: 与 `p(rint) expr` 相同，但使用 `pprint` 模块的漂亮打印。非常适合结构化数据
• `a(rgs)`: 打印当前函数的参数列表

执行命令：
• `s(tep)`: 执行当前行并步入下一行源代码，即使它位于函数内部
• `n(ext)`: 执行当前行并步进到当前函数中的下一行
• `r(eturn)`: 继续执行直到当前函数返回
• `c(ontinue)`: 继续执行直到下一个断点。当与 `--trace` 一起使用时，继续执行直到下一个测试的开始
• `unt(il) lineno`: 继续执行直到给定的行号

继续调试我们的测试，我们将使用 `ll` 列出当前函数：

`(Pdb) ll`

我们可以使用 `until 8` 在调用 `list_done_cards()` 之前打断点：

`(Pdb) until 8`

然后使用 `step` 进入函数：

`(Pdb) step`

让我们再次使用 `ll` 查看整个函数：

`(Pdb) ll`

现在让我们继续执行直到此函数即将返回：

`(Pdb) return`

我们可以使用 `p` 或 `pp` 查看 `done_cards` 的值：

`(Pdb) pp done_cards`

如果我们继续退出调试器，进入调用测试并检查返回值，我们可以进一步确认：

`(Pdb) step`
`(Pdb) ll`
`(Pdb) pp the_list`

如果我们停止调试器，向 `list_done_cards()` 添加 `return done_cards`，并重新运行失败的测试，我们可以看看是否修复了问题：

`(Pdb) exit`
`$ pytest --lf -x -v --tb=no`

### 结合使用 pdb 和 tox

我们已经在 Cards 的 `tox.ini` 文件中设置好了。

```ini
# cards_proj/tox.ini
[tox]
envlist = py39, py310
isolated_build = True
skip_missing_interpreters = True
[testenv]
deps =
    pytest
    faker
    pytest-cov
commands = pytest --cov=cards --cov=tests --cov-fail-under=100 {posargs}
```

让我们运行一次并使用 `-e py310 -- --pdb --no-cov` 在失败点停止。（`--no-cov` 用于禁用覆盖率报告。）

`$ tox -e py310 -- --pdb --no-cov`

我们可以在测试中添加空格，并重新运行带有单个测试失败的 tox 环境：

`$ tox -e py310 -- --lf --tb=no --no-cov -v`

## 第 14 章. 第三方插件

### 寻找插件

你可以从几个地方找到第三方 pytest 插件。

[插件列表](https://docs.pytest.org/en/latest/reference/plugin_list.html)
pytest 的主要文档网站包含了一个按字母顺序排列的插件列表，这些插件是从 PyPI 获取的。列表非常长。

[PyPI](https://pypi.org)
Python 包索引 (PyPI) 是获取大量 Python 包的好地方，同时也是寻找 pytest 插件的好去处。当你寻找 pytest 插件时，搜索 pytest、pytest- 或 -pytest 应该能得到不错的结果，因为大多数 pytest 插件要么以 pytest- 开头，要么以 -pytest 结尾。你也可以通过分类器 "Framework :: Pytest" 进行过滤，这样会包括那些虽然不以 pytest- 或 -pytest 命名但包含 pytest 插件的包，例如 Hypothesis 和 Faker。

[pytest-dev](https://github.com/pytest-dev)
GitHub 上的 pytest-dev 组织存放着 pytest 的源代码。这也是许多流行的 pytest 插件所在的地方。对于插件而言，pytest-dev 组织旨在作为一个集中位置，以维护一些流行的 pytest 插件，并分担一些维护责任。更多信息请参考 docs.pytest.org 网站上的 "提交插件到 pytest-dev"。

[pytest 主文档网站](https://docs.pytest.org/en/latest/how-to/plugins.html)
pytest 的主文档网站有一个页面讨论了如何安装和使用 pytest 插件，并列出了几个常用的插件。

### 安装插件

pytest 插件是通过 pip 安装的，就像你在本书前面章节中安装的其他 Python 包一样。

`$ pip install pytest-cov`

### 探索 pytest 插件的多样性

#### 改变常规测试运行流程的插件

有时改变测试的运行顺序是有好处的。以下是一些改变常规测试运行流程的插件：

* pytest-order — 允许我们使用标记来指定顺序
* pytest-randomly — 随机化测试的顺序，首先是文件级别，然后是类级别，最后是测试级别
* pytest-repeat — 方便重复运行单一测试或多个测试特定次数
* pytest-rerunfailures — 重新运行失败的测试。对于不可靠的测试很有帮助
* pytest-xdist — 并行运行测试，可以在一台机器上的多个 CPU 上或多个远程机器上运行

#### 改变或增强输出的插件

常规的 pytest 输出主要显示通过测试的点以及其他输出的字符。如果你传递 `-v` 参数，你会看到测试名称和结果的列表。但是，有一些插件可以改变输出形式。

* pytest-instafail — 添加了一个 `--instafail` 标志，可以在测试失败后立即报告失败测试的堆栈跟踪和输出。通常情况下，pytest 在所有测试完成后才报告这些信息。
* pytest-sugar — 用绿色勾号代替通过测试的点，并且有一个漂亮的进度条。它也会像 pytest-instafail 一样立即显示失败情况。
* pytest-html — 允许生成 HTML 报告。报告可以扩展额外的数据和图像，例如失败案例的屏幕截图。

#### 用于 Web 开发的插件

pytest 广泛用于测试 Web 项目，因此有许多插件可以帮助进行 Web 测试也就不足为奇了。

* pytest-selenium — 提供了便于配置基于浏览器的测试的固定件。Selenium 是一个流行的浏览器测试工具。
* pytest-splinter — 基于 Selenium 构建了一个更高级别的接口，使得更容易地从 pytest 使用 Splinter。
* pytest-django 和 pytest-flask — 使使用 pytest 测试 Django 和 Flask 应用程序变得更加容易。Django 和 Flask 是 Python 最流行的 Web 框架之一。

#### 生成假数据的插件

有几个插件满足这一需求。

* Faker — 为你生成假数据。提供了 faker 固定件用于 pytest
* model-bakery — 生成带有假数据的 Django 模型对象。
* pytest-factoryboy — 包含了 Factory Boy 的固定件，这是一个数据库模型数据生成器
* pytest-mimesis — 生成类似 Faker 的假数据，但 Mimesis 的速度要快得多

#### 扩展 pytest 功能的插件

所有的插件都扩展了 pytest 的功能，但我正在耗尽好的分类名称。这里是一些各种很酷的插件。

* pytest-cov — 在测试的同时运行覆盖率分析
* pytest-benchmark — 对测试中的代码进行基准计时
* pytest-timeout — 不让测试运行太久
* pytest-asyncio — 测试异步函数
* pytest-bdd — 使用 pytest 编写行为驱动开发 (BDD) 风格的测试
* pytest-freezegun — 冻结时间，使得任何读取时间的代码在测试期间都能得到相同的时间值。你还可以设置特定的日期或时间。
* pytest-mock — 对 unittest.mock 的补丁 API 的轻量级封装

### 并行运行测试

如果你的测试不需要访问共享资源，可以通过并行运行多个测试来加速测试会话。`pytest-xdist` 插件允许你这么做。你可以指定多个处理器并并行运行许多测试。你甚至可以把测试推送到其他机器上，并使用多台计算机。

我们可以使用 `-n=auto` 来尽可能多地利用 CPU 核心：

`$ pytest --count=10 -n=auto test_parallel.py`

下面是使用 `-n=6` 对于 60 个测试的情况：

`$ pytest --count=60 -n=6 test_parallel.py`

pytest-xdist 插件还附带了一个很好的特性：`--looponfail` 标志。`--looponfail` 标志允许你在子进程中反复运行测试。

### 随机化测试顺序

pytest-randomly 插件非常适合随机化测试顺序。它还随机化了其他随机工具（如 Faker 和 Factory Boy）的种子值。

要随机化测试顺序，请安装 pytest-randomly：

`$ pip install pytest-randomly`
`$ pytest -v`

定期随机化你的测试可以帮助你避免某些问题。

## 第 15 章. 构建插件

### 从一个酷点子开始

这里是我们的想法（文档实际上使用 `--runslow`，但我们使用 `--slow`，因为它更短，而且在我看来似乎是一个更好的标志）：

* 使用 `@pytest.mark.slow` 标记那些运行非常慢以至于你不希望总是运行它们的测试。
* 当 pytest 收集要运行的测试时，通过向所有标记为 `@pytest.mark.slow` 的测试添加额外的标记 `@pytest.mark.skip(reason="需要 --slow 选项来运行")` 来拦截这个过程。这样，这些测试默认会被跳过。
* 添加 `--slow` 标志，以便用户能够覆盖这种行为并实际运行慢速测试。在正常情况下，每当运行 pytest 时，标记为慢速的测试将会被跳过。然而，`--slow` 标志将会运行所有测试，包括慢速测试。
* 要仅运行慢速测试，仍然可以使用 `-m slow` 来选择标记，但必须与 `--slow` 结合使用，所以 `-m slow --slow` 将只运行慢速测试。

我们已经可以使用标记来选择或排除特定测试。有了 `--slow`，我们只是试图改变默认行为，即排除标记为“慢”的测试：

| 行为 | 没有插件 | 有插件 |
| -- | -- | -- |
| 排除慢速 | pytest -m "not slow" | pytest |
| 包含慢速 | pytest | pytest --slow |
| 仅慢速 | pytest -m slow | pytest -m slow --slow |

测试文件看起来像这样：

```python
# just_markers/test_slow.py
import pytest

def test_normal():
    pass

@pytest.mark.slow
def test_slow():
    pass
```

这里是声明“慢”标记的配置文件：

```python
# just_markers/pytest.ini
[pytest]
markers = slow: 标记测试为慢速运行
```

我们想要更容易实现的行为，即避免慢速测试，看起来像这样：

```shell
cd path/to/code/just_markers
pytest -v -m "not slow"
```

### 构建本地的 conftest 插件

我们将从修改 conftest.py 文件开始，并在将代码移到插件之前在当地测试我们的更改。

为了修改 pytest 的工作方式，我们需要利用 pytest 的钩子函数。钩子函数是 pytest 提供的功能入口点，允许插件开发者在某些点拦截 pytest 的行为并做出改变。pytest 文档列出了很多钩子函数。

* `pytest_configure()` — 允许插件和 conftest 文件执行初始配置。我们将使用这个钩子函数预声明慢速标记，这样用户就不必在他们的配置文件中添加 `slow`。
* `pytest_addoption()` — 用来注册选项和设置。我们将使用这个钩子添加 `--slow` 标志。
* `pytest_collection_modifyitems()` — 在收集测试之后调用，可用于过滤或重新排序测试项。我们需要这个来查找慢速测试，以便我们可以标记它们以跳过。

让我们从 `pytest_configure()` 开始，声明慢速标记，并使用 `pytest_addoption()` 添加 `--slow` 标志：

```python
# conftest.py
import pytest

def pytest_configure(config):
    config.addinivalue_line("markers", "slow: 标记测试为慢速运行")

def pytest_addoption(parser):
    parser.addoption(
        "--slow", action="store_true", help="包含标记为慢速的测试"
    )

def pytest_collection_modifyitems(config, items):
    if not config.getoption("--slow"):
        skip_slow = pytest.mark.skip(reason="需要 --slow 选项来运行")
        for item in items:
            if item.get_closest_marker("slow"):
                item.add_marker(skip_slow)
```

### 创建可安装的插件

顶级目录的名称并不重要。我们将称其为 `pytest_skip_slow`：

```
pytest_skip_slow
├── examples
│   └── test_slow.py
└── pytest_skip_slow.py
```

我们首先安装 Flit 并在一个虚拟环境中以及新目录中运行 `flit init`：

```shell
cd path/to/code/pytest_skip_slow
pip install flit
flit init
```

现在让我们看看 `flit init` 后 `pyproject.toml` 的样子：

```toml
[build-system]
requires = ["flit_core >=3.2,<4"]
build-backend = "flit_core.buildapi"

[project]
name = "pytest-skip-slow"
authors = [{name = "您的名字", email = "your.name@example.com"}]
readme = "README.md"
classifiers = [
    "License :: OSI Approved :: MIT License",
    "Framework :: Pytest"
]
dynamic = ["version", "description"]
dependencies = ["pytest>=6.2.0"]
requires-python = ">=3.7"

[project.urls]
Home = "https://github.com/okken/pytest-skip-slow"

[project.entry-points.pytest11]
skip_slow = "pytest_skip_slow"

[project.optional-dependencies]
test = ["tox"]

[tool.flit.module]
name = "pytest_skip_slow"
```

我们几乎准备好构建我们的包了。不过还有一些缺失的东西，我们需要：

1. 在 `pytest_skip_slow.py` 的顶部添加一个描述插件的文档字符串。
2. 在 `pytest_skip_slow.py` 中添加一个 `__version__` 字符串。
3. 创建一个 `README.md` 文件。（它不必很花哨；我们可以稍后再添加内容。）

这是 `pytest_skip_slow.py` 中的文档字符串和版本：

```python
# pytest_skip_slow_final/pytest_skip_slow.py
"""
一个 pytest 插件，用于默认跳过 `@pytest.mark.slow` 的测试。
使用 `--slow` 包含慢速测试。
"""
import pytest

__version__ = "0.0.1"
# ... 我们的插件剩余部分的代码 ...
```

这是一个简单的 `README.md` 文件：

```
# pytest_skip_slow_final/README.md
# pytest-skip-slow
一个 pytest 插件，用于默认跳过 `@pytest.mark.slow` 的测试。
使用 `--slow` 包含慢速测试。
```

现在我们可以使用 `flit build` 来构建一个可安装的包：

```shell
flit build
```

我们可以直接安装轮子文件来自行尝试：

```shell
pip install dist/pytest_skip_slow-0.0.1-py3-none-any.whl
pytest examples/test_slow.py
pytest --slow examples/test_slow.py
```

如果我们打算在这里停下来，还需要记住做以下几步：

* 确保 `.gitignore` 文件中忽略了 `__pycache__` 和 `dist` 目录。
* 提交 `LICENSE`、`README.md`、`pyproject.toml`、`examples/test_slow.py` 和 `pytest_skip_slow.py` 到版本控制系统。

### 使用 pytester 测试插件

插件是需要像其他代码一样进行测试的代码。然而，测试对测试工具的更改有些棘手。当我们手动使用 `test_slow.py` 测试插件时，

* 使用 `-v` 运行以确保标记为慢速的测试被跳过，
* 使用 `-v --slow` 运行以确保所有测试都运行了，
* 使用 `-v -m slow --slow` 运行以确保仅慢速测试运行。

我们首先需要在 `conftest.py` 中启用它：

```python
# pytest_skip_slow_final/tests/conftest.py
pytest_plugins = ["pytester"]
```

`pytester` 文档列出了一系列函数来帮助填充这个目录：

* `makefile()` 创建任何类型的文件。
* `makepyfile()` 创建 Python 文件。这通常用来创建测试文件。
* `makeconftest()` 创建 `conftest.py`。
* `makeini()` 创建 `tox.ini`。
* `makepyprojecttoml()` 创建 `pyproject.toml`。
* `maketxtfile()` … 你明白了吧。
* `mkdir()` 和 `mkpydir()` 创建包含或不包含 `__init__.py` 的测试子目录。
* `copy_example()` 从项目的目录复制文件到临时目录。这是我最喜欢的，也是我们将用来测试我们的插件的方式。

让我们来看一个例子：

```python
# pytest_skip_slow_final/tests/test_plugin.py
import pytest

@pytest.fixture()
def examples(pytester):
    pytester.copy_example("examples/test_slow.py")

def test_skip_slow(pytester, examples):
    result = pytester.runpytest("-v")
    result.stdout.fnmatch_lines(
        [
            "*test_normal PASSED*",
            "*test_slow SKIPPED (需要 --slow 选项来运行)*",
        ]
    )
    result.assert_outcomes(passed=1, skipped=1)

def test_run_slow(pytester, examples):
    result = pytester.runpytest("--slow")
    result.assert_outcomes(passed=2)

def test_run_only_slow(pytester, examples):
    result = pytester.runpytest("-v", "-m", "slow", "--slow")
    result.stdout.fnmatch_lines(["*test_slow PASSED*"])
    outcomes = result.parseoutcomes()
    assert outcomes["passed"] == 1
    assert outcomes["deselected"] == 1

def test_help(pytester):
    result = pytester.runpytest("--help")
    result.stdout.fnmatch_lines(
        ["*--slow * 包含标记为慢速的测试*"]
    )
```

在运行此测试之前，让我们先针对可编辑代码进行测试：

```shell
cd /path/to/code/ch15/pytest_skip_slow_final
pip uninstall pytest-skip-slow
pip install -e .
```

### 使用 tox 测试多个 Python 和 pytest 版本

这是我们插件的 `tox.ini` 文件：

```ini
# pytest_skip_slow_final/tox.ini
[pytest]
testpaths = tests

[tox]
envlist = py{37, 38, 39, 310}-pytest{62,70}
isolated_build = True

[testenv]
deps =
    pytest62: pytest==6.2.5
    pytest70: pytest==7.0.0
commands = pytest {posargs:tests}
description = 运行 pytest
```

现在我们只需要运行它：

```shell
tox -q --parallel
```

### 发布插件

要发布你的插件，你可以：

* 将插件代码推送到 Git 存储库并从那里安装。
  * 例如：pip install git+<https://github.com/okken/pytest-skip-slow>
  * 注意，你可以在 `requirements.txt` 文件和 `tox.ini` 中列出多个 `git+https://...` 存储库作为依赖项。
* 将轮子文件 `pytest_skip_slow-0.0.1-py3-none-any.whl` 复制到某个共享目录并从那里安装。
  * cp dist/*.whl path/to/my_packages/
  * pip install pytest-skip-slow --no-index --find-links=path/to/my_packages/
* 发布到 PyPI。
  * 查看 Python 关于打包的文档中的“上传发行存档”部分。
  * 参阅 Flit 文档中的“控制包上传”部分。
