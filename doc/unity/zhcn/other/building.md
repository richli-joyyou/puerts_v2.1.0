# PuerTS Unity Plugin 编译指南
在unity里使用PuerTS时，Plugins目录的puerts.dll,puerts.bundle等，都是C++编译成的native plugin。

如果遇到了需要Debug版plugin、或者其他需要修改C++代码的需求，你可以根据这个文档操作。

## 2.0.0以上版本指引
2.0版本后，PuerTS采用Node.js驱动cmake编译PuerTS的native plugin。你只需要进入`unity/native_src/`目录，输入以下命令即可完成编译
```
node ../cli make --backend v8_9.4 --platform win --arch x64
```
上述命令会有一点长，你也可以参见`unity/cli/cmd.mts`查找quickcommand相关的信息，比如以上命令可以简化为`node ../cli make vw6`

## 1.x版本指引

### PuerTS 官方是怎么编译 C++ plugin 的？

PuerTS采用`github action`进行自动化构建。因此，你可以很容易地在 .github/workflows/build_unity_plugins.yml 中找到编译 C++ plugins 的步骤。

以 macos 为例。你可以在 build_unity_plugins.yml 中找到构建的几大步骤:
1. 下载v8/quickjs/node依赖库
```
- name: Download artifact
  uses: dawidd6/action-download-artifact@v2
    with:
      workflow: build_v8.yml
      name: v8_bin
      repo: puerts/backend-v8
      path: unity/native_src/
```
说明：
PuerTS负责构建v8依赖库的仓库是https://github.com/puerts/backend-v8（你也可以该仓库所属组织下找到负责构建QuickJS和NodeJS依赖库的仓库）
而上述action的这个步骤，就会把v8依赖库仓库下名为`build_v8`的action的产物下载到`native_src`目录下。

2. 执行构建
```
- name: Build
  run: |
    cd unity/native_src
    npx ts-node --esm make.mts --platform osx
```
说明：
这个`npx ts-node --esm make.mts`会调用`cmake`启动c++编译，最终在 `unity/native_src` 目录下生成build_osx_v8目录。里面就包含着我们需要的构建产物：puerts.bundle。

> 1.3版本及以前，这里用的是sh make_osx.sh

### 本地构建
如果要本地构建puerts，只需要按照上述github action workflow的流程跑一遍即可。

当然，在此之前你需要先准备好相关的构建编译工具，比如XCode、VisualStudio、CMake等，这部分内容在互联网上很容易能搜到，就不多表述了。

1. 将 https://github.com/puerts/backend-v8 或者 https://github.com/puerts/backend-quickjs 等JS引擎的构建产物放入`unity/native_src`目录，它们可以在github项目首页的`Actions`tab里找到。（其中需要注意的是，因为github action页面的编排的问题，鼠标滚动的时候需要将鼠标放在页面比较边缘的位置）
2. 执行`npx ts-node --esm make.mts`，并将你需要的JS引擎作为参数传入，比如`npx ts-node --esm make.mts --backend quickjs`。如果不传最后的参数，默认为v8。`npx ts-node --esm make.mts -h`可以看到更多的编译选项，如编译其他平台的plugin或者是调试版本plugin