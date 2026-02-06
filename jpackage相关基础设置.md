# jpackage相关基础设置

## 1.设置应用的“内部名称 / 安装标识名”

### `--name` 是干嘛的？

```xml
<arg value="--name"/>
<arg value="${app.name}"/>
```

**#一句话：**
👉 设置**应用的“内部名称 / 安装标识名”**

它会直接影响：

* 安装包名称
* 安装目录名
* 开始菜单分组
* exe / app 名称
* 卸载器名称

📌 可以理解为：**操作系统层面的唯一 ID**

---

### Windows 下它具体影响什么？

假设：

```xml
<app.name>MyApp</app.name>
```

#### 1️⃣ 生成的安装器文件名

```text
MyApp-1.0.0.exe
```

---

#### 2️⃣ 默认安装目录

##### 单用户安装

```text
C:\Users\xxx\AppData\Local\Programs\MyApp
```

##### 全局安装

```text
C:\Program Files\MyApp
```

---

#### 3️⃣ 开始菜单结构

```text
开始
 └─ MyApp
    └─ MyApp
```

（如果你用了 `--win-menu`）

---

#### 4️⃣ exe 文件名

```text
MyApp.exe
```

---

#### 5️⃣ 卸载器显示名称

```text
卸载 MyApp
```

---

### macOS 下它影响什么？

#### 1️⃣ 应用包名

```text
MyApp.app
```

---

#### 2️⃣ dmg / pkg 名称

```text
MyApp-1.0.0.dmg
MyApp-1.0.0.pkg
```

---

#### 3️⃣ Launchpad / Spotlight 显示名

```text
MyApp
```

---

### Linux 下它影响什么？

* 可执行命令名
* `.desktop` 文件名
* 安装目录名

---

### `--name` 的**硬性规则**（很重要）

#### ✅ 推荐写法

* 英文
* 无空格
* 无特殊符号
* PascalCase / camelCase

```text
MyApp
FileTool
DataManager
```

---

#### ❌ 不推荐 / 容易出事

```text
My App        ❌ 空格
我的工具        ❌ 非 ASCII（跨平台风险）
My-App        ⚠️ 某些平台会被处理
my_app        ⚠️ Linux OK，Windows/mac 不统一
```

📌 **显示给用户看的名字 ≠ `--name`**

---

### 那用户看到的“好看名字”怎么办？

### 👉 用 `--app-image-name` / `--mac-package-name` / `--description`

但最常用、最稳的是：

#### ✅ mac / Windows 都支持的方式

```xml
<arg value="--name"/>
<arg value="myapp"/>

<arg value="--description"/>
<arg value="My App 文件管理工具"/>
```

或者在 mac 上：

```xml
<arg value="--mac-package-name"/>
<arg value="My App"/>
```

---

### 强烈建议的做法（产品级）

### 🔒 内部名 vs 显示名分离

```xml
<properties>
    <app.id>myapp</app.id>
    <app.name>My App</app.name>
</properties>
```

```xml
<arg value="--name"/>
<arg value="${app.id}"/>

<arg value="--description"/>
<arg value="${app.name}"/>
```

这样你未来：

* 改品牌名
* 改显示文案

都 **不影响安装结构**

---

### 常见坑（真实案例）

### ❌ 后期改了 `--name`

结果：

* Windows 认为是**另一个软件**
* 无法覆盖安装
* 用户机器上出现两个版本

👉 **一旦发布，`--name` 基本不能再改**

---

### 一句话总结（记住这句）

> `--name` = **操作系统认你的“真名”**

---

## 2.设置操作系统识别用的应用版本号

### `--app-version` 是干嘛的？

```xml
<arg value="--app-version"/>
<arg value="${app.version}"/>
```

**一句话：**
👉 设置**操作系统识别用的应用版本号**

这是 **系统级版本**，不是给你代码用的。

---

### Windows 下它具体影响什么？

#### 1️⃣ 安装器文件名

```text
MyApp-1.2.3.exe
```

---

#### 2️⃣「程序和功能」里的版本号

路径：

```text
控制面板 → 程序 → 程序和功能
```

显示为：

