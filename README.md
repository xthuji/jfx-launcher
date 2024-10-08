# JFX-Launcher

<a href="https://github.com/unclezs/jfx-launcher/actions/workflows/gradle.yml">
<img src="https://img.shields.io/github/actions/workflow/status/uncle-novel/jfx-launcher/gradle.yml" alt="gradle build"/>
</a>
<a href="https://openjdk.java.net/">
<img src="https://img.shields.io/badge/version-1.1.10-blue" alt="版本"/>
</a>
<a href="https://github.com/unclezs/jfx-launcher/blob/main/LICENSE">
<img src="https://img.shields.io/github/license/unclezs/jfx-launcher?color=%2340C0D0&label=License" alt="GitHub license"/>
</a>
<a href="https://github.com/unclezs/jfx-launcher/issues">
<img src="https://img.shields.io/github/issues/unclezs/jfx-launcher?color=orange&label=Issues" alt="GitHub issues"/>
</a>
<a href="https://openjdk.java.net/">
<img src="https://img.shields.io/badge/OpenJDK-11-green" alt="JDK Version"/>
</a>

一个openJfx的自动更新器，采用模块化API加载模块。
<p>
<img src="https://gitee.com/unclezs/image-blog/raw/master/start.png" />
<img src="https://gitee.com/unclezs/image-blog/raw/master/20210409094616.png"/>
</p>

## 引入依赖

### maven

```xml
<dependency>
  <groupId>com.unclezs</groupId>
  <artifactId>jfx-launcher</artifactId>
  <version>1.1.10</version>
</dependency>
```

### gradle

```groovy
implementation "com.unclezs:jfx-launcher:1.1.10"
```

## 原理

在Launcher启动的时候，会对比本地配置与服务端配置是否一致，如果服务端配置与本地不一致，则进行拉取同步。 对比条件：

1. 版本号是否一致
2. 各个文件大小是否发生改变

同步完成之后，通过无需调用java指令再去启动，直接通过ModuleApi加载依赖模块，支持打破模块规则的参数， 如：add-exports、add-opens、add-reads可以在配置文件中进行设置

支持加载本地Native库，通过指定资源类型为 NATIVE、NATIVE_SYS 区分系统库与自定义库

## 简介

### 指定启动参数

可以通过程序的启动参数来运行启动器：

- --configPath=配置文件路径(conf/app.json) *
- --appName=应用名称 *
- --url=资源下载地址 *
- --launchClass=启动类 *
- --launchModule=启动类所属模块 *

### 配置介绍

- **url**： 资源下载地址
- **configPath**： 相对于url的配置路径
- **configUrl**： 直接指定配置全路径 ， 指定了将忽略configPath
- **appName**： 应用名称
- **version**： 版本号
- **changeLog**： 更新日志
- **launchModule**： 启动类所属模块
- **launchClass**： 启动类
- **moduleOptions**： 模块的一些打破规则的参数 ： add-exports、add-opens、add-reads
- **resources**: 资源列表，升级时候可以自动更新的，可以指定JAR、NATIVE、NATIVE_SYS、FILE类型的，根据不同类型采取不同的加载策略

### 注意

如果添加了打破模块的规则，并且源模块不属于当前加载的layer，需要添加VM参数允许反射

```
--add-modules ALL-SYSTEM --add-opens=java.base/java.lang=com.unclezs.jfx.launcher
```

## 打包
### 打包jar文件到本地maven仓库
在Gradle的publishing repositories 配置中添加 mavenLocal()，发布时将jar同步到Maven的本地仓库
打包命令 `gradle publishToMavenLocal`

### 将打包的jar文件复制到gradle的cache目录
*如果Gradle优先使用的是Maven的本地仓库，可以忽略这个*

Mac上 gradle的cache目录路径: `~/.gradle/caches/modules-2/files-2.1/`

```shell
jarVersion="1.1.11-SNAPSHOT"
sourceDirPath="$HOME/work/MyProject/code_mine/github/my-folder/jfx-launcher/build/libs"
targetDirPath="$HOME/.gradle/caches/modules-2/files-2.1/com.unclezs/jfx-launcher"
ls -1 "${sourceDirPath}" | while read line ; do
    originJarPath="${sourceDirPath}/${line}"
    jarName="${line}"
    jarDirName=$(sha1sum "${originJarPath}" | awk '{print $1}')
    if [ ! -d "${targetDirPath}/${jarVersion}/$jarDirName/" ]; then
        mkdir -p "${targetDirPath}/${jarVersion}/$jarDirName/"
    fi
    echo "cp -pf \"${originJarPath}\" \"${targetDirPath}/${jarVersion}/$jarDirName/${jarName}\""
    cp -pf "${originJarPath}" "${targetDirPath}/${jarVersion}/$jarDirName/${jarName}"
done
```
