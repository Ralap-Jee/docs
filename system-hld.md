# AkkanMore HLD 高层次描述

# 欢迎来到AkkanMore的技术文档

[AkkanMore](https://github.com/Ralap-Jee/akkanmore/) 是一个基于 [Akka](https://akka.io/) 的分布式系统异步类脑芯片系统级仿真器，提供了一套完整的异步类脑芯片架构仿真系统的开发解决方案

## 安装说明

推荐在课题组GPU服务器上安装，该服务器包含了所有的依赖库和配置文件

* 通过配置全局环境变量,指定Java/Scala/Sbt库:

```shell
#!/bin/bash

echo "Setting up sbt environment..."
sbt_setup=/data/zhangjian22/akkanmore/setup/
export JAVA_HOME=$sbt_setup/jdk1.8.0_161
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:JAVA_HOME/lib/dt.jar:$JAVA_HOME/bin/tools.jar

export SCALA_HOME=sbt_setup/scala-2.13.1
export PATH=$PATH:$SCALA_HOME/bin

export SBT_HOME=$sbt_setup/sbt
export PATH=$PATH:$SBT_HOME/bin
export SBT_OPTS="-Dsbt.override.build.repos=true"
```

保存成文件如`sbt_setup.sh`，在每次运行仿真器前执行`source sbt_setup.sh`即可

## 项目结构

    docs/                   # The documention directory.
    logs/                   # The simulation ouput log directory.
    project/                # The building files of the project using Sbt.
    src/main/               # The main source code directory.
    ├── resources/
    │   ├── logback.xml     # The logback configuration file.
    ├── scala/
    │   ├── akkanmore/      # The main package of the project.
    │   │   ├── AsyncLib/   # The asynchronous library package.
    │   │   │   ├── ...
    │   │   ├── NoC/        # The network-on-chip modeling package.
    │   │   │   ├── ...
    │   │   ├── PE/         # The processing element modeling package.
    │   │   │   ├── ...
    │   │   ├── config/     # The configuration parameter package.
    │   │   │   ├── ...
    │   │   ├── example/    # The example usage.
    │   │   │   ├── ...
    │   │   ├── util/       # The utility package.
    │   │   │   ├── ...
    └────
    target/                 # The building output directory.
    README.md               # The README file of the project.
    build.sbt               # The building configuration file of the project.

## 项目信息

开发人员名单可见于 [贡献者](https://github.com/Ralap-Jee/akkanmore/graphs/contributors)