```text
MyApp
版本：1.2.3
```

---

#### 3️⃣ 升级 / 覆盖安装（非常关键）

* 新版本 **version > 旧版本**
  👉 Windows 认为是升级
* version 一样
  👉 覆盖行为不可控
* version 更小
  👉 安装可能被拒绝

📌 **这是 Windows Installer 的核心判断依据**

---

### macOS 下它影响什么？

#### 1️⃣ pkg / dmg 文件名

```text
MyApp-1.2.3.pkg
MyApp-1.2.3.dmg
```

---

#### 2️⃣ Finder / 关于此 Mac / 系统信息

* App 版本号
* 用于更新判断

---

#### 3️⃣ Gatekeeper / 签名一致性

macOS 会把：

* `--name`
* `--app-version`

一起作为 App Identity 的一部分

---

### Linux 下它影响什么？

* deb / rpm 的版本字段
* 包管理器升级逻辑

---

### `--app-version` 的**硬性规则**

#### ✅ 推荐格式（跨平台最稳）

```text
X.Y.Z
```

例如：

```text
1.0.0
1.2.3
2.0.1
```

---

#### ⚠️ 可用但有风险

```text
1.0
1.0.0-beta
1.0.0-SNAPSHOT
```

不同平台处理不一致，**不推荐用于正式发布**

---

#### ❌ 明确不推荐

```text
v1.0.0        ❌ 带字母前缀
2026-02-06    ❌ 日期格式
1.0.0+build  ❌ SemVer 扩展
```

Windows Installer 可能直接拒绝。

---

### `pom.xml` 里的最佳实践

#### ❌ 不要直接用 `${project.version}`（坑）

```xml
<version>1.0.0-SNAPSHOT</version>
```

会导致：

```text
jpackage 报错 / Windows 无法识别
```

---

#### ✅ 正确姿势（你现在这个就对）

```xml
<properties>
    <app.version>1.0.0</app.version>
</properties>
```

```xml
<arg value="--app-version"/>
<arg value="${app.version}"/>
```

---

### 调试版 vs 发布版（强烈建议）

#### 🧪 调试 / 内测

```text
1.0.0
```

（内部版本体现在文件名或 about 页面）

---

#### 🚀 正式发布

```text
1.0.1
1.1.0
2.0.0
```

---

### 升级策略（产品级经验）

#### 🔹 永远只增不减

* 即使回滚功能
* 版本号也继续往上走

---

#### 🔹 和 `--name` 绑定

> **`--name + --app-version` = 系统认为的“同一个软件”**

其中任意一个变了：

* Windows / mac 都会当成新应用

---

### 常见坑（真实踩过）

#### ❌ 改了版本，但安装没覆盖

原因：

* 安装类型不同（per-user ↔ global）
* name / vendor 变了
* 版本格式不合法

---

### 一句话总结（记住这个）

> `--app-version` = **系统用来判断“你是不是新版本”的唯一依据**

---
## 3. jpackage 会从这个目录里找你要打包的所有文件

### `--input` 是干嘛的？

```xml
<arg value="--input"/>
<arg value="${project.build.directory}"/>
```

**一句话：**
👉 指定 **jpackage 的输入目录**
也就是：**jpackage 会从这个目录里找你要打包的所有文件**

---

### jpackage 会在 `--input` 里干什么？

在这个目录里，jpackage 会：

* 找 `--main-jar` 指定的 jar
* 复制 **你显式放进去的文件**
* 打包进最终 app image / 安装器

📌 **不会递归去 Maven 仓库找依赖**

---

### 你现在这个写法的含义

```xml
<arg value="${project.build.directory}"/>
```

等价于：

```text
target/
```

前提是：

```text
target/
 ├─ your-app.jar
 ├─ dependency1.jar
 ├─ dependency2.jar
```

否则就会翻车。

---

### `--input` + `--main-jar` 是一对

#### 必须满足的条件

```xml
<arg value="--main-jar"/>
<arg value="${project.build.finalName}.jar"/>
```

意味着：

> `${project.build.directory}/${project.build.finalName}.jar`
> **必须真实存在**

否则报错：

```text
Error: Main jar does not exist
```

