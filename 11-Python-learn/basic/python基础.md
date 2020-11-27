逻辑运算符

| 逻辑运算符 | 结果                                     | 说明                                                         |
| ---------- | ---------------------------------------- | ------------------------------------------------------------ |
| a and b    | if a is false, then a, else x            | 短路运算符，只有在第一个参数为真值时才会对第二个参数求值。   |
| a or b     | if a is false, then b, else a            | 短路运算符，只有在第一个参数为假值时才会对第二个参数求值。   |
| not a      | if a is false, then `True`, else `False` | `not` 的优先级比非布尔运算符低<br>`not a == b` 会被解读为 `not (a == b)` 而 `a == not b` 会引发语法错误。 |

比较运算

| 运算     | 含义           |
| :------- | :------------- |
| `<`      | 严格小于       |
| `<=`     | 小于或等于     |
| `>`      | 严格大于       |
| `>=`     | 大于或等于     |
| `==`     | 等于           |
| `!=`     | 不等于         |
| `is`     | 对象标识       |
| `is not` | 否定的对象标识 |

行边界列表

| 表示符         | 描述               |
| :------------- | :----------------- |
| `\n`           | 换行               |
| `\r`           | 回车               |
| `\r\n`         | 回车 + 换行        |
| `\v` 或 `\x0b` | 行制表符           |
| `\f` 或 `\x0c` | 换表单             |
| `\x1c`         | 文件分隔符         |
| `\x1d`         | 组分隔符           |
| `\x1e`         | 记录分隔符         |
| `\x85`         | 下一行 (C1 控制码) |
| `\u2028`       | 行分隔符           |
| `\u2029`       | 段分隔符           |

转换旗标为：

| 标志  | 含义                                                         |
| :---- | :----------------------------------------------------------- |
| `'#'` | 值的转换将使用“替代形式”（具体定义见下文）。                 |
| `'0'` | 转换将为数字值填充零字符。                                   |
| `'-'` | 转换值将靠左对齐（如果同时给出 `'0'` 转换，则会覆盖后者）。  |
| `' '` | (空格) 符号位转换产生的正数（或空字符串）前将留出一个空格。  |
| `'+'` | 符号字符 (`'+'` 或 `'-'`) 将显示于转换结果的开头（会覆盖 “空格” 旗标）。 |

转换类型为：

| 转换符 | 含义                                                         | 注释 |
| :----- | :----------------------------------------------------------- | :--- |
| `'d'`  | 有符号十进制整数。                                           |      |
| `'i'`  | 有符号十进制整数。                                           |      |
| `'o'`  | 有符号八进制数。                                             | (1)  |
| `'u'`  | 过时类型 – 等价于 `'d'`。                                    | (6)  |
| `'x'`  | 有符号十六进制数（小写）。                                   | (2)  |
| `'X'`  | 有符号十六进制数（大写）。                                   | (2)  |
| `'e'`  | 浮点指数格式（小写）。                                       | (3)  |
| `'E'`  | 浮点指数格式（大写）。                                       | (3)  |
| `'f'`  | 浮点十进制格式。                                             | (3)  |
| `'F'`  | 浮点十进制格式。                                             | (3)  |
| `'g'`  | 浮点格式。 如果指数小于 -4 或不小于精度则使用小写指数格式，否则使用十进制格式。 | (4)  |
| `'G'`  | 浮点格式。 如果指数小于 -4 或不小于精度则使用大写指数格式，否则使用十进制格式。 | (4)  |
| `'c'`  | 单个字符（接受整数或单个字符的字符串）。                     |      |
| `'r'`  | 字符串（使用 [`repr()`](https://docs.python.org/zh-cn/3.6/library/functions.html#repr) 转换任何 Python 对象）。 | (5)  |
| `'s'`  | 字符串（使用 [`str()`](https://docs.python.org/zh-cn/3.6/library/stdtypes.html#str) 转换任何 Python 对象）。 | (5)  |
| `'a'`  | 字符串（使用 [`ascii()`](https://docs.python.org/zh-cn/3.6/library/functions.html#ascii) 转换任何 Python 对象）。 | (5)  |
| `'%'`  | 不转换参数，在结果中输出一个 `'%'` 字符。                    |      |

内置异常的类层级结构如下：

```python
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration
      +-- StopAsyncIteration
      +-- ArithmeticError
      |    +-- FloatingPointError
      |    +-- OverflowError
      |    +-- ZeroDivisionError
      +-- AssertionError
      +-- AttributeError
      +-- BufferError
      +-- EOFError
      +-- ImportError
      |    +-- ModuleNotFoundError
      +-- LookupError
      |    +-- IndexError
      |    +-- KeyError
      +-- MemoryError
      +-- NameError
      |    +-- UnboundLocalError
      +-- OSError
      |    +-- BlockingIOError
      |    +-- ChildProcessError
      |    +-- ConnectionError
      |    |    +-- BrokenPipeError
      |    |    +-- ConnectionAbortedError
      |    |    +-- ConnectionRefusedError
      |    |    +-- ConnectionResetError
      |    +-- FileExistsError
      |    +-- FileNotFoundError
      |    +-- InterruptedError
      |    +-- IsADirectoryError
      |    +-- NotADirectoryError
      |    +-- PermissionError
      |    +-- ProcessLookupError
      |    +-- TimeoutError
      +-- ReferenceError
      +-- RuntimeError
      |    +-- NotImplementedError
      |    +-- RecursionError
      +-- SyntaxError
      |    +-- IndentationError
      |         +-- TabError
      +-- SystemError
      +-- TypeError
      +-- ValueError
      |    +-- UnicodeError
      |         +-- UnicodeDecodeError
      |         +-- UnicodeEncodeError
      |         +-- UnicodeTranslateError
      +-- Warning
           +-- DeprecationWarning
           +-- PendingDeprecationWarning
           +-- RuntimeWarning
           +-- SyntaxWarning
           +-- UserWarning
           +-- FutureWarning
           +-- ImportWarning
           +-- UnicodeWarning
           +-- BytesWarning
           +-- ResourceWarning
```

参考

[Python-100-Days](https://github.com/jackfrued/Python-100-Days)

[Python文档目录](https://docs.python.org/zh-cn/3.6/contents.html)