# 题库02 · Python 与 pytest

## 模块说明

**考频**：⭐⭐⭐⭐⭐ 测开岗必考，通常笔试+面试coding。占面试权重约 30%-40%。
**面试占比**：Python 基础 1-2 题 + pytest 1-2 题 + 算法手写 1 题，常是现场白板/共享文档写代码。
**你的优势**：独立全栈开发过 FastAPI+Vue3 环境信息模块，写 Python 是日常；做过报文比对自动化、自动部署模块，对"写工具解决测试痛点"有真实作品；熟练 Claude Code 等 AI 提效，可讲"AI 辅助写测试代码"。
**你的短板**：算法题（快排/链表反转）需复习手写熟练度；装饰器/生成器等进阶语法的标准表述要背牢。
**怎么用本文件**：
1. 代码题先在本地跑通，再默写一遍。
2. 概念题背【参考答案】加粗关键词。
3. 算法题记住"思路+复杂度+易错点"三件套。
4. 考前只看"二、必背清单"。

---

## 一、高频题（35 题）

### Q1 可变对象 vs 不可变对象（哪些是可变哪些不可变）
- 【考点】Python 内存模型、函数传参陷阱。
- 【参考答案】
  **不可变对象（immutable）**：创建后内容不可改，任何"修改"都生成新对象、原对象不变。包括 **int、float、bool、str、tuple、frozenset、bytes**。例如 `a=1; b=a; a+=1` 后 b 仍为 1。
  **可变对象（mutable）**：内容可原地修改，id 不变。包括 **list、dict、set、bytearray、自定义类实例**。例如 `l=[1]; l.append(2)` 后 l 还是原对象。
  关键陷阱在**函数传参**：可变对象作为参数，函数内修改会影响外部（共享引用）；不可变对象则不会。这也是为什么默认参数**绝不能用可变对象**（如 `def f(x=[])` 会跨调用累积）。
- 【结合你的经验】
  我写 FastAPI 接口和测试脚本时常踩这个坑。比如批量导入 Excel 时，默认参数我一定写 `def parse(rows=None): rows = rows or []`，绝不用 `[]` 当默认值，否则多笔请求会串数据。这种细节在金融批量处理里很要命，我养成习惯了。
- 【追问链】
  - 为什么 tuple 是不可变但能放 list？→ tuple 存的是引用，引用不可变、指向的对象可变。
  - 字符串拼接为什么慢？→ 不可变，每次 + 生成新对象，应用 join。
- [ ] 已掌握

### Q2 深拷贝 vs 浅拷贝（配代码）
- 【考点】引用复制、嵌套结构。
- 【参考答案】
  **浅拷贝（copy.copy / 切片 / list() / dict.copy）**只复制第一层，嵌套对象仍共享引用；修改拷贝中的嵌套对象会影响原对象。
  **深拷贝（copy.deepcopy）**递归复制所有层，完全独立。
  ```python
  import copy
  a = {'x': [1, 2], 'y': 3}
  b = copy.copy(a)      # 浅拷贝
  b['x'].append(3)      # a['x'] 变 [1,2,3]，因为共享
  c = copy.deepcopy(a)  # 深拷贝
  c['x'].append(4)      # a 不受影响
  ```
  性能：浅拷贝快、深拷贝慢（递归）。仅一层结构时两者等价。
- 【结合你的经验】
  报文比对自动化里，我会深拷贝原始期望报文再做"忽略节点"的裁剪，避免污染基线模板——因为模板要被 40+ 渠道复用，一旦被浅拷贝共享改坏，所有渠道比对都错。这个坑我踩过，现在一律 deepcopy。
- 【追问链】
  - 切片是深还是浅？→ 浅拷贝（只第一层独立）。
  - 循环引用 deepcopy 会怎样？→ Python 有记忆机制防死循环。
- [ ] 已掌握

### Q3 列表去重的几种方式（配代码）
- 【考点】集合、有序保持、性能。
- 【参考答案】
  常见四种：
  ```python
  lst = [3, 1, 3, 2, 1]
  # 1. 集合（无序，最快）
  list(set(lst))
  # 2. 保持顺序（字典 fromkeys，Py3.7+ 保序）
  list(dict.fromkeys(lst))          # [3,1,2]
  # 3. 列表推导（保持顺序，可读）
  seen = set(); [x for x in lst if not (x in seen or seen.add(x))]
  # 4. 循环 append
  res = []; [res.append(x) for x in lst if x not in res]
  ```
  去重需按"值"还是"对象"？对象去重要用 `key=` 或 `(id, hash)`。性能：set 为 O(n)，优先。
- 【结合你的经验】
  我在环境信息模块做 Excel 批量导入去重时用 `dict.fromkeys` 保序，因为用户期望"按原表顺序保留第一条"。还有渠道号去重，直接用 set 即可。pytest 参数化里也常用 set 对用例 ID 去重，防止重复执行。
- 【追问链】
  - 按对象属性去重怎么做？→ 用 key 函数或 dict 按属性分组。
  - 大数据量去重？→ 流式处理+set，注意内存。
- [ ] 已掌握

### Q4 列表 vs 元组 vs 集合 vs 字典区别与使用场景
- 【考点】数据结构选型。
- 【参考答案】
  - **list**：有序、可重复、可变。场景：序列数据、需增删改。
  - **tuple**：有序、可重复、**不可变**。场景：固定结构记录（坐标、配置项）、字典 key、函数返回多值。
  - **set**：无序、不可重复、**可变**（frozenset 不可变）。场景：去重、成员判断、交集并集。
  - **dict**：键值对、键唯一、可变、Py3.7+ 保序。场景：映射、缓存、配置。
  选型原则：要顺序+改→list；要不可变/做 key→tuple；要查重/集合运算→set；要映射→dict。
- 【结合你的经验】
  报文比对里"忽略节点列表"我用 tuple（不可变、可放心当配置）；测试用例集用 list；渠道去重用 set；环境配置映射用 dict。选对结构代码既清晰又少 bug——这也是我写 FastAPI 时的习惯。
