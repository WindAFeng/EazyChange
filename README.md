# 易产 · Eazy Change

> **简单地描述，稳定地解析，自由地实现。**

**易产（Eazy Change，简称 EC）** 是一种轻量级、行式、面向内容描述的开放文档格式，默认使用 `.ec` 作为文件扩展名。

EC 的目标不是成为一种复杂的标记语言，也不是取代 JSON、YAML 等数据格式，而是提供一种同时对 **人类、程序、Web 与生成式 AI** 友好的文档描述方式。

EC 强调：

- 简单、直观的语法
- 良好的源码可读性
- 极低的解析复杂度
- 高容错与局部错误隔离
- 高效的流式读取与渲染
- 对 HTML 渲染友好
- 对 AI 流式输出友好
- 长期稳定的向下兼容
- 对低性能、小型设备友好
- 开放的格式与扩展生态

---

## Hello EC

一个最简单的 EC 文档：

```ec
![title,level=1]Hello EC[end]
![text]这是我的第一份易产文档。[end]
![text,type=bold]Easy for humans, easy for machines.[end]
```

EC 的基本思想非常简单：

```text
![Element,参数...]内容[end]
```

例如：

```ec
![title,level=2]这是一个二级标题[end]
```

```ec
![text,color=red]这是一段红色文本[end]
```

```ec
![text,type=bold]这是一段粗体文本[end]
```

EC 以行为基本单位。

大多数情况下，一个 Element 就是一行内容。

---

## 为什么是 EC？

Markdown 非常适合人类快速书写。

JSON、YAML 非常适合描述结构化数据和配置。

EC 并不试图替代它们擅长的领域。

EC 更关注：

> **如何用一种简单、明确、稳定并且机器容易理解的方式描述文档内容。**

例如：

```ec
![text,type=bold,color=red]Hello EC[end]
```

人类可以直接理解：

- `text`：文本
- `type=bold`：粗体
- `color=red`：红色
- `Hello EC`：内容
- `[end]`：当前内容结束

机器同样可以非常容易地解析这些信息。

EC 宁愿使用少量冗余，也不希望通过大量特殊符号增加解析和学习成本。

---

## 容错优先

EC 的一个核心原则是：

> **可以降级，不应崩溃。**

假设：

```ec
![text,colro=red]Hello EC[end]
```

`colro` 是一个无法识别的参数。

EC Renderer 应当忽略它，并继续渲染：

```text
Hello EC
```

而不是让整个文档解析失败。

如果 Element 本身无法识别：

```ec
![something]Hello EC[end]
```

则应尽可能降级为普通 `text`：

```text
Hello EC
```

EC 希望做到：

```text
未知参数
    ↓
忽略参数
    ↓
继续

未知 Element
    ↓
降级为 text
    ↓
继续

局部内容异常
    ↓
局部处理
    ↓
继续
```

一个 Element 的问题不应该轻易影响整个文档。

---

## 表格

EC 提供简单的表格描述方式。

例如一个三行四列的表格：

```ec
![table,3,4]
![row,1]A&&B&&C&&D
![row,2]E&&F&&G&&H
![row,3]I&&J&&K&&L
[end]
```

EC 不追求让所有结构共享完全相同的解析方式。

对于 `table` 等特殊 Element，可以由对应解析策略进行处理。

---

## Group

默认情况下，每一个普通 Element 表示一个独立的视觉行。

```ec
![text]Hello[end]
![text,type=bold]World[end]
```

默认表现为：

```text
Hello
World
```

如果多个连续 Element 属于同一个 `group`，则可以共同组成一个视觉行：

```ec
![text,group=example]Hello [end]
![text,group=example,type=bold]World[end]
![text,group=example]![end]
```

表现为：

```text
Hello World!
```

EC 源文件仍然保持扁平的 Element Stream，而不需要复杂的嵌套文本结构。

---

## 外部文件

对于体系化技术文档，EC **推荐将代码、JSON、配置示例等内容保存在独立文件中**，然后通过 `file` 引入。

例如：

