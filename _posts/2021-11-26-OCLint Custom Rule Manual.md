---
title:  "OCLint Custom Rule Manual" 
toc: true

---

---

OCLint is a static code analysis tool for improving quality and reducing defects by inspecting C, C++ and Objective-C code and looking for potential problems

---

从官方的解释来看，[OCLint](https://oclint.org) 是一种通过检查 C、C++、Objective-C 代码来寻找潜在问题，来提高代码质量并减少缺陷的静态代码分析工具。

## 下载和环境配置

有 3 种方式安装，分别为 Homebrew、下载安装包和源代码编译。 区别：

- 如果需要自定义 Lint 规则，则需要下载源码编译安装
- 如果仅仅是使用自带的规则来 Lint，那么以上3种安装方式都可以

这里需要自定义规则，所以介绍源码编译安装方式，另外两种按需后边补充。

### 源码编译

homebrew 安装 CMake 和 Ninja 这 2 个编译工具。

```
brew install cmake ninja
```

拉 OCLint 到本地

```
git clone https://github.com/oclint/oclint.git
```

进入 oclint-scripts 目录，执行 ./make 命令。

```
./make
```

这一步的时间非常长。会下载 oclint-json-compilation-database、oclint-xcodebuild、llvm 源码以及 clang 源码，进行相关的编译得到 oclint。

在同级的 build 目录下可以看到 oclint-release 。进入 oclint/build/oclint-release 目录执行。

```
cp <OCLint path>/build/oclint-release/bin/oclint* /usr/local/bin/
ln -s <OCLint path>/build/oclint-release/lib/oclint /usr/local/lib
ln -s <OCLint path>/build/oclint-release/lib/clang /usr/local/lib
```

使用 ln -s 把 lib 中的 clang 和 oclint 链接到 /usr/local/bin 目录下。目的是为了后面如果编写了自己创建的 lint 规则，不必要每次更新自定义的 rule 库。

cd 到根目录，配置编译环境，比如我是 zsh ，对应修改 .zshrc 文件：

```
OCLint_PATH=<OCLint path>/build/oclint-release
export PATH=$OCLint_PATH/bin:$PATH
```

source 下 .zhsrc

```
source .zshrc
```

## 生成工程

1. 进入 oclint 目录，执行:

   ```
   oclint-scripts/scaffoldRule CustomLint -t ASTVisitor
   ```

   **-t** 可以选择：ASTVisitor、SourceCodeReader、ASTMatcher。

   **CustomLint** 就是自定义的规则名称。

   ```
   RuleBase
    |
    |-AbstractASTRuleBase
    |      |_ AbstractASTVisitorRule
    |             |_AbstractASTMatcherRule
    |
    |-AbstractSourceCodeReaderRule
   ```

   - AbstractSourceCodeReaderRule：eachLine 方法，读取每行的代码，如果想编写的规则是需要针对每行的代码内容，则可以继承自该类。

   - AbstractASTVisitorRule：可以访问 AST 上特定类型的所有节点，可以检查特定类型的所有节点是递归实现的。在 **apply** 方法内可以看到代码实现。开发者只需要重载 bool visit* 方法来访问特定类型的节点。其值表明是否继续递归检查。
   - AbstractASTMatcherRule：实现 setUpMatcher 方法，在方法中添加 matcher，当检查发现匹配结果时会调用 callback 方法。然后通过 callback 方法来继续对匹配到的结果进行处理。

2. 在 oclint 根目录创建 oclint-xcoderules 文件夹，在  oclint-xcoderules 内创建 create-xcode-rules.sh，添加下面内容:

   ```shell
   #! /bin/sh -e
   
   cmake -G Xcode \
       -D CMAKE_CXX_COMPILER=../build/llvm-install/bin/clang++  \
       -D CMAKE_C_COMPILER=../build/llvm-install/bin/clang \
       -D OCLINT_BUILD_DIR=../build/oclint-core \
       -D OCLINT_SOURCE_DIR=../oclint-core \
       -D OCLINT_METRICS_SOURCE_DIR=../oclint-metrics \
       -D OCLINT_METRICS_BUILD_DIR=../build/oclint-metrics \
       -D LLVM_ROOT=../build/llvm-install/ ../oclint-rules
   ```

   M1 电脑这里貌似找不到 llvm 路径，需要将路径将相对路径为绝对路径。

   执行 create-xcode-rules.sh

   ```shell
   sh ./create-xcode-rules.sh
   ```

   运行完成后会生成 OCLINT_RULES 工程。

3. 在 oclint 根目录创建 oclint-xcodedriver 文件夹，在 oclint-xcodedriver 内创建 creat_xcode_driver.sh， 添加脚本：

   ```shell
   #! /bin/sh -e
   
   cmake -G Xcode \
       -D CMAKE_CXX_COMPILER=../build/llvm-install/bin/clang++  \
       -D CMAKE_C_COMPILER=../build/llvm-install/bin/clang \
       -D OCLINT_BUILD_DIR=../build/oclint-core \
       -D OCLINT_SOURCE_DIR=../oclint-core \
       -D LLVM_ROOT=../build/llvm-install/ ../oclint-driver
   ```

   M1 同上，需要将路径将相对路径为绝对路径。

   执行 creat_xcode_driver.sh

   ```shell
   sh ./creat_xcode_driver.sh
   ```

   运行完会生成 OCLINT_DRIVER 工程。

   在当前目录的 bin 目录下，看是否有 lib 文件夹，如果没有从根目录 **oclint -> build -> oclint-release** 将 lib 拷贝到 **oclint -> oclint-xcodedriver -> bin** 下，否则后边会报 report type 的错。

   ![image-20211126181618476](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126181618476.png?raw=true)

4. 打开 OCLINT_DRIVER.xcodeproj ，将 OCLINT_RULES.xcodeproj 拖进工程

   ![image-20211126183026290](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126183026290.png?raw=true)

   M1 需要将 Build system 设置为 New Build System，否则后边编译会报错提示。

5. target 选择 oclint，设备选择 Mac ，如果是 M1 的电脑需要选择 Rosetta 模式。

   ![image-20211126185902646](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126185902646.png?raw=true)

6. 执行 oclint 需要参数，参考以下内容：

   ```
   -R=/Users/xxx/Documents/Project/oclint/oclint-xcoderules/rules.dl/Debug --report-type=html -o /Users/xxx/Documents/temp/result.html  /Users/xxx/Documents/Project/oclint/oclint-xcodedriver/OCLintTest.m -- -fmodules -x objective-c -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator15.0.sdk
   ```

   [参数详细介绍看官方](https://docs.oclint.org/en/stable/manual/oclint.html) 

   isysroot 可以通过建一个空工程编译模拟器看编译日志获取。

   将内容修改好粘贴到 scheme 设置里

   ![image-20211126190930005](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126190930005.png?raw=true)

7. 在工程里设置要编译的依赖 rule（搜索你创建的名字，不一定是 CustomLint），如下：

   ![image-20211126191435729](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126191435729.png?raw=true)

   添加后可以编译一下，会得到 result.html。由于检查规则没有输出内容，所以 html 里没有内容。

接下来开发自定义规则。

## 自定义规则

1. 可以在工程目录里搜索之前创建的自定义规则名，然后进行自定义开发。

2. 这个文件很大，你忍一下。这个文件可以分成两部分，以 **setUp()** 和 **tearDown()** 为分界。上边是一些设置属性，下边是大量的 visit 函数。每一个 visit 的节点都是 ast 分析出的一种节点。

3. 创建一个 test 的 .m 文件，然后使用 clang 命令：

   ```
   clang -fmodules -fsyntax-only -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS15.0.sdk -Xclang -ast-dump OCLintTest.m
   ```

   会得到结果：

   ![image-20211126201353072](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126201353072.png?raw=true)

   前面就是它的节点类型，后边能获取到 loc、 name 等属性。节点分为 Decl 和 Stmt（还有一个它的子类 Expr），详细了解可以看 [Clang AST 基础学习](https://blog.csdn.net/hatter110/article/details/107282596)

4. 根据 ast-dump 代码文件，看节点的特征写出在对应的 visit 代码块里，这里先拿 ObjCInterfaceDecl 举个例子：

   ![image-20211126184024420](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126184024420.png?raw=true)

   ```
   addViolation(node->getBeginLoc(), node->getBeginLoc(), this, "hahahahahhaa");
   ```

   这里是检查到此节点时，进行分析逻辑，如果需要报错则使用 addViolation ，return 表示是否继续扫描。

5. 添加一个断点可以看到每一步执行的过程。

   ![image-20211126202600580](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126202600580.png?raw=true)

6. 查看 result.html 。

   ![image-20211126202736372](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/oclint/image-20211126202736372.png?raw=true)

这样就可以根据需求写对应的自定义规则。

## 应用到工程

每个工程的应用方式都不一样

[官方配合 xcode 的使用手册](https://docs.oclint.org/en/stable/guide/xcode.html)

参考链接：

[OCLint 实现 Code Review - 给你的代码提提质量](https://juejin.cn/post/6844903853775650830)

[OCLint 官网](https://oclint.org)

[Clang  AST](https://clang.llvm.org/docs/IntroductionToTheClangAST.html)

[Clang AST 基础学习](https://blog.csdn.net/hatter110/article/details/107282596)