---

### 常见踩坑 1：只有一个瘦 jar

如果你的项目是：

* 普通 `maven-jar-plugin`
* **没有打依赖**

那 `target/` 里只有：

```text
myapp.jar
```

运行后会：

```text
NoClassDefFoundError
```

📌 **jpackage 不会帮你拉依赖**

---

### 解决方案（必选其一）

#### ✅ 方案 1：Fat / Uber Jar（最简单）

用：

* `maven-shade-plugin`
* `maven-assembly-plugin`

输出：

```text
target/myapp.jar   （已包含全部依赖）
```

`--input target` 就完全 OK。

---

#### ✅ 方案 2：依赖 + 主 jar 分开放（更专业）

结构：

```text
target/
 ├─ app.jar
 ├─ lib/
 │   ├─ a.jar
 │   ├─ b.jar
```

jpackage 参数：

```xml
<arg value="--input"/>
<arg value="${project.build.directory}"/>

<arg value="--main-jar"/>
<arg value="app.jar"/>

<arg value="--java-options"/>
<arg value="-cp app.jar;lib/*"/>
```

（Windows 分号，mac/Linux 用冒号）

---

#### ✅ 方案 3：用 jlink（进阶）

* 先用 jlink 生成 runtime
* 再给 jpackage 用

适合：

* 大型应用
* 对体积 / 启动速度敏感

---

### 常见踩坑 2：以为 resources 会自动进来

❌ 错误认知：

> “我 resources 在 jar 里，jpackage 应该能用吧？”

✔ **前提是你真的把它们打进 jar 了**

否则：

* 配置文件
* native 文件
* 外部资源

必须 **手动拷到 `--input`**

---

### 工程级推荐目录结构（很稳）

```text
target/jpackage-input/
 ├─ app.jar
 ├─ lib/
 │   └─ *.jar
 ├─ config/
 └─ native/
```

```xml
<arg value="--input"/>
<arg value="${project.build.directory}/jpackage-input"/>
```

👉 **不要直接用裸 `target/`**（杂）

---

### 常见踩坑 3：路径对，但还是找不到

原因：

* `--input` 是 **相对 jpackage 执行目录**
* AntRun / Maven 的 cwd 不是你想的那个

👉 解决：**永远用绝对路径**

```xml
<arg value="${project.build.directory}"/>
```

（你现在这个是对的 👍）

---

### 一句话总结（记住这句）

> `--input` = **“我允许 jpackage 打包的文件范围”**

---

## 4. 指定 jpackage 启动时要执行的主 JAR 文件

### `--main-jar` 是干嘛的？

```xml
<arg value="--main-jar"/>
<arg value="${project.build.finalName}.jar"/>
```

**一句话：**
👉 指定 **jpackage 启动时要执行的主 JAR 文件**

也就是：

> **用户双击 exe / app，本质就是在运行这个 jar**

---

### 它和 `--input` 的关系（必须一起理解）

你前面写的是：

```xml
<arg value="--input"/>
<arg value="${project.build.directory}"/>
```

这两者组合起来，实际含义是：

```text
jpackage 会在：
target/
里寻找：
${project.build.finalName}.jar
```

📌 **不是 classpath，不是 Maven 依赖**
👉 就是一个真实存在的文件

---

### `${project.build.finalName}` 是什么？

默认规则：

```text
finalName = ${artifactId}-${version}
```

比如：

```xml
<artifactId>jpackahe</artifactId>
<version>1.0-SNAPSHOT</version>
```

那么实际文件名是：

```text
jpackahe-1.0-SNAPSHOT.jar
```

⚠️ 如果你 **同时又定义了 `app.version`**，
那 jar 名 ≠ 安装器版本，这是两个系统。

---

### 常见踩坑 1：jar 名对不上（最常见）

#### ❌ 错误现象

```text
Error: Main jar does not exist
```

#### 原因

* 你改过 `<finalName>`
* 或用了 shade / assembly 插件
* 或版本号带 `-SNAPSHOT`

#### ✅ 稳定做法（强烈推荐）

**明确指定 finalName：**

```xml
<build>
  <finalName>${app.id}</finalName>
</build>
```