- 【追问链】
  - dict 为什么快？→ 哈希表，平均 O(1)。
  - tuple 能做 dict key 为什么 list 不能？→ tuple 可 hash（不可变），list 不可 hash。
- [ ] 已掌握

### Q5 字符串常用操作（拼接/切分/格式化/常用方法）
- 【考点】字符串 API 熟练度。
- 【参考答案】
  ```python
  # 拼接：优先 join（不可变，+ 慢）
  ','.join(['a','b'])          # 'a,b'
  # 切分
  'a,b,c'.split(',')           # ['a','b','c']
  'a.b.c'.rsplit('.',1)        # ['a.b','c']
  # 格式化：f-string（Py3.6+ 推荐）
  f'{name}-{age:03d}'          # 数字补零
  # 常用方法
  s.strip() / s.lower() / s.upper()
  s.replace('a','b')
  s.startswith('x') / s.endswith('.log')
  s.find('x')                  # 找不到返回 -1
  s.count('x')
  '{}'.format(x)
  ```
  易错：字符串不可变，所有方法返回新串。
- 【结合你的经验】
  报文里字段截取、银行渠道名匹配我经常用这些。比如把回盘报文按分隔符 split 后逐字段比对，用 f-string 拼凑期望报文。还有大小写严格匹配——某银行节点名校验大小写，我用 `.lower()` 统一归一化再比对，拦过一次生产事故。
- 【追问链】
  - join 为什么比 + 快？→ 预先算长度一次分配，+ 多次拷贝。
  - 怎么判断子串？→ in 操作符，比 find 直观。
- [ ] 已掌握

### Q6 列表推导式（配 3 个例子）
- 【考点】Pythonic 写法、可读性。
- 【参考答案】
  列表推导式 `[expr for item in iterable if cond]`，比 for+append 更简洁、通常更快。
  ```python
  # 1. 平方
  [x*x for x in range(5)]                 # [0,1,4,9,16]
  # 2. 带过滤：偶数的平方
  [x*x for x in range(10) if x % 2 == 0] # [0,4,16,36,64]
  # 3. 嵌套扁平化
  [n for row in [[1,2],[3,4]] for n in row]  # [1,2,3,4]
  ```
  注意：逻辑复杂时（多层 if-else）应改回普通循环，可读性优先。
- 【结合你的经验】
  我写测试数据构造时大量用推导式。比如从 1900+ 用例里筛 P0：`[c for c in cases if c.priority=='P0']`；构造批量支付报文：`[{**base, 'amt': a} for a in amounts]`。读起来一行顶五行的 for，效率也高。
- 【追问链】
  - 推导式 vs map/filter？→ 推导式可读性好，map 配 lambda 偏函数式。
  - 字典/集合推导式？→ {k:v for...} / {x for...}。
- [ ] 已掌握

### Q7 生成器 vs 迭代器（yield 是什么）
- 【考点】惰性求值、内存优化。
- 【参考答案】
  **迭代器（Iterator）**是实现了 `__iter__` 和 `__next__` 的对象，用 `next()` 逐个取值，耗尽抛 StopIteration。可迭代对象（list 等）用 `iter()` 转迭代器。
  **生成器（Generator）**是一种简易迭代器，用**含 `yield` 的函数**创建；每次 `next()` 执行到 yield 暂停并返回值，保留局部状态，下次从暂停处继续。优势：**惰性、省内存**（不一次性生成全部）。
  ```python
  def gen(n):
      for i in range(n):
          yield i*i
  g = gen(3)
  next(g)  # 0, 1, 4
  ```
  大数据/流式（如读取大文件、批量报文）首选生成器。
- 【结合你的经验】
  报文比对处理大批量回盘时，我用生成器逐条 yield 解析后的报文，而不是一次性 load 进 list，内存从几个 G 降到几十 M。还有日志回放，用生成器边读边比，特别适合渠道几十万笔的场景。
- 【追问链】
  - 生成器只能遍历一次？→ 是，用完即尽，需重新创建。
  - yield from 干嘛用？→ 委托子生成器，扁平嵌套。
- [ ] 已掌握

### Q8 装饰器原理（手写：计时装饰器，完整代码）
- 【考点】高阶函数、闭包、语法糖。
- 【参考答案】
  装饰器本质是**接受函数、返回新函数的高阶函数**，用于在不改原函数代码下增强功能。`@dec` 等价于 `f = dec(f)`。
  ```python
  import time

  def timer(func):
      def wrapper(*args, **kwargs):
          start = time.perf_counter()
          result = func(*args, **kwargs)
          print(f'{func.__name__} 耗时 {time.perf_counter()-start:.4f}s')
          return result
      return wrapper

  @timer
  def add(a, b):
      return a + b

  add(1, 2)
  ```
  关键点：`*args,**kwargs` 透传参数并 `return result` 保留返回值；可用 `functools.wraps` 保留元信息。
- 【结合你的经验】
  我写测试工具时离不开装饰器。`@timer` 用来量每个接口用例耗时；我还写 `@retry` 做失败重试、`@log_call` 记出入参。这些在我自动部署模块和报文比对里都是基础设施，可以说是"测开标配"。
- 【追问链】
  - 为什么要用 functools.wraps？→ 保留原函数名/文档，利于调试。
  - 带参数的装饰器？→ 三层嵌套：外层收参数，内层收函数。
- [ ] 已掌握

### Q9 手写重试装饰器（测试中重试失败用例场景，完整代码）
- 【考点】装饰器实战、测试稳定性。
- 【参考答案】
  Flaky 用例/偶发网络抖动时，重试能提升稳定性。带次数与间隔：
  ```python
  import time
  from functools import wraps

  def retry(times=3, delay=1, exceptions=(Exception,)):
      def deco(func):
          @wraps(func)
          def wrapper(*args, **kwargs):
              last = None
              for i in range(times):
                  try:
                      return func(*args, **kwargs)
                  except exceptions as e:
                      last = e
                      if i < times - 1:
                          time.sleep(delay)
              raise last
          return wrapper
      return deco

  @retry(times=3, delay=2)
  def call_bank_api():
      ...
  ```
  注意：重试只适合**幂等**操作，且要限制次数防雪崩。
