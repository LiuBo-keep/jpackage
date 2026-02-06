# windows系统常用设置

## 1.安装完成后，**把你的应用注册到 Windows「开始菜单」里**

### `--win-menu` 是干嘛的？

```xml
<arg value="--win-menu"/>
```

**作用一句话版：**
👉 安装完成后，**把你的应用注册到 Windows「开始菜单」里**

------

### 实际效果（用户看到的）

安装完成后，用户可以：

- 点 **Windows 键**
- 搜索：`MyApp`
- 在「开始菜单」里看到你的程序
- 固定到开始 / 任务栏

📌 **这一步非常重要**
不加的话，用户只能去安装目录双击 exe，很不 Windows。

------

### `--win-menu` 单独用 vs 配套用

#### 1️⃣ 只用 `--win-menu`（不推荐）

```xml
<arg value="--win-menu"/>
```

效果：

- 开始菜单里有一个入口
- 分组名不确定（可能乱）

------

#### 2️⃣ 推荐组合（95% 场景）

```xml
<arg value="--win-menu"/>
<arg value="--win-menu-group"/>
<arg value="${app.name}"/>
```

效果：

```text
开始
 └─ MyApp
    └─ MyApp
```

👉 看起来像「正经软件」

------

### 和 `--win-shortcut` 的区别

| 参数             | 位置     |
| ---------------- | -------- |
| `--win-menu`     | 开始菜单 |
| `--win-shortcut` | 桌面     |

📌 两个**互不冲突**，强烈建议一起用：

```xml
<arg value="--win-menu"/>
<arg value="--win-shortcut"/>
```

------

### 常见误解 & 坑

#### ❌ 误解 1：`--win-menu` 会创建桌面图标

➡️ 不会
👉 桌面图标必须 `--win-shortcut`

------

#### ❌ 误解 2：不加也能搜索到

➡️ 有时候能，但**不稳定**
👉 取决于 exe 是否被系统索引

------

#### ❌ 误解 3：卸载不会清理菜单

➡️ 会的
jpackage 生成的卸载器会自动清理开始菜单项

------

### 什么时候可以不加？

| 场景         | 是否需要 |
| ------------ | -------- |
| GUI 工具     | ✅ 必加   |
| 普通桌面软件 | ✅ 必加   |
| 纯后台服务   | ❌        |
| CLI 工具     | ❌        |

------

### 推荐写法（你可以直接用）

```xml
<!-- Windows 开始菜单 -->
<arg value="--win-menu"/>
<arg value="--win-menu-group"/>
<arg value="${app.name}"/>
```

------

### 一句话总结

> `--win-menu` = **让用户“找得到你”**

---

## 2. 安装完成后，**在 Windows 桌面创建一个快捷方式**

### `--win-shortcut` 是干嘛的？

```xml
<arg value="--win-shortcut"/>
```

**作用一句话版：**
👉 安装完成后，**在 Windows 桌面创建一个快捷方式**

用户双击就能启动你的应用。

------

### 实际效果（用户视角）

- 桌面出现：`MyApp.lnk`
- 图标 = 你设置的 `--icon`
- 双击启动 = 调用安装目录下的 exe

📌 对 **非技术用户** 来说，极其重要。

------

### 推荐搭配（几乎必选）

```xml
<arg value="--win-shortcut"/>
<arg value="--win-menu"/>
<arg value="--win-menu-group"/>
<arg value="${app.name}"/>
```

效果：

- 桌面有
- 开始菜单有
- 搜索能搜到

👉 **标准 Windows 软件体验**

------

### 常见误区 & 坑点

#### ❌ 误区 1：不加图标，桌面还是好看的

➡️ 不对
没 `--icon`，桌面会是 **Java 默认白图标**，很丑 😅

------

#### ❌ 误区 2：卸载后桌面图标会残留

➡️ 不会
jpackage 生成的卸载器会自动清理

------

#### ❌ 误区 3：快捷方式位置不可控

➡️ 桌面位置由 Windows 决定
你不能指定到某个子目录（这是系统行为）

------

### 和 `--win-console` 的关系

如果你是 **GUI 程序**：

```xml
<arg value="--win-console"/>
<arg value="false"/>
```

否则：

- 双击桌面图标
- 会先闪一个黑框
- 再启动 GUI（很不专业）

------

### 什么时候不要用？

| 场景           | 是否用 |
| -------------- | ------ |
| 桌面 GUI 工具  | ✅ 必须 |
| 面向小白用户   | ✅      |
| CLI / 开发工具 | ❌      |
| 后台服务       | ❌      |

------

### 进阶注意点（很少人知道）

#### 🔹 每用户 vs 全局安装

- `--win-per-user-install`
  → 快捷方式在**当前用户桌面**