然后：

```xml
<arg value="--main-jar"/>
<arg value="${app.id}.jar"/>
```

👉 一眼就知道在跑哪个 jar

---

### 常见踩坑 2：main class 没在 jar 里

即使 jar 文件存在，如果：

* `Main-Class` 没配置
* 或你忘了 `--main-class`

就会启动失败。

#### 两种正确方式（二选一）

##### ✅ 方式 1：Manifest 里声明（推荐）

```xml
<plugin>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>com.example.App</mainClass>
      </manifest>
    </archive>
  </configuration>
</plugin>
```

然后 **jpackage 不用再写 `--main-class`**

---

##### ✅ 方式 2：jpackage 指定

```xml
<arg value="--main-jar"/>
<arg value="app.jar"/>

<arg value="--main-class"/>
<arg value="com.example.App"/>
```

---

### 常见踩坑 3：依赖没进来（必踩）

#### 表现

* 安装成功
* 双击没反应 / 秒退

#### 原因

你的 jar 是 **瘦 jar**：

```text
只包含你自己的 class
```

#### 解决方案

##### ✅ 推荐（最简单）

用 `maven-shade-plugin` 生成 fat jar
让 `${project.build.finalName}.jar` 包含所有依赖

---

### 工程级推荐组合（直接可用）

```xml
<!-- 输入目录 -->
<arg value="--input"/>
<arg value="${project.build.directory}"/>

<!-- 主 jar -->
<arg value="--main-jar"/>
<arg value="${project.build.finalName}.jar"/>

<!-- 主类（可选，二选一） -->
<arg value="--main-class"/>
<arg value="${main.class}"/>
```

---

### 一句话总结（记住这句）

> `--main-jar` = **“用户双击时，真正被执行的那个 jar”**

它必须：

1. 真存在于 `--input`
2. 名字完全一致
3. 能独立启动（依赖齐全）
4. 有入口（Main-Class）

## 5. 指定 JAR 启动入口类（public static void main）

### `--main-class` 是干嘛的？

```xml
<arg value="--main-class"/>
<arg value="${main.class}"/>
```

**一句话：**
👉 指定 **JAR 启动入口类（public static void main）**

也就是告诉 jpackage：

> “这个 jar 启动时，请从哪个类开始跑”

---

### 什么时候**必须写**？

#### ✅ 情况 1：JAR 里 **没有 Main-Class**

你没有在 `MANIFEST.MF` 里配置：

```text
Main-Class: com.example.App
```

那就 **必须** 写：

```xml
<arg value="--main-class"/>
<arg value="com.example.App"/>
```

---

#### ✅ 情况 2：一个 jar 里 **有多个 main 方法**

比如：

* `App`
* `DevTool`
* `CliMain`

jpackage 不知道选哪个，
👉 你必须明确告诉它。

---

### 什么时候**不该写**？（很多人不知道）

#### ❌ 情况 3：Manifest 里已经声明了 Main-Class

如果你已经用：

```xml
<maven-jar-plugin>
  <manifest>
    <mainClass>com.example.App</mainClass>
  </manifest>
</maven-jar-plugin>
```

那：

* `java -jar app.jar` 能直接跑
* jpackage **会自动识别**

👉 **此时写不写都行，但建议不写，避免不一致**

---

### `--main-class` 和 `--main-jar` 的关系

这两个是 **一对**

```xml
<arg value="--main-jar"/>
<arg value="app.jar"/>

<arg value="--main-class"/>
<arg value="com.example.App"/>
```

含义是：

> 在 `app.jar` 里，用 `com.example.App` 作为入口

📌 **`--main-class` 不是全局的，只作用于 `--main-jar`**

---

### JavaFX 项目的特别说明（你很可能遇到）

#### ⚠️ JavaFX 真实入口 vs 表面入口

很多 JavaFX 项目：

```java
public class App extends Application {
    public static void main(String[] args) {
        launch(args);
    }
}
```

这种是 **OK 的**

但如果你是：

```java
public class App extends Application {
}
```

没有 `main()`，那：

❌ `--main-class App` 会直接启动失败