- 【结合你的经验】
  银行渠道偶发超时，我在接口自动化里就用这个 `@retry` 包一层，3 次间隔 2 秒重试，把"假失败"滤掉。但我会严格区分：真正逻辑错（断言失败）不重试，只有网络/超时类异常才重试，避免掩盖真 bug。
- 【追问链】
  - 断言失败也重试吗？→ 不，断言失败是逻辑错，重试无意义。
  - 重试会不会放大问题？→ 会，必须限次+只重试幂等操作。
- [ ] 已掌握

### Q10 手写日志装饰器（完整代码）
- 【考点】装饰器+日志+AOP 思维。
- 【参考答案】
  统一记录函数调用入参、出参、异常，便于排查：
  ```python
  import logging, time
  from functools import wraps

  logging.basicConfig(level=logging.INFO,
      format='%(asctime)s %(levelname)s %(message)s')

  def log_call(func):
      @wraps(func)
      def wrapper(*args, **kwargs):
          logging.info(f'→ {func.__name__} args={args} kwargs={kwargs}')
          try:
              result = func(*args, **kwargs)
              logging.info(f'← {func.__name__} return={result}')
              return result
          except Exception as e:
              logging.exception(f'✗ {func.__name__} raised {e}')
              raise
      return wrapper

  @log_call
  def parse_xml(s):
      return s.upper()
  ```
  这是 AOP 思路，在测试工具中极常见。
- 【结合你的经验】
  我的报文比对工具每个解析函数都挂 `@log_call`，生产排查时一眼能看到"哪个渠道、哪笔报文、入参出参是什么"。结合 k8s 日志，我能快速定位到某银行回盘字段映射问题。这套是我在客户现场支持 6 个月磨出来的排障习惯。
- 【追问链】
  - 为什么 except 里还要 raise？→ 日志归日志，异常仍需向上抛。
  - 日志写到文件？→ 配 FileHandler，测试用临时日志。
- [ ] 已掌握

### Q11 闭包是什么
- 【考点】嵌套函数、变量捕获。
- 【参考答案】
  **闭包**是"内层函数 + 其引用的外层函数局部变量"的组合。内层函数即使在外层函数返回后，仍能访问那些被捕获的变量（存在 `__closure__` 中）。
  ```python
  def outer(x):
      def inner(y):
          return x + y   # x 被捕获
      return inner
  f = outer(10)
  f(5)   # 15，x 仍为 10
  ```
  用途：装饰器、工厂函数、保持状态（如计数器）。注意循环里闭包引用循环变量要用默认参数固化，否则全捕获最后一值。
- 【结合你的经验】
  我写装饰器（计时/重试/日志）本质都靠闭包捕获 func 和参数。还有做接口 base_url 工厂：返回一个带上固定 host 的请求函数，就是闭包。理解闭包对我写测试框架 helper 很有用。
- 【追问链】
  - 闭包和类有什么区别？→ 闭包轻量存状态，类更结构化。
  - late binding 坑？→ 循环变量在调用时才取值，用默认参数固化。
- [ ] 已掌握

### Q12 *args 和 **kwargs
- 【考点】参数打包、透传。
- 【参考答案】
  `*args` 收集**位置参数**为元组，`**kwargs` 收集**关键字参数**为字典。用于函数接收不定参数或透传给其他函数。
  ```python
  def f(*args, **kwargs):
      print(args, kwargs)
  f(1, 2, name='x')   # (1, 2) {'name':'x'}

  def wrapper(*a, **k):
      return target(*a, **k)   # 透传
  ```
  顺序固定：`def f(普通, *args, 默认=, **kwargs)`。调用侧 `*` 解包列表、`**` 解包字典。
- 【结合你的经验】
  我写装饰器时 `def wrapper(*args, **kwargs): return func(*args, **kwargs)` 就是用它透传任意参数。测试夹具里也常用 `**kwargs` 收额外配置，比如 `@retry(times=3)` 这种参数化装饰器。
- 【追问链】
  - args 一定是元组吗？→ 是，位置参数打包成 tuple。
  - 只想要关键字参数？→ 用 `*,` 强制关键字（Python 3）。
- [ ] 已掌握

### Q13 Python 异常处理（try/except/else/finally，自定义异常）
- 【考点】异常机制、健壮代码。
- 【参考答案】
  ```python
  try:
      risky()
  except ValueError as e:     # 捕获指定异常
      handle(e)
  except (A, B) as e:         # 多异常
      ...
  else:
      print('无异常才执行')     # try 成功走
  finally:
      print('始终执行')         # 收尾/释放资源
  ```
  `else` 在无异常时执行（try 体不该太胖）；`finally` 常用于关连接/删临时文件。**自定义异常**继承 Exception，便于精确捕获：
  ```python
  class BankChannelError(Exception): pass
  raise BankChannelError('节点名不匹配')
  ```
  原则：捕具体异常别裸 `except:`，避免吞掉 KeyboardInterrupt/SystemExit。
- 【结合你的经验】
  报文比对里我自定义 `CompareDiffError`、`NodeMissingError`，断言差异时 raise 具体异常，pytest 能精准分类失败原因。finally 里我一定关掉数据库连接和临时文件——金融批量处理忘了关连接会撑爆连接池，我吃过亏。
- 【追问链】
  - else 和 finally 区别？→ else 仅成功执行，finally 总执行。
  - 为什么不用裸 except？→ 会吞掉系统退出信号，难调试。
- [ ] 已掌握

### Q14 with 上下文管理器原理
- 【考点】资源管理、__enter__/__exit__。
- 【参考答案】
  `with` 用于**自动资源管理**（文件、连接、锁），即使异常也能正确释放。原理是实现**上下文管理协议**：`__enter__` 进入时调用并返回资源，`__exit__(exc_type,exc, tb)` 退出时调用（返回 True 吞异常）。
  ```python
  with open('a.txt') as f:   # 自动 close
      data = f.read()
  ```
  也可用 `@contextmanager` 装饰器快速写：
  ```python
  from contextlib import contextmanager
  @contextmanager
  def tag(name):
      print(f'<{name}>'); yield; print(f'</{name}>')
  ```
  pytest 的 fixture 底层就是上下文管理思想。
