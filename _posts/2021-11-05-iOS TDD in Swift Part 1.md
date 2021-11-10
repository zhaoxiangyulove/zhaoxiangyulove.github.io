---
title:  "iOS TDD in Swift Part 1" 
---

---

我在开发中，经常会遇到的两种奇怪现象：

- 为了尽快提测，经常提测后才开始写 Unit Test Case (后边简称 UT)， QA 和 UT 几乎同时进行。当写 UT 时发现代码被乱放，需要调整位置，这可能会引起逻辑改动，导致了需要回测，浪费了开发和测试更多的时间。
- 当写完需求再去写 UT，当时是通过的，但随着需求迭代，有人在函数里加了不影响 UT 的代码，由于不强制在写代码前写好 Test Case，则可能会出现 UT 检测的和函数执行不一致。时间久了就会出现 UT 无法使用的情况。

Test-driven development(后边简称 TDD) 正好能解决上边两个现象，能让我写出 testable 和 reasonable 的 code，这就够了。

## Section I: Hello, TDD

### 1. Introduction To TDD

#### 1.1 What is TDD

TDD 是一种通过迭代地进行许多由支持测试的小变更来开发软件的迭代方式。这个是 TDD 循环（后边会详细介绍，先有个印象）

<img src="https://assets.alexandria.raywenderlich.com/books/itdd/images/1fd7eb6808587a2d64b149794f3228d347d9d62c10d15dba04ce068afce474aa/original.png" alt="img" style="zoom: 25%;" />

TDD 循环分为四步:

1. 写一个会失败的 test。
2. 让这个 test 通过。
3. 重构。
4. 重复执行所有 tests。

TDD 能确保彻底并精确地测试你的代码，因为你的代码是被 Test 驱动的。表面上看，TDD 似乎非常简单，实际也是如此。

#### 1.2 Why should you use TDD

TDD 是确保你的代码能执行成功和将来能执行成功最好的方式。

关于测试代码，并非必须使用 TDD，你当然可以完成业务开发后再写 UT，或者你可以直接手动测试你的代码（想起之前在的一个团队每两周都要发版一次，然后让开发们手动执行一遍 case 🐶）。为什么 TDD 比其他的方式好？

好的 test 能保证你的 App 按照期望运行，然而，不是所有的 Test 都是"好的"。好的 test 应该是可失败的、可重复的、快速运行的和可维护的。

TDD 提供了方法来确保你的 test 是好的：

- 第一步就是写一个会失败的 test，不失败的 Test 只是在浪费 CPU。
- 当你想写新的 test，需要确保所有之前的 tests 都通过，这就确保你的 tests 是可重复的。你不能只执行你正在开发的 test，建议执行全部 tests。
- 由于要经常执行所有的 tests，所以要确保 test 是快速执行的，如果太久，没人会执行所有的 tests。
- 当你在重构时，你需要同时更新了生产代码和测试代码，你需要不断地保持它们到最新。
- 通过同时迭代更新生产代码和测试代码，你确保了代码是可测试的，如果你是完成开发后才写 test，似乎会因为完成 UT 需要一些代码重构。

尽管如此，你可能认为即便不使用 TDD，你也能写出好的 test。你当然可以，短期很容易做到，但是长期下来会很难。不久，你会创造一些方法来确保你在写好的 test，你会发现你就在做 TDD！

#### 1.3 What should you test

更高的测试覆盖率不代表你的 App 正在被更好的测试，下边是一些建议：

- 建议为自动化不能捕获的代码写 test，比如你自定义类里的函数、自定义了 getters 和 setters 和大多数你自己写的代码，不建议为生成出的代码写 test，比如生成出 getters 和 setters，Swift 已经做的很好，你可以相信它能正常 work。
- 不建议为编译器能捕获的问题来写 test。
- 不建议为基础库或第三方库写 test，这些库的作者应该写 test，举个🌰，你不应该为 UIKit 的类来写 tests，然而你需要为你所继承的子类写 tests。这些是你自定义出来的代码，所以你应该写对应的 tests。

有两个例外，一种情况是你想知道一个 framework 是如何 work，那么写一些 test 是有帮助的，但是这些 tests不会持续很久。另一种是第三方的代码非常不稳定，所以需要写 tests 来确保，那么你应该考虑要不要使用这个库了。

#### TDD 花费太多时间！

最常见的抱怨了。

其实，当你习惯使用 TDD 来开发后会比根本不写 tests 更快，因为后者你最终写了更多的代码。TDD 会在开始消耗一些时间。