👉 你需要一个 **真正带 main 的启动类**：

```java
public class Launcher {
    public static void main(String[] args) {
        Application.launch(App.class, args);
    }
}
```

然后：

```xml
<arg value="--main-class"/>
<arg value="com.example.Launcher"/>
```

---

### 常见踩坑（血坑）

#### ❌ 包名写错 / 大小写错

```xml
com.example.app   ❌
com.example.App   ✅
```

Linux / mac 会直接找不到类。

---

#### ❌ 写成文件路径

```xml
com/example/App.java ❌
```

必须是 **完全限定类名**

---

#### ❌ 和 Manifest 不一致

Manifest 是：

```text
Main-Class: A
```

jpackage 写成：

```text
B
```

👉 结果不可预测

---

### 工程级推荐做法（最稳）

#### ✅ 方式 1（我最推荐）

* **只在 jar 里声明 Main-Class**
* jpackage **不写 `--main-class`**

优点：

* 单一事实来源
* jar / jpackage 行为一致

---

#### ✅ 方式 2（明确控制）

```xml
<properties>
    <main.class>com.example.Launcher</main.class>
</properties>
```

```xml
<arg value="--main-class"/>
<arg value="${main.class}"/>
```

适合：

* 多入口程序
* CLI + GUI 共存

---

### 最稳组合（GUI 桌面应用）

```xml
<arg value="--main-jar"/>
<arg value="${project.build.finalName}.jar"/>

<arg value="--main-class"/>
<arg value="${main.class}"/>

<arg value="--win-console"/><arg value="false"/>
```

---

### 一句话总结（记住这句）

> `--main-class` = **“程序从哪一行 `main()` 开始跑”**

写错 = 秒退
不该写却写 = 未来维护噩梦

## 6.指定 jpackage 最终生成物的输出目录

### `--dest` 是干嘛的？

```xml
<arg value="--dest"/>
<arg value="${project.build.directory}/installer"/>
```

**一句话：**
👉 指定 **jpackage 最终生成物的输出目录**

也就是：

> **安装器 / app-image 被放到哪里**

---

### 你现在这个写法的真实效果

```xml
<arg value="${project.build.directory}/installer"/>
```

等价于：

```text
target/installer/
```

里面会出现：

#### Windows

```text
target/installer/
 └─ MyApp-1.0.0.exe
```

#### macOS

```text
target/installer/
 └─ MyApp-1.0.0.dmg
```

#### Linux

```text
target/installer/
 └─ myapp_1.0.0_amd64.deb
```

📌 **非常干净，非常推荐**

---

### `--dest` 不会干什么？（重要）

* ❌ 不会影响安装路径
* ❌ 不会影响程序运行
* ❌ 不会参与升级判断

它只是 **构建产物目录**。

---

### 为什么不推荐省略 `--dest`？

如果你不写：

```xml
<arg value="--dest"/>
```

jpackage 会：

```text
把产物丢在当前工作目录
```

在 Maven 里这通常是：

```text
项目根目录
```

结果就是：

```text
pom.xml
MyApp-1.0.0.exe  ❌
src/
target/
```

👉 非常不专业

---

### 工程级推荐目录结构

#### ✅ 推荐（90% 项目）

```text
target/
 ├─ jpackage-input/
 ├─ installer/
 │   └─ MyApp-1.0.0.exe
 └─ ...
```

---

#### 🚀 CI / 发布友好型

```xml
<arg value="--dest"/>
<arg value="${project.build.directory}/dist"/>
```

然后 CI 只需要：

```bash
upload target/dist/*
```

---

### 常见踩坑

#### ❌ 目录不存在会失败吗？

👉 **不会**

jpackage 会自动创建目录。

---

#### ❌ 相对路径会乱吗？

会。

```xml
<arg value="installer"/>
```

在 CI / IDE / 命令行下行为不一致。

👉 **强烈建议绝对路径**

你现在这个写法是 **正确示范** 👍

---

#### ❌ 多平台同时打包会覆盖吗？

如果你在同一目录里打：

* Windows exe
* mac dmg

👉 **不会覆盖**（后缀不同）

但：