- 【结合你的经验】
  我读大报文文件、连数据库都用 `with`，绝不会漏关。pytest 的 fixture yield 也是同一套——`yield` 前准备、`yield` 后清理。我在接口自动化里用 fixture 管理数据库连接，和 with 是相通的，理解这个对写 fixture 很有帮助。
- 【追问链】
  - exit 返回 True 会怎样？→ 吞掉异常，慎用。
  - 多个 with 怎么写？→ 用反斜杠或嵌套，Py3.10+ 支持多括号。
- [ ] 已掌握

### Q15 GIL 是什么？多线程/多进程/协程怎么选？（IO 密集 vs CPU 密集）
- 【考点】并发模型、性能选型。
- 【参考答案】
  **GIL（全局解释器锁）**是 CPython 的互斥锁，保证同一时刻只有一个线程执行字节码，导致**多线程无法真正并行 CPU 计算**（多核无效）。
  - **CPU 密集**（计算、加解密）：用**多进程**（multiprocessing）绕过 GIL，吃满多核；或换 C 扩展/NumPy。
  - **IO 密集**（网络、文件、数据库）：用**多线程**或**协程（asyncio）**，等待 IO 时释放 GIL，线程切换开销小；协程单线程高并发更优。
  测试场景：JMeter 压测是另说；Python 写并发请求用 `concurrent.futures.ThreadPoolExecutor` 或 asyncio 更合适。
- 【结合你的经验】
  我批量回放报文做比对时，是 IO 密集（等银行模拟响应），用 `ThreadPoolExecutor` 开多线程并发，比单线程快几倍；而报文解析计算量大时我用多进程。pytest-xdist 多进程跑用例也是吃多核，我做回归提速就是靠它。
- 【追问链】
  - GIL 在 PyPy/Jython 有吗？→ 仅 CPython 有。
  - asyncio 适合什么？→ 高并发 IO，如万级连接爬虫。
- [ ] 已掌握

### Q16 Python 文件读写与 os/pathlib 常用操作
- 【考点】文件 API、路径处理。
- 【参考答案】
  ```python
  # 读
  with open('a.txt', encoding='utf-8') as f:
      lines = f.readlines()
  # 写
  with open('b.txt','w',encoding='utf-8') as f:
      f.write('hi')
  # json
  import json
  json.dump(obj, open('c.json','w')); json.load(open('c.json'))
  # os
  import os
  os.path.exists(p); os.makedirs(p, exist_ok=True)
  os.listdir(p); os.remove(p)
  # pathlib（推荐，面向对象）
  from pathlib import Path
  p = Path('data/a.xml')
  p.read_text(encoding='utf-8')
  p.write_text('x')
  p.parent.mkdir(parents=True, exist_ok=True)
  list(p.parent.glob('*.xml'))
  ```
  pathlib 跨平台、链式调用更优雅。
- 【结合你的经验】
  报文比对里我用 pathlib 批量扫描渠道报文目录 `Path(dir).rglob('*.xml')`，读出来做差异比对，比 os 拼路径清爽太多。Excel 批量导入模块也用 pathlib 处理上传文件。金融系统文件名常带日期，用 glob 按 `2026*.xml` 筛选很方便。
- 【追问链】
  - read 和 readlines 区别？→ 前者整体字符串，后者按行列表。
  - 大文件怎么读？→ 迭代文件对象逐行，或分块。
- [ ] 已掌握

---

### Q17 pytest vs unittest 区别？为什么 pytest 是主流？
- 【考点】框架对比、选型理由。
- 【参考答案】
  **unittest** 是 Python 标准库，类式风格（TestCase/assert*），需继承、写 `self.assertEqual`，boilerplate 多；**pytest** 是第三方，函数式、自动发现、断言用原生 `assert`、fixture 依赖注入、丰富插件（xdist/allure/parametrize）。
  pytest 主流原因：①**极简**：原生 assert+自动发现；②**fixture** 灵活可复用、作用域清晰；③**parametrize** 数据驱动优雅；④**插件生态**强大（并行、报告、失败重试）；⑤**兼容 unittest**。测开项目几乎标配 pytest。
- 【结合你的经验】
  我做接口自动化和环境信息模块的后台测试都选 pytest。尤其 fixture 做 token 管理和数据库连接、parametrize 做多银行多场景，比 unittest 的 setUp/tearDown 好用太多。我们 1900+ 用例的回归如果有自动化，pytest 是底座。
- 【追问链】
  - pytest 能跑 unittest 用例吗？→ 能，自动兼容。
  - 缺点？→ 需额外装，极大型项目需自己规划目录。
- [ ] 已掌握

### Q18 pytest 用例命名规则与发现机制
- 【考点】用例识别、约定。
- 【参考答案】
  pytest 自动发现规则：
  - **文件**：`test_*.py` 或 `*_test.py`
  - **函数/类**：`test_*` 开头；类中方法也 `test_*`，且类名以 `Test` 开头（**不能带 `__init__`**）
  - **目录**：含 `__init__.py` 的包也能被收集
  执行：`pytest` 递归收集当前目录；`pytest test_api.py::TestLogin::test_ok` 精确定位；`-k "login"` 按关键字；`-m smoke` 按标记。
  断言失败 pytest 会显示详细 diff，无需手写消息。
- 【结合你的经验】
  我接渠道回归时，按银行分目录 `tests/icbc/test_pay.py`，函数 `test_pay_success`、`test_pay_timeout`，收集清晰。配合 `-k` 能只跑某渠道，提速排查。这套命名规范是我维护多银行用例库的基线。
- 【追问链】
  - 想自定义发现规则？→ 配 pytest.ini 的 python_files/python_classes。
  - 类为什么不能有 __init__？→ 会被当普通类不收集。
- [ ] 已掌握

