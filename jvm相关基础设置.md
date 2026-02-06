# jvm相关基础设置

```xml
<arg value="--java-options"/>
<arg value="-Xmx512m"/>

<arg value="--java-options"/>
<arg value="-Dfile.encoding=UTF-8"/>
```

等价于用户最终运行：

```bash
java -Xmx512m -Dfile.encoding=UTF-8 -jar MyApp.jar
```

✔ JVM 最大堆内存 512MB
✔ Windows / mac / Linux 统一 UTF-8
✔ 对 exe / dmg / deb **全部生效**

---

## 这两项各自解决了什么问题？

### `-Xmx512m`

* 防止 JavaFX / Swing 内存突然顶满
* 大文件、图片、字体加载更稳
* 不会无限吃用户内存（很重要）

### `-Dfile.encoding=UTF-8`

* Windows 中文路径 / 文件名不乱码
* 配置文件 / 日志统一编码
* 避开系统默认编码差异

👉 **这两个参数 = 桌面应用“保命符”**

---

## 你现在“还可以补”的 3 个高性价比参数

### ① JavaFX 显卡兼容（强烈推荐）

```xml
<arg value="--java-options"/>
<arg value="-Dprism.order=sw"/>
```

作用：

* 解决老显卡 / 远程桌面 / 虚拟机黑屏
* 性能略降，但稳定性暴增

📌 **发布版比跑分更重要**

---

### ② 语言环境锁定（可选）

```xml
<arg value="--java-options"/>
<arg value="-Duser.language=zh"/>
<arg value="--java-options"/>
<arg value="-Duser.country=CN"/>
```

防止：

* 系统语言不同导致 UI / 时间格式乱跳

---

### ③ GC 稳定性（可选）

```xml
<arg value="--java-options"/>
<arg value="-XX:+UseG1GC"/>
```

桌面应用里 **G1 比默认更稳**

---

## 常见坑（你这段已经避开了）

### ❌ 写在一个 arg 里（你没有）

```xml
<arg value="-Xmx512m -Dfile.encoding=UTF-8"/> ❌
```

### ❌ 少了 `--java-options`（你没有）

```xml
<arg value="-Xmx512m"/> ❌
```

### ❌ 想让用户自己改（做不到）

jpackage 打包后：

* 用户基本 **不能改 JVM 参数**
* 所以：**发布前一定想清楚**

---

## 发布级推荐组合（直接抄）

### 普通 GUI 应用（最稳）

```xml
<arg value="--java-options"/><arg value="-Xmx512m"/>
<arg value="--java-options"/><arg value="-Dfile.encoding=UTF-8"/>
<arg value="--java-options"/><arg value="-Dprism.order=sw"/>
```

---

### 重 UI / 大文件版

```xml
<arg value="--java-options"/><arg value="-Xmx1024m"/>
<arg value="--java-options"/><arg value="-Dfile.encoding=UTF-8"/>
<arg value="--java-options"/><arg value="-XX:+UseG1GC"/>
```