- 全局安装
  → 所有用户桌面

------

#### 🔹 多入口程序？

`jpackage` **只能生成一个快捷方式**
如果你要：

- GUI + CLI
- 多启动模式

需要：

- 多次 jpackage
  或
- 程序内自己做启动选项

------

### 推荐写法（直接抄）

```xml
<!-- 桌面快捷方式 -->
<arg value="--win-shortcut"/>
<arg value="--win-console"/><arg value="false"/>
<arg value="--icon"/><arg value="icon.ico"/>
```

------

### 一句话总结

> `--win-shortcut` = **让用户“一眼就能点开你”**

## 3. 这是一个 GUI 程序，不要分配控制台窗口

### `--win-console false` 是干嘛的？

```xml
<arg value="--win-console"/>
<arg value="false"/>
```

**一句话：**
👉 告诉 Windows：**这是一个 GUI 程序，不要分配控制台窗口**

------

### 用户看到的区别（立竿见影）

#### ❌ 不加 / 默认

- 双击 exe
- 先闪一个 **黑色 CMD 窗口**
- 再启动 GUI
- 有时黑框一直挂着

#### ✅ 加了 `false`

- 双击 exe
- **直接启动 GUI**
- 没有任何黑窗口

📌 对普通用户来说：

> “有没有黑框 = 像不像正经软件”

------

### 底层原理（你是程序员，这段很值）

Windows 有两种程序子系统：

| 类型       | 子系统    |
| ---------- | --------- |
| 控制台程序 | `CONSOLE` |
| GUI 程序   | `WINDOWS` |

`jpackage --win-console false` 做的事是：

- 生成的 exe 使用 **WINDOWS 子系统**
- 启动时不绑定控制台

这不是 Java 参数，是 **exe 级别行为**

------

### 什么时候一定要用？

| 场景           | 是否用 |
| -------------- | ------ |
| JavaFX / Swing | ✅ 必须 |
| 桌面工具       | ✅      |
| 给普通用户     | ✅      |
| CLI 工具       | ❌      |

------

### 什么时候千万别用？

#### ❌ 命令行工具

如果你的程序靠：

```java
System.out.println()
```

那加了 `false` 之后：

- 输出看不见
- 错误也看不见
- 调试非常痛苦

👉 CLI 工具 **不要加**

------

### 调试期 vs 发布期（推荐做法）

#### 🧪 调试期

```xml
<arg value="--win-console"/>
<arg value="true"/>
```

#### 🚀 发布期

```xml
<arg value="--win-console"/>
<arg value="false"/>
```

你甚至可以用 Maven profile 切换。

------

### 常见误解 & 坑

#### ❌ 误解 1：会影响日志

➡️ 不会
日志正常写文件（log4j / jul）

------

#### ❌ 误解 2：程序崩溃没法看错误

➡️ 正确
**必须有日志文件**
否则用户只会说一句：

> “打不开”

------

### 强烈推荐的配套参数

```xml
<arg value="--win-console"/><arg value="false"/>
<arg value="--java-options"/><arg value="-Dfile.encoding=UTF-8"/>
<arg value="--java-options"/><arg value="-Dapp.env=prod"/>
```

------

### 一句话总结

> `--win-console false` = **“这是一个 Windows GUI 程序” 的声明**

## 4. 在安装过程中，**允许用户自己选择安装目录**

### `--win-dir-chooser` 是干嘛的？

```xml
<arg value="--win-dir-chooser"/>
```

**一句话：**
👉 在安装过程中，**允许用户自己选择安装目录**

------

### 用户实际看到的流程

#### ❌ 不加（默认）

- 安装路径固定

  ```text
  C:\Users\xxx\AppData\Local\Programs\MyApp
  ```

- 用户不能改

#### ✅ 加了

- 安装向导中出现：

  > “请选择安装位置”

- 用户可改到：

  - D:\Apps\MyApp
  - 自定义路径

📌 很多用户 **非常在意装哪儿**

------

### 和其他参数的关系（非常重要）

#### 1️⃣ 和 `--win-per-user-install`

```xml
<arg value="--win-per-user-install"/>
<arg value="--win-dir-chooser"/>
```

效果：

- 不需要管理员权限
- 但 **只能选用户有权限的目录**

⚠️ 选 `C:\Program Files` 会被拒绝

------

#### 2️⃣ 不加 `--win-per-user-install`

```xml
<arg value="--win-dir-chooser"/>
```

效果：

- 默认全局安装
- 会弹 UAC（管理员权限）
- 可选 `Program Files`

📌 更“传统软件”，但失败率更高

------

### 什么时候该用？

| 场景              | 建议   |
| ----------------- | ------ |
| 内部工具 / 开发者 | ✅      |
| 面向技术用户      | ✅      |
| 企业分发          | ✅      |
| 小白用户          | ⚠️ 可选 |
| 免安装绿色软件    | ❌      |