```text
同名 + 同类型 = 会覆盖
```

---

### 高级玩法（可选）

#### 按平台拆目录

```xml
<properties>
  <dist.dir>${project.build.directory}/installer/${os.detected.name}</dist.dir>
</properties>
```

```xml
<arg value="--dest"/>
<arg value="${dist.dir}"/>
```

生成：

```text
installer/
 ├─ windows/
 ├─ mac/
 └─ linux/
```

---

### 一句话总结（记住这句）

> `--dest` = **你“最终交付给用户”的那个文件放哪**

不影响运行，
但 **非常影响工程质量**。

## 7. 指定 生成哪一种安装包格式

### `--type exe` 是干嘛的？

```xml
<arg value="--type"/>
<arg value="exe"/>
```

**一句话：**
👉 指定 **生成哪一种安装包格式**

在 Windows 上：

| type        | 结果               |
| ----------- | ---------------- |
| `exe`       | `.exe` 安装器（最常用）  |
| `msi`       | `.msi` 安装包（企业部署） |
| `app-image` | 免安装目录（调试 / 便携）   |

---

### `exe` 的真实行为（你现在用的）

生成：

```text
MyApp-1.0.0.exe
```

用户双击后：

* 出现 **安装向导**
* 可选安装目录（如果开了 `--win-dir-chooser`）
* 可创建桌面 / 开始菜单快捷方式
* 安装到 `Program Files`

👉 **最符合普通用户直觉**

---

### 什么时候该用 `exe`？

#### ✅ 个人 / 商业桌面软件

* JavaFX / Swing
* ToC 应用
* 官网下载

#### ❌ 不适合：

* 企业批量部署（用 `msi`）
* 绿色免安装（用 `app-image`）

---

### `exe` 能配合哪些 Windows 专属参数？

你前面提到的这些，**只有 exe / msi 才生效** 👇

#### 常用（强烈推荐）

```xml
<arg value="--win-menu"/>
<arg value="--win-shortcut"/>
<arg value="--win-dir-chooser"/>
<arg value="--win-console"/><arg value="false"/>
```

效果：

* 开始菜单 ✔
* 桌面快捷方式 ✔
* 安装目录可选 ✔
* 不弹黑框 ✔（GUI 程序）

---

#### 品牌 / 合规相关

```xml
<arg value="--icon"/><arg value="app.ico"/>
<arg value="--description"/><arg value="My desktop application"/>
<arg value="--vendor"/><arg value="Your Company"/>
<arg value="--license-file"/><arg value="LICENSE.txt"/>
```

---

### `exe` vs `msi`（必须知道）

| 对比项     | exe  | msi |
| ------- | ---- | --- |
| 用户友好    | ⭐⭐⭐⭐ | ⭐⭐  |
| UI 可定制  | 较多   | 很少  |
| 企业部署    | ❌    | ✅   |
| 静默安装    | 较难   | 很好  |
| 签名 / 策略 | 一般   | 更规范 |

👉 **个人 & 独立软件：exe**
👉 **公司 IT 环境：msi**

---

### 常见坑（一定会遇到）

#### ❌ 在非 Windows 系统用 `exe`

```text
Error: Invalid or unsupported type: exe
```

👉 `exe` **只能在 Windows 上构建**

---

#### ❌ 写死 `exe`，CI 打多平台炸了

❌ 不推荐：

```xml
<arg value="exe"/>
```

✅ 推荐：

```xml
<properties>
  <jpackage.type>exe</jpackage.type>
</properties>
```

或者：

```xml
<arg value="${jpackage.type}"/>
```

---

### 多平台正确写法（工程级）

#### Windows profile

```xml
<profile>
  <id>windows</id>
  <properties>
    <jpackage.type>exe</jpackage.type>
  </properties>
</profile>
```

#### macOS

```xml
<jpackage.type>dmg</jpackage.type>
```

#### Linux

```xml
<jpackage.type>deb</jpackage.type>
```

---

### 一句话总结（记住这句）

> `--type exe` = **Windows 用户最熟悉的安装器**

用得最多，
但 **只能在 Windows 构建**，
最好 **不要写死在通用 pom 里**。








