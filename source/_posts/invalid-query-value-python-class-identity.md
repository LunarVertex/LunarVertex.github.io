---
title: "一次诡异的 invalid query value：我最终追到了 Python 的 Class Identity"
date: 2026-08-30 12:00:00
tags:
  - Python
  - 调试
  - MongoEngine
---

> 从 MongoEngine，到 `isinstance`，再到 `sys.modules` 与 `load_module()`

---

最近遇到一个生产问题，现象特别反直觉。

一个更新接口偶发失败，日志最终指向 MongoEngine：

```text
invalid query value
```

但奇怪的是，传入数据库字段的对象，明明就是目标 `EmbeddedDocument` 类型。

更诡异的是：**同一个请求，重试几次，有时候又成功了。**

一开始我以为是数据问题：某个字段是 `None`？历史版本里有脏数据？但查了一圈，这些方向都对不上。

真正让我改变判断的，是一个很不起眼的事实：

```python
type(value).__name__      == TargetClass.__name__      # True
type(value).__module__    == TargetClass.__module__    # True
isinstance(value, TargetClass)                          # False
```

打印出来，它们都是：

```text
app.models.policy.ResourcePolicy
```

但 `isinstance` 却是 `False`。

再进一步：

```python
id(type(value)) != id(TargetClass)
# True
```

**同一个名字、同一个模块，甚至看起来是同一个 Class，为什么 Python 认为它们不是同一个类型？**

这个问题把我从 MongoEngine 一路拉进了 Python Runtime 的底层。

## 第一个错误方向：数据问题

最开始我怀疑某个阈值字段是 `None`，或者数据库里有旧版本数据。

但这些假设都有一个共同的缺陷：**它们只能解释"为什么失败"，却无法解释"为什么重试后可能成功"。**

如果数据本身就是错的，重试不会改变结果。

所以这些方向被排除了。

## 第二个错误方向：MongoEngine 本身

接着我怀疑是不是 MongoEngine 的 bug。

但如果是库本身的 bug，应该更稳定地出现，而不是偶发。

而且我后来在容器里单独执行：

```bash
python3 -c "..."
```

测试：

```python
TargetClass is Field.document_type
```

结果竟然是 `True`。

这让我一度以为"重复 Class 应该不是问题"。

但后来发现，这个测试根本没复现真实环境——它启动的是一个全新的 Python 进程，没有经历应用启动时的动态加载路径。

**诊断代码必须和故障运行环境处于同一个状态空间。**

## 真正有用的探针

最后我把诊断信息直接放进了实际请求路径，拿到了这样的输出：

```text
cls_mod=app.models.policy
cls_id=94387591399664

dt_mod=app.models.policy
dt_id=94387582836832

isinstance=False
```

`module` 相同，`name` 相同，但 `id` 不同。

这时我才意识到：这两个 `ResourcePolicy` 只是字符串表示完全一样，运行时其实是两个不同的 Class Object。

证据链一下子清晰了：

```text
现象
  ↓
isinstance(value, TargetClass) == False
  ↓
type(value) is not TargetClass
  ↓
type(value).__module__ == TargetClass.__module__
  ↓
type(value).__name__ == TargetClass.__name__
  ↓
id(type(value)) != id(TargetClass)
  ↓
确认：两个 Class Object
  ↓
追踪 module loading
  ↓
发现 load_module()
```

从这一刻开始，问题已经不再是 MongoDB 问题，而变成了 Python Runtime 问题。

## Python 为什么会出现两个 Class Object

在 Python 中，`class` 是一个可执行语句。执行到：

```python
class ResourcePolicy(EmbeddedDocument):
    ...
```

解释器会在当前命名空间中创建一个新的类对象。这个类对象是 `type` 的实例，拥有自己的 `__dict__`、`__mro__`、`__bases__`。它不仅仅是名字。

`isinstance(obj, cls)` 会判断对象是否是 `cls` 或其子类的实例。对于普通 Python 类而言，两个独立执行 `class` 语句产生的类对象，即使拥有相同的 `__name__`、`__module__` 和 `__qualname__`，仍然是不同的类对象。因此一个类创建的实例并不会自动被另一个同名类识别为实例。

当模块代码被重复执行时：

```text
第一次执行 → ResourcePolicy A → id 0x123456
第二次执行 → ResourcePolicy B → id 0x789012
```

`A.__name__ == B.__name__` 为 `True`，`A.__module__ == B.__module__` 为 `True`，但 `A is B` 为 `False`。于是 `isinstance(B(), A)` 为 `False`。

## 同一份源码为什么执行了两次？

正常 `import` 会经过 `sys.modules` 缓存：

```text
import
  ↓
sys.modules
  ↓
已存在？
  ├── Yes → 复用 module object
  └── No  → 执行模块代码，创建 module object
```

在遵循标准 import 语义的前提下，`sys.modules` 是 Python 复用已加载模块对象的核心缓存。

但系统中存在一个旧式动态加载函数，简化后类似：

