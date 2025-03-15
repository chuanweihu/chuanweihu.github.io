---
layout: post
title:  使用python性能剖析工具
categories: [python]
comments: true
tags: [python, profile]
---

> 本文主要介绍一个自定义的函数时间剖析程序和line_profiler库。

* TOC
{:toc}

<!--more-->

## 函数整体的性能剖析工具(`func_profile`)

> func_profile模块能够对Python函数进行时间剖析。

### 定义

```python
# func_profile.py
"""
Basic usage is to import func_profile and decorate your function with
func_profile.  By default this does nothing, it's a no-op decorator.
However, if you run with the environment variable ``FUNC_PROFILE=1`` or if
``'--func-profile' in sys.argv'``, then it enables profiling and at the end of your
script it will output the profile text.

Here is a minimal example that will write a script to disk and then run it
with profiling enabled or disabled by various methods:

    echo "---"
    echo "## Base Case: Run without any profiling"
    python demo.py

    echo "---"
    echo "## Option 1: Enable profiler with the command line"
    python demo.py --func-profile

    echo "---"
    echo "## Option 2: Enable profiler with an environment variable" (Linux)
    FUNC_PROFILE=1 python demo.py
"""
import functools
import inspect
import os
import pathlib
import sys
from time import strftime, time

_FALSY_STRINGS = {'', '0', 'off', 'false', 'no'}


def log_display(*args):
    print(strftime("[%H:%M:%S]"), end=" ")
    print(*args)


def is_classmethod(f):
    return isinstance(f, classmethod)


class FuncProfiler:
    """
    Manages a profiler that will output on interpreter exit.

    The :py:obj:`line_profile.profile` decorator is an instance of this object.

    Attributes:
        setup_config (Dict[str, List[str]]):
            Determines how the implicit setup behaves by defining which
            environment variables / command line flags to look for.

        enabled (bool | None):
            True if the profiler is enabled (i.e. if it will wrap a function
            that it decorates with a real profiler). If None, then the value
            defaults based on the ``setup_config``, :py:obj:`os.environ`, and
            :py:obj:`sys.argv`.

    Example:
        >>> self = FuncProfiler()
    """

    def __init__(self):
        self.setup_config = {
            'environ_flags': ['FUNC_PROFILE'],
            'cli_flags': ['--func-profile'],
        }
        self.enabled = None
        self.logfile = None

    def open_logfile(self, filename='profile.log'):
        if self.logfile is not None:
            self.close_logfile()
        try:
            self.logfile = open(filename, 'w')
        except IOError as e:
            print(f"Error opening log file: {e}", file=sys.stderr)
            self.logfile = None

    def close_logfile(self):
        if self.logfile is not None:
            try:
                self.logfile.close()
            except IOError as e:
                print(f"Error closing log file: {e}", file=sys.stderr)
            finally:
                self.logfile = None

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self.close_logfile()

    def _logfile_setup(self):
        from datetime import datetime as datetime_cls
        now = datetime_cls.now()
        timestamp = now.strftime('%Y-%m-%dT%H%M%S')
        filename = pathlib.Path(f'profile_{timestamp}.log')
        self.open_logfile(filename)

    def _implicit_setup(self):
        """
        Called once the first time the user decorates a function with
        ``line_profiler.profile`` and they have not explicitly setup the global
        profiling options.
        """
        environ_flags = self.setup_config['environ_flags']
        cli_flags = self.setup_config['cli_flags']
        is_profiling = any(os.environ.get(f, '').lower() not in _FALSY_STRINGS
                           for f in environ_flags)
        is_profiling |= any(f in sys.argv for f in cli_flags)
        if is_profiling:
            self.enable()
        else:
            self.disable()

    def enable(self):
        """
        Explicitly enables global profiler and controls its settings.
        """
        self.enabled = True
        self._logfile_setup()

    def disable(self):
        """
        Explicitly initialize and disable this global profiler.
        """
        self.enabled = False
        self.logfile = None

    def dump_stats(self, *args):
        if self.logfile is not None:
            try:
                print(strftime("[%H:%M:%S]"), end=" ", file=self.logfile)
                print(*args, file=self.logfile)
                self.logfile.flush()
            except IOError as e:
                print(f"Error writing to log file: {e}", file=sys.stderr)

    def wrap_classmethod(self, func):
        """
        Wrap a classmethod to profile it.
        """

        @functools.wraps(func)
        def wrapper(*args, **kwds):
            self.enable_by_count()
            try:
                start_time = time()
                result = func.__func__(func.__class__, *args, **kwds)
                end_time = time()
                exec_time = end_time - start_time
                frame = inspect.stack()[1]
                filename = os.path.basename(frame[1])

                self.dump_stats(f"Current Line: File {filename}, line ({frame[2]}),"
                                f"sum of time for {func.__name__} is {exec_time: .2e}(s)")
            finally:
                self.dump_stats('-' * 79)
            return result

        return wrapper

    def wrap_function(self, func):
        """ Wrap a function to profile it.
        """

        @functools.wraps(func)
        def wrapper(*args, **kwds):
            try:
                start_time = time()
                result = func(*args, **kwds)
                end_time = time()
                exec_time = end_time - start_time
                frame = inspect.stack()[1]
                filename = os.path.basename(frame[1])
                self.dump_stats(f"Current Line: File '{filename}', line ({frame[2]}), "
                                f"sum of time for '{func.__name__}' is {exec_time: .2e}(s)")
            finally:
                self.dump_stats('-' * 79)
            return result

        return wrapper

    def __call__(self, func):
        """
        If the global profiler is enabled, decorate a function to start the
        profiler on function entry and stop it on function exit. Otherwise
        return the input.

        Args:
            func (Callable): the function to profile

        Returns:
            Callable: a potentially wrapped function
        """
        if self.enabled is None:
            # Force a setup if we haven't done it before.
            self._implicit_setup()
        if not self.enabled:
            return func
        """
        # Decorate a function to start the profiler on function entry and stop
        it on function exit.
        """
        if is_classmethod(func):
            wrapper = self.wrap_classmethod(func)
        else:
            wrapper = self.wrap_function(func)
        return wrapper


# Construct the global profiler
func_profile = FuncProfiler()
```

