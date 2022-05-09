# 文件和目录访问

## os.path

## os.pathlib

### [目录](https://docs.python.org/zh-cn/3.8/contents.html)

- `pathlib` --- 面向对象的文件系统路径
  - [基础使用](https://docs.python.org/zh-cn/3.8/library/pathlib.html#basic-use)
  - 纯路径
    - [通用性质](https://docs.python.org/zh-cn/3.8/library/pathlib.html#general-properties)
    - [运算符](https://docs.python.org/zh-cn/3.8/library/pathlib.html#operators)
    - [访问个别部分](https://docs.python.org/zh-cn/3.8/library/pathlib.html#accessing-individual-parts)
    - [方法和特征属性](https://docs.python.org/zh-cn/3.8/library/pathlib.html#methods-and-properties)
  - 具体路径
    - [方法](https://docs.python.org/zh-cn/3.8/library/pathlib.html#methods)
  - [对应的 `os` 模块的工具https://github.com/python/cpython/blob/3.8/Doc/library/pathlib.rst)

**源代码** [Lib/pathlib.py](https://github.com/python/cpython/tree/3.8/Lib/pathlib.py)

------

该模块提供表示文件系统路径的类，其语义适用于不同的操作系统。路径类被分为提供纯计算操作而没有 I/O 的 [纯路径](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pure-paths)，以及从纯路径继承而来但提供 I/O 操作的 [具体路径](https://docs.python.org/zh-cn/3.8/library/pathlib.html#concrete-paths)。

![../_images/pathlib-inheritance.png](https://docs.python.org/zh-cn/3.8/_images/pathlib-inheritance.png)

如果你以前从未使用过此模块或者不确定在项目中使用哪一个类是正确的，则 [`Path`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path) 总是你需要的。它在运行代码的平台上实例化为一个 [具体路径](https://docs.python.org/zh-cn/3.8/library/pathlib.html#concrete-paths)。

在一些用例中纯路径很有用，例如：

1. 如果你想要在 Unix 设备上操作 Windows 路径（或者相反）。你不应在 Unix 上实例化一个 [`WindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.WindowsPath)，但是你可以实例化 [`PureWindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PureWindowsPath)。
2. 你只想操作路径但不想实际访问操作系统。在这种情况下，实例化一个纯路径是有用的，因为它们没有任何访问操作系统的操作。

参见

 

[**PEP 428**](https://www.python.org/dev/peps/pep-0428)：pathlib 模块 -- 面向对象的的文件系统路径。

参见

 

对于底层的路径字符串操作，你也可以使用 [`os.path`](https://docs.python.org/zh-cn/3.8/library/os.path.html#module-os.path) 模块。

### 基础使用

导入主类:

\>>>

```
>>> from pathlib import Path
```

列出子目录:

\>>>

```
>>> p = Path('.')
>>> [x for x in p.iterdir() if x.is_dir()]
[PosixPath('.hg'), PosixPath('docs'), PosixPath('dist'),
 PosixPath('__pycache__'), PosixPath('build')]
```

列出当前目录树下的所有 Python 源代码文件:

\>>>

```
>>> list(p.glob('**/*.py'))
[PosixPath('test_pathlib.py'), PosixPath('setup.py'),
 PosixPath('pathlib.py'), PosixPath('docs/conf.py'),
 PosixPath('build/lib/pathlib.py')]
```

在目录树中移动:

\>>>

```
>>> p = Path('/etc')
>>> q = p / 'init.d' / 'reboot'
>>> q
PosixPath('/etc/init.d/reboot')
>>> q.resolve()
PosixPath('/etc/rc.d/init.d/halt')
```

查询路径的属性:

\>>>

```
>>> q.exists()
True
>>> q.is_dir()
False
```

打开一个文件:

\>>>

```
>>> with q.open() as f: f.readline()
...
'#!/bin/bash\n'
```



### 纯路径

纯路径对象提供了不实际访问文件系统的路径处理操作。有三种方式来访问这些类，也是不同的风格：

- *class* `pathlib.``PurePath`(**pathsegments*)

  一个通用的类，代表当前系统的路径风格（实例化为 [`PurePosixPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePosixPath) 或者 [`PureWindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PureWindowsPath)）:>>>`>>> PurePath('setup.py')      # Running on a Unix machine PurePosixPath('setup.py') `每一个 *pathsegments* 的元素可能是一个代表路径片段的字符串，一个返回字符串的实现了 [`os.PathLike`](https://docs.python.org/zh-cn/3.8/library/os.html#os.PathLike) 接口的对象，或者另一个路径对象:>>>`>>> PurePath('foo', 'some/path', 'bar') PurePosixPath('foo/some/path/bar') >>> PurePath(Path('foo'), Path('bar')) PurePosixPath('foo/bar') `当 *pathsegments* 为空的时候，假定为当前目录:>>>`>>> PurePath() PurePosixPath('.') `当给出一些绝对路径，最后一位将被当作锚（模仿 [`os.path.join()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.join) 的行为）:>>>`>>> PurePath('/etc', '/usr', 'lib64') PurePosixPath('/usr/lib64') >>> PureWindowsPath('c:/Windows', 'd:bar') PureWindowsPath('d:bar') `但是，在 Windows 路径中，改变本地根目录并不会丢弃之前盘符的设置:>>>`>>> PureWindowsPath('c:/Windows', '/Program Files') PureWindowsPath('c:/Program Files') `假斜线和单独的点都会被消除，但是双点 （`‘..’`） 不会，以防改变符号链接的含义。>>>`>>> PurePath('foo//bar') PurePosixPath('foo/bar') >>> PurePath('foo/./bar') PurePosixPath('foo/bar') >>> PurePath('foo/../bar') PurePosixPath('foo/../bar') `（一个很 naïve 的做法是让 `PurePosixPath('foo/../bar')` 等同于 `PurePosixPath('bar')`，如果 `foo` 是一个指向其他目录的符号链接那么这个做法就将出错）纯路径对象实现了 [`os.PathLike`](https://docs.python.org/zh-cn/3.8/library/os.html#os.PathLike) 接口，允许它们在任何接受此接口的地方使用。*在 3.6 版更改:* 添加了 [`os.PathLike`](https://docs.python.org/zh-cn/3.8/library/os.html#os.PathLike) 接口支持。

- *class* `pathlib.``PurePosixPath`(**pathsegments*)

  一个 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 的子类，路径风格不同于 Windows 文件系统:>>>`>>> PurePosixPath('/etc') PurePosixPath('/etc') `*pathsegments* 参数的指定和 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 相同。

- *class* `pathlib.``PureWindowsPath`(**pathsegments*)

  [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 的一个子类，路径风格为 Windows 文件系统路径:>>>`>>> PureWindowsPath('c:/Program Files/') PureWindowsPath('c:/Program Files') `*pathsegments* 参数的指定和 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 相同。

无论你正运行什么系统，你都可以实例化这些类，因为它们提供的操作不做任何系统调用。

### 通用性质

路径是不可变并可哈希的。相同风格的路径可以排序与比较。这些性质尊重对应风格的大小写转换语义:

\>>>

```
>>> PurePosixPath('foo') == PurePosixPath('FOO')
False
>>> PureWindowsPath('foo') == PureWindowsPath('FOO')
True
>>> PureWindowsPath('FOO') in { PureWindowsPath('foo') }
True
>>> PureWindowsPath('C:') < PureWindowsPath('d:')
True
```

不同风格的路径比较得到不等的结果并且无法被排序:

\>>>

```
>>> PureWindowsPath('foo') == PurePosixPath('foo')
False
>>> PureWindowsPath('foo') < PurePosixPath('foo')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: '<' not supported between instances of 'PureWindowsPath' and 'PurePosixPath'
```

### 运算符

斜杠 `/` 操作符有助于创建子路径，就像 [`os.path.join()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.join) 一样:

\>>>

```
>>> p = PurePath('/etc')
>>> p
PurePosixPath('/etc')
>>> p / 'init.d' / 'apache2'
PurePosixPath('/etc/init.d/apache2')
>>> q = PurePath('bin')
>>> '/usr' / q
PurePosixPath('/usr/bin')
```

文件对象可用于任何接受 [`os.PathLike`](https://docs.python.org/zh-cn/3.8/library/os.html#os.PathLike) 接口实现的地方。

\>>>

```
>>> import os
>>> p = PurePath('/etc')
>>> os.fspath(p)
'/etc'
```

路径的字符串表示法为它自己原始的文件系统路径（以原生形式，例如在 Windows 下使用反斜杠）。你可以传递给任何需要字符串形式路径的函数。

\>>>

```
>>> p = PurePath('/etc')
>>> str(p)
'/etc'
>>> p = PureWindowsPath('c:/Program Files')
>>> str(p)
'c:\\Program Files'
```

类似地，在路径上调用 [`bytes`](https://docs.python.org/zh-cn/3.8/library/stdtypes.html#bytes) 将原始文件系统路径作为字节对象给出，就像被 [`os.fsencode()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.fsencode) 编码一样:

\>>>

```
>>> bytes(p)
b'/etc'
```

注解

 

只推荐在 Unix 下调用 [`bytes`](https://docs.python.org/zh-cn/3.8/library/stdtypes.html#bytes)。在 Windows， unicode 形式是文件系统路径的规范表示法。

### 访问个别部分

为了访问路径独立的部分 （组件），使用以下特征属性：

- `PurePath.``parts`

  一个元组，可以访问路径的多个组件:>>>`>>> p = PurePath('/usr/bin/python3') >>> p.parts ('/', 'usr', 'bin', 'python3') >>> p = PureWindowsPath('c:/Program Files/PSF') >>> p.parts ('c:\\', 'Program Files', 'PSF') `（注意盘符和本地根目录是如何重组的）

### 方法和特征属性

纯路径提供以下方法和特征属性：

- `PurePath.``drive`

  一个表示驱动器盘符或命名的字符串，如果存在:>>>`>>> PureWindowsPath('c:/Program Files/').drive 'c:' >>> PureWindowsPath('/Program Files/').drive '' >>> PurePosixPath('/etc').drive '' `UNC 分享也被认作驱动器:>>>`>>> PureWindowsPath('//host/share/foo.txt').drive '\\\\host\\share' `

- `PurePath.``root`

  一个表示（本地或全局）根的字符串，如果存在:>>>`>>> PureWindowsPath('c:/Program Files/').root '\\' >>> PureWindowsPath('c:Program Files/').root '' >>> PurePosixPath('/etc').root '/' `UNC 分享一样拥有根:>>>`>>> PureWindowsPath('//host/share').root '\\' `

- `PurePath.``anchor`

  驱动器和根的联合:>>>`>>> PureWindowsPath('c:/Program Files/').anchor 'c:\\' >>> PureWindowsPath('c:Program Files/').anchor 'c:' >>> PurePosixPath('/etc').anchor '/' >>> PureWindowsPath('//host/share').anchor '\\\\host\\share\\' `

- `PurePath.``parents`

  An immutable sequence providing access to the logical ancestors of the path:>>>`>>> p = PureWindowsPath('c:/foo/bar/setup.py') >>> p.parents[0] PureWindowsPath('c:/foo/bar') >>> p.parents[1] PureWindowsPath('c:/foo') >>> p.parents[2] PureWindowsPath('c:/') `

- `PurePath.``parent`

  此路径的逻辑父路径:>>>`>>> p = PurePosixPath('/a/b/c/d') >>> p.parent PurePosixPath('/a/b/c') `你不能超过一个 anchor 或空路径:>>>`>>> p = PurePosixPath('/') >>> p.parent PurePosixPath('/') >>> p = PurePosixPath('.') >>> p.parent PurePosixPath('.') `注解 这是一个单纯的词法操作，因此有以下行为:>>>`>>> p = PurePosixPath('foo/..') >>> p.parent PurePosixPath('foo') `如果你想要向上移动任意文件系统路径，推荐先使用 [`Path.resolve()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.resolve) 来解析符号链接以及消除 `".."` 组件。

- `PurePath.``name`

  一个表示最后路径组件的字符串，排除了驱动器与根目录，如果存在的话:>>>`>>> PurePosixPath('my/library/setup.py').name 'setup.py' `UNC 驱动器名不被考虑:>>>`>>> PureWindowsPath('//some/share/setup.py').name 'setup.py' >>> PureWindowsPath('//some/share').name '' `

- `PurePath.``suffix`

  最后一个组件的文件扩展名，如果存在:>>>`>>> PurePosixPath('my/library/setup.py').suffix '.py' >>> PurePosixPath('my/library.tar.gz').suffix '.gz' >>> PurePosixPath('my/library').suffix '' `

- `PurePath.``suffixes`

  路径的文件扩展名列表:>>>`>>> PurePosixPath('my/library.tar.gar').suffixes ['.tar', '.gar'] >>> PurePosixPath('my/library.tar.gz').suffixes ['.tar', '.gz'] >>> PurePosixPath('my/library').suffixes [] `

- `PurePath.``stem`

  最后一个路径组件，除去后缀:>>>`>>> PurePosixPath('my/library.tar.gz').stem 'library.tar' >>> PurePosixPath('my/library.tar').stem 'library' >>> PurePosixPath('my/library').stem 'library' `

- `PurePath.``as_posix`()

  返回使用正斜杠（`/`）的路径字符串:>>>`>>> p = PureWindowsPath('c:\\windows') >>> str(p) 'c:\\windows' >>> p.as_posix() 'c:/windows' `

- `PurePath.``as_uri`()

  将路径表示为 `file` URL。如果并非绝对路径，抛出 [`ValueError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#ValueError)。>>>`>>> p = PurePosixPath('/etc/passwd') >>> p.as_uri() 'file:///etc/passwd' >>> p = PureWindowsPath('c:/Windows') >>> p.as_uri() 'file:///c:/Windows' `

- `PurePath.``is_absolute`()

  返回此路径是否为绝对路径。如果路径同时拥有驱动器符与根路径（如果风格允许）则将被认作绝对路径。>>>`>>> PurePosixPath('/a/b').is_absolute() True >>> PurePosixPath('a/b').is_absolute() False >>> PureWindowsPath('c:/a/b').is_absolute() True >>> PureWindowsPath('/a/b').is_absolute() False >>> PureWindowsPath('c:').is_absolute() False >>> PureWindowsPath('//some/share').is_absolute() True `

- `PurePath.``is_reserved`()

  在 [`PureWindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PureWindowsPath)，如果路径是被 Windows 保留的则返回 `True`，否则 `False`。在 [`PurePosixPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePosixPath)，总是返回 `False`。>>>`>>> PureWindowsPath('nul').is_reserved() True >>> PurePosixPath('nul').is_reserved() False `当保留路径上的文件系统被调用，则可能出现玄学失败或者意料之外的效应。

- `PurePath.``joinpath`(**other*)

  调用此方法等同于将每个 *other* 参数中的项目连接在一起:>>>`>>> PurePosixPath('/etc').joinpath('passwd') PurePosixPath('/etc/passwd') >>> PurePosixPath('/etc').joinpath(PurePosixPath('passwd')) PurePosixPath('/etc/passwd') >>> PurePosixPath('/etc').joinpath('init.d', 'apache2') PurePosixPath('/etc/init.d/apache2') >>> PureWindowsPath('c:').joinpath('/Program Files') PureWindowsPath('c:/Program Files') `

- `PurePath.``match`(*pattern*)

  将此路径与提供的通配符风格的模式匹配。如果匹配成功则返回 `True`，否则返回 `False`。如果 *pattern* 是相对的，则路径可以是相对路径或绝对路径，并且匹配是从右侧完成的：>>>`>>> PurePath('a/b.py').match('*.py') True >>> PurePath('/a/b/c.py').match('b/*.py') True >>> PurePath('/a/b/c.py').match('a/*.py') False `如果 *pattern* 是绝对的，则路径必须是绝对的，并且路径必须完全匹配:>>>`>>> PurePath('/a.py').match('/*.py') True >>> PurePath('a/b.py').match('/*.py') False `与其他方法一样，是否大小写敏感遵循平台的默认规则:>>>`>>> PurePosixPath('b.py').match('*.PY') False >>> PureWindowsPath('b.py').match('*.PY') True `

- `PurePath.``relative_to`(**other*)

  计算此路径相对 *other* 表示路径的版本。如果不可计算，则抛出 ValueError:>>>`>>> p = PurePosixPath('/etc/passwd') >>> p.relative_to('/') PurePosixPath('etc/passwd') >>> p.relative_to('/etc') PurePosixPath('passwd') >>> p.relative_to('/usr') Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "pathlib.py", line 694, in relative_to    .format(str(self), str(formatted))) ValueError: '/etc/passwd' does not start with '/usr' `

- `PurePath.``with_name`(*name*)

  返回一个新的路径并修改 [`name`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.name)。如果原本路径没有 name，ValueError 被抛出:>>>`>>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz') >>> p.with_name('setup.py') PureWindowsPath('c:/Downloads/setup.py') >>> p = PureWindowsPath('c:/') >>> p.with_name('setup.py') Traceback (most recent call last):  File "<stdin>", line 1, in <module>  File "/home/antoine/cpython/default/Lib/pathlib.py", line 751, in with_name    raise ValueError("%r has an empty name" % (self,)) ValueError: PureWindowsPath('c:/') has an empty name `

- `PurePath.``with_suffix`(*suffix*)

  返回一个新的路径并修改 [`suffix`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.suffix)。如果原本的路径没有后缀，新的 *suffix* 则被追加以代替。如果 *suffix* 是空字符串，则原本的后缀被移除:>>>`>>> p = PureWindowsPath('c:/Downloads/pathlib.tar.gz') >>> p.with_suffix('.bz2') PureWindowsPath('c:/Downloads/pathlib.tar.bz2') >>> p = PureWindowsPath('README') >>> p.with_suffix('.txt') PureWindowsPath('README.txt') >>> p = PureWindowsPath('README.txt') >>> p.with_suffix('') PureWindowsPath('README') `



### 具体路径

具体路径是纯路径的子类。除了后者提供的操作之外，它们还提供了对路径对象进行系统调用的方法。有三种方法可以实例化具体路径:

- *class* `pathlib.``Path`(**pathsegments*)

  一个 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 的子类，此类以当前系统的路径风格表示路径（实例化为 [`PosixPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PosixPath) 或 [`WindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.WindowsPath)）:>>>`>>> Path('setup.py') PosixPath('setup.py') `*pathsegments* 参数的指定和 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 相同。

- *class* `pathlib.``PosixPath`(**pathsegments*)

  一个 [`Path`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path) 和 [`PurePosixPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePosixPath) 的子类，此类表示一个非 Windows 文件系统的具体路径:>>>`>>> PosixPath('/etc') PosixPath('/etc') `*pathsegments* 参数的指定和 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 相同。

- *class* `pathlib.``WindowsPath`(**pathsegments*)

  [`Path`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path) 和 [`PureWindowsPath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PureWindowsPath) 的子类，从类表示一个 Windows 文件系统的具体路径:>>>`>>> WindowsPath('c:/Program Files/') WindowsPath('c:/Program Files') `*pathsegments* 参数的指定和 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath) 相同。

你只能实例化与当前系统风格相同的类（允许系统调用作用于不兼容的路径风格可能在应用程序中导致缺陷或失败）:

\>>>

```
>>> import os
>>> os.name
'posix'
>>> Path('setup.py')
PosixPath('setup.py')
>>> PosixPath('setup.py')
PosixPath('setup.py')
>>> WindowsPath('setup.py')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "pathlib.py", line 798, in __new__
    % (cls.__name__,))
NotImplementedError: cannot instantiate 'WindowsPath' on your system
```

### 方法

除纯路径方法外，实体路径还提供以下方法。 如果系统调用失败（例如因为路径不存在）这些方法中许多都会引发 [`OSError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#OSError)。

*在 3.8 版更改:* 对于包含 OS 层级无法表示字符的路径，[`exists()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.exists), [`is_dir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_dir), [`is_file()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_file), [`is_mount()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_mount), [`is_symlink()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_symlink), [`is_block_device()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_block_device), [`is_char_device()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_char_device), [`is_fifo()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_fifo), [`is_socket()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_socket) 现在将返回 `False` 而不是引发异常。

- *classmethod* `Path.``cwd`()

  返回一个新的表示当前目录的路径对象（和 [`os.getcwd()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.getcwd) 返回的相同）:>>>`>>> Path.cwd() PosixPath('/home/antoine/pathlib') `

- *classmethod* `Path.``home`()

  返回一个表示当前用户家目录的新路径对象（和 [`os.path.expanduser()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.expanduser) 构造含 `~` 路径返回的相同）:>>>`>>> Path.home() PosixPath('/home/antoine') `*3.5 新版功能.*

- `Path.``stat`()

  返回一个 [`os.stat_result`](https://docs.python.org/zh-cn/3.8/library/os.html#os.stat_result) 对象，其中包含有关此路径的信息，例如 [`os.stat()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.stat)。 结果会在每次调用此方法时重新搜索。>>>`>>> p = Path('setup.py') >>> p.stat().st_size 956 >>> p.stat().st_mtime 1327883547.852554 `

- `Path.``chmod`(*mode*)

  改变文件的模式和权限，和 [`os.chmod()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.chmod) 一样:>>>`>>> p = Path('setup.py') >>> p.stat().st_mode 33277 >>> p.chmod(0o444) >>> p.stat().st_mode 33060 `

- `Path.``exists`()

  此路径是否指向一个已存在的文件或目录:>>>`>>> Path('.').exists() True >>> Path('setup.py').exists() True >>> Path('/etc').exists() True >>> Path('nonexistentfile').exists() False `注解 如果路径指向一个符号链接， [`exists()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.exists) 返回此符号链接是否指向存在的文件或目录。

- `Path.``expanduser`()

  返回展开了包含 `~` 和 `~user` 的构造，就和 [`os.path.expanduser()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.expanduser) 一样:>>>`>>> p = PosixPath('~/films/Monty Python') >>> p.expanduser() PosixPath('/home/eric/films/Monty Python') `*3.5 新版功能.*

- `Path.``glob`(*pattern*)

  解析相对于此路径的通配符 *pattern*，产生所有匹配的文件:>>>`>>> sorted(Path('.').glob('*.py')) [PosixPath('pathlib.py'), PosixPath('setup.py'), PosixPath('test_pathlib.py')] >>> sorted(Path('.').glob('*/*.py')) [PosixPath('docs/conf.py')] `"`**`" 模式表示 “此目录以及所有子目录，递归”。换句话说，它启用递归通配:>>>`>>> sorted(Path('.').glob('**/*.py')) [PosixPath('build/lib/pathlib.py'), PosixPath('docs/conf.py'), PosixPath('pathlib.py'), PosixPath('setup.py'), PosixPath('test_pathlib.py')] `注解 在一个较大的目录树中使用 "`**`" 模式可能会消耗非常多的时间。

- `Path.``group`()

  返回拥有此文件的用户组。如果文件的 GID 无法在系统数据库中找到，将抛出 [`KeyError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#KeyError) 。

- `Path.``is_dir`()

  如果路径指向一个目录（或者一个指向目录的符号链接）则返回 `True`，如果指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_file`()

  如果路径指向一个正常的文件（或者一个指向正常文件的符号链接）则返回 `True`，如果指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_mount`()

  如果路径是一个 *挂载点 <mount point>*：在文件系统中被其他不同的文件系统挂载的地点。在 POSIX 系统，此函数检查 *path* 的父级 —— `path/..` 是否处于一个和 *path* 不同的设备中，或者 file:path/.. 和 *path* 是否指向相同设备的相同 i-node —— 这能检测所有 Unix 以及 POSIX 变种上的挂载点。 Windows 上未实现。*3.7 新版功能.*

- `Path.``is_symlink`()

  如果路径指向符号链接则返回 `True`， 否则 `False`。如果路径不存在也返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_socket`()

  如果路径指向一个 Unix socket 文件（或者指向 Unix socket 文件的符号链接）则返回 `True`，如果指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_fifo`()

  如果路径指向一个先进先出存储（或者指向先进先出存储的符号链接）则返回 `True` ，指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_block_device`()

  如果文件指向一个块设备（或者指向块设备的符号链接）则返回 `True`，指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``is_char_device`()

  如果路径指向一个字符设备（或指向字符设备的符号链接）则返回 `True`，指向其他类型的文件则返回 `False`。当路径不存在或者是一个破损的符号链接时也会返回 `False`；其他错误（例如权限错误）被传播。

- `Path.``iterdir`()

  当路径指向一个目录时，产生该路径下的对象的路径:>>>`>>> p = Path('docs') >>> for child in p.iterdir(): child ... PosixPath('docs/conf.py') PosixPath('docs/_templates') PosixPath('docs/make.bat') PosixPath('docs/index.rst') PosixPath('docs/_build') PosixPath('docs/_static') PosixPath('docs/Makefile') `子条目会以任意顺序生成，并且不包括特殊条目 `'.'` 和 `'..'`。 如果有文件在迭代器创建之后在目录中被移除或添加，是否要包括该文件对应的路径对象并没有规定。

- `Path.``lchmod`(*mode*)

  就像 [`Path.chmod()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.chmod) 但是如果路径指向符号链接则是修改符号链接的模式，而不是修改符号链接的目标。

- `Path.``lstat`()

  就和 [`Path.stat()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.stat) 一样，但是如果路径指向符号链接，则是返回符号链接而不是目标的信息。

- `Path.``mkdir`(*mode=0o777*, *parents=False*, *exist_ok=False*)

  新建给定路径的目录。如果给出了 *mode* ，它将与当前进程的 `umask` 值合并来决定文件模式和访问标志。如果路径已经存在，则抛出 [`FileExistsError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileExistsError)。如果 *parents* 为 true，任何找不到的父目录都会伴随着此路径被创建；它们会以默认权限被创建，而不考虑 *mode* 设置（模仿 POSIX 的 `mkdir -p` 命令）。如果 *parents* 为 false（默认），则找不到的父级目录会导致 [`FileNotFoundError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileNotFoundError) 被抛出。如果 *exist_ok* 为 false（默认），则在目标已存在的情况下抛出 [`FileExistsError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileExistsError)。如果 *exist_ok* 为 true， 则 [`FileExistsError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileExistsError) 异常将被忽略（和 POSIX `mkdir -p` 命令行为相同），但是只有在最后一个路径组件不是现存的非目录文件时才生效。*在 3.5 版更改:* *exist_ok* 形参被加入。

- `Path.``open`(*mode='r'*, *buffering=-1*, *encoding=None*, *errors=None*, *newline=None*)

  打开路径指向的文件，就像内置的 [`open()`](https://docs.python.org/zh-cn/3.8/library/functions.html#open) 函数所做的一样:>>>`>>> p = Path('setup.py') >>> with p.open() as f: ...     f.readline() ... '#!/usr/bin/env python3\n' `

- `Path.``owner`()

  返回拥有此文件的用户名。如果文件的 UID 无法在系统数据库中找到，则抛出 [`KeyError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#KeyError)。

- `Path.``read_bytes`()

  以字节对象的形式返回路径指向的文件的二进制内容:>>>`>>> p = Path('my_binary_file') >>> p.write_bytes(b'Binary file contents') 20 >>> p.read_bytes() b'Binary file contents' `*3.5 新版功能.*

- `Path.``read_text`(*encoding=None*, *errors=None*)

  以字符串形式返回路径指向的文件的解码后文本内容。>>>`>>> p = Path('my_text_file') >>> p.write_text('Text file contents') 18 >>> p.read_text() 'Text file contents' `文件先被打开然后关闭。有和 [`open()`](https://docs.python.org/zh-cn/3.8/library/functions.html#open) 一样的可选形参。*3.5 新版功能.*

- `Path.``rename`(*target*)

  将文件或目录重命名为给定的 *target*，并返回一个新的指向 *target* 的 Path 实例。 在 Unix 上，如果 *target* 存在且为一个文件，如果用户有足够权限，则它将被静默地替换。 *target* 可以是一个字符串或者另一个路径对象:>>>`>>> p = Path('foo') >>> p.open('w').write('some text') 9 >>> target = Path('bar') >>> p.rename(target) PosixPath('bar') >>> target.open().read() 'some text' `目标路径可能为绝对或相对路径。 相对路径将被解释为相对于当前工作目录，而 *不是* 相对于 Path 对象的目录。*在 3.8 版更改:* 添加了返回值，返回新的 Path 实例。

- `Path.``replace`(*target*)

  将文件名目录重命名为给定的 *target*，并返回一个新的指向 *target* 的 Path 实例。 如果 *target* 指向一个现有文件或目录，则它将被无条件地替换。目标路径可能为绝对或相对路径。 相对路径将被解释为相对于当前工作目录，而 *不是* 相对于 Path 对象的目录。*在 3.8 版更改:* 添加了返回值，返回新的 Path 实例。

- `Path.``resolve`(*strict=False*)

  将路径绝对化，解析任何符号链接。返回新的路径对象:>>>`>>> p = Path() >>> p PosixPath('.') >>> p.resolve() PosixPath('/home/antoine/pathlib') `"`..`" 组件也将被消除（只有这一种方法这么做）:>>>`>>> p = Path('docs/../setup.py') >>> p.resolve() PosixPath('/home/antoine/pathlib/setup.py') `如果路径不存在并且 *strict* 设为 `True`，则抛出 [`FileNotFoundError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileNotFoundError)。如果 *strict* 为 `False`，则路径将被尽可能地解析并且任何剩余部分都会被不检查是否存在地追加。如果在解析路径上发生无限循环，则抛出 [`RuntimeError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#RuntimeError)。*3.6 新版功能:* 加入*strict* 参数（3.6之前的版本相当于strict值为True）

- `Path.``rglob`(*pattern*)

  这就像调用 `Path.glob`时在给定的相对 *pattern* 前面添加了"``**/`()`">>>`>>> sorted(Path().rglob("*.py")) [PosixPath('build/lib/pathlib.py'), PosixPath('docs/conf.py'), PosixPath('pathlib.py'), PosixPath('setup.py'), PosixPath('test_pathlib.py')] `

- `Path.``rmdir`()

  移除此目录。此目录必须为空的。

- `Path.``samefile`(*other_path*)

  返回此目录是否指向与可能是字符串或者另一个路径对象的 *other_path* 相同的文件。语义类似于 [`os.path.samefile()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.samefile) 与 [`os.path.samestat()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.samestat)。如果两者都以同一原因无法访问，则抛出 [`OSError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#OSError)。>>>`>>> p = Path('spam') >>> q = Path('eggs') >>> p.samefile(q) False >>> p.samefile('spam') True `*3.5 新版功能.*

- `Path.``symlink_to`(*target*, *target_is_directory=False*)

  将此路径创建为指向 *target* 的符号链接。在 Windows 下，如果链接的目标是一个目录则 *target_is_directory* 必须为 true （默认为 `False`）。在 POSIX 下， *target_is_directory* 的值将被忽略。>>>`>>> p = Path('mylink') >>> p.symlink_to('setup.py') >>> p.resolve() PosixPath('/home/antoine/pathlib/setup.py') >>> p.stat().st_size 956 >>> p.lstat().st_size 8 `注解 参数的顺序（link, target) 和 [`os.symlink()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.symlink) 是相反的。

- `Path.``link_to`(*target*)

  创建硬链接 *target* 指向此路径。警告 此函数不是将此路径设为指向 *target* 的硬链接，这与函数和参数名的本义不同。 参数顺序 (target, link) 与 [`Path.symlink_to()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.symlink_to) 相反，而与 [`os.link()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.link) 一致。*3.8 新版功能.*

- `Path.``touch`(*mode=0o666*, *exist_ok=True*)

  将给定的路径创建为文件。如果给出了 *mode* 它将与当前进程的 `umask` 值合并以确定文件的模式和访问标志。如果文件已经存在，则当 *exist_ok* 为 true 则函数仍会成功（并且将它的修改事件更新为当前事件），否则抛出 [`FileExistsError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileExistsError)。

- `Path.``unlink`(*missing_ok=False*)

  移除此文件或符号链接。如果路径指向目录，则用 [`Path.rmdir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.rmdir) 代替。如果 *missing_ok* 为假值（默认），则如果路径不存在将会引发 [`FileNotFoundError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileNotFoundError)。如果 *missing_ok* 为真值，则 [`FileNotFoundError`](https://docs.python.org/zh-cn/3.8/library/exceptions.html#FileNotFoundError) 异常将被忽略（和 POSIX `rm -f` 命令的行为相同）。*在 3.8 版更改:* 增加了 *missing_ok* 形参。

- `Path.``write_bytes`(*data*)

  将文件以二进制模式打开，写入 *data* 并关闭:>>>`>>> p = Path('my_binary_file') >>> p.write_bytes(b'Binary file contents') 20 >>> p.read_bytes() b'Binary file contents' `一个同名的现存文件将被覆盖。*3.5 新版功能.*

- `Path.``write_text`(*data*, *encoding=None*, *errors=None*)

  将文件以文本模式打开，写入 *data* 并关闭:>>>`>>> p = Path('my_text_file') >>> p.write_text('Text file contents') 18 >>> p.read_text() 'Text file contents' `同名的现有文件会被覆盖。 可选形参的含义与 [`open()`](https://docs.python.org/zh-cn/3.8/library/functions.html#open) 的相同。*3.5 新版功能.*

### 对应的 [`os`](https://docs.python.org/zh-cn/3.8/library/os.html#module-os) 模块的工具

以下是一个映射了 [`os`](https://docs.python.org/zh-cn/3.8/library/os.html#module-os) 与 [`PurePath`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath)/[`Path`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path) 对应相同的函数的表。

注解

 

尽管 [`os.path.relpath()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.relpath) 和 [`PurePath.relative_to()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.relative_to) 拥有相同的重叠的用例，但是它们语义相差很大，不能认为它们等价。

| os 和 os.path                                                | pathlib                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`os.path.abspath()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.abspath) | [`Path.resolve()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.resolve) |
| [`os.chmod()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.chmod) | [`Path.chmod()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.chmod) |
| [`os.mkdir()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.mkdir) | [`Path.mkdir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.mkdir) |
| [`os.rename()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.rename) | [`Path.rename()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.rename) |
| [`os.replace()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.replace) | [`Path.replace()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.replace) |
| [`os.rmdir()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.rmdir) | [`Path.rmdir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.rmdir) |
| [`os.remove()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.remove), [`os.unlink()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.unlink) | [`Path.unlink()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.unlink) |
| [`os.getcwd()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.getcwd) | [`Path.cwd()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.cwd) |
| [`os.path.exists()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.exists) | [`Path.exists()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.exists) |
| [`os.path.expanduser()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.expanduser) | [`Path.expanduser()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.expanduser) 和 [`Path.home()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.home) |
| [`os.listdir()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.listdir) | [`Path.iterdir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.iterdir) |
| [`os.path.isdir()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.isdir) | [`Path.is_dir()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_dir) |
| [`os.path.isfile()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.isfile) | [`Path.is_file()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_file) |
| [`os.path.islink()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.islink) | [`Path.is_symlink()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.is_symlink) |
| [`os.link()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.link) | [`Path.link_to()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.link_to) |
| [`os.symlink()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.symlink) | [`Path.symlink_to()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.symlink_to) |
| [`os.stat()`](https://docs.python.org/zh-cn/3.8/library/os.html#os.stat) | [`Path.stat()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.stat), [`Path.owner()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.owner), [`Path.group()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.group) |
| [`os.path.isabs()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.isabs) | [`PurePath.is_absolute()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.is_absolute) |
| [`os.path.join()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.join) | [`PurePath.joinpath()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.joinpath) |
| [`os.path.basename()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.basename) | [`PurePath.name`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.name) |
| [`os.path.dirname()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.dirname) | [`PurePath.parent`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.parent) |
| [`os.path.samefile()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.samefile) | [`Path.samefile()`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.Path.samefile) |
| [`os.path.splitext()`](https://docs.python.org/zh-cn/3.8/library/os.path.html#os.path.splitext) | [`PurePath.suffix`](https://docs.python.org/zh-cn/3.8/library/pathlib.html#pathlib.PurePath.suffix) |

© [版权所有](https://docs.python.org/zh-cn/3.8/copyright.html) 2001-2022, Python Software Foundation.
This page is licensed under the Python Software Foundation License Version 2.
Examples, recipes, and other code in the documentation are additionally licensed under the Zero Clause BSD License.

The Python Software Foundation is a non-profit corporation. [Please donate.](https://www.python.org/psf/donations/)

最后更新于 3月 17, 2022. [Found a bug](https://docs.python.org/3/bugs.html)?
Created using [Sphinx](https://www.sphinx-doc.org/) 2.4.4.