```python
import pkgutil

def load_component(module_name):
    loader = pkgutil.find_loader(module_name)
    return loader.load_module(module_name)
```

这里用的是 `loader.load_module()`，而不是 `importlib.import_module()`。

`load_module()` 使用的是已经废弃的旧式模块加载协议。在特定加载路径下，它可能绕开现代 import system 的完整语义，使同一份源代码被重新执行，从而产生多个 module/class object。

调用链大致是：

```text
Application Startup
        │
        ▼
register_components()
        │
        ▼
load_component()
        │
        ▼
loader.load_module()
        │
        ▼
重新执行某个 service module
        │
        ▼
import app.models.policy
        │
        ▼
重新创建 ResourcePolicy
```

于是进程中出现：

```text
                  app.models.policy
                         │
             ┌───────────┴───────────┐
             │                       │
          Class A                 Class B
             │                       │
        MongoEngine Field         Runtime 构造
             │                       │
             └───────────┬───────────┘
                         │
                 isinstance(B, A)
                         │
                         ▼
                       False
```

MongoEngine 内部正是依赖 `isinstance(value, document_type)` 判断字段值是否匹配。当 `value` 来自 `Class B`，而 Field 的 `document_type` 是 `Class A` 时，判断失败，走入 `_from_son` 处理逻辑，最终抛出 `invalid query value`。

## 为什么它表现成"偶发"

Web 服务通常是多 worker 模型：

```text
                    Load Balancer
                   /      |      \
             Worker A  Worker B  Worker C
                │         │         │
             Process    Process    Process
                │         │         │
           sys.modules  sys.modules  sys.modules
```

在多 worker 部署中，每个进程拥有独立的 import state。只要不同 worker 的模块加载顺序或加载路径不同，就可能形成不同的运行时状态。

所以从外部看，同一请求时好时坏，实际上是请求被路由到了不同的进程状态。这不是随机，而是状态不一致。

可以用抽象后的 worker 证据来说明：

```text
Request #1 → Worker A → Class A != Class B → FAIL
Request #2 → Worker B → Class A == Class B → SUCCESS
Request #3 → Worker A → Class A != Class B → FAIL
```

## 修复

修复很简单，将动态加载改为标准 import：

```python
import importlib

def load_component(module_name):
    return importlib.import_module(module_name)
```

对比：

```text
load_module()
    ↓
可能重新执行模块
    ↓
重新创建 Class Object
    ↓
破坏类型 identity

import_module()
    ↓
遵循标准 import 机制
    ↓
复用 sys.modules
    ↓
保持 Class identity
```

修复后验证，`cls_id == dt_id`，`isinstance(value, ResourcePolicy)` 为 `True`，连续请求不再出现 `invalid query value`。

## 这次真正学到的东西

**1. 不要只解释一个现象，要解释全部现象。**

最开始 `invalid query value` 可以有很多解释，但一个好的根因模型必须同时解释：

```text
为什么 invalid query value？
为什么 isinstance=False？
为什么名字一样？
为什么 id 不一样？
为什么重试有时成功？
为什么独立 python3 -c 无法复现？
为什么改 importlib 后恢复？
```

最终 `Class Identity + Import State` 这个模型可以一次性解释所有现象。这比"找到一个能让测试通过的 workaround"重要得多。

**2. "偶发"往往意味着状态，而不是随机。**

如果同样的输入，有时成功、有时失败，首先应该寻找隐藏状态：

```text
Process State
Thread State
Cache State
Connection State
Initialization Order
Global State
Import State
```

这已经不仅是 Python 经验，而是一个通用的 Systems Debugging Principle。

**3. 诊断环境必须和故障环境属于同一个状态空间。**

这次 `kubectl exec ... python3 -c` 得到 `True`，但线上 Worker 是 `False`。

这说明："我在容器里复现了" ≠ "我复现了线上进程状态"。

以后遇到 import、cache、connection pool、singleton、global state、K8s informer cache 等问题，都应该问：

> 我现在观察到的状态，和真正出问题的进程是同一个状态空间吗？

问题表面是 MongoEngine 的 `invalid query value`，本质却是 Python Runtime 中 **Module Identity 与 Class Identity 被破坏**。

这次排查让我重新意识到：复杂系统里的异常，往往只是最底层不变量被破坏后的一个表现。

真正值得做的不是解释"为什么这次请求失败"，而是建立一个能够同时解释**失败、成功、偶发性以及修复后行为**的模型。

---

**Don't just explain the symptom. Build a model that explains all the symptoms.**

---

## 附：最小复现代码

```python
# app/models/policy.py
class ResourcePolicy:
    pass
```

```python
import pkgutil, sys
import app.models.policy as m1

A = m1.ResourcePolicy
print(id(A))   # 例如 32590304

loader = pkgutil.find_loader('app.models.policy')
m2 = loader.load_module('app.models.policy')
B = m2.ResourcePolicy
print(id(B))   # 例如 32595168

print(A is B)                         # False
print(A.__module__ == B.__module__)   # True
print(isinstance(B(), A))             # False
```