```ec
![file,from=src/Main.java,type=java][end]
```

即使文件扩展名不是对应语言：

```ec
![file,from=example.txt,type=java][end]
```

EC 仍然可以通过 `type=java` 告诉 Renderer 应当按照 Java 内容进行处理。

### 指定范围

`interval` 可以指定需要展示的行区间：

```ec
![file,from=src/Main.java,type=java,interval=10:20][end]
```

其中：

```text
10:20 → [10, 20)
```

即从第 10 行开始，到第 20 行之前。

### 异常显示

`display` 用于描述指定内容存在格式问题时的展示行为：

```ec
![file,from=test.json,type=json,display=true][end]
```

具体文件读取、路径解析、访问权限以及运行环境相关行为由对应 EC Engine / Runtime 负责。

EC 只负责描述：

> **这里有什么。**

---

## 内嵌 Content

EC 支持直接内嵌代码或其他原始内容，但 **官方非常不建议在体系化文档中使用这种方式保存代码。**

推荐：

```ec
![file,from=Main.java,type=java][end]
```

而不是：

```ec
![content,define=null,type=java]
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello EC");
    }
}
[content-end]
```

`content` 主要用于：

- AI 流式生成
- 临时内容
- 无法建立外部文件引用的环境
- 特殊兼容场景

长期维护的代码应优先保存为独立文件，再通过 `file` 引入。

---

## 面向流式处理

EC 天然适合逐行读取。

```text
EC Stream
    ↓
读取一行
    ↓
识别 Element
    ↓
解析参数
    ↓
Render
    ↓
继续下一行
```

普通 Element 可以在完成后立即提交给 Renderer。

特殊 Block，例如：

```text
table
content
```

可以在 Block 完成后统一提交。

因此 EC 非常适合：

- Web 动态文档
- AI 流式输出
- 实时预览
- WebSocket 文档传输
- 低内存环境
- 嵌入式或小型设备

EC 并不意味着所有操作的时间复杂度都是 O(1)。

完整读取一个文档仍然至少与文档大小相关。

EC 追求的是：

> **让普通 Element 可以独立解析、独立处理和独立重新渲染。**

---

## HTML

EC 非常适合作为 HTML 的上层内容描述格式。

例如：

```ec
![title,level=1]Eazy Change[end]
![text]Hello EC[end]
```

Renderer 可以生成：

```html
<h1>Eazy Change</h1>
<p>Hello EC</p>
```

推荐的架构：

```text
.ec
 ↓
EC Parser
 ↓
Element Stream
 ↓
Renderer
 ↓
HTML
```

EC 正文本身不应被直接视为 HTML。

HTML 的生成方式由 Renderer 决定。

---

## 版本

EC 支持版本声明：

```ec
![version,v1][end]
```

版本声明应位于文档第一条有效 Element。

如果没有指定版本，推荐使用当前最新稳定规范进行处理。

EC 的稳定版本遵循一个重要原则：

> **已有稳定语法只增不改。**

一项语法一旦进入 EC Stable Specification，其既有含义不应在后续版本中被重新定义。

新版本主要通过增加新的 Element、参数和能力进行演进。

因此新的 EC Engine 应尽可能自然地兼容旧 EC 文档。

而旧 Engine 遇到未来的新语法时，则通过：

```text
未知参数 → ignore
未知 Element → text fallback
```

尽可能保持文档可读。

---

# 开放原则

## EC 属于大家

EC 是一种开放文档格式。

任何人都可以：

- 使用 EC
- 学习 EC
- 实现 EC
- 传播 EC
- 将 EC 用于商业项目
- 将 EC 用于非商业项目
- 开发收费的 EC 软件或服务
- 开发自己的 EC Parser
- 开发自己的 EC Renderer
- 开发闭源 EC Engine
- 开发闭源 EC Editor

EC 格式与任何具体实现相互独立。

**实现可以属于任何人，格式应当保持开放。**

---

## 新的想法属于生态

任何人都可以提出新的：

```text
Element
Parameter
Syntax
Extension
```

官方也欢迎社区提出新的设计。

