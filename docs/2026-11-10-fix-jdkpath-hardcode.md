# 2026-11-10 Java CI（make 构建）修复

## 现象

`Java CI with Maven` workflow 中 "Build with Make" 失败：

```
/bin/sh: 1: /Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/bin/javac: not found
make[1]: *** [Makefile:13: together] Error 127
```

## 根因

仓库内所有子目录 Makefile（src/Makefile、src/{global,btree,bufmgr,...}/Makefile）把 `JDKPATH` 硬编码为作者当时的 **macOS JDK1.7 路径**：

```make
JDKPATH = /Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home
```

在 CI 的 Linux runner 上该路径不存在，`javac` 404 → Error 127。

## 修复

1. 全部 Makefile：`JDKPATH = /Library/...` → `JDKPATH ?= $(JAVA_HOME)`（默认取环境 JAVA_HOME，可被命令行覆盖）；
2. `.github/workflows/maven.yml`：
   - JDK 由 17 → **8**（1997 年代教学代码，先用低版本 JDK 缩小兼容面）；
   - `runs-on` 显式改 `ubuntu-24.04`（避开 ubuntu-latest→26.04 迁移期镜像兼容问题）。

## 后续

若 JDK8 编译仍报 Java 语法/类库问题，将按照 flake8 首跑日志继续迭代（本文件随每次修复更新）。

## 验证记录
| Run | 结果 |
|---|---|
| 42832065678 (d54b73f) | ✅ success |