### Q19 fixture 是什么？4 种作用域详解（配代码）
- 【考点】fixture 核心、作用域。
- 【参考答案】
  **fixture** 是 pytest 的"依赖注入"机制，用 `@pytest.fixture` 标记函数，作为测试函数的参数传入，负责**准备与清理**（替代 setUp/tearDown）。
  **4 种作用域（scope）**：
  - `function`（默认）：每个测试函数一次
  - `class`：每个测试类一次
  - `module`：每个 .py 文件一次
  - `session`：整个测试会话一次（如全局登录 token、DB 连接）
  ```python
  @pytest.fixture(scope='session')
  def db():
      conn = create_conn()
      yield conn          # 测试用
      conn.close()        # 清理
  def test_a(db):
      assert db.query(1) == 1
  ```
  yield 前准备、yield 后清理，异常也保证清理（类似 try/finally）。
- 【结合你的经验】
  我做接口自动化时，把"登录拿 token"做成 `scope='session'` 的 fixture，全会话只登录一次；数据库连接 `scope='module'`。这大幅提速——不像 setUp 每个用例重连。我推的回归提速，fixture 作用域优化是关键技术点之一。
- 【追问链】
  - 怎样让 fixture 只在失败时清理？→ 用 yield + 条件判断；或 request 对象。
  - fixture 间能依赖吗？→ 能，fixture 参数可引用其他 fixture。
- [ ] 已掌握

### Q20 conftest.py 机制（多个目录怎么共享 fixture）
- 【考点】fixture 共享、层级。
- 【参考答案】
  `conftest.py` 是 pytest 的**本地插件文件**，无需 import，放在目录中即可让该目录及子目录的测试自动使用其中的 fixture 和 hook。
  **共享规则（就近原则）**：测试文件会向上逐级查找父目录的 conftest.py，最近一级优先。例如：
  ```
  tests/
    conftest.py        # 全局 fixture（登录、db）
    channel/
      conftest.py      # 渠道专用 fixture
      icbc/test_pay.py
  ```
  `test_pay.py` 可用 tests/conftest 和 channel/conftest 的全部 fixture。注意：conftest.py 不要写 `__init__.py`，避免变成包导入问题。
- 【结合你的经验】
  我按"全局+渠道"两级 conftest 组织：根 conftest 放登录 token、DB 连接；每个银行目录 conftest 放该渠道专属配置（如证书路径）。这样 40+ 渠道复用一套底座，专属各自扩展——和我的"模块化用例库"思路一致。
- 【追问链】
  - conftest 能跨项目吗？→ 用 pytest 插件或根 conftest 共享。
  - 放 fixture 还是放普通函数？→ fixture 放 conftest，工具函数放 utils。
- [ ] 已掌握

### Q21 @pytest.mark.parametrize 数据驱动（配登录多组数据完整代码）
- 【考点】参数化、数据驱动。
- 【参考答案】
  `parametrize(argnames, argvalues)` 把多组数据注入同一测试函数，每组生成独立用例，失败互不干扰，Allure 可区分。
  ```python
  import pytest

  login_cases = [
      ("user1", "pwd1", True,  "正常登录"),
      ("",      "pwd1", False, "用户名为空"),
      ("user1", "",     False, "密码为空"),
      ("user1", "err",  False, "密码错误"),
  ]

  @pytest.mark.parametrize("user,pwd,ok,desc", login_cases)
  def test_login(user, pwd, ok, desc):
      r = do_login(user, pwd)
      assert r.success == ok
  ```
  也可叠多层 parametrize（笛卡尔积）；`ids=` 自定义用例名便于阅读。
- 【结合你的经验】
  我接银行渠道时，把"各渠道参数组合"做成 parametrize 数据源，一条测试函数覆盖 40+ 渠道，新增渠道只加数据不写代码——这正是数据驱动。结合 conftest 的 fixture 注入 base_url，维护成本极低，和我"用例库结构化"理念一脉相承。
- 【追问链】
  - 多参数笛卡尔积？→ 多个 parametrize 装饰器叠加。
  - 怎么给用例起好名字？→ ids= 函数或列表。
- [ ] 已掌握

### Q22 fixture 和 setup/teardown 对比
- 【考点】新旧机制差异。
- 【参考答案】
  | 维度 | unittest setup/tearDown | pytest fixture |
  |---|---|---|
  | 写法 | 类方法，需继承 | 装饰器函数，参数注入 |
  | 复用 | 靠继承，耦合高 | 任意函数参数引用，解耦 |
  | 作用域 | 方法/类两级 | function/class/module/session 四级 |
  | 依赖 | 难组合 | fixture 可依赖其他 fixture |
  | 清理 | tearDown 固定 | yield 后清理，灵活 |
  fixture 优势明显：更 Pythonic、可组合、作用域细、命名直观。setup/tearDown 仅在与 unittest 兼容时保留。
- 【结合你的经验】
  我从 unittest 切到 pytest 后，最大的爽点是 fixture 能"按需注入"——一个测试需要 DB 就加个参数，不需要的不连。不像 setUp 把所有资源都准备好，浪费且慢。我做的回归提速里，fixture 作用域优化功不可没。
- 【追问链】
  - 能混用吗？→ 能，pytest 兼容 unittest 的 setup。
  - fixture 命名冲突？→ 就近原则，子目录覆盖父目录。
- [ ] 已掌握

### Q23 pytest.mark 常用标记（skip/xfail/自定义 mark）
- 【考点】标记机制、用例分类。
- 【参考答案】
  - `@pytest.mark.skip(reason)`：无条件跳过（如环境不满足）。
  - `@pytest.mark.skipif(cond, reason)`：条件跳过（如 Py 版本）。
  - `@pytest.mark.xfail(reason)`：预期失败，通过算意外、失败算预期，不计入红。
  - `@pytest.mark.parametrize`：数据驱动。
  - **自定义 mark**：如 `@pytest.mark.smoke`，配合 `pytest -m smoke` 筛选；需在 pytest.ini 注册避免警告：
    ```ini
    [pytest]
    markers =
        smoke: 冒烟用例
        channel: 渠道回归
    ```
  标记用于分层执行（冒烟/回归/性能），是 CI 中按门禁跑用例的基础。
- 【结合你的经验】
  我给 1900+ 用例打 `smoke`/`channel`/`p0` 等标记。CI 里先跑 `-m smoke` 做准入，再全量回归。新渠道接入只跑 `-m channel and icbc` 精准验证。这套标记体系让我"时间紧也能按风险挑用例"，和我题库01讲的 RBT 策略呼应。