值得讨论的问题是：开发花费的真实时间！他不止是写一个版本，也包括后边的新需求，变更代码，修复 bugs 等等，在这个长跑过程中，相比于没有 TDD ，TDD 花费了更少的时间，因为它生产更相关的代码，带来更少的 bugs。

另一个需要考虑的问题：线上 bug 的影响。如果 bug 是开发过程中发现，会非常容易修复，但是如果过了几周才发现，你会花费更多时间 debg 代码来寻找根本原因，你的 tests 会帮助你对抗这种情况。

#### 1.4 When should you use TDD

如果你的 App将要有 release 的版本，同时/或者你的工程有复杂逻辑，那么你最好及时使用 TDD。

如果你只是用来写一个 demo 工程或者临时的工程， 那么你随意。

总之，TDD 是一个工具，你自己来决定什么时候来使用它。

#### 1.5 Key points

在这个小节，你应该了解了 TDD 是什么，为什么使用它，哪些代码应该被测试和什么时间去使用它。

- TDD 提够了写好的 tests 的方式。
- 好的 test 是可失败的、可重复的、快速运行的和可维护的。
- 开发真正的时间花费包含初始开发，添加新需求，修改现有逻辑和修复 bugs 等等。TDD 减少维护成本和保证了 bugs 的可控
- TDD 对于一个长期的项目是非常有帮助的。

### 2. Introduction To TDD Cycle

#### 2.1Getting Started

在这个小节，你讲继续关注于 TDD 代替 Xcode 的设置，你可以使用 playground，创建一个 **CashRegister.playground** 。"File" -> "New" -> "Playground", 选择 "Blank"。

![image-20211110182716628](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/image-20211110182716628.png?raw=true)

删掉所有的代码。

开始使用 TDD。

#### 2.2 Red: Write a falling test

在你写任何生产代码前，你需要写一个会失败的 test。

首先你需要建一个 test 的类，复制下边的代码到 playground。

```swift
import XCTest

class CashRegisterTests: XCTestCase {

}
```

编译，如果报错 **error: unable to load standard library for target 'arm64-apple-macos11.0'** 需要将 platform 设置为 macOS，运行通过。

接下来在 playground 最下方添加

```swift
CashRegisterTests.defaultTestSuite.run()
```

这是告诉 playground 去跑 <hl>CashRegisterTests</hl> 下所有的 tests，然而还没有任何 tests，所以在里边添加一个会引起编译报错的 test。

![WX20211110-185352](https://github.com/zhaoxiangyulove/zhaoxiangyulove.github.io/blob/main/assets/pic/WX20211110-185352.png?raw=true)

1. 解释下函数命名：

- XCTest：需要所有的测试函数以 <hl>test</hl> 开头。
- test：接下来是函数要测试的名字，这里是 <hl>init</hl>，用 "_"和下个部分区分。
- 最后的部分是期望的输出或结果，这里是 <hl>createsCashRegister</hl> 。

这种命名容易理解同时提供有用的信息，如果 test 失败了，Xcode 会告诉你 test 的类和函数名，通过这种命名方式可以快速了解问题。

2. 你试图初始化一个<hl>CashRegister</hl> 的实例。通过 <hl>XCTAssertNotNil</hl> 表达式来判断，当参数不为 nil 则通过。

然而，第七行并不能编译，因为你还没有创建<hl>CashRegister</hl> 的类，在 TDD 里有这样一条规则，编译失败视为 test 失败，所以你已经完成了 TDD 循环的第一步 Red，现在进行下一步。

#### 2.3 Green: Make the test pass

你只能写最少的代码让这个 test 通过，如果你写了比下边的代码更多，你的 test 将会落后于你的生产代码！那最少的代码能解决这个编译问题的代码是什么？声明一个<hl>CashRegister</hl>类！

在<hl>CashRegisterTest</hl>上方添加下面的代码：

```swift
class CashRegister {
  
}
```

点击执行，你会看到控制台输出：

```swift
Test Suite 'CashRegisterTests' started at 2021-11-10 22:20:45.074
Test Case '-[__lldb_expr_18.CashRegisterTests testInit_createsCashRegister]' started.
Test Case '-[__lldb_expr_18.CashRegisterTests testInit_createsCashRegister]' passed (0.003 seconds).
Test Suite 'CashRegisterTests' passed at 2021-11-10 22:20:45.078.
	 Executed 1 test, with 0 failures (0 unexpected) in 0.003 (0.004) seconds
```

好耶！你已经让 test 通过了，下一步，重构你的代码。

#### 2.4 Refactor: Clean up your code

#### 2.5 Repeat: Do it again

#### 2.6 Challenge

#### 2.7 Key points