------

### 常见坑 & 误解

#### ❌ 误解 1：可以随便选任何目录

➡️ 不行
Windows 权限说了算

------

### #❌ 误解 2：选了目录，卸载会乱

➡️ 不会
卸载器会记住安装路径

------

#### ❌ 误解 3：快捷方式会失效

➡️ 不会
快捷方式指向注册的 exe

------

### 实际项目里的推荐组合

#### ✅ 最稳妥（90% 桌面软件）

```xml
<arg value="--win-per-user-install"/>
<arg value="--win-dir-chooser"/>
```

#### ✅ 偏“传统安装器”

```xml
<arg value="--win-dir-chooser"/>
```

------

### 产品层面的取舍（经验）

- **给普通用户**
  👉 不选目录，降低决策成本
- **给高级用户**
  👉 必须给选择权
- **公司内部分发**
  👉 通常要求固定盘符

------

### 一句话总结

> `--win-dir-chooser` = **“我尊重用户的磁盘洁癖”**
---

## 5. 在安装过程中，**插入一个“许可协议（License）页面”**

### `--license-file` 是干嘛的？

```xml
<arg value="--license-file"/>
<arg value="LICENSE.txt"/>
```

**一句话：**
👉 在安装过程中，**插入一个“许可协议（License）页面”**
用户必须 **同意** 才能继续安装。

------

### 用户实际看到的效果

安装流程会变成：

```text
欢迎
↓
许可协议（滚动文本，需勾选同意）
↓
安装路径（可选）
↓
安装
```

📌 这是 **jpackage 唯一支持的自定义页面**

------

### 这个页面你能改什么？

#### ✅ 能改的

- **全部文字内容**
- 语言（中文 / 英文 / 中英混合）
- 排版（纯文本 / 换行）

#### ❌ 不能改的

- 页面布局
- 标题文案
- 按钮文字
- 字体 / 字号
- 背景 / Logo

👉 **只能改“文本内容”**

------

### License 文件格式要求（非常重要）

#### ✅ 推荐格式

- `.txt`
- UTF-8 编码
- 纯文本

```text
MyApp License Agreement

1. This software is provided "as is" ...
2. You may use it for free ...
```

⚠️ 注意：

- ❌ 不支持 HTML
- ❌ 不支持 Markdown
- ❌ 不支持富文本

------

### 中文 License 的坑（必看）

#### ⚠️ 编码问题

- **必须 UTF-8（无 BOM）**
- 否则会乱码

Windows 记事本：

> 另存为 → 编码 → UTF-8

------

### 常见使用场景

#### 1️⃣ 开源软件（强烈建议）

```text
MIT License
Apache License 2.0
GPL v3
```

------

#### 2️⃣ 免费但非开源

```text
本软件可免费使用
禁止反编译、商业转售
```

------

#### 3️⃣ 企业内部工具

```text
仅限公司内部使用
禁止外传
```

------

### 如果你不加会怎样？

| 情况                  | 结果                      |
| --------------------- | ------------------------- |
| 不加 `--license-file` | **不会显示 License 页面** |
| 加了                  | 用户必须同意才能安装      |

📌 **不是强制的，但专业软件基本都会加**

------

### Maven 中的推荐写法

```xml
<arg value="--license-file"/>
<arg value="${project.basedir}/LICENSE.txt"/>
```

📁 放在项目根目录最稳

------

### 和法律 / 合规的关系（现实提醒）

- **有 License ≠ 自动合法**
- 但：
  - 能明确责任
  - 能减少纠纷
  - 企业环境里几乎是必备

------

### 一句话总结（记住这句）

> `--license-file` = **jpackage 唯一允许你“说话”的页面**

## 6. 设置**应用的描述信息（Description）**

### `--description` 是干嘛的？

```xml
<arg value="--description"/>
<arg value="My desktop application"/>
```

**一句话：**
👉 设置**应用的描述信息（Description）**

------

### 这个描述会显示在哪里？

#### 1️⃣ Windows「程序和功能」

路径：

```text
控制面板 → 程序 → 程序和功能
```

你的软件条目下会显示：

- 程序名
- 版本
- **描述（Description）**
- 发布者（Vendor）

------

#### 2️⃣ 安装器的“详细信息区域”

- 某些 Windows 版本
- 安装过程中 / 安装完成提示

📌 **不是欢迎页正文**

------

### 它不会显示在哪里？（重要）

❌ 安装欢迎界面的大段文字
❌ 开始菜单
❌ 桌面快捷方式
❌ 程序窗口标题

------

### 描述写得好 vs 写得随便

#### ❌ 不推荐

```text
My desktop application
```

太泛，毫无信息量