- 【追问链】
  - xfail 和 skip 区别？→ xfail 仍执行、预期失败；skip 不执行。
  - 不注册自定义 mark 会怎样？→ 警告，建议注册。
- [ ] 已掌握

### Q24 pytest 的 hook 机制
- 【考点】插件扩展、pytest 生命周期。
- 【参考答案】
  pytest 通过**钩子函数（hook）**开放生命周期扩展点，命名 `pytest_*` 放在 conftest.py 或插件中，框架在特定时机调用。常用：
  - `pytest_collection_modifyitems`：收集后改写用例（如自动加 mark、按 tag 排序）。
  - `pytest_runtest_setup/makereport`：用例前后/报告钩子。
  - `pytest_addoption`：注册命令行参数（如 `--env`）。
  - `pytest_configure`：读取配置。
  例：收集时给所有用例加 smoke 标记或按优先级排序：
  ```python
  def pytest_collection_modifyitems(items):
      for item in items:
          if 'login' in item.name:
              item.add_marker(pytest.mark.smoke)
  ```
- 【结合你的经验】
  我做多环境切换就用 `pytest_addoption` 加 `--env staging/prod`，conftest 读它决定 base_url 和证书。还用 `pytest_collection_modifyitems` 自动按用例优先级排序执行，让 P0 先跑、早暴露问题。这些 hook 让我把测试框架做得像产品。
- 【追问链】
  - hook 和 fixture 区别？→ hook 改框架行为，fixture 注资源。
  - 冲突的 hook 执行顺序？→ 按插件注册顺序，可指定。
- [ ] 已掌握

### Q25 接口自动化中 token 依赖怎么处理？（fixture 提取 token 完整代码）
- 【考点】鉴权处理、session 级 fixture。
- 【参考答案】
  登录 token 应在全会话获取一次，供所有用例复用，用 `scope='session'` 的 fixture 缓存：
  ```python
  import pytest, requests

  @pytest.fixture(scope='session')
  def token():
      r = requests.post('https://api/login',
                        json={'user':'t','pwd':'p'})
      assert r.status_code == 200
      return r.json()['token']

  @pytest.fixture(scope='session')
  def auth_header(token):
      return {'Authorization': f'Bearer {token}'}

  def test_query(auth_header):
      r = requests.get('https://api/balance',
                       headers=auth_header)
      assert r.status_code == 200
  ```
  若 token 过期，可在 fixture 内加失效判断或 `autouse` 刷新。我的环境信息模块正是 JWT+RBAC，理解这套正好。
- 【结合你的经验】
  我做接口回归时就是这套：session 级 token fixture + auth_header 注入。我们系统用 JWT，token 有效期长，session 级够用；若短时效，我会用 `request` 对象检测 401 自动刷新。这也和我独立开发的 JWT 鉴权模块经验对得上。
- 【追问链】
  - token 过期怎么刷新？→ fixture 内捕获 401 重新登录。
  - 多用户怎么办？→ parametrize 不同账号的 token fixture。
- [ ] 已掌握

### Q26 自动化用例不稳定（Flaky）6 种解法
- 【考点】稳定性治理、工程素养。
- 【参考答案】
  Flaky（偶发失败）6 类解法：
  1. **显式等待**：用 `WebDriverWait`/轮询接口状态，别用固定 `sleep`。
  2. **重试机制**：`@retry` 或 pytest-rerunfailures，仅限幂等/环境抖动。
  3. **数据隔离**：每用例独立测试数据，避免相互污染（用 fixture 造数+清理）。
  4. **环境稳定**：统一测试数据、Mock 第三方（银行渠道用 Mock 服务）。
  5. **断言精准**：等状态真正就绪再断言，避免竞态（如"处理中"误判）。
  6. **并行隔离**：pytest-xdist 下保证用例间无共享状态/端口冲突。
  根因通常是：竞态、脏数据、弱网、时序——定位要先复现再治。
- 【结合你的经验】
  银行渠道偶发超时就是 Flaky 元凶，我用 `@retry` 滤网络抖动，但只重试异常不重试断言失败。更关键是 Mock 渠道：我搭了模拟银行回盘服务，让回归不依赖真实网络，稳定性从"时好时坏"到"全绿"。这套经验是我做报文比对自动化的底气。
- 【追问链】
  - 重试掩盖真 bug 怎么办？→ 只对网络异常重试，断言失败不重试+记录。
  - 怎么量化 Flaky？→ 统计同用例多次运行失败率。
- [ ] 已掌握

### Q27 pytest-xdist 并行执行 + allure 报告集成
- 【考点】并行、报告、CI。
- 【参考答案】
  **pytest-xdist** 多进程并行：`pytest -n 4`（或 `-n auto` 按 CPU）。注意用例须**相互独立**（无共享状态），否则并行出错；可用 `--dist=loadscope` 按模块分进程。
  **allure** 生成美观报告：装 `allure-pytest`，用例加 `@allure.title/severity/feature`，运行 `pytest --alluredir=tmp` 后 `allure serve tmp` 看报告。
  ```bash
  pip install pytest-xdist allure-pytest
  pytest -n auto --alluredir=report
  allure serve report
  ```
  二者常接 CI（Jenkins/GitHub Actions），并行提速+报告留痕，是测开交付标准。
- 【结合你的经验】
  我把 1900+ 用例的回归用 `-n auto` 并行，配合报文比对 0.5 人天兜住渠道回归，整体回归从 2 人天压到 0.5 人天，并行是重要一环。Allure 报告我贴在测试报告里，领导能直观看通过率和失败分布——这也是我讲"测试价值量化"的素材。
- 【追问链】
  - 并行下 token fixture 怎么不冲突？→ session 级共享，或用锁。
  - allure 怎么标严重度？→ @allure.severity 分级。
- [ ] 已掌握

---

