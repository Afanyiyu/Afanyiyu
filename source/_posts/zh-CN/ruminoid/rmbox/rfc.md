---
title: Ruminoid Toolbox 草稿
date: 2021-06-15 17:47:01
categories:
  - Ruminoid
  - Rmbox
tags:
  - Ruminoid
  - Rmbox
---

这里是 Ruminoid Toolbox 的草稿。

<!-- more -->

# 定义

Ruminoid Toolbox 是一个 **「媒体处理工具箱」（Media Processing Toolbox）**。此处的「媒体」指 [传播媒体](https://zh.wikipedia.org/wiki/%E4%BC%A0%E6%92%AD%E5%AA%92%E4%BD%93)（即 [Media (communication)](https://en.wikipedia.org/wiki/Media_(communication))）；「处理」指 [线性剪接](https://zh.wikipedia.org/wiki/%E7%B7%9A%E6%80%A7%E5%89%AA%E6%8E%A5)（即 [Linear video editing](https://en.wikipedia.org/wiki/Linear_video_editing)）。Rmbox 是 Ruminoid Toolbox 的简称。

Ruminoid Toolbox 在 [这里](https://github.com/Ruminoid/Toolbox) 开源。

## 这是……

- Rmbox 是一个传统意义上的「视频工具箱」。目前，Rmbox 具有与 [小丸工具箱](https://maruko.appinn.me/) 相似的功能。

- Rmbox 是一个传统意义上的「多功能工具箱」。由于 Rmbox 具有插件系统，因此可以被扩展多种多样的功能。

- Rmbox 是一个线性处理工具，参见上文提到的「线性剪接」。

## 这不是……

- Rmbox 不是 [非线性剪辑](https://zh.wikipedia.org/wiki/%E9%9D%9E%E7%B7%9A%E6%80%A7%E5%89%AA%E8%BC%AF) 软件。Rmbox 不具有非线性编辑功能。

# 特点

- **CLI/GUI**：Rmbox 同时支持以 GUI 的方式启动，和以 CLI 的方式调用。

- **插件系统**：Rmbox 支持使用 C#、Python、JS、Lua 四种语言编写插件。插件可以为 Rmbox 添加新的功能，使其能够执行更多操作。

- **多种操作方式**：Rmbox 支持简单操作、批量操作和链式操作等多种操作方式。

- **任务队列**：Rmbox 具有一个串行任务队列，可以使操作按照顺序逐个执行。

- **线性处理**：Rmbox 是一个线性处理工具。Rmbox 在操作开始前可以指定操作的相关参数，但在操作开始之后则无法对操作进行调整和改变。

- **模板和预设**：Rmbox 支持将调节好的参数保存为预设，并在需要的时候再次加载使用。

- **监视文件夹**：Rmbox 支持监视文件夹更改，并在发现新文件时自动启动指定的操作。

- **跨平台**：Rmbox 同时支持 Windows、macOS 和 Linux 三个平台。

# 结构简述

## 项目组成

Rmbox 的主项目由以下三部分组成。

### `rmbox-plugbase`

`rmbox-plugbase` 是 Rmbox 的公共库，所有的基本类和接口都在此库中定义。所有 Rmbox 组件和 Rmbox 插件均依赖此库。

### `rmbox`（CLI）

`rmbox` 是 Rmbox 的命令行用户界面，也是 **操作（Operation）** 的实际执行实例。

大部分情况下，`rmbox` 的单个实例仅会负责单个操作（批量操作等操作类型也属于单个操作）或单个任务（如监视文件夹）的运行。

### `rmbox-shell`（GUI）

`rmbox-shell` 是 Rmbox 的图形用户界面。`rmbox` 在无参数提供的情况下默认启动 `rmbox-shell` 实例。

图形用户界面可以方便用户调节各 **配置项（ConfigSection）** 的参数，且可以多次运行 `rmbox` 实例。

## 核心实现

### `rmbox-plugbase`

`rmbox-plugbase` 有如下核心概念：

#### 操作（Operation）

**操作（Operation）** 是用户希望 Rmbox 所执行的任务，如「压制」或「封装」。单个的操作由 Rmbox 插件实现 [`IOperation`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Core/Interfaces/IOperation.cs) 接口。

注意此处的 **操作（Operation）** 与 Rmstd 中的操作是不同的概念。

下面是 `IOperation` 接口的定义：

```cs
public interface IOperation
{
    public List<TaskCommand> Generate(Dictionary<string, JToken> sectionData);
    public Dictionary<string, JToken> RequiredConfigSections { get; }
}
```

其中，`RequiredConfigSections` 向 Rmbox 提供该操作所需的 **配置项（ConfigSection）** 的类型签名列表以及对每个配置项可选的自定义选项；`Generate` 方法用于接收 **配置项（ConfigSection）** 提供的参数并生成 **任务命令（TaskCommand）** 列表。

#### 任务命令（TaskCommand）

**任务命令（TaskCommand）** 简称「**命令**」，是操作在执行时的原子单位。一个任务命令通常可以视作一行可以在 Shell 内执行的字符串，类似 `echo test`。单个的任务命令是 [`TaskCommand`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Core/TaskCommand.cs) 类的一个单例。

下面是 `TaskCommand` 类的定义：

```cs
public record TaskCommand
{
    public readonly string Target = ""; // 需要执行的实用工具
    public readonly string Args = ""; // 向工具传递的参数
    public readonly string Formatter = ""; // 需要使用的格式器（Formatter）
}
```

此外，`TaskCommand` 类还带有与对应 `ValueTuple` 类型的隐式转换，方便在对象使用过程中直接获取或指定对应的字段。

#### 配置项（ConfigSection）

**配置项（ConfigSection）** 提供了与 **操作（Operation）** 解耦，但可供 **操作（Operation）** 使用的用户自定义配置，如「输入/输出」和「视频质量」。单个的配置项由 Rmbox 继承并实现 [`ConfigSectionBase`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Core/Interfaces/ConfigSectionBase.cs) 基类。

下面是 `ConfigSectionBase` 类的定义：

```cs
public abstract class ConfigSectionBase : UserControl
{
    public abstract object Config { get; } // 配置项的参数对象
    public string CurrentHelpText { get; }
}
```

`ConfigSectionBase` 类继承自 Avalonia 的 `UserControl` 类，使得继承的实例可以直接作为一个用户控件在 GUI 中呈现。

每个配置项在构造函数内需要接受一个 `JToken` 类型的参数，用于接收操作传来的自定义选项。

#### 格式器（Formatter）

**格式器（Formatter）** 用于将命令行实用工具（如 `ffmpeg`）输出的信息转换为 Rmbox 使用的 [`FormattedEvent`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Formatting/FormattedEvent.cs) 描述方式。单个的格式器由 Rmbox 插件实现 [`IFormatter`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Formatting/IFormatter.cs) 接口。

下面是 `FormattedEvent` 类的定义：

```cs
public record FormattedEvent
{
    public string Target; // 执行的实用工具
    public double Progress; // 在0-100之间用来表示进度的浮点数
    public bool IsIndeterminate; // 进度是否不确定
    public string Summary; // 执行操作的简述
    public string Detail; // 执行操作的详细信息
}
```

下面是 `IFormatter` 接口的定义：

```cs
public interface IFormatter
{
    public FormattedEvent Format(string target, string data, Dictionary<string, object> sessionStorage);
}
```

#### 元数据（Meta）

**元数据（Meta）** 描述了单个 Rmbox 插件的注册信息。一个 Rmbox 插件内只能有一个类实现 [`IMeta`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-plugbase/Core/Interfaces/IMeta.cs) 接口。

下面是 `IMeta` 接口的定义：

```cs
public interface IMeta
{
    public string Name { get; } // 插件的名称
    public string Description { get; } // 插件的描述
    public string Author { get; } // 插件的作者
}
```

#### 工具（Tool）

TODO。

### `rmbox`

`rmbox` 有如下核心概念：

#### 项目模型（ProjectModel）

**项目模型（ProjectModel）** 是 `rmbox` 所需要执行的操作的结构化表述，序列化使用 JSON 方式存储。

一般来讲，只需要给 `rmbox` 实例提供单个 **项目模型（ProjectModel）** 文件作为参数，`rmbox` 便可自行解析并执行指定的操作。

下面是一个运行 `rmbox` 实例的例子：

```sh
rmbox proj.json
```

需要注意的是，Rmbox 在解析 **项目模型（ProjectModel）** 的时候，并不会直接进行反序列化，而是会根据 `type` 字段手动解析。

下面是一个 **项目模型（ProjectModel）** 的例子：

```json
{
    "version": 1, // 项目模型的版本
    "type": "project", // 项目的类型（如 project、batch、chain 等）
    "operation": "Ruminoid.Toolbox.Plugins.FFmpeg.Operations.FFmpegEncodeOperation", // 项目要执行的操作
    "sections": [ // 配置项参数
        {
            "type": "Ruminoid.Toolbox.Plugins.Common.ConfigSections.IOConfigSection",
            "data": {
                "input": "flv-src.flv",
                "output": "flv-out.mp4"
            }
        }
    ]
}
```

### `rmbox-shell`

`rmbox-shell` 有如下核心概念：

#### 导出器（Exporter）

**导出器（Exporter）** 负责将 `rmbox-shell` 内 **操作（Operation）** 的 `ViewModel` 导出为 `rmbox` 可以解析的 **项目模型（ProjectModel）**，由 [`Exporter`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-shell/Core/Exporter.cs) 类实现。

#### 队列服务（QueueService）

**队列服务（QueueService）** 是 `rmbox-shell` 内负责将任务排队并逐个运行的服务，由 [`QueueService`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox-shell/Services/QueueService.cs) 类实现。

# 功能简述

## 组合

**Rmbox 组合** 是 Rmbox 用来加载插件的方式，位于 `Ruminoid.Toolbox.Composition` 命名空间。**Rmbox 组合** 的核心是 **「插件服务接口」**—— [`IPluginService`](https://github.com/Ruminoid/Toolbox/blob/master/src/rmbox/Composition/Services/IPluginService.cs)。使用该接口，Rmbox 的各个组件可以轻松地获取到由 Rmbox 插件所提供的各项功能和组件，如 **操作**、**配置项** 和 **工具**。

在 Rmbox 的任一二进制实例（如 `rmbox` 和 `rmbox-shell`）启动时，**Rmbox 组合** 便会预先启动，遍历、扫描并加载所有的插件，逐个解析插件中包含的所有组件并将其添加到 **插件服务接口** 中对应的集合中。

## 主要功能

Rmbox 的 **主要功能** 有两个，分别是 **执行操作** 和 **运行工具**。

### 执行操作

Rmbox 通过丰富的插件提供了多种多样的 **操作（Operation）**，**执行操作** 就是接收所提供的参数并运行这些操作的过程。

#### `rmbox`（CLI）

`rmbox` 接受一个 **项目模型（ProjectModel）**  作为参数并运行 **项目模型（ProjectModel）** 内指定的 **操作（Operation）**。

#### `rmbox-shell`（GUI）

**Rmbox 组合** 在 `rmbox-shell` 启动时即会加载，之后 `rmbox-shell` 将成功加载的插件按照分类列出一个树形列表，供用户选择和创建 **操作（Operation）**。

用户创建操作时，`rmbox-shell` 使用 **Rmbox 组合** 解析该操作所需的 **配置项（ConfigSection）** 并按照顺序加载，供用户调整参数。操作所提供的针对每个配置项的自定义选项通过配置项的构造函数传递。

用户调整完参数并点击「添加到队列」后，`rmbox-shell` 将对应的操作交由 **导出器（Exporter）** 导出 **项目模型（ProjectModel）** 并交由 **队列服务（QueueService）** 运行。在用户点击之后，`rmbox-shell` 会直接提取每个配置项的 `Config` 属性作为参数，因此这要求配置项在用户每一次调节参数之后都立即计算 `Config` 属性中的计算属性（Computed Properties）。

### 运行工具

Rmbox 通过丰富的插件提供了多种多样的 **工具（Tool）**，**运行工具** 就是对工具进行调用的过程。

TODO。