------

#### ✅ 推荐（真实产品风格）

```text
A lightweight desktop tool for file processing
A JavaFX-based utility for managing local data
Offline desktop application built with Java
```

------

#### ✅ 中文示例（完全 OK）

```text
本地运行的桌面工具，用于文件处理与管理
基于 JavaFX 的轻量级桌面应用
```

------

### 写 `--description` 的最佳实践（经验）

#### 📏 长度

- **1 句**
- 60～120 字符以内

------

#### 🧠 内容建议

包含至少 1 个：

- 功能
- 使用场景
- 技术特点（如 Offline / Local）

------

#### ❌ 不要写

- 欢迎语
- 免责声明
- 更新日志
- 宣传语

------

### 和 `--vendor` 的组合（强烈推荐）

```xml
<arg value="--vendor"/>
<arg value="Aidan Studio"/>

<arg value="--description"/>
<arg value="Offline desktop tool built with JavaFX"/>
```

Windows 中显示效果：

```text
发布者：Aidan Studio
说明：Offline desktop tool built with JavaFX
```

👉 一下子就“正规软件”了

------

### Maven 中的推荐写法

```xml
<arg value="--description"/>
<arg value="${project.description}"/>
```

然后在 POM 里：

```xml
<description>
本地运行的桌面工具，用于文件处理
</description>
```

更统一、更好维护 👍

------

### 一句话总结

> `--description` = **Windows 系统层面对你软件的官方简介**

不是给用户看的欢迎词，而是给**系统、管理员、运维**看的。
---
## 7. 设置**应用的图标**

### `--icon` 是干嘛的？

```xml
<arg value="--icon"/>
<arg value="src/main/resources/icon/app.ico"/>
```

**一句话：**
👉 指定**应用图标**，用于：

* 安装器
* 生成的 exe
* 桌面快捷方式
* 开始菜单

📌 没它 = 白色 Java 默认图标（非常拉胯）

---

### Windows 对 `.ico` 的真实要求（重点）

#### ✅ 必须是多尺寸 ICO（不是单图！）

一个合格的 Windows ico **至少包含**：

| 用途       | 尺寸      |
| -------- | ------- |
| 文件 / exe | 16×16   |
| 开始菜单     | 32×32   |
| 桌面       | 48×48   |
| 高分屏      | 256×256 |

👉 推荐组合：

```text
16x16
32x32
48x48
256x256
```

❌ 只有一个 256×256 的 ico
❌ 从 png 直接改后缀

都会导致：

* 桌面模糊
* 开始菜单不显示
* exe 图标白块

---

### 路径写法的坑（你这个很常见）

你现在写的是：

```xml
<arg value="src/main/resources/icon/app.ico"/>
```

⚠️ 注意：
`jpackage` **不是在 classpath 里找文件**，而是：

> **以执行 Maven 时的工作目录为基准找真实文件**

#### ✅ 推荐写法（最稳）

```xml
<arg value="--icon"/>
<arg value="${project.basedir}/src/main/resources/icon/app.ico"/>
```

或者干脆：

```xml
<arg value="--icon"/>
<arg value="${project.build.directory}/icon/app.ico"/>
```

（打包前拷贝过去）

---

### icon 放哪里最合理？（工程实践）

#### ✅ 推荐结构

```text
project
 ├─ src/main/resources
 │   └─ icon
 │       └─ app.ico
 └─ pom.xml
```

⚠️ 但记住：

* **不是给程序用**
* 是给 **jpackage 用**
* 不需要打进 jar

---

### 常见问题 & 解决

#### ❌ 图标不显示 / 显示成白图

原因：

* ico 只有一个尺寸
* ico 格式不标准

👉 用 ImageMagick 或专业工具生成

---

#### ❌ 桌面图标 OK，exe 还是老图标

原因：

* Windows 图标缓存

👉 解决：

```text
重启资源管理器
或
重启电脑
```

---

#### ❌ 图标在 mac / linux 失效

正常：

* Windows 只认 `.ico`
* mac 要 `.icns`
* linux 要 `.png`

👉 用 profile 区分

---

### 跨平台推荐写法（进阶）

```xml
<profiles>
  <profile>
    <id>windows</id>
    <properties>
      <app.icon>${project.basedir}/src/main/resources/icon/app.ico</app.icon>
    </properties>
  </profile>

  <profile>
    <id>mac</id>
    <properties>
      <app.icon>${project.basedir}/src/main/resources/icon/app.icns</app.icon>
    </properties>
  </profile>
</profiles>
```

```xml
<arg value="--icon"/>
<arg value="${app.icon}"/>
```

---

### 一句话总结（记住这句）

> `--icon` = **你软件的“脸”**

做桌面软件：

* 图标不对
* 再好的功能都显得业余