### 使用

1. 在需要调试优化的代码中引用`func_profile`装饰器

```python
from func_profiler import func_profile

@func_profile
def plus(a, b):
    return a + b


@func_profile
def fib(n):
    a, b = 0, 1
    while a < n:
        a, b = b, plus(a, b)
    return b


def main():
    log_display('start calculating')
    n = 3
    result = fib(n)
    log_display(f'done calculating. fib({n}) = {result}.')


if __name__ == '__main__':
    main()
```

2. 启动

终端运行`python demo.py --func-profile`命令

## 函数内的性能剖析工具(`line_profiler`)

> line_profiler模块能够对Python函数进行逐行的时间剖析，以帮助开发者找出代码中的性能瓶颈。
> - [Github](https://github.com/pyutils/line_profiler)
> - [Pypi](https://pypi.org/project/line_profiler)
> - [ReadTheDocs](https://kernprof.readthedocs.io/en/latest/)

#### 安装

`python3 -m pip install -i https://mirrors.cloud.tencent.com/pypi/simple line_profiler`

#### 使用

1. 在需要调试优化的代码中引用`func_profile`装饰器

```python
from line_profiler import profile


@profile
def plus(a, b):
    return a + b


@profile
def fib(n):
    a, b = 0, 1
    while a < n:
        a, b = b, plus(a, b)
    return b


def main():
    print('start calculating')
    n = 3
    result = fib(n)
    print(f'done calculating. fib({n}) = {result}.')


if __name__ == '__main__':
    main()
```

2. 启动

终端运行`python demo.py --line-profile`命令

3. 查看结果

终端运行`python -m line_profiler -rtmz profile_output.lprof`命令