但新的设计不会因为存在就自动进入 EC 官方规范。

官方 EC Specification 对新增能力保持谨慎。

一项设计只有经过充分审查并确认符合 EC 的长期设计理念后，才可能进入 Stable Specification。

如果一个提案不适合官方 EC，它仍然可以作为第三方扩展存在。

---

## 第三方扩展

任何人都可以 Fork EC Engine 或自行实现 Engine，并增加官方 EC 不支持的功能。

实现代码可以保持闭源。

但是：

> **如果一种新的 EC 格式扩展被用于公开的 EC 文档交换，那么它的语法本身应当保持公开。**

其他人应当能够知道：

- Element 的名称
- Element 的用途
- 支持的参数
- 参数的含义
- 内容结构
- 解析方式
- 推荐的降级行为
- 兼容版本

目标非常简单：

> **你可以拥有自己的 EC Engine，但不应该创造一种只有你的 Engine 才知道如何理解的公开 EC 格式。**

---

## 社区共同维护开放性

EC 的开放性不属于某个个人、公司或组织，也不属于 EC 最初的设计者。

如果任何个人或组织公开使用新的 EC 格式扩展，却拒绝公开足以让第三方理解和独立实现该格式的规范，任何人都可以依据 EC 的开放原则对此提出质询、要求说明或要求公开相关格式规范。

不需要获得 EC 原作者或官方维护者的事先授权。

这项原则同样适用于 EC 官方自身。

如果未来 EC 官方违反了 EC 的开放原则，社区同样有权提出质询。

EC 的开放原则不应被用于骚扰、威胁、攻击或干扰任何个人与组织。

---

# EC 的设计原则

EC 希望长期坚持以下原则：

**Simple**

保持语法简单，不为了少量功能无限增加语言复杂度。

**Readable**

即使不了解 EC，也应该能够通过源码大致理解文档表达的内容。

**Fault-tolerant**

可以降级，不应因为一个局部问题导致整个文档不可用。

**Stable**

已经进入稳定规范的语法不应被随意修改。

**Stream-friendly**

优先保证普通 Element 能够独立读取、解析和渲染。

**Implementation-independent**

EC 不属于任何一个 Parser、Renderer、Engine 或 Editor。

**Open**

实现可以闭源，服务可以收费，但公开使用的 EC 格式本身应当让所有人能够学习和实现。

---

# EC 不是什么？

EC 不是为了取代所有格式而设计的。

如果需要：

```text
配置文件        → JSON / YAML 等格式可能更加合适
复杂数据交换    → JSON / Protobuf 等格式可能更加合适
程序执行        → 使用真正的编程语言
```

EC 主要关注：

```text
文档
文章
技术文档
Web 内容
AI 生成内容
轻量出版
实时文档
跨设备内容渲染
```

**选择适合问题的工具，比让一种格式解决所有问题更加重要。**

---

# 项目状态

> EC 当前仍处于早期设计阶段。

在第一个 Stable Specification 发布之前，语法仍可能发生调整。

进入 Stable 的语法将遵循 EC 的长期兼容原则。

当前建议：

```text
Specification: Draft
Stable: Not Yet
```

---

# 未来

EC 计划逐步提供：

- EC Specification
- 官方 Rust EC Engine
- EC → HTML Renderer
- 静态文档渲染
- 动态文档渲染
- WebSocket 跨设备渲染
- EC Editor
- Conformance Tests
- 更多语言的第三方实现

EC 不要求所有实现使用同一种编程语言。

我们希望未来能够看到：

```text
Rust EC Engine
Python EC Parser
Java EC Parser
JavaScript EC Renderer
Web EC Editor
Desktop EC Editor
Embedded EC Reader
AI EC Generator
```

它们通过同一份公开 EC Specification 理解同一种文档。

---

# 最后

EC 的实现可以属于任何人。

EC 的思想可以来自任何人。

**EC 的规范应当始终让所有人都能够知道、学习和实现。**

> **易产 —— 让内容更容易被生产，也更容易被机器理解。**