### Q28 字符串反转与回文判断
- 【考点】字符串操作、双指针。
- 【参考答案】
  ```python
  # 反转
  def reverse_str(s: str) -> str:
      return s[::-1]

  # 回文判断（忽略非字母数字+大小写）
  def is_palindrome(s: str) -> bool:
      t = ''.join(c.lower() for c in s if c.isalnum())
      return t == t[::-1]
  ```
  **复杂度**：时间 O(n)，空间 O(n)（切片新建）。回文也可用双指针 O(1) 额外空间。
  **面试注意**：①是否忽略大小写/标点；②空串、单字符算回文；③中文回文直接比。先问清边界再写。
- 【结合你的经验】
  这类题我常用在报文校验里——比如回盘报文某些字段要求正反一致。虽然业务里不会真写回文，但手写算法题时我会先和面试官确认"是否忽略大小写/符号"，展示工程严谨，而不是上来就写。
- 【追问链】
  - 双指针怎么写？→ left,right 向中间比，遇不等返回 False。
  - 空间 O(1) 回文？→ 双指针不建新串。
- [ ] 已掌握

### Q29 两数之和（哈希法）
- 【考点】哈希表、O(n)。
- 【参考答案】
  ```python
  def two_sum(nums, target):
      seen = {}              # 值 -> 索引
      for i, x in enumerate(nums):
          need = target - x
          if need in seen:
              return [seen[need], i]
          seen[x] = i
      return []
  ```
  **复杂度**：时间 O(n)，空间 O(n)。比暴力双重循环 O(n²) 优。
  **注意**：返回下标（非值）；假设恰有一解；可返回任意一组。先说明假设。
- 【结合你的经验】
  算法题我习惯先讲思路再写。两数之和我先说"暴力 O(n²) 会超时，用哈希把查找从 O(n) 降到 O(1)"，再写。面试官看的是"优化意识"，和我做报文比对用哈希索引加速匹配是同一思维。
- 【追问链】
  - 返回所有解？→ 收集列表，注意去重。
  - 有序数组更快？→ 双指针 O(n) 无需哈希。
- [ ] 已掌握

### Q30 斐波那契数列（3 种写法对比）
- 【考点】递归、记忆化、迭代。
- 【参考答案】
  ```python
  # 1. 朴素递归（慢，O(2^n) 重复计算）——仅展示
  def fib_rec(n):
      return n if n < 2 else fib_rec(n-1) + fib_rec(n-2)

  # 2. 记忆化（lru_cache，O(n)）
  from functools import lru_cache
  @lru_cache
  def fib_memo(n):
      return n if n < 2 else fib_memo(n-1) + fib_memo(n-2)

  # 3. 迭代（最优，O(n) 空间 O(1)）
  def fib_iter(n):
      a, b = 0, 1
      for _ in range(n):
          a, b = b, a + b
      return a
  ```
  **对比**：递归易读但指数爆炸；记忆化改平方/线性；迭代最稳，面试优先写迭代或记忆化，并说明复杂度。
- 【结合你的经验】
  这题我必提"避免重复计算"——和我做报文比对用缓存避免重复解析同一报文是同一思路。面试我会主动说"生产里绝不用朴素递归，会用 lru_cache 或迭代"，体现工程落地意识而非只背答案。
- 【追问链】
  - 超大 n 怎么算？→ 矩阵快速幂 O(log n) 或取模。
  - lru_cache 注意什么？→ 参数须可哈希。
- [ ] 已掌握

### Q31 冒泡排序
- 【考点】基础排序、稳定性。
- 【参考答案】
  ```python
  def bubble_sort(arr):
      n = len(arr)
      for i in range(n):
          swapped = False
          for j in range(n - 1 - i):
              if arr[j] > arr[j + 1]:
                  arr[j], arr[j + 1] = arr[j + 1], arr[j]
                  swapped = True
          if not swapped:        # 提前退出优化
              break
      return arr
  ```
  **复杂度**：平均/最坏 O(n²)，最好 O(n)（已近有序+优化）；空间 O(1)；**稳定**。
  **注意**：加 `swapped` 标志可提前结束；讲清稳定性（相等不交换）。
- 【结合你的经验】
  冒泡是基础题，我写时一定会加"提前退出"优化并说出最好情况 O(n)，这才是测开该有的"不仅写出来还讲优化"的素养。我写工具排序用例数据时用过类似思路。
- 【追问链】
  - 为什么稳定？→ 相等不交换，相对位置不变。
  - 和选择排序区别？→ 选择不稳定、交换少。
- [ ] 已掌握

### Q32 二分查找
- 【考点】有序数组、边界。
- 【参考答案】
  ```python
  def binary_search(arr, target):
      lo, hi = 0, len(arr) - 1
      while lo <= hi:
          mid = (lo + hi) // 2
          if arr[mid] == target:
              return mid
          elif arr[mid] < target:
              lo = mid + 1
          else:
              hi = mid - 1
      return -1
  ```
  **复杂度**：时间 O(log n)，空间 O(1)。前提：**数组有序**。
  **注意**：易错在 `lo<=hi` 还是 `<`、`mid±1`。变体：找左/右边界、找插入位置（用 `bisect` 模块）。
- 【结合你的经验】
  二分查找我常类比"资金对账里在有序流水里定位某笔交易"。面试我会强调"前提是有序，否则先排序 O(nlogn)"，以及边界 `lo<=hi`。这类题考的是边界严谨，正好是测试该有的思维。
- 【追问链】
  - 找第一个等于 target 的位置？→ 命中时 hi=mid-1 继续。
  - Python 内置？→ bisect 模块。
- [ ] 已掌握

### Q33 快速排序
- 【考点】分治、递归、复杂度。
- 【参考答案】
  ```python
  def quick_sort(arr):
      if len(arr) <= 1:
          return arr
      pivot = arr[len(arr) // 2]
      left  = [x for x in arr if x < pivot]
      right = [x for x in arr if x > pivot]
      mid   = [x for x in arr if x == pivot]
      return quick_sort(left) + mid + quick_sort(right)
  ```
  **复杂度**：平均 O(n log n)，最坏 O(n²)（已有序+末位选轴）；空间 O(log n)（递归栈）；**不稳定**。
  **工程版**用原地 partition + 随机轴防最坏。面试可先写易懂版，再补原地版。
- 【结合你的经验】
  快排我习惯先写"列表推导版"展示思路清晰，再补一句"生产用随机轴避免 O(n²)"。和我做大数据报文排序一样——算法选型要看数据特征，这是工程思维，不只是刷题。
- 【追问链】
  - 最坏怎么避免？→ 随机选轴 / 三数取中。
  - 为什么不稳定？→ 跨越交换打乱相等元素顺序。
- [ ] 已掌握

### Q34 链表反转
- 【考点】指针操作、边界。
- 【参考答案】
  ```python
  class ListNode:
      def __init__(self, v=0, nxt=None):
          self.val, self.next = v, nxt

  def reverse_list(head):
      prev = None
      cur = head
      while cur:
          nxt = cur.next     # 暂存下一个
          cur.next = prev    # 反转指向
          prev = cur
          cur = nxt
      return prev            # 新头
  ```
  **复杂度**：时间 O(n)，空间 O(1)。
  **注意**：空链表、单节点要正确处理；务必先存 `nxt` 再改 `next`，否则断链。
- 【结合你的经验】
  链表题考指针熟练度。我写时一定先画三步（存 nxt→改指向→推进），避免断链——这和我做报文节点遍历/反转解析是同样的指针思维。哪怕不常用链表，手写不出错是测开基本功。
- 【追问链】
  - 递归反转怎么写？→ 递归到尾，回溯时改 next。
  - 反转部分链表？→ 定位区间再反转。
- [ ] 已掌握

### Q35 统计字符串中每个字符出现次数（Counter）
- 【考点】计数、collections。
- 【参考答案】
  ```python
  from collections import Counter

  def char_count(s):
      return Counter(s)

  # 纯手写版（不依赖库）
  def char_count_raw(s):
      d = {}
      for c in s:
          d[c] = d.get(c, 0) + 1
      return d

  print(char_count("abracadabra")['a'])  # 5
  ```
  **复杂度**：时间 O(n)，空间 O(k)（k 为字符种类）。
  **注意**：Counter 还能 `most_common(n)` 取TopN；区分大小写（`'A'!='a'`），按需 `.lower()`。
- 【结合你的经验】
  Counter 我在报文差异统计里真用过——统计回盘字段各类状态出现次数、Top 异常排行，直接 `Counter(status_list).most_common(5)` 给领导看。算法题里的"计数"在金融业务里就是"对账统计"，我很熟。
- 【追问链】
  - 统计单词频率？→ s.split() 后 Counter。
  - 中文怎么统计？→ 遍历字符，Counter 同样适用。
- [ ] 已掌握

---

## 二、必背清单（浓缩速记卡）

```
【可变】list/dict/set/bytearray/类实例；【不可变】int/float/str/tuple/frozenset/bytes
【拷贝】copy=浅(共享嵌套) / deepcopy=递归全复制
【去重】set(无序)/dict.fromkeys(保序)/推导
【结构】list序改 / tuple序不可变做key / set查重 / dict映射
【生成器】yield 惰性省内存；迭代器 __next__ 耗尽 StopIteration
【装饰器】高阶函数 @dec=f=dec(f)；计时/重试/日志三件套
【闭包】内层+捕获外层变量；装饰器底座
【异常】try/except/else(成功)/finally(必执行)；自定义继承Exception
【with】__enter__/__exit__ 自动释放；fixture 同理
【GIL】CPython 锁，多线程不并行CPU；CPU密集→多进程，IO密集→多线程/协程
【pytest优势】原生assert+fixture+parametrize+插件生态
【发现】test_*.py / test_* / Test*类(无__init__)
【fixture】4作用域 function/class/module/session；yield 前准备后清理
【conftest】就近共享，无需import
【parametrize】数据驱动，多组独立用例
【mark】skip/xfail/自定义(需注册)
【hook】pytest_* 改框架行为（addoption/collection_modifyitems）
【token】session级fixture缓存+auth_header注入
【Flaky6解】显式等待/重试/数据隔离/Mock/精准断言/并行隔离
【并行报告】pytest -n auto + allure
【算法】两数之和哈希O(n)/快排O(nlogn)/二分O(logn)/链表反转O(1)空间
```

---

## 三、实战演练（3 道场景题 + 答题框架）

### 场景1：现场写"接口自动化框架"怎么搭（综合题）
**框架**：
1. 目录分层：tests/（用例）、conftest.py（全局 fixture：登录/token/DB）、utils/（请求封装）、data/（参数化数据）、pytest.ini（mark 注册）。
2. 鉴权：session 级 token fixture + auth_header 注入（参考 Q25）。
3. 数据驱动：parametrize 多银行多场景（参考 Q21）。
4. 稳定性：Mock 渠道 + 重试 + 显式等待（参考 Q26）。
5. 执行：`-n auto` 并行 + `--alluredir` 报告（-m smoke 准入）。
**话术钩子**："我独立开发过 FastAPI+Vue3 模块，pytest 这套我会从框架设计到 CI 落地讲清楚。"

### 场景2：手写"读取目录下所有 XML 报文并比对差异"函数
**框架**：
1. `Path(dir).rglob('*.xml')` 遍历（参考 Q16）。
2. 用生成器惰性解析大文件（参考 Q7）。
3. 深拷贝期望模板再裁剪忽略节点（参考 Q2）。
4. 差异用自定义异常 `CompareDiffError` 抛出（参考 Q13）。
5. 写 pytest 用例 + parametrize 多文件。
**话术钩子**："这就是我报文比对自动化的核心逻辑，我做过、跑过、省过人天。"

### 场景3：算法题"给有序交易流水，找金额为 target 的任一笔"（二分）
**框架**：
1. 确认前提：数组有序（参考 Q32）。
2. 写二分 `lo<=hi`，`mid=(lo+hi)//2`，命中返回否则缩半。
3. 说复杂度 O(log n)，并补"若无序先排序 O(nlogn)"。
4. 边界：空、单元素、重复值（返回任一下标）。
**话术钩子**："这题和我资金对账里在有序流水定位某笔是同一思路，边界我会先和面试官对齐。